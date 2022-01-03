# nelua-bbtree

BBTree is a Bounded Balance Tree library for the [Nelua programming language](https://github.com/edubart/nelua-lang).

## Why?

I've used implementation of a BBTree data structure library as a way to learn and explore
several interesting programming languages, including Nim, Lobster, Koka, and some toy languages
of my own. The project is big enough to explore several aspects of a programming language, but
contained enough to do in a short burst.

Nelua provides a new challenge, a language that can be used with or without automatic memory
management. I wondered if a BBTree library could be developed that worked well in both modes.
The persistent BBTree needs to share structure among trees to be asymptotically optimal. Memory
management is intractable to do manually by the library user, so to be useful, the Nelua BBTree
library would need to manage its own memory with or without the Nelua garbage collector. The
design is described in [wbtgc](docs/wbtgc.md).

## Status

The project has basic BBTree functions working, but is incomplete: some set functions are not
coded, and unit testing is incomplete. It is not ready for use.

## BBTree Overview

BBTrees, also known as Weight Balanced Trees, or Adams Trees, are a persistent data structure
with a nice combination of properties:

* Generic (parameterized) key,value map
* Insert (`insert`), lookup (`get`), and delete (`delete`) in O(log(N)) time
* Key-ordered iterators (`pairs` and `revpairs`)
* Lookup by relative position from beginning or end (`getnth`) in O(log(N)) time
* Get the position (`getrank`) by key in O(log(N)) time
* Efficient set operations using tree keys
* Map extensions to set operations with optional value merge control for duplicates

By "persistent" we mean that the BBTree always preserves the previous version of itself when it is modified.
As such it is effectively immutable, as the operations do not (visibly) update the structure in-place,
but instead always yield a new updated structure. When insertions or deletions to the tree are made, we
attempt to reuse as much of the old structure as possible.

Because Nelua has an included garbage collector, Nelua programs can run without Nelua's gc. To
accommodate them, `wbtforestT` manages its own heap of nodes and list of tree roots. Operations on
trees always return a new root, called a `wbtmapT`. In garbage collected environments, the `__gc`
metamethods on `wbtmapT` and `wbtforestT` will clean up internal `wbtforestT` resources. In non-gc
environments, the owner of the `wbtforestT` and its `wbtmapT`s is responsible for calling
`wbtforestT`'s and `wbtmapT`'s `destroy` methods, which are also connected to
`__close` metamethods for convenience.

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

See the docs directory. Here is the [API](docs/api.md)

### Installation

None, simply include the files in your project

### Test

See `wbtforest-test.nelua`

### License

MIT. See file LICENSE.
