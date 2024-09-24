## From Developer Provided Docker Image

1. Get the unpatched version via `docker pull grammatech/ddisasm:1.5.4`
2. Run `ddisasm hello_world --asm hello_world.s` to disassemble `hello_world`
3. Compile `hello_world.s` back to `hello_world_mod`
4. The reassembled `hello_world_mod` has an executable stack.
5. Run `docker pull grammatech/ddisasm:latest` to get the patched version and repeat steps 2~4. No executable stack in reassembled `hello_world_mod`.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in `hello_world.s`.

