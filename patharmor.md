## From Provided VirtualBox Image

1. Download VirtualBox image from [badass vm](https://zenodo.org/records/15467218)
2. Boot the VM and select Linux kernel 3.16.0-77 in grub menu
3. Login with password "badass"
4. In `~/badass-vm/patharmor/shared/`, we can find `libwrapper.so` with GNU_STACK set to RWX
5. Run `sh lkm-start.sh` under `~/badass-vm/patharmor` to load modules
6. Run `sh ~/badass-vm/patharmor/run-app.sh ~/badass-vm/patharmor/toy-bug/toy`, it will print the pid (e.g., 2345) of `toy`.
7. Run `cat /proc/2345/maps` and we can see the process has an executable stack.


## From Source Code

1. Get source code from [PathArmor repo](https://github.com/vusec/patharmor).
2. Follow [official guideline](https://github.com/vusec/patharmor/blob/master/INSTALL.md) to build PathArmor. 
    
    2.1. Since the VM image and kernel mentioned in the blog are no longer accessible, we replace them with similar versions: Ubuntu 14.04 and Linux kernel 3.16.0-77.
    
    2.2. The original link of libdwarf-20130126.tar.gz is not accessible, replace all `http://www.paradyn.org/libdwarf/libdwarf-20130126.tar.gz` in `~/badass-vm/patharmor/Dyninst-8.2.1/install-dir/libdwarf/src/LibDwarf-stamp/download-LibDwarf.cmake` with `https://www.prevanders.net/libdwarf-20130126.tar.gz`

3. In `patharmor/shared/`, we can find `libwrapper.so` with GNU_STACK set to RWX.
4. In `patharmor/toy_bug/toy.c`, add code to print pid and sleep for 1000 seconds.
5. Run `sh ~/badass-vm/patharmor/run-app.sh ~/badass-vm/patharmor/toy-bug/toy`, it will print the pid (e.g., 2345) of `toy`.
6. Run `cat /proc/2345/maps` and we can see the process has an executable stack.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in `shared/callback.S` and `shared/libenter.S`, which make the stack of any protected program executable.

## Timeline

* 11/13/2023: report to the author
* 11/14/2023: issue confirmed