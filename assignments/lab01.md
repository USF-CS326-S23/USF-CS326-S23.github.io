---
layout: default
title: Lab01
nav_order: 1
parent: Assignments
permalink: /assignments/lab01
---

# Lab01 - Getting started with xv6

## Deliverables due Mon Jan 30 in your Lab01 GitHub repo

- A copy of xv6 with new user programs described below

## Overview

In this class we will be studying, modifying, and extending the xv6 Unix-like operating developed at MIT. Specifically, we will be using the latest RISC-V version of xv6. In this lab you will learn how to build and run xv6 using the Qemu machine emulator. You will also add some user-level programs to xv6. Here are some useful links:

[xv6-riscv on GitHub](https://github.com/mit-pdos/xv6-riscv)  
[xv6-riscv-book](book-riscv-rev3.pdf)

The xv6 operating system is a rewrite of Unix Version 6. Like most OS kernels, xv6 is written in C with some Assembly Language. The xv6 kernel and user programs total just under 10,000 lines of code, yet it is a fairly complete kernel supporting common Unix system calls, processes, virtual memory, and a modern file system. Originally, xv6 was written to work on Intel x86 processors, but it was ported a few years ago to RISC-V. RISC-V is a modern processor architecure that is open for public use, like the Linux kernel. At USF we started using RISC-V in CS 315 Computer Architecture in the Fall of 2022. Knowing RISC-V is not required, but is helpful in understanding some of the lower-level pieces of the kernel code. We will review aspects of RISC-V when looking at these parts of the kernel.

In order to compile and run xv6 you will need to have access to a RISC-V cross-compiler toolchain and Qemu. You can do all your xv6 work for this class using a dedicated USFCS lab machine we have for CS 326 called ```griffin.cs.usfca.edu```. This lab will explain how to use griffin for our xv6 work. Alternatively, you can install a toolchain and Qemu locally on your own machine. On macOS and Linux this is straightforward. On Windows it is more complicated. See below for more details. My recommentation is to get yourself setup on griffin, then explore setting up a local build environment.

## Console-based Editors

Given this is a systems course, you should dedicate yourself to becoming familiar with a good console-based editor. Such editors include vim, micro, and emacs (not nano). I personally use all three of these editors, but more recently I use micro the most. Knowing a console-based editor is especially important when you are working on a maching remotely using ssh.

## Getting started with xv6 on griffin

The ```griffin.cs.usfca.edu``` lab machine is an 8-core Intel-based machine with 32GB of RAM running AlmaLinux, a RedHat and Centos compatible version of Linux. You should access griffin using ssh from a terminal. If you are using Windows, you can install git-bash, WSL, or run Linux in a virtual machine. You can access it via ssh, but if you are not on the CSLabs network, you will need to ssh into ```stargate.cs.usfca.edu``` first, then ssh into ```griffin.cs.usfca.edu``` like this:

```text
% ssh benson@stargate.cs.usfca.edu
benson@stargate's password:
...
Picture, lots of text
...
[benson@stargate ~]$ ssh griffen.cs.usfca.edu
benson@griffin's password:
...
Picture, lots of text
...
Last login: Tue Jan 24 00:13:49 2023 from stargate.cs.usfca.edu
[benson@griffin ~]$
```

Note: on my Mac, my shell prompt is '%' because I'm using zsh. Your prompt might be '%' if you are using bash or another shell.

If you are going to use griffin often, then you don't want to have to type your password twice each time you want to login. We can use ssh keys and some ssh configuration so that we can just type:

```text
% ssh griffin
```

to login remotely.

## SSH Keys and Configuration

We can create an ssh keypair and put your public key on stargate, which also puts it on griffin because your home directory is shared on both machines. If you already have a keypair that you use, then you can skip the key creation step. However, it's not a bad idea to create a keypair specifically for use in this class. 

In a terminal on your computer, cd into your .ssh directory:

```text
% cd
% cd .ssh
```

Note that when you type cd with no arguments, this will put you into your home directory. If you don't have a .ssh directory, then you can create one:

```text
% mkdir .ssh
% chmod 700 .ssh
```

The chmod command restricts access to the .ssh directory and is needed by ssh to work properly. Now you can cd into your .ssh directory and create a new keypair:

```text
% cd .ssh
% ssh-keygen -t ed25519 -C "your_email@dons.usfca.edu" -f id_ed25519_cs326_2023s
```

You will be asked to enter a passphrase, which is a just a password, but you should make it multple words or a sentence that you can remember.

This will create two files:

```text
id_ed25519_cs326_2023s
id_ed25519_cs326_2023s.pub
```

You should change the permissions for these new files:

```
% chmod 600 id_ed25519_cs326_2023s*
```

Now we need to configure your local ssh by editing the ```.ssh/config``` file. If you already have a ```.ssh/config``` file, then you can add to it, otherwise you can create it using a text editor. You want to put the following into your ```.ssh/config```:

```text
IgnoreUnknown UseKeychain
UseKeychain yes

Host github.com
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs326_2023s
  User <your_username>

Host stargate
  HostName stargate.cs.usfca.edu
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs326_2023s
  User <your_username>

Host griffin
  HostName griffin
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs326_2023s
  User <your_username>
  ProxyCommand ssh -W %h:%p stargate
```

Notes:
- UseKeychain only applies to macOS
- The github.com entry will be used to securing access your GitHub repos
- Make sure to put your username where you see <your_username>

Once you have editor or created ```.ssh/config```, set its permissions:

```text
% chmod 600 config
```

Now you have everything ready on your local machine. We need to add your public key to your ```.ssh/authorized_keys``` file on stargate. You can do this using the ```ssh-copy-id``` command:

```text
% ssh-copy-id -i ~/.ssh/id_ed25519_cs326_2023s <username>@stargate.cs.usfca.edu```

If all goes well, you should now be able to ssh into stargate or griffin by just typing:

```text
% ssh stargate
```
or
```text
% ssh griffin
```

You will be asked to enter your passphrase the first time you connect to either machine.

## Adding your key to GitHub 

You can add your new key to GitHub by going to ```github.com``` in a browser. To add your key, follow the directions [here](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account). This will allow you to access your GitHub repos via ssh from the command line.

## Learn a Console-based editor: micro

The micro editor is a small but powerful console-based editor with intuitive key commands. We have micro installed on griffin. To edit files, just type:

```text
$ micro <filename>
```

Here is a quick summary of commands:
```text
CTRL-Q - quit
CTRL-S - save
Shift-Arrow - select text
CTRL-C - copy selection
CTRL-X - cut selection
CTRL-V - paste selection
CTRL-F - find
```

You can learn more about micro and how to configure it [here](https://micro-editor.github.io).

## Configure your PATH on griffen

In order to compile xv6, you need to put the cross-compiler toolchain binaries on your PATH. In order to do this, you need to edit your ```~/.bash_profile``` on griffin:

```text
% ssh griffin
% micro .bash_profile
```

Then add the following:

```text
PATH=$PATH:/opt/riscv/bin
```

Logout of griffin and then back in to get the updated PATH environment variable. You can check if your PATH is correct by typing and then getting:

```text
$ riscv64-unknown-linux-gnu-gcc -v
Using built-in specs.
COLLECT_GCC=riscv64-unknown-linux-gnu-gcc
COLLECT_LTO_WRAPPER=/opt/riscv/libexec/gcc/riscv64-unknown-linux-gnu/12.2.0/lto-wrapper
Target: riscv64-unknown-linux-gnu
Configured with: /home2/afedosov/riscv-gnu-toolchain/gcc/configure --target=riscv64-unknown-linux-gnu --prefix=/opt/riscv --with-sysroot=/opt/riscv/sysroot --with-pkgversion=g2ee5e430018 --with-system-zlib --enable-shared --enable-tls --enable-languages=c,c++,fortran --disable-libmudflap --disable-libssp --disable-libquadmath --disable-libsanitizer --disable-nls --disable-bootstrap --src=/home2/afedosov/riscv-gnu-toolchain/gcc --disable-multilib --with-abi=lp64d --with-arch=rv64imafdc --with-tune=rocket --with-isa-spec=2.2 'CFLAGS_FOR_TARGET=-O2   -mcmodel=medlow' 'CXXFLAGS_FOR_TARGET=-O2   -mcmodel=medlow'
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 12.2.0 (g2ee5e430018)
```

## Clone xv6

For labs and projects, I will provide initial starter xv6 code, but for now you can clone your own copy of xv6 to compile and run. Because you will be creating several copies of xv6 it will take quite a bit of disk space. You will not have enough room in your normal home directory to do all of the labs and projects for this class. Instead, you have access to a locally mounted disk for your work.

```text
$ cd /home2/<username>
$ mkdir test-xv6 
$ cd test-xv6
$ git clone git@github.com:mit-pdos/xv6-riscv.git
```

Now, you can cd into the xv6-riscv directory and compile and run xv6:

```text
$ cd xv6-riscv
$ make qemu
```

After lots of file are compiled, you should see:

```text
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -global virtio-mmio.force-legacy=false -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$
```

You can try some basic command in xv6:

```text
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2305
cat            2 3 32816
echo           2 4 31696
forktest       2 5 15816
grep           2 6 36256
init           2 7 32168
kill           2 8 31616
ln             2 9 31432
ls             2 10 34760
mkdir          2 11 31672
rm             2 12 31664
sh             2 13 54240
stressfs       2 14 32544
usertests      2 15 180304
grind          2 16 47424
wc             2 17 33744
zombie         2 18 31032
console        3 19 0
$ wc README
49 325 2305 README
```

To quit xv6 (actually Qemu), type:

```text
CTRL-a x
```

That is type ```CTRL-a``` then ```x```.

And you should see something like:

```text
$ QEMU: Terminated
$
```

## Adding new user programs to xv6

Once you have everything setup so you can access griffin and build/run xv6, you can now added new user-level programs to xv6. You will add two programs to xv6: ```hello.c``` and ```sumargs.c```.

Consider ```hello.c```:

```text
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  printf("Hello, xv6!\n");
  exit(0);
}
```

Add this file to the user directory in the xv6 repo. Then edit the Makefile in the root directory of the xv6 repo and look for the UPROGS variable:

```text
UPROGS=\
        $U/_cat\
        $U/_echo\
        $U/_forktest\
        $U/_grep\
        $U/_init\
        $U/_kill\
        $U/_ln\
        $U/_ls\
        $U/_mkdir\
        $U/_rm\
        $U/_sh\
        $U/_stressfs\
        $U/_usertests\
        $U/_grind\
        $U/_wc\
        $U/_zombie\
```

Add ```$U/_hello\``` after ```$U/_grep\``` to keep the list in alphabetical order.

Once you've added ```hello.c``` and modified the Makefile, you should be able to type ```make qemu``` then run ```hello``` the xv6 shell prompt.

Now write a programc called ```sumargs.c``` that will sum up all of the integer arguments provided on the command line:

```text
$ sumargs 1 2 3 4 5
15
```

