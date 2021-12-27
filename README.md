# nelua-bbtree

BBTree is a Bounded Balance Tree library for the [Nelua programming language](https://github.com/edubart/nelua-lang).

## Status

The project is incomplete and not ready for use.

## BBTree Overview

BBTrees, also known as Weight Balanced Trees, or Adams Trees, are a persistent data structure
with a nice combination of properties:

* Generic (parameterized) key,value map
* Insert (`add`), lookup (`get`), and delete (`del`) in O(log(N)) time
* Key-ordered iterators (`inorder` and `revorder`)
* Lookup by relative position from beginning or end (`getNth`) in O(log(N)) time
* Get the position (`rank`) by key in O(log(N)) time
* Efficient set operations using tree keys
* Map extensions to set operations with optional value merge control for duplicates

By "persistent" we mean that the BBTree always preserves the previous version of itself when it is modified.
As such it is effectively immutable, as the operations do not (visibly) update the structure in-place,
but instead always yield a new updated structure. When insertions or deletions to the tree are made, we
attempt to reuse as much of the old structure as possible.

Because Nelua has an optional garbage collector, some Nelua programs expect to run without gc. To
accommodate them, `wbtforest` manages its own heap of nodes and list of tree roots. Operations on
trees always return a new root, called a `wbtree`. In garbage collected environments, the `__gc`
metamethod on `wbtree` will clean up internal `wbtforest` resources. In non-gc environments, the
owner of the `wbtree` is responsible for calling `wbtree.destroy`, which is also connected to the
`__close` metamethod for convenience.

## BBTree Credits

References:

*Implementing Sets Efficiently in a Functional Language*
Stephen Adams
CSTR 92-10
Department of Electronics and Computer Science University of Southampton Southampton S09 5NH

*Adams’ Trees Revisited Correct and Efficient Implementation*
Milan Straka
Department of Applied Mathematics Charles University in Prague, Czech Republic

[Weight-balanced trees](https://en.wikipedia.org/wiki/Weight-balanced_tree) on Wikipedia

## BBTree Details

### Documentation

See the doc directory. Here is the [API](https://github.com/dcurrie/nelua-bbtree/blob/master/doc/api.md)

### Installation

None, simply include the files in your project

### Test

See `wbtforest-test.nelua`

### License

MIT. See file LICENSE.
