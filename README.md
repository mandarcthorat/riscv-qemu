riscv-qemu [![Build Status](https://travis-ci.org/riscv/riscv-qemu.svg?branch=master)](https://travis-ci.org/riscv/riscv-qemu)
=========

**About:**

The `riscv64-softmmu` target for full system RV64G emulation is currently
supported.  It supports booting Linux from the `priv-1.9` branch of
[riscv-linux] and passes the compatibility tests from [riscv-tests].
A `riscv32-softmmu` target for full system RV32G emulation is also supported.
It currently passes all tests from [riscv-tests]. See Method 1 below.

Support for `riscv64-linux-user` and `riscv32-linux-user` is also present.
These pass the tests from [riscv-qemu-tests]. See Method 2 below.

**RISC-V Port Contributors:**

* Sagar Karandikar (sagark@eecs.berkeley.edu)
* Alex Suykov (alex.suykov@gmail.com)
* Bastian Koppelmann (kbastian@mail.uni-paderborn.de)

**Upstream QEMU Version:**

* 2.7.50, Last rebase: Sept 27, 2016
* *Note*: As we proceed with upstreaming, rebasing will happen regularly

**Privileged Specification Version:**

This version of QEMU adheres to the RISC-V v1.9 Privileged Specification as
described in [Technical Report No. UCB/EECS-2016-129](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-129.pdf)
and commit 745e74afb56ecba090669615d4ac9c9b9b96c653 ([the priv-1.9-rc0 release](https://github.com/riscv/riscv-tools/releases/tag/priv-1.9-rc0))
of riscv-tools.

Please note that QEMU tracks released drafts of the RISC-V Privileged
Specification, not work-in-progress changes like Spike does.

Installation
--------------

Prerequisites:

    $ sudo apt-get install gcc libc6-dev pkg-config bridge-utils uml-utilities zlib1g-dev libglib2.0-dev autoconf automake libtool libsdl1.2-dev

Jump to Method 1 if you want full-system simulation, or Method 2a/b for linux-user 
mode.

### Method 1 \(Full-System Simulation\):

####Step 1: Build QEMU

    $ git clone https://github.com/riscv/riscv-qemu
    $ cd riscv-qemu
    $ git submodule update --init pixman
    $ ./configure --target-list=riscv64-softmmu,riscv32-softmmu [--prefix=INSTALL_LOCATION]
    $ make
    $ [make install] # if you supplied prefix above

####Step 2: Obtain Images

You can build `vmlinux` from the `priv-1.9` branch of the [riscv-linux] repo and
create an initramfs for your root filesystem, then supply the resulting vmlinux
as a payload for bbl. Alternatively, you can use the prebuilt copy linked
below. This single file contains bbl with the Linux kernel as a payload. The
included copy of the Linux kernel also has an initramfs with busybox.

**a)** [bblvmlinuxinitramfs_dynamic](https://people.eecs.berkeley.edu/~skarandikar/host/qemu/1.9/bblvmlinuxinitramfs_dynamic)

####Step 3: Run QEMU

To boot Linux (assuming you are in the `riscv-qemu` directory):

    $ ./riscv64-softmmu/qemu-system-riscv64 -kernel bblvmlinuxinitramfs_dynamic -nographic

Notes about arguments:
* `-kernel bblvmlinuxinitramfs_dynamic`: This is the path to the binary to run. In this case, it contains the bbl bootloader, vmlinux, and an initramfs containing busybox.

Useful optional arguments:
* `-m 2048M`: Set size of memory, in this example, 2048 MB

<!--**IMPORTANT**: To cleanly exit this system, you must enter `halt` at the prompt
and then hit `ctrl-a x`. Otherwise, the root filesystem will likely be corrupted.-->

####Current limitations:

* The current RISC-V board definition provides only an HTIF console device.
Support for other HTIF-based devices has been removed from [riscv-linux]; as a
result, QEMU no longer supports them either.

### Method 2a \(Fedora 24 Userland with User Mode Simulation, Recommended\):

To avoid having to build the RISC-V toolchain and programs yourself, use Stefan O'Rear's [RISC-V Fedora Docker Image](https://hub.docker.com/r/sorear/fedora-riscv-wip/) to obtain a Fedora 24 Userland for RISC-V, packaged with riscv-qemu.

### Method 2b \(Manual User Mode Simulation\):

####Step 1: Build QEMU

    $ git clone https://github.com/riscv/riscv-qemu
    $ cd riscv-qemu
    $ git submodule update --init pixman
    $ ./configure --target-list=riscv64-linux-user,riscv32-linux-user [--prefix=INSTALL_LOCATION]
    $ make
    $ [make install] # if you supplied prefix above

####Step 2: Setup Compiler, Run a Program

You will need a compiler to build programs for RISC-V, as well as a sysroot
that contains the appropriate libraries. Follow the instructions in the README
of the [riscv-tools] repo (make sure you use the linked commit!) to build the
`riscv64-unknown-linux-gnu-gcc` compiler. `$RISCV` below refers to the
installation directory you are instructed to create in the aforementioned
README.

Now, build a hello world program with `riscv64-unknown-linux-gnu-gcc` and run
it like so:

    $ riscv64-unknown-linux-gnu-gcc hello.c -o hello
    $ ./riscv64-linux-user/qemu-riscv64 -L $RISCV/sysroot hello


Running RISC-V Tests on softmmu:
---------------------

A script (`run-rv-tests.py`) for running the RV64/RV32 tests from [riscv-tests]
is included in the `hacking_files` directory. All RV64/RV32 tests (listed in
`hacking_files/rv64-tests-list` and `hacking_files/rv32-tests-list`) are
expected to pass on their respective targets. 

Running RISC-V Tests on linux-user:
---------------------

Please see [riscv-qemu-tests].

Using QEMU to Debug RISC-V Code:
--------------------------------

QEMU works with RISC-V GDB to enable remote debugging.

To use this, start QEMU with the additional flags `-S -s`:

    $ ./riscv64-softmmu/qemu-system-riscv64 -S -s -kernel PROGRAM -nographic

This will start QEMU, but immediately pause and wait for a gdb connection.

Separately, start `riscv64-unknown-elf-gdb`:

    $ riscv64-unknown-elf-gdb [optional binary]

At the prompt, connect to QEMU:

    (gdb) target remote localhost:1234

At this point, you can use regular gdb commands to singlestep, set breakpoints,
read/write registers, etc. If you type `continue` in gdb, you can return to QEMU
and interact with the machine as if you were using it without GDB attached.

TODOs:
------

* See target-riscv/TODO

Notes
-----

- Files/directories of interest:
  - target-riscv/
  - hw/riscv/
  - linux-user/riscv

[riscv-linux]:https://github.com/riscv/riscv-linux/tree/priv-1.9
[Buildroot]:https://github.com/a0u/buildroot
[riscv-tests]:https://github.com/riscv/riscv-tests
[proxy kernel]:https://github.com/riscv/riscv-pk
[riscv-qemu-tests]:https://github.com/arsv/riscv-qemu-tests
[riscv-tools]:https://github.com/riscv/riscv-tools/tree/745e74afb56ecba090669615d4ac9c9b9b96c653