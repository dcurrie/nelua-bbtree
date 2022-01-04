# BBTree GC

The BBTree experiment with Nelua focused on building a BBTree abstraction that
worked in a Nelua GC environment or a Nelua noGC environment. That meant that
the code needed to manage memory on its own.

## The Problem

Since the persistent nature of BBTrees requires structure sharing and
immutability, we need to keep track of live references to the roots of the trees
we can mark reachable tree nodes.

Note that the implementation chosen uses a "forest" of trees stored in a span
of tree nodes. This reduces the size of nodes since we can use indexes into the
span rather than full sized pointers. All trees in the forest are the same type,
generated from the generic type specialized on concrete key and value types.

## Three approaches

Note that the word "roots" is a bit overloaded here. There are three kinds of
roots: *Nelua roots* for Nelua's GC (the stack, registers, static variables, etc.),
the *tree roots* (from which all other nodes of that tree are reachable), and
the *live forest roots* that are inputs to the mark phase of the forest GC. All
the *live forest roots* are *tree roots*, but not all *tree roots* are
*live forest roots*; the *live forest roots* are those reachable from *Nelua roots*.

### Weak Pointers

If Nelua had weak pointers, that might be a good approach. The forest could keep
a table of tree roots keyed by a tree token object returned to the caller of
tree creating functions. When all references to the tree token are gone, only
the weak pointer in the table is left, and it is removed from the table by the
Nelua GC. Iterating the table identifies all the live tree roots. Without weak
references, though, we needed another way.

### Surrogate Records to a List of Tree Roots

The second approach considered, and the first implemented, is to keep a doubly
linked circular list of tree roots, and return a `wbtree` record that points to
the list entry and the `wbtforest` to identify the tree. A `__gc` metamethod on
the `wbtree` record removes the root from the list.

A bug: the test code for this approach mostly worked, but for one case where the
`wbtforest` is destroyed before all the issued `wbtree` records. This occurred
in the unit tests when a stack allocated `wbtforest` went out of scope. One
could argue that this is a programmer error, but does Nelua offer any guarantees
about order of finalization if a `wbtforest` and `wbtree` records become garbage
in the same gc cycle?

### A Safer Approach

A surrogate record for the `wbtforest` is used to prevent access to `destroy`'d
forests. This surrogate is called the `wbttrail` and contains a pointer to the
`wbtforest`, and a copy of the memory allocator used by the forest. The allocator
is used to `destroy` the list items and eventually the `wbttrail` itself once the
list of tree roots is empty. The `wbttrail` has it's `wbtforest` pointer set to
`nilptr` when the forest is destroyed.

This `wbttrail` record is used in place of a direct pointer to the `wbtforest`
when a `wbtree` is created. The `wbtree` API functions check that the `wbttrail`'s
forest pointer is not `nilptr` before accessing the forest.

#### Forest Structure

![wbtforest structure](wbtforest_1.drawio.png)

#### Deallocated Forest with Live Trees

![wbtforest structure](wbtforest_2.drawio.png)

## `wbtforestT` GC

This section describes the details of the `wbtforestT` GC using the "weak roots"
mechanism described above.

### GC Design

The `wbtforestT` has span of tree nodes, a mark/free bitarray, and a list of
`wbtrootT` root records. All node references are indexes into the span of tree nodes.

The GC is non-moving; this allows `wbtforestT` functions to hold node indexes into
the node span across GC calls, which may occur during allocations when space is
exhausted within the function.

The GC uses a bit span to hold mark bits. This provides some cache locality for
searching for free nodes during the sweep. The mark phase uses the list of
`wbtrootT` root records in a classic recursive mark algorithm. Since all nodes
have the same structure the function is a very simple tree walk for each root,
bailing out early when an already marked node is found. The mark phase keeps a
count of marked (live) nodes. At the end of the phase, if the number of marked
nodes leaves too little free space, the spans holding the nodes and mark bits
are doubled in size using `xspanrealloc` and `xspanrealloc0`.

The sweep phase itself is deferred to the "mutator," in other words the
`wbtforestT` node allocator scans the mark bits to find a free node. It keeps a
`rover` index to the most recently allocated node to start the search for a free
node to avoid re-scanning from the beginning.

The data structures are not protected with locks, and assume single threaded
behavior; this is consistent with other Nelua libraries.

Nodes created during `wbtforestT` (e.g., `insert`, `delete`, *et al*) may have
no gc root until the tree is stitched up, so we keep a stack `gcwip` of node
allocations. This stack, implemented as a span, is scanned during the mark phase
of the gc just like the `wbtrootT` root records. The `gcwip` stack is cleared
before top level return from `wbtmapT` functions, at which point all live nodes
are in complete trees.

### Rejected Ideas

I considered marking based on (Deutsch-)Schorr-Waite DFS, but since the
`wbtforestT` algorithms use recursion on trees, there is no big advantage to
stackless marking relative to the stack used by the primary algorithms. The
stack depth is the same as the tree height, proportional to the log base 2 of
the tree size.

I considered a compacting Jonkers or Johannes Martin style mark-compact approach.
These use pointer reversal and rotation for a stackless compacting gc. These
are especially efficient in persistent heaps where the "pointers" all point in
one direction (since nodes are not mutated, they can only point to older nodes).
There are two drawbacks to this approach: the pointer reversal and rotation
algorithms are non-intuitive (to me! and I've implemented these style gcs a few
times!), difficult to get right, and require an extra bit per "pointer." The
biggest drawback, though, is that all "pointers" in the Nelua stack during a
`wbtforestT` operation need to be found and updated... a no go.

One way to make the mark-compact approach work would be to **prevent** GC during
`wbtforestT` operations so there is no need to find and update references to the
nodes in the Nelua stack. This can be done by ensuring there is sufficient node
space available before starting each `wbtforestT` operation, which can be
calculated with a factor for each operation based on the number of nodes that
can be created at each level (for stitching up the new tree, and for rotations),
and the tree depth.

* For a BBTree, the max depth is log2(n) / log2(1 + 1/ω)
* With OMEGA at 3, log2(1 + 1/ω) = log2(1 + 1/3) = 0.41503749927884
* The upper bound on depth is 2.41 * log2(n)

I used this approach in other implementations, but it is fussy, and prefer the
`gcwip` design described above.

I implemented a bump allocation index in parallel with the scan of mark bits
that was really only useful when the node span was newly allocated or just after
a grow. It would come into its own if we implemented a compacting gc, but with
the non-moving gc, it was more trouble than it was worth. A separate compactor
could be provided that may be called at top level. See the description in
[the TODO maybe list](todo_maybe.md).
