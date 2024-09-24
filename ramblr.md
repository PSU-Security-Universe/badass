## From Provided Docker Image

1. We provide a docker image for ERIM, Ramblr (patcherex), RetroWrite, and Donky. Run `docker pull hengkaiye/badass-image` to download.
2. In `/badass/patcherex/`, run `python reassemble.py` to generate `hello_world_mod`
3. Convert `hello_world_mod` to ELF format via `cgc2elf/cgc2elf hello_world_mod`
4. Run `objdump -f ./hello_world_mod | grep "start address"` to get the start address of `hello_world_mod`. For example, we assume the address is 0x080480c0
5. In GDB, set breakpoint at start address via `b *0x080480c0` and run.
6. When GDB stops at the breakpoint, run `cat /proc/$(pgrep hello_world_mod)/maps`. We can see the stack is executable.

## From Source

1. Install `compilerex`
```bash
git clone https://github.com/mechaphish/compilerex.git
cd compilerex
pip install -e .
mv compilerex to /Path/to/installed/python/packages
```
2. Get Ramblr source code from [patcherex repo](https://github.com/angr/patcherex.git).
3. Comment line 14 `povsim` and line 15 `compilerex` in `setup.py`.
5. Run `dpkg --add-architecture i386`, `apt-get update`, `apt install lib32z1 libtinfo5:i386`
4. Follow the official instructions to build patcherex.
5. The original patcherex works on CGC binary, so we need to convert CGC binary to ELF. Get code from [cgc2elf](https://github.com/CyberGrandChallenge/cgc2elf.git) and compile `cgc2elf.c` to `cgc2elf`.
6. We also need to convert ELF to CGC later. Replace line 8 in `cgc2elf.c` with `#define      C_IDENT "\177CGC\x01\x01\x01\x43\x01\x4d\x65\x72\x69\x6e\x6f` and compile it to `elf2cgc`.
7. Prepare an assembly file `hello_world.s`, compile it via `as --32 hello_world.s -o hello_world.o` and `ld -m elf_i386 hello_world.o -o hello_world`. Then, convert `hello_world` to CGC format via `cgc2elf hello_world`.
```asm
      .data
hello:
      .string "Hello World!\n"

.text
.globl _start
_start:
      movl $4, %eax
      movl $1, %ebx
      movl $hello, %ecx
      movl $13, %edx
      int $0x80

      movl $1, %eax
      movl $0, %ebx
      int $0x80

.section        .note.GNU-stack,"",@progbits
```
8. Create a python file named `reassemble.py` as follows and reassemble `hello_world` via `python3 ./reassemble.py`
```py
import patcherex
from patcherex.backends.detourbackend import DetourBackend
from patcherex.backends.reassembler_backend import ReassemblerBackend
from patcherex.patches import *

# the detour backend can be used as well:
backend = ReassemblerBackend("hello_world")
patches = []

transmit_code = '''
  pusha
  mov ecx,eax
  mov edx,ebx
  mov eax,0x2
  mov ebx,0x1
  mov esi,0x0
  int 0x80
  popa
  ret
  '''
patches.append(AddCodePatch(transmit_code, name="transmit_function"))
patches.append(AddRODataPatch(b"HI!\x00", name="transmitted_string"))
# the following code is going to be executed just before the original instruction at 0x8048166
injected_code = '''
; at this code location, it is fine to clobber eax and ebx
mov eax, {transmitted_string} ; a patch can refer to another patch address, by putting its name between curly brackets
mov ebx, 4
call {transmit_function}
'''
patches.append(InsertCodePatch(0x8048166,injected_code,name="injected_code_after_receive"))

# now we ask to the backend to inject all our patches
backend.apply_patches(patches)
# and then we save the file
backend.save("hello_world_mod")
```
9. Convert `hello_world_mod` to ELF format via `cgc2elf hello_world_mod`.
10. Run `objdump -f ./hello_world_mod | grep "start address"` to get the start address of `hello_world_mod`. For example, here we assum the address is 0x080480c0
11. In GDB, set breakpoint at start address via `b *0x080480c0` and run.
12. When GDB stops at the breakpoint, run `cat /proc/$(pgrep hello_world_mod)/maps`. We can see the stack is executable.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in disassembled assembly file.

## Timeline

* 11/13/2023: report to the author
* 11/13/2023: issue confirmed