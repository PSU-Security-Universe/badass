## From Provided Docker Image

1. We provide a docker image for ERIM, Ramblr (patcherex), RetroWrite, and Donky. Run `docker pull hengkaiye/badass-image` to download.
2. Run `make -C DonkyLib PLATFORM=x86_64 RELEASE=1 TIMING=1 SIM=pk clean run` to build the static DonkyLib and run `make -C DonkyLib PLATFORM=x86_64 RELEASE=1 TIMING=1 SIM=pk SHARED=1 clean run` to build the shared DonkyLib.
3. Donky compiles both library and user-side tests into a single executable: `x.elf`. We can see the GNU_STACK segment of `x.elf` has been set to RWX.

## From Source Code

1. Donky works on both RISC-V and x86 architecture. We only examine Donky on x86 here.
2. Get source code from [Donky repo](https://github.com/IAIK/Donky.git).
3. Follow https://github.com/IAIK/Donky/blob/master/README.md to build static and shared DonkyLib.
4. Donky compiles both library and user-side tests into a single executable: `x.elf`. We can see the GNU_STACK segment of `x.elf` has been set to RWX.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in `pku_handler.S`, `pk_handler.S`, and `pku_api_wrapper.S`, which make the static libraries (`pk.a` and `usr.a`) and dynamic libraries (`libpk.a` and `libpku.so`) have executable stacks. The protected process will have an executable stack by linking or loading the above libraries.

## Timeline

* 11/13/2023: report to authors
* 11/17/2023: issue confirmed & no real security risk