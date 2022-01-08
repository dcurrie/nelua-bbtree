# e Notes

## C compiler and --sanitize leaks

macOS default clang 13.0 compiler doesn't support --sanitize leaks

`brew install llvm` gives this message

```
To use the bundled libc++ please add the following LDFLAGS:
LDFLAGS="-L/usr/local/opt/llvm/lib -Wl,-rpath,/usr/local/opt/llvm/lib"

llvm is keg-only, which means it was not symlinked into /usr/local,
because macOS already provides this software and installing another
 version in parallel can cause all kinds of trouble.

If you need to have llvm first in your PATH, run:
echo 'export PATH="/usr/local/opt/llvm/bin:$PATH"' >> ~/.zshrc

For compilers to find llvm you may need to set:
export LDFLAGS="-L/usr/local/opt/llvm/lib"
export CPPFLAGS="-I/usr/local/opt/llvm/include"
```

So, we can build Nelua code with, e.g.,:
`nelua --cc=/usr/local/opt/llvm/bin/clang --ldflags="-L/usr/local/opt/llvm/lib" --cflags="-I/usr/local/opt/llvm/include" --sanitize -DeDEBUG wbtforest-test.nelua`

Running that way didn't detect leaks, but when I used:
`ASAN_OPTIONS=detect_leaks=1 ~/.cache/nelua/wbtforest-test`
it worked, so Eduardo added that env setting automatically when `--sanitize` is used.

See `nel` bash script that provides the necessary llvm compiler options.

## Stress Test

2022-01-06

```
nelua-bbtree e% ./nel -DSTRESS_TEST_N=20000 -DBB_ITERATIONS=100000  wbtforest-stress.nelua
warning: using error handling module, it is highly experimental and incomplete!
        .................................................................................................
        CPU time 3282.504314 s for 100000 loops, averaging 0.032825 s per loop
        random q: 54996  inserts, 45004 deletes, 9992 qintree, 9992 final size
[PASS] wbtforest | stress test
[====] wbtforest | 1 successes / 3282.504440 seconds
nelua-bbtree e%
```

2022-01-07 overnight, questionable timing?

```
nelua-bbtree e% ./nel -DSTRESS_TEST_N=20000 -DBB_ITERATIONS=500000  wbtforest-stress.nelua
warning: using error handling module, it is highly experimental and incomplete!
        ........................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
        CPU time 59717.410381 s for 500000 loops, averaging 0.119435 s per loop
        random q: 254998  inserts, 245002 deletes, 9996 qintree, 9996 final size
[PASS] wbtforest | stress test
[====] wbtforest | 1 successes / 59717.410563 seconds
nelua-bbtree e%
```
2022-01-08 while machine in use

```
nelua-bbtree e% ./nel -DSTRESS_TEST_N=20000 -DBB_ITERATIONS=5000  wbtforest-stress.nelua
warning: using error handling module, it is highly experimental and incomplete!
        ....
        CPU time 29.894521 s for 5000 loops, averaging 0.005979 s per loop
        random q: 4435  inserts, 565 deletes, 3870 qintree, 3870 final size
        Allocated nodes: 8192 roots: 3 gcwip: 64
[PASS] wbtforest | stress test
[====] wbtforest | 1 successes / 29.894645 seconds
nelua-bbtree e% ./nel -DSTRESS_TEST_N=20000 -DBB_ITERATIONS=50000 wbtforest-stress.nelua
warning: using error handling module, it is highly experimental and incomplete!
        ................................................
        CPU time 1354.365573 s for 50000 loops, averaging 0.027087 s per loop
        random q: 30028  inserts, 19972 deletes, 10056 qintree, 10056 final size
        Allocated nodes: 16384 roots: 3 gcwip: 64
[PASS] wbtforest | stress test
[====] wbtforest | 1 successes / 1354.365699 seconds
nelua-bbtree e% ./nel -DSTRESS_TEST_N=20000 -DBB_ITERATIONS=100000 wbtforest-stress.nelua
warning: using error handling module, it is highly experimental and incomplete!
        .................................................................................................
        CPU time 2465.951925 s for 100000 loops, averaging 0.024660 s per loop
        random q: 54996  inserts, 45004 deletes, 9992 qintree, 9992 final size
        Allocated nodes: 16384 roots: 2 gcwip: 64
[PASS] wbtforest | stress test
[====] wbtforest | 1 successes / 2465.952037 seconds
nelua-bbtree e%
```
