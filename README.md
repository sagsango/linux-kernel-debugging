# Linux Kernel Debugging
How to create a setup for linux kernel debugging using buildroot, qemu and gdb.

# Part 1: Compile linux kernel and rootfs
We are going to compile linux kernel and rootfs using buildroot.
Buildroot supplies all the toolchain which needed for automate the process of compiling linux kernel and rootfs.
Buildroot was created for creating linux embedded/minimal systems.
However, if your purpose is developing or debugging the linux kernel its really good solution.

First, Clone buildroot repository (latest version):

1. cd ~/workspace
2. git clone https://github.com/buildroot/buildroot.git    
   NOTE: current top commit id where I am building: 6b86f07
3. cd buildroot

## Config buildroot
Now we need to configure buildroot in order to build every packages with debug symbols.
In order to be able to ssh to the vm we'll add the openssh package.

4. make qemu_x86_64_defconfig // Generate buildroot default config
5. make menuconfig


| Option | Corresponding config symbols                                                                                             |
| ------ | -------------------------------------------------------------------------------------------------------------------------|
| In Build options, toggle “build packages with debugging symbols”          | `BR2_ENABLE_DEBUG`                                    |
| In System configuration, untoggle "Run a getty (login prompt) after boot" | `BR2_TARGET_GENERIC_GETTY`                            |
| In System configuration, enter root password                              | `BR2_TARGET_GENERIC_ROOT_PASSWD`                      |
| In Target packages, Networking applications, toggle "openssh"             | `BR2_PACKAGE_OPENSSH`                                 |
| In Filesystem images, change to ext4 root filesystem                      | `BR2_TARGET_ROOTFS_EXT2` & `BR2_TARGET_ROOTFS_EXT2_4` |

> The path to the options may change between buildroot versions, if an option is missing validate the symbols
> are set appropriately using `cat .config | grep <symbol>` from buildroot's folder 

#### Notes: 
If you want to tell buildroot to download and compile antoher version of linux kernel:
* In Toolchain, change “linux version” to <version_you_want>
* In Toolchain, change “Custom kernel version headers series” to <version_you_want>
* In Kernel, change “Kernel version" to <version_you_want>

## Config linux kernel
Now we are going to configure linux kernel in order to compile it with debug symbols.
Before opening the menuconfig it will trigger buildroot to download linux kernel source code.

6. make linux-menuconfig

| Option | Corresponding config symbols                                                                                             |
| ------ | -------------------------------------------------------------------------------------------------------------------------|
| In “Kernel hacking”, toggle “Kernel debugging”                                                            | `CONFIG_DEBUG_KERNEL` |
| In “Kernel hacking/Compile-time checks and compiler options”, toggle “Compile the kernel with debug info” | `DEBUG_INFO`          |
| In “Kernel hacking”, toggle “Compile the kernel with frame pointers”                                      | `FRAME_POINTER`       |
| In “Kernel hacking”, toggle “Provide GDB scripts for kernel debugging“                                    |                       |


## Compile linux kernel and rootfs
Now lets compile everything:

7. make -j8

Important files:

* output/build/linux-<version> contains the downloaded kernel source code
* output/images/bzImage is the compressed kernel image
* output/images/rootfs.ext4 is the rootfs
* output/build/linux-<version>/vmlinux is the raw kernel image

# Part 2: Debugging linux kernel using qemu and gdb
8. Run the qemu
   * $ pwd
   * /local_home/ssing214/public/buildroot/output/images
   * $ vim start-qemu.sh
   * add " -s -S" in the qemu cmd.
   * final output may look like this: exec qemu-system-x86_64 -M pc -kernel bzImage -drive file=rootfs.ext2,if=virtio,format=raw -append "rootwait root=/dev/vda console=tty1 console=ttyS0"  -net nic,model=virtio -net user  ${EXTRA_ARGS} -s -S "$@"
   * $ ./start-qemu.sh
10. Run the gdb
   * $ pwd
   * /local_home/ssing214/public/buildroot
   * $ gdb ./output/build/linux-6.6.32/vmlinux
   * <gdb> target remote :1234
   * <gdb> break start_kernel
11. Adding breakpoints for x86_64
   * https://syscalls.mebeim.net/?table=x86/64/x64/latest

## Start the debugging session
Now we are going to attach to our vm and the debug the kernel, we will also use our symbols to the kernel. 

12. cd output/build/linux-{version}
13. gdb ./vmlinux
14. target remote :1234

And now you got a kernel debugging session. 

**DONE!!!**
