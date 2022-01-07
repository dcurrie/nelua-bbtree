# e Notes

## C compiler and --sanitize leaks

macOS default clang 13.0 compiler doesn't support --sanitize leaks

`brew install llvm`

>    To use the bundled libc++ please add the following LDFLAGS:
>    LDFLAGS="-L/usr/local/opt/llvm/lib -Wl,-rpath,/usr/local/opt/llvm/lib"
>
>    llvm is keg-only, which means it was not symlinked into /usr/local,
>    because macOS already provides this software and installing another version in
>    parallel can cause all kinds of trouble.
>
>    If you need to have llvm first in your PATH, run:
>    echo 'export PATH="/usr/local/opt/llvm/bin:$PATH"' >> ~/.zshrc
>
>    For compilers to find llvm you may need to set:
>    export LDFLAGS="-L/usr/local/opt/llvm/lib"
>    export CPPFLAGS="-I/usr/local/opt/llvm/include"

Then you can build Nelua code with, e.g.,:
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
