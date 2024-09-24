## From Provided Docker Image

1. We provide a docker image for ERIM, Ramblr (patcherex), RetroWrite, and Donky. Run `docker pull hengkaiye/badass-image` to download.
2. In `/badass/retrowrite`, run `./retrowrite hello_world hello_world.s`.
3. Compile `hello_world.s` to `hello_world_mod`: `gcc hello_world.s -o hello_world_mod`.
4. The reassembled `hello_world_mod` has an executable stack.

## From Source Code

1. Get code from [RetroWrite repo](https://github.com/HexHive/retrowrite.git).
2. Switch to unpatched version: `git checkout d722ec5`.
3. Follow the official instructions to install dependencies.
4. Use RetroWrite to disassemble `hello_world`: `./retrowrite hello_world hello_world.s`.
5. Compile `hello_world.s` to `hello_world_mod`: `gcc hello_world.s -o hello_world`.
6. The reassembled `hello_world_mod` has an executable stack.
7. Switch to patched version via `git checkout c31896e` and repeat 4~5. The assembly directive is added to `hello_world.s` and no exectable stack in reassembled `hello_world_mod`.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in disassembled assembly file.

## Timeline

* 11/13/2023: report to authors
* 11/16/2023: issue confirmed
* 11/27/2023: issue fixed by commit [c31896e](https://github.com/HexHive/retrowrite/commit/c31896e6814b61e6ef1227f02545ea3cb0d369d4)