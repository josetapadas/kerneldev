# Kernel Tinkering

## Setting up QEMU dev environment

1) Create a qemu image to act as the volume for our development:

```
qemu-img create -f qcow2 kernel-dev.qcow2 40G
```

2) Install (Ubuntu) Linux on it

```
qemu-system-x86_64 -enable-kvm -m 4G -smp 4 -cpu host -hda kernel-dev.qcow2 -cdrom ./ubuntu-24.04.4-live-server-amd64.iso -boot d -net nic -net user -vga virtio -display sdl
```

3) Boot into the system

```
qemu-system-x86_64 -enable-kvm -m 4G -smp 4 -cpu host -hda kernel-dev.qcow2 -net nic -net user,hostfwd=tcp::2223-:22 -vga virtio -display sdl
```

(for me somehow `2222` was already being used by some process)

4) On the HOST, install sshfs and create a mounting point for local development

```
sudo apt install sshfs
mkdir ~/kerneldev-mount
sshfs jose@localhost:~/kernel-modules ~/kerneldev-mount -p 2223
```

5) On the VM, install the required development tools (for 6.8):

```
sudo apt update && sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev dwarves git linux-headers-$(uname -r)
```

6) On the VM, build and install the module:

```
make
sudo insmod hello_kernel
sudo rmmod hello_kernel
```

And check it in `dmesg`:

```
jose@kerneldev:~/kernel-modules$ sudo dmesg | tail -5
[    7.740393] audit: type=1400 audit(1772731226.046:107): apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/lib/snapd/snap-confine//mount-namespace-capture-helper" pid=903 comm="apparmor_parser"
[ 1376.791174] hello_kernel: loading out-of-tree module taints kernel.
[ 1376.791180] hello_kernel: module verification failed: signature and/or required key missing - tainting kernel
[ 1376.791337] [-] hello_kernel: module loaded!
[ 1412.785446] [-] hello_kernel: module unloaded!
```
