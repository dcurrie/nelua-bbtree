# Nelua Observations

Nelua is a very nice project, and a pleasure to use. Eduardo is a very bright
programmer, and is very kind to Nelua users. I really like the language and the
community. So, please don't take any criticisms in these observations as
disparaging... for one thing they may be wrong (I'm very new to Nelua), and
they're just my opinions.

## Documentation

Nelua is in flux. That's a good thing and active development is a part of
why I am taking a look at it. However, it means there are features that exist
and are not really documented. One I needed and found via Discord was
`<forwarddecl>` to make circular references among record definitions.

## `__atindex`

The `__atindex` metamethod is ugly. My first Nelua experience was a port of my
Lobster Robin Hood Hashmap to Nelua. The Robin Hood Hashmap moves entries
around on insertion and deletion to optimize lookup time. The problem with
`__atindex` is that it is used for array-like operations on tables whether as
a source or sink of values. I.e., both `a[x] = 1` and `y = a[x]` use the
same `__atindex` metamethod. So, reading indexed on a key puts the key into the
table! Ugh. It also rearranges the table, maybe for nothing. Double ugh.

Futhermore, the `a[x] = 1` form could be used to insert a new value into the
table or modify an existing one. This makes it useless for persistent structures,
where it might have been useful for lookups like `y = a[x]` if there were
separate `__at_r` and `__at_w` metamethods.

## Zero is Initialization

Getting accustomed to "Zero is Initialization" policy was interesting.

One thing I learned is that using `index < size` was a better policy than
using `index ~= special_nil_value`. Three advantages:

1. `index < size` leads to the exception case when both `index` and `size` are
zero, which is a good "not hot path" place to do the necessary allocations and
initializations

2. `index < size` is not much more expensive than `index ~= special_nil_value`
because the span pointer will need to get into the CPU cache anyway, and the
size field is adjacent to that pointer

3. `index < size` is good to check for buffer/heap overflows and other bugs in
any case

After coming to terms with having to check for initialization state in several
places in the code (e.g., see the **three** places I use `preprootslist`), I see
that Eduardo is implementing a `__new` metamethod to do the record initialization.
Hmm.

## Doubly linked lists

I prefer the circular version. There are fewer special cases to worry about (like
head and tail NILs), any node can be a fast way into the list, and any node can
be left as the only remaining piece of the list, as I did with the `wbtrootT`
records. It is just better suited to my use-case, so I didn't use the `list`
standard library.

## Type System and Generics

The meta-programming approach to generic types is an interesting point in the
type system design space. I certainly don't see all the implications. Here are
a few things that I am not confident that I understand well (or at all).

### Private vs Public

The `wbtforestT` implementation has a lot of internal mechanisms and data, and
it would be nice to protect some of them from misuse. In a Lua module it would
be easy to make some methods `local` to the module and not expose them to users
of the module. It's not clear to me if this is possible in Nelua Generics.

The `wbtforestT` is a generic factory that creates `wbtmapT` instances that are
also generic. This all seems to work, but is a bit magical. I'm on the fence
about whether to make the record definition for `wbtmapT` local to the module
(unlike methods, record definitions can be local). If they are local, users of
the library can't declare variables to be `wbtmapT` despite being able to get
instances of `wbtmapT` from the `wbtforestT` factory. It seems odd that there
is no way to name the type of something you can create and manipulate.

I also found it odd that all `wbtmapT`s use the same type name despite being
generated by `wbtforestT` factories of different concrete types.
