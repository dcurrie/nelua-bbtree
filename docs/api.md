# BBTree API for Nelua

Bounded Balance Trees a.k.a. Weight Balanced Trees

a persistent data structure providing a generic (parameterized) key,value map
 * Insert (``insert``), lookup (``get``), and delete (``delete``) in O(log(N)) time
 * Key-ordered iterators (``pairs`` and ``revpairs``)
 * Lookup by relative position from beginning or end (``getNth``) in O(log(N) time
 * Get the position (``getrank``) by key in O(log(N)) time
 * Efficient set operations using tree keys
 * Set operations with map extensions for options on value merge for duplicates

## Types

Create `wbtforestT` instances using

```lua
require 'wbtforest'
local forest : wbtforest(keytype, valtype)
-- or
local forest : wbtforest(keytype, valtype, comparefun)
-- or
local forest : wbtforest(keytype, valtype, comparefun, allocator)
-- or
local forest = new(@wbtforest(keytype, valtype))
-- or
local forest : new(@wbtforest(keytype, valtype, comparefun))
-- or
local forest : new(@wbtforest(keytype, valtype, comparefun, allocator))
```
Of course you can use `global` in place of `local` above. Using `nil` in place
of the `comparefun` or `allocator`, or omitting them entirely, will use the
defaults.

The `comparefun` must take two arguments, and return -1 if the first is less than
the second argument, return 1 if the first is greater than the second argument,
and return 0 if the arguments are equal. If omitted, the `keytype` must support
the `__lt` and `__gt` metamethods.

Instances of `wbtforestT` are intended to be immutable opaque data structures.

The `wbtforestT` is a "factory" that can be used to make instances of `wbtmapT`.
Assuming your forest is named `forest` as create above, create `wbtmapT`s using

```lua
local wbt = forest:makewbtmap()
```
Instances of `wbtmapT` are intended to be immutable opaque data structures.

New instances of `wbtmapT` are also returned by functions that manipulate the trees.

## Memory Management

The `wbtforest` library is designed to work with either automatic or manual
memory management, i.e., in default or in `-Pnogc` modes.

### Default (automatic) memory management mode

No worries! Well, no worries as long as you use `new` to make `wbtforestT`
instances. Using `new` will allocate the `wbtforestT` instance in the heap
where it is automatically managed. In particular, there is a `__gc`
metamethod on `wbtforestT` and on `wbtmapT` to `destroy` all resources used,
and manage the links among them.

#### Danger, Will Robinson!

You *can* stack allocate a `wbtforestT` instance. In this case it is essential
to call `wbtforestT.destroy` on that instance before the stack frame is exited.
The most reliable way to manage this is to annotate the allocation with `<close>`
as in:

```lua
require 'wbtforest'

local function tbd()
    local forest : wbtforest(keytype, valtype) <close>
    -- your code here!
end
```
This works because the `wbtforestT.__close` metamethod also calls
`wbtforestT.destroy`

If the code does not directly or indirectly call `wbtforestT.destroy` before
the stack frame is exited, and there are live `wbtmapT` instances, you will
likely get stack corruption when those instances are garbage collected.

### Optional (`-Pnogc`) manual memory management mode

In manual memory management mode you are responsible to `Allocator:delete` all
instances of `wbtforestT` and `wbtmapT`. This can be done in any order. The
`Allocator` must match the one passed to the `wbtforest` function used to create
the `wbtforestT`, if any, or the `default_allocator` otherwise. Note that the
Nelua `delete` function is shorthand for `default_allocator:delete`.

The `wbtforestT` and `wbtmapT` instances have a `__delete` metamethod that does
the required `wbtforestT.destroy` step when deleted.

See the caveat above about stack allocating a `wbtforestT` instance. In the
manual memory management mode you must also call `wbtforestT.destroy` on
`wbtforestT` instances created on the stack (only call `delete` on
`wbtforestT` instances created with `new`, and on `wbtmapT` instances, which
are always created with `new`).

## Functions

### Basic Functions for *key,value* mapping and inquiry

| Basic Functions for *key,value* mapping and inquiry |
|:---------------------|
| **insert**(self: wbtmapT, key: K, value: V) -> wbtmapT |
| Returns a new tree with the (`key`, `value`) pair added, or replaced if `key` is already in the tree `root`. O(log N) |
| **delete**(self: wbtmapT, key: K) -> wbtmapT |
| Deletes `key` from tree `root`. Does nothing if the key does not exist. O(log N) |
| **delmax**(self: wbtmapT) -> wbtmapT |
| Delete the maximum element from tree `root`. O(log N) |
| **delmin**(self: wbtmapT) -> wbtmapT |
| Delete the minimum element from tree `root`. O(log N) |
| **get**(self: wbtmapT, key: K, vdefault: V) -> V |
| Retrieves the value for `key` in the tree `root` iff `key` is in the tree.  Otherwise, `vdefault` is returned. O(log N) |
| **getmax**(self: wbtmapT, kdefault: K, vdefault: V) -> K, V |
| Retrieves the key,value pair with the largest key in the tree `root`. For an empty tree `kdefault, vdefault` is returned. O(log N) |
| **getmin**(self: wbtmapT, kdefault: K, vdefault: V) ->  K, V |
| Retrieves the key,value pair with the smallest key in the tree `root`. For an empty tree `kdefault, vdefault` is returned. O(log N) |
| **getNth**(self: wbtmapT, index: int, kdefault: K, vdefault: V) -> K, V |
| Get the key,value pair of the 0-based `index` key in the tree `root` when index is positive and less than the tree length. Or get the tree length plus `index` key in the tree `root` when the `index` is negative and greater than the negative tree length. Otherwise, `kdefault, vdefault` is returned. O(log N) |
| **getnext**(self: wbtmapT, key K, kdefault: K, vdefault: V) -> K, V |
| Returns the key,value pair with smallest key > `key`. It is almost inorder successor, but it also works when `key` is not present. If there is no such successor key in the tree, `kdefault, vdefault` is returned. O(log N) |
| **getprev**(self: wbtmapT, key K, kdefault: K, vdefault: V) -> K, V |
| Returns the key,value pair with largest key < `key`. It is almost inorder predecessor, but it also works when `key` is not present. If there is no such predecessor key in the tree, `kdefault, vdefault` is returned. O(log N) |
| **len**(self: wbtmapT) -> int |
| Returns the number of keys in tree at `root`.  O(1) |
| **getrank**(self: wbtmapT, key: K, rdefault: int) -> int |
| Retrieves the 0-based index of `key` in the tree `root` iff `key` is in the tree. Otherwise, `rdefault` is returned. O(log N) |

### Iterators

| Iterators |
|:---------------------|
| **pairs**(self: wbtmapT) |
| Used in `for k,v in pairs(wbt) do` loops to iterate through the tree entries in *key* order. |
| **revpairs**(self: wbtmapT) |
| Used in `for k,v in wbt:revpairs() do` loops to iterate through the tree entries in reverse *key* order. |

### Set operation functions on *key,value* maps

| Set operation functions on *key,value* maps |
|:---------------------|
| **contains**(self: wbtmapT, key: K) -> boolean |
| Returns `true` if `key` is in the tree `root` otherwise `false`. O(log N) |
| **union**(tree1: wbtmapT, tree2: wbtmapT) -> wbtmapT |
| Returns the union of the sets represented by the keys in `tree1` and `tree2`.  When viewed as maps, returns the key,value pairs that appear in either tree; if a key appears in both trees, the value for that key is selected from `tree1`, so this function is asymmetrical for maps. If you need more control over how the values are selected for duplicate keys, see `unionmerge`. O(M + N) but if the minimum key of one tree is greater than the maximum key of the other tree then O(log M) where M is the size of the larger tree. |
| **unionmerge**(tree1: wbtmapT, tree2: wbtmapT, merge: (K,V,V)->V) -> wbtmapT |
| Returns the union of the sets represented by the keys in `tree1` and `tree2`. When viewed as maps, returns the key,value pairs that appear in either tree; if a key appears in both trees, the value for that key is the result of the supplied `merge` function, which is passed the common key, and the values from `tree1` and `tree2` respectively. O(M + N) but if the minimum key of one tree is greater than the maximum key of the other tree then O(log M) where M is the size of the larger tree. |
| **intersection**(tree1: wbtmapT, tree2: wbtmapT) -> wbtmapT |
| Returns the set intersection of `tree1` and `tree2`. In other words, returns the keys that are in both trees. When viewed as maps, returns the key,value pairs for keys that appear in both trees; the value each key is selected from `tree1`, so this function is asymmetrical for maps. If you need more comtrol over how the values are selected for duplicate keys, see `intersectionmerge`. O(M + N) |
| **intersectionmerge**(tree1: wbtmapT, tree2: wbtmapT, merge: (K,V,V)->V) -> wbtmapT |
| Returns the set intersection of `tree1` and `tree2`. In other words, returns the keys that are in both trees. When viewed as maps, returns the key,value pairs for keys that appear in both trees; the value for each key is the result of the supplied `merge` function, which is passed the common key, and the values from `tree1` and `tree2` respectively.  O(M + N) |
| **difference**(tree1: wbtmapT, tree2: wbtmapT) -> wbtmapT |
| Returns the asymmetric set difference between `tree1` and `tree2`. In other words, returns the keys that are in `tree1`, but not in `tree2`. O(M + N) |
| **symmetricdifference**(tree1: wbtmapT, tree2: wbtmapT) -> wbtmapT |
| Returns the symmetric set difference between `tree1` and `tree2`. In other words, returns the keys that are in `tree1`, but not in `tree2`, union the keys that are in `tree2` but not in `tree1`.  O(M + N) |
| **issubset**(tree1: wbtmapT, tree2: wbtmapT) -> boolean |
| Returns true iff the keys in `tree1` form a subset of the keys in `tree2`. In other words, if all the keys that are in `tree1` are also in `tree2`. O(N) where N is `len(tree1)`. Use `ispropersubset` instead to determines that there are keys in `tree2` that are not in `tree1`. |
| **ispropersubset**(tree1: wbtmapT, tree2: wbtmapT) -> boolean |
| Returns true iff the keys in `tree1` form a proper subset of the keys in `tree2`. In other words, if all the keys that are in `tree1` are also in `tree2`, but there are keys in `tree2` that are not in `tree1`.  O(N) where N is `len(tree1)` |
| **setequal**(tree1: wbtmapT, tree2: wbtmapT) -> boolean |
| Returns true if both `tree1` and `tree2` have the same keys. O(N) where N is `len(tree1)` |
| **disjoint**(tree1: wbtmapT, tree2: wbtmapT) -> boolean |
| Returns true iff `tree1` and `tree2` have no keys in common. O(N) where N is `len(tree1)` |
