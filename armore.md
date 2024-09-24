## From Source Code

1. ARMore works on aarch64 and only has BADASS issue when the kernel is older than v5.8-rc1.
2. We use QEMU to simulate an aarch64 Ubuntu virtual machine. The instructions are as follows.
```
sudo apt install -y qemu-system-arm

wget https://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.6-server-arm64.iso

wget https://releases.linaro.org/components/kernel/uefi-linaro/16.02/release/qemu64/QEMU_EFI.fd

qemu-img create -f raw -o size=30G ubuntu_aarch64.img

qemu-system-aarch64 \
-m 2048 \
-cpu cortex-a57 \
-smp 2 \
-M virt \
-bios QEMU_EFI.fd \
-nographic \
-drive if=none,file=ubuntu-18.04.6-server-arm64.iso,id=cdrom,media=cdrom \
-device virtio-scsi-device \
-device scsi-cd,drive=cdrom \
-drive if=none,file=ubuntu_aarch64.img,id=hd0 \
-device virtio-blk-device,drive=hd0

qemu-system-aarch64 \
-m 2048 \\
-cpu cortex-a57  \
-smp 2 \
-M virt \
-bios QEMU_EFI.fd \
-nographic \
-drive if=none,id=system,format=raw,media=disk,file=ubuntu_aarch64.img \
-device ramfb \
-device qemu-xhci,id=xhci -usb \
-device usb-kbd \
-device usb-mouse \
-device usb-tablet \
-k en-us \
-device virtio-blk,drive=system,bootindex=0
```

3. Get code from [ARMore repo](https://github.com/HexHive/retrowrite.git).
4. Switch to unpatched version: `git checkout d722ec5`.
5. Use ARMore to disassemble `hello_world`: `./retrowrite --asan hello_world hello_world.s`.
6. Compile `hello_world.s` back to `hello_world_mod`: `./retrowrite -a hello_world.s -o hello_world_mod`.
7. `hello_world_mod` does not have GNU_STACK segment. The missing of GNU_STACK segment on aarch64 will trigger READ_IMPLES_EXEC. Check the memory via `cat /proc/$(pgrep hello_world_mod)/maps` and we can see all readable memory is executable.

## Root Cause

Missing `.section .note.GNU-stack,"",@progbits` in `hello_world.s`.

## Timeline

* 11/13/2023: report to authors
* 11/16/2023: issue confirmed
* 11/30/2023: issue fixed by commit [c31896e](https://github.com/HexHive/retrowrite/commit/ef4e541ac47fd032d30148fc61dfe48e94d5cd19)