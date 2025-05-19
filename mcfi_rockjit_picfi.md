## From Provided VirtualBox Image

1. Download VirtualBox image from [badass vm](https://zenodo.org/records/15467218)
2. Boot the VM and select Linux kernel 4.4.0-142-generic in grub menu
3. Login with password "badass"
4. In `~/badass-vm/MCFI/`, run `~/MCFI/toolchain/clang hello_world.c -o hello_world`
5. Start a new terminal and run `cat /proc/$(pgrep hello_world)/maps`. We can see the process has an executable stack.

## From Source Code

1. Build MCFI/RockJIT/πCFI on a Ubuntu 14.04 virtual machine due to compatibility issues.
2. Get source code from [MCFI repo](https://github.com/mcfi/MCFI).
3. Switch to the version before patch: `git checkout 7c9de0c`.
4. Follow [official guideline](https://github.com/mcfi/MCFI/blob/master/README.md) to build the tool.
5. Find the MCFI compiler at `toolchain/clang`.
6. Compile hello_world: `toolchain/clang hello_world.c -o hello_world`.
7. Check stack permission from readelf: `readelf -l hello_world`. We can see the GNU_STACK segment has been set to RWX.
8. Check in-memory stack permission: `cat /proc/$(pgrep hello_world)/maps`. We can see the stack region is readable, writable, and executable. Besides, the executable stack triggers the kernel feature READ_IMPLES_EXEC, which sets all readable memory to executable.
9. Apply the patch via `git checkout c5eb60e`, rebuild the toolchain, and compile `hello_world` again. The GNU_STACK flag is RW and the in-memory stack is readable, writable, but not executable.

## Root Cause

`lib/cxxstart/build.sh` generates an empty assembly file `crtend.S`, which makes the stack of any protected program executable. Besides, `runtime/src/start.s` and `runtime/src/runtime_interface.S` miss `.section .note.GNU-stack,"",@progbits`, which makes the stack of the loader `rock` executable.

## Timeline

* 11/13/2023: report to the author
* 11/13/2023: issue confirmed
* 11/13/2024: issue fixed by commit [9537dd9](https://github.com/mcfi/MCFI/commit/9537dd9822c874f3a134d42e355b67b70de1888e)