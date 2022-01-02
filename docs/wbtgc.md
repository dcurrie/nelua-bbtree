# BBTree GC

The BBTree experiment with Nelua focused on building a BBTree abstraction that
worked in a Nelua GC environment or a Nelua noGC environment. That meant that
the code needed to manage memory on its own. Since the persistent nature of
BBTrees requires structure sharing and immutability, we need to keep track of
references to the roots of the trees so we can mark reachable tree nodes. The
alternative of reference counting in each tree node is also possible, but the
bookkeeping involved was not appealing.

Note that the implementation chosen uses a "forest" of trees stored in a span
of tree nodes. This reduces the size of nodes since we can use indexes into the
span rather than full sized pointers. All trees in the forest are the same type,
generated from the generic type specialized on concrete key and value types.

If Nelua had weak pointers, that might be a good approach. The forest could keep
a table of tree roots keyed by a tree token object returned to the caller of
tree creating functions. When all references to the tree token are gone, only
the weak pointer in the table is left, and it is removed from the table by the
Nelua GC. Iterating the table identifies all the live tree roots. Without weak
references, though, we needed another way.

![wbtforest structure]((https://github.com/dcurrie/nelua-bbtree/blob/master/docs/wbtforest_1.drawio.png))

This is it.



