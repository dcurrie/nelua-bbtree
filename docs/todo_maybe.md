# TODO, maybe

## Compact the node space

The BBTree GC page discusses the GC design, and mentions that a compacting GC
was considered and rejected. Nevertheless, a separate compact operation on the
`wbtforestT` is a reasonable thing, and not too hard to do, e.g., using the LISP
two-finger algorithm.
According to [Jemma Issrof's Ruby Deep Dive](https://jemma.dev/blog/gc-compaction)

> This algorithm was actually first written about in “The Programming Language LISP:
> Its Operation and Applications” by Edmund C. Berkeley and Daniel G. Bobrow in 1964.
> It’s quite incredible that this is the algorithm Ruby uses today (57 years later!),
> so I thought it’d be fun to share some of the original text they wrote:
>
>> Two pointers are set, one to the top of free storage and one to the bottom. The
>> top pointer scans words, advancing downward, looking for one not marked. When one
>> is found, the bottom pointer scans words, advancing upward, looking for a marked
>> word. When one is found, it is moved into the location identified by the other pointer.
>> A pointer is left at the location the word was moved from, pointed to whether it
>> was moved to. The top pointer is then advanced as before. The process terminates
>> when the two pointers meet.

In our case, we can scan the `marks` bitvector from both ends to do the same thing.

The references are then patched up by scanning from the start of the storage; any reference
to beyond the (now equal) fingers is replaced with the forwarding pointer in the node
above the fingers. In the `wbtforestT` case, we do the same for the root indexes in the
`wbtrootT` list entries.

## Destructive versions of `wbtmap` functions

Many times we know that a `wbtmapT` passed to a function, like `insert`, will never be
used again. We can avoid the allocation and later gc of the new `wbtrootT` by modifying
the argument to the function instead of creating a new `wbtrootT`.

Using a function name suffix to indicate "destructive" could be used to differentiate
the two versions of the function; too bad Nelua doesn't support use of
symbols in function names like Scheme's use of a `!` suffix for destructive. Perhaps:

```lua
    wbt1:insert_D(k,v)
```

The underlying `wbtforestT` functions doing the tree manipulations remain persistent.
Essentially the above would be equivalent to:

```lua
    local wbt2 = wbt1:insert(k,v)
    wbt1, wbt2 = wbt2, wbt1
    wbt2:destroy()
```

The same could be done for two-tree functions, such as `union`, but in that case one
tree would be persisted and one destructively updated. Perhaps suffixes like `_Dp` and
`_pD` could be used to indicate which is which. For `union`, only one of the those two
is needed since the arguments to `union` can be swapped, but for asymmetric operations
like `difference` we'd need both.

Would this mix of persistent and destructive `wbtmapT`s be too confusing?

## Not implemented; TBD if they will be

### Cross Forest Functions

It's perfectly reasonable to have functions that work on trees in different forests. For example,
copying a tree and type-casting or mapping it's values to a new type, or combining two trees via
set operations with a `merge` function that combines values of the two trees into a third type.

### HOFs

These Higher Order Functions are not really in Nelua style, and also in some cases have the same
concerns as the Cross Forest Functions above.

| HOF |
|:---------------------|
| **foldl**(self: wbtmapT, base: T, f: (K,V,T)->T)) -> T |
| Applies the function `f` to each key value pair of tree `root` in order (left ro right). Uses the `base` value for the first leftmost operand. So, for example, to construct a sum of the tree values, you could use ``foldl(root, 0): _a; _b + _c`` and this will print the keys in order prefixed by "keys:" and separated by ":": `print(foldl(tree, "keys") k, v, b: v; concat_string([b, k],":"))` |
| **foldr**(self: wbtmapT, base: T, f: (K,V,T)->T)) -> T |
| Applies the function `f` to each key value pair of tree `root` in reverse order (right to left). Uses the `base` value for the first rightmost operand. |
| **foreach**(self: wbtmapT, f: (K,V)->void) -> void |
| Applies function `f` to each key and corresponding value of `root`, discarding the results. |
| **map**(self: wbtmapT, f: (K,V,V)->V) -> wbtmapT |
| Returns a new tree with the keys of tree `root` and values that are the result of applying `f` to each key and corresponding value in `root`. |
| **toset**(keys: [K]) -> BBTree<K,bool> |
| Creates a BBTree set that contains the given `keys` with value `true`. |
