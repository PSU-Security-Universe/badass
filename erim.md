# From Provided Docker Image

1. We provide a docker image for ERIM, Ramblr (patcherex), RetroWrite, and Donky. Run `docker pull hengkaiye/badass-image` to download.
2. Set compiler to Clang via `export CC=clang`
3. Run `make` under `/badass/erim/src/tem/ptrace/test` run `cat /proc/$(pidof test_ptracespeed)/maps` in another shell. The hardened process has an executable stack.

# From Source Code

1. Get source code from [ERIM repo](https://github.com/vahldiek/erim.git).
2. Switch to unpatched version: `git checkout f1d4a28` and set compiler to Clang via `export CC=clang`
3. Add `getchar();` at line 50 of `libtem.c` to pause the process.
4. Run `make` under `erim/src/`, `erim/src/tem/libtem/`, and `erim/src/tem/ptrace/`
5. GNU_STACK segment of both `erim/bin/tem/libtem/libtem-lsm.so` and `erim/bin/tem/libtem/libtem-ptrace.so` has been set to RWX.
6. Run `make` under `erim/src/tem/ptrace/test/` and run `cat /proc/$(pidof test_ptracespeed)/maps` in another shell. We can see the hardened process has an executable stack.
7. Switch to patched version: `git checkout 7232f47` and repeat 3~6. We can see the stack is no longer executable.

## Root Cause

Missing `section .note.GNU-stack noalloc noexec nowrite progbits` in `libtem_trampsignal.asm`, which makes the stack of `libtem-lsm.so` and `libtem-ptrace.so` to be executable. The protected process will further have an executable stack if it loads one of these two shared libraries to enable TEM.

## Timeline

* 11/13/2023: report to the author
* 11/13/2023: issue confirmed
* 11/13/2023: issue fixed by commit [7232f47](https://github.com/vahldiek/erim/commit/7232f4762c5ff51035116f9664ec6e7af3b236af)