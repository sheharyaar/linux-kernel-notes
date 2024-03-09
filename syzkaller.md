- [Syzkaller](#syzkaller)
  - [Building Kernel for Syzbot](#building-kernel-for-syzbot)
  - [Build FS Image](#build-fs-image)
  - [Running QEMU](#running-qemu)
    - [Troubleshooting](#troubleshooting)
  - [Build Syzkaller](#build-syzkaller)
  - [References](#references)

# Syzkaller

Bugs found through dynamic analysis (fuzzing) are usually more critical than bugs found through static analysis. Fuzzing the kernel is nothing more than running it with different configurations and checking if it crashes at some point under certain circumstances

**Syzkaller** is a coverage-guided kernel fuzzer. It is a tool that generates system calls and checks if the kernel crashes. It is a very powerful tool to find bugs in the kernel. Tt generates **reports** from the bugs it finds, and sometimes even **reproducers** (programs that can be used to trigger the bugs).

**Syzbot**, which is a higher-level tool that continuously build kernels with different configurations on a number of architectures and deploys Syzkaller instances to test them.

You require the following to run Syzkaller:
- A kernel image
- A filesystem image (from Syzkaller)
- QEMU

## Building Kernel for Syzbot

- Build the kernel with Clang/LLVM, which will allow us to profit from a nice tool like `KMSAN` (not supported by gcc at the moment of writing). For other tools, you can build the kernel with gcc. Learn how to build the kernel with both gcc and llvm
  
1. Install `clang` and `llvm` according to your distribution. For example, in Ubuntu, you can install them with `sudo apt install clang llvm`.

2. Generate default configs
```shell
cd $KERNEL
make defconfig
make kvm_guest.config
```
Or if you want to specify a compiler.
```shell
cd $KERNEL
make CC="$GCC/bin/gcc" defconfig
make CC="$GCC/bin/gcc" kvm_guest.config
```

3. Enable required config options. It's not required to enable all of them, but at the very least you need:
```shell
# Coverage collection.
CONFIG_KCOV=y

# Debug info for symbolization.
CONFIG_DEBUG_INFO_DWARF4=y

# Memory bug detector
CONFIG_KASAN=y
CONFIG_KASAN_INLINE=y

# Required for Debian Stretch and later
CONFIG_CONFIGFS_FS=y
CONFIG_SECURITYFS=y
```

> Edit .config file manually and enable them (or do that through make menuconfig if you prefer).

4. Since enabling these options results in more sub options being available, we need to regenerate config:
```shell
make olddefconfig
```
Or if you want to specify a compiler.
```shell
make CC="$GCC/bin/gcc" olddefconfig
```

You might also be interested in disabling the Predictable Network Interface Names mechanism. This can be disabled either in the syzkaller configuration (see details here) or by updating these kernel configuration parameters:

```shell
CONFIG_CMDLINE_BOOL=y
CONFIG_CMDLINE="net.ifnames=0"
```

5. Build the kernel : `make -j$(nproc)`. Or if you want to specify a compiler : `make CC="$GCC/bin/gcc" -j$(nproc)`

6. Now you should have vmlinux (kernel binary) and bzImage (packed kernel image):
```shell
ls $KERNEL/vmlinux
ls $KERNEL/arch/x86/boot/bzImage
```

## Build FS Image

1. Install debootstrap: `sudo apt install debootstrap`
2. Create Debian Bullseye Linux image

Create a Debian Bullseye Linux image with the minimal set of required packages.
```shell
mkdir $IMAGE
cd $IMAGE/
wget https://raw.githubusercontent.com/google/syzkaller/master/tools/create-image.sh -O create-image.sh
chmod +x create-image.sh
./create-image.sh

# The result should be $IMAGE/bullseye.img disk image.
```

To create a Debian image with a different version (e.g. buster, stretch, sid), specify the --distribution option : `./create-image.sh --distribution buster`

3. Image extra tools : Sometimes it's useful to have some additional packages and tools available in the VM even though they are not required to run syzkaller. To install a set of tools we find useful do (feel free to edit the list of tools in the script): `./create-image.sh --feature full`

4. To install perf (not required to run syzkaller; requires $KERNEL to point to the kernel sources): `./create-image.sh --add-perf`

## Running QEMU

1. Make sure the kernel boots and sshd starts.
```shell
qemu-system-x86_64 \
	-m 2G \
	-smp 2 \
	-kernel $KERNEL/arch/x86/boot/bzImage \
	-append "console=ttyS0 root=/dev/sda earlyprintk=serial net.ifnames=0" \
	-drive file=$IMAGE/bullseye.img,format=raw \
	-net user,host=10.0.2.10,hostfwd=tcp:127.0.0.1:10021-:22 \
	-net nic,model=e1000 \
	-enable-kvm \
	-nographic \
	-pidfile vm.pid \
	2>&1 | tee vm.log
```
```log
early console in setup code
early console in extract_kernel
input_data: 0x0000000005d9e276
input_len: 0x0000000001da5af3
output: 0x0000000001000000
output_len: 0x00000000058799f8
kernel_total_size: 0x0000000006b63000

Decompressing Linux... Parsing ELF... done.
Booting the kernel.
[    0.000000] Linux version 4.12.0-rc3+ ...
[    0.000000] Command line: console=ttyS0 root=/dev/sda debug earlyprintk=serial
...
[ ok ] Starting enhanced syslogd: rsyslogd.
[ ok ] Starting periodic command scheduler: cron.
[ ok ] Starting OpenBSD Secure Shell server: sshd.
```

2. After that you should be able to ssh to QEMU instance in another terminal.
```shell
ssh -i $IMAGE/bullseye.id_rsa -p 10021 -o "StrictHostKeyChecking no" root@localhost
```
### Troubleshooting

- If this fails with "**too many tries**", ssh may be passing default keys before the one explicitly passed with -i. Append option `-o "IdentitiesOnly yes"`.

- To kill the running QEMU instance press **Ctrl+A and then X** or run:
```shell
kill $(cat vm.pid)
```

> If QEMU works, the kernel boots and ssh succeeds, you can shutdown QEMU and try to run syzkaller.

## Build Syzkaller

Build syzkaller as described [here](/docs/linux/setup.md#go-and-syzkaller).
Then create a manager config like the following, replacing the environment
variables `$GOPATH`, `$KERNEL` and `$IMAGE` with their actual values.

``` json
{
	"target": "linux/amd64",
	"http": "127.0.0.1:56741",
	"workdir": "$GOPATH/src/github.com/google/syzkaller/workdir",
	"kernel_obj": "$KERNEL",
	"image": "$IMAGE/bullseye.img",
	"sshkey": "$IMAGE/bullseye.id_rsa",
	"syzkaller": "$GOPATH/src/github.com/google/syzkaller",
	"procs": 8,
	"type": "qemu",
	"vm": {
		"count": 4,
		"kernel": "$KERNEL/arch/x86/boot/bzImage",
		"cpu": 2,
		"mem": 2048
	}
}
```

Run syzkaller manager:

``` bash
mkdir workdir
./bin/syz-manager -config=my.cfg
```

Now syzkaller should be running, you can check manager status with your web browser at `127.0.0.1:56741`.

If you get issues after `syz-manager` starts, consider running it with the `-debug` flag.
Also see [this page](/docs/troubleshooting.md) for troubleshooting tips.

## References 

- [Ubuntu host, QEMU vm, x86-64 kernel - syzkaller docs](https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md)
- [Fixing bugs in the Linux kernel with Syzbot, Qemu and GDB - Blog by Javier](https://javiercarrascocruz.github.io/syzbot)
- [A simple workflow to debug the Linux Kernel - Blog by Ricardo B. Marliere](https://marliere.net/posts/1/)
- [A better workflow for kernel debugging - Blog by Ricardo B. Marliere](https://marliere.net/posts/2/)