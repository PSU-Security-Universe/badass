## From Developer Provided Docker Image

1. Uroboros provides a [docker image](https://github.com/s3team/uroboros/tree/master/Docker)
2. In docker container, feed Uroboros `hello_world` via `python uroboros.py hello_world`
3. Check the disassembled output `final.s` and find **.section .note.GNU-stack,"",@progbits** missing.
4. Compile `final.s` back to `hello_world_mod`.
5. The reassembled `hello_world_mod` has an executable stack.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in disassembled assembly file.

## Timeline

* 11/13/2023: report to the author
* 11/16/2023: issue confirmed