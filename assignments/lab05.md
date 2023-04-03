---
layout: default
title: Lab05
nav_order: 7
parent: Assignments
permalink: /assignments/lab05
---

# Lab05 Process Status (ps)

## Deliverables due Thu Apr 6 by 11:59pm in your lab05 GitHub repo

- A copy of xv6 with your modified implementation of ps
- Your implemenation should pass all of the Lab05 Autograder tests.
- Your source code should conform to xv6 formatting conventions.
- For this lab you need to include your output for ```ps a``` (see below).
- Your Lab05 repo should not have any extraneous files or build artifacts
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.

# Contents
1. [Overview](#overview)
2. [Process Status](#process-status)
3. [Implementation](#implementation)

## Overview

The Process Status (ps) command enables users to see all of the currently exeucting processes on a machine. The xv6 kernel provides a direct keyboard shortcut (CTRL-P) to see a list of the currently running processes. While this was intended as quick way to see all processes, it has limitations. With the keyboard shortcut, there is no way to provide command line options to control the information provided in the process listing. Also, because the shortcut causes the kernel to print the process list directly to the console device, you cannot redirect the output of ps to a file or via a pipe to another process (like grep).

We are going to build a proper user-level ps command for xv6 that requires us to implement a new system call similar to the ```fstat()``` system call to get file status information from the kernel. The in addition to learning about ps functionality, the goal of this lab is to get your familiar with implementing system calls and how send arguments and data from user level to kernel level and vice versa.

### Process Status

Here is the basic output of our new ps command:

```text
$ ps
PID=1 NAME=init SIZE=24576
PID=2 NAME=sh SIZE=28672
PID=3 NAME=ps SIZE=24576
```

We will use this ```field=value``` form to keep formatting simple.

You will support the ```a``` option to print more information about the running processes:

```text
$ ps a
PID=1 NAME=init SIZE=24576 STATE=sleep NSCHED=22 NTICKS=0
PID=2 NAME=sh SIZE=28672 STATE=sleep NSCHED=24 NTICKS=1
PID=4 NAME=ps SIZE=24576 STATE=run NSCHED=2 NTICKS=1
```

In this case we can see how many times a process has been selected by the schedule to run (```NSCHED```) and the number of ticks a process has executed (```NTICKS```). I've provided a new test program called ```busymany``` that makes it easy to create several processes that will run for a specified number of ticks:

```text
busymany <NTICKS> <NPROCESSES>
```

Here are some examples:

```text
init: starting sh
$ busymany 200 10
$ ps
PID=1 NAME=init SIZE=24576
PID=2 NAME=sh SIZE=28672
PID=14 NAME=ps SIZE=24576
PID=4 NAME=busy[A] SIZE=24576
PID=5 NAME=busy[B] SIZE=24576
PID=6 NAME=busy[C] SIZE=24576
PID=7 NAME=busy[D] SIZE=24576
PID=8 NAME=busy[E] SIZE=24576
PID=9 NAME=busy[F] SIZE=24576
PID=10 NAME=busy[G] SIZE=24576
PID=11 NAME=busy[H] SIZE=24576
PID=12 NAME=busy[I] SIZE=24576
PID=13 NAME=busy[J] SIZE=24576
```

```text
init: starting sh
$ busymany 200 10
$ ps a
PID=1 NAME=init SIZE=24576 STATE=sleep NSCHED=23 NTICKS=0
PID=2 NAME=sh SIZE=28672 STATE=sleep NSCHED=24 NTICKS=1
PID=14 NAME=ps SIZE=24576 STATE=run NSCHED=17 NTICKS=1
PID=4 NAME=busy[A] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=5 NAME=busy[B] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=6 NAME=busy[C] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=7 NAME=busy[D] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=8 NAME=busy[E] SIZE=24576 STATE=sleep NSCHED=29 NTICKS=0
PID=9 NAME=busy[F] SIZE=24576 STATE=sleep NSCHED=29 NTICKS=0
PID=10 NAME=busy[G] SIZE=24576 STATE=sleep NSCHED=29 NTICKS=0
PID=11 NAME=busy[H] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=12 NAME=busy[I] SIZE=24576 STATE=sleep NSCHED=28 NTICKS=0
PID=13 NAME=busy[J] SIZE=24576 STATE=sleep NSCHED=29 NTICKS=0
```
Note that ```NTICKS=0``` in this example is ok. The busymany program "sleeps" while busy waiting, so the processes are not consuming CPU time or very little.

Note that the values of NSCHED and NTICKS are not deterministic, so I can't provide autograder tests. Instead you will submit two output files in your lab04.

```text
$ python3 runxv6.py "ps a" > ps1.out
$ python3 runxv6.py "busymany 200 10" "ps a" > ps2.out
```
Add these to your xv6 directory in your lab05 GitHub Repo.

### Implementation

The lab05 starter code contains all the code we have developed so far in class. I've created the lab05-starter repo by first committed a fresh version of xv6-riscv, then I added files and made modifications in further commits. In this way you can see how we evolved the xv6 code base. In particular here are the things that have been added:

- ```busy.c``` - our first version of process busy waiting for testing
- ```busymany.c``` - a extended version of busy.c that allow multiple processes to be created at once. This version uses busy sleeping instead of busy waiting so the processes do not consume CPU time, which can slow the system down with many processes.
- New systems calls:
  - ```sname(const char *name)``` - change the name of the currently running process. This is used by ```busymany.c``` so we can differentiate the processes create in the ps output.
  - ```uproc(int pid, struct uproc *up_p)``` - This the system call we developed in class to retrieve process information by pid.
- ```uproc.c``` - a user-level program that calls the new ```uproc()``` to get and print process information given a pid as a command line argument.

Your job will be to implement ```ps.c``` that provides the output given above. To do this you need to create a new user-level command in the ```user``` directory called ```ps.c```. You can start with the ```uproc.c``` program. To implement ```ps.c``` you will need to implement a new system call, ```sproc()```. The goal of ```sproc()``` will be to retrieve process information using the ```uproc struct```.

You have two choices for implementing ```sproc()```:

1. ```int sproc(int slot, struct uproc *up_p)```
In this version, the ```sproc()``` system call will retrieve process information at index ```slot``` in the proc table using ```struct uproc *up_p```. That is, this will retrieve information about one process and your ```ps.c``` implementation will need to call ```sproc()``` ```NPROC``` times (64 by default) to get information about every process.
2. ```int sproc(struct uproc *up_p)``` In this version, ```up_p``` is a pointer to a user-level ```struct uproc array```. In this system call, the kernel will populate the entire user-level arrar in one call. In this way, ```ps.c``` can iterate over the local ```uproc``` array to print information about every process.

Both versions have pros and cons. The first version is more like the ```uproc()`` system call we developed in class, but it introduces a lot of extra overhead because we need to issue 64 system calls to get all the process information. The first version could potentially display process information that is changing as ps is running. The second version, is faster because only one system call is issued. However, the second version requires enough memory for the entire process array. For 64 processes this is not a big deal, but if we had 10,000 processes it would be a different story.

Note you shoud follow the steps we have developed in class for adding the new ```sproc()``` system call:
1. [USER] Add the ```sproc()``` prototype to ```user/user.h```
  - The specific prototype will depend on the ```sproc()``` version you choose to implement. It will either be ```int sproc(int slot, struct uproc *up_p)``` or ```int sproc(struct uproc *up_p)```. This allows user programs to see the correct type signature for the ```sproc()``` system call so the compiler can generate the correct code to call the ```sproc()``` function implemented in RISC-V assembly language.
2. [USER] Add a new entry for ```sproc``` in user/usys.pl:
  - ```entry("sproc")```
  - This allows the ```usys.pl``` Perl script to generate the assembly code for the new ```sproc()``` system call.
3. [KERNEL] In ```kernel/syscall.h``` add a new system call number for ```sproc()```:
  - ```#define SYS_sproc 25```
  - If you have other systems calls, the actually number you pick may be different than 25.
4. [KERNEL] In ```kernel/syscall.c``` you need to make two modifications:
  - Add ```extern uint64 sys_sproc()``` after the other externs for the other system calls.
  - Add an entry to the ```syscalls[]``` array at the end: ```[SYS_sproc]    sys_sproc,```
5. [KERNEL] In ```kernel/sysproc.c``` add the ```sys_sproc()``` function. You can base this off of the ```sys_uproc()``` function in the same file. Note that it's implementation will depend on how you decide to implement ```sproc()```. It will either grab the slot number in arg 0 like ```sys_uproc()``` using ```argint()``` and then get the pointer to the user-level ```uproc struct``` using ```argaddr()```. Otherwise it will just get the pointer to the user-level ```uproc struct``` using ```argaddr()```.
6. [KERNEL] In ```kernel/proc.c``` Add a new ```sproc()``` function based on the existing ```uproc()``` function implementation. Again, how you implement this will depend on your choice of reading one slot at a time versus reading all the slots at once.
7. [KERNEL] In ```kernel/defs.h``` in the ```// proc.c``` section add your prototype for ```sproc()```. This is needed so that the ```sysproc.c:sys_sproc()``` function can know the type signature for ```sproc()```.

***IMPORTANT*** You should not modify the existing ```uproc()``` system call. You need to leave this as is and add a new ```sproc()``` system call.

After you have all this implemented you can start coding your ```user/ps.c``` based on the given ```user/uproc.c``` code. Note that you may find as you are coding ```ps.c``` you find problems in your ```sproc()``` system call implementation. This is normal and it will be an iterative process.



