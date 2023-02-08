---
layout: default
title: Lab03
nav_order: 3
parent: Assignments
permalink: /assignments/lab03
---

# Lab03 - Adding more xv6 Commands

## Deliverables due Mon Feb 13 by 11:59pm in your Lab03 GitHub repo

- A copy of xv6 with all of your lab02 commands: ```wc```, ```cat```, and ```sort```
- Three additional commands: ```head```, ```tail```, and ```seq```
- Your Lab03 should be able to pass all the Lab02 Autograder tests
- Your Lab03 should be able to pass all the Lab03 Autograder tests
- You source code should conform to xv6 formatting conventions
- Your Lab03 repo should not have any extraneous files or build artifacts

# Contents
1. [Overview](#overview)
2. [Preliminaries](#preliminaries)
3. [head](#head)
4. [tail](#tail)
5. [seq](#seq)


## Overview

To get more practice writing user-level Unix commands you are going to implement three more useful commands: ```head```, ```tail```, and ```seq```. See below for the details of each of the these commands. For each of these you need to add a new source file to the xv6 user directory and update the xv6 Makefile to have these commands included in the xv6 file system.

## Preliminaries

The starter code for Lab03 is the same as the starter code for Lab02, with the following exceptions:
- The Makefile now turns off compiler optimizations in CFLAGS to allow you to use gdb for debugging your programs (```-O0```)
- The Makefile includes ```$U/printf.o``` in the rule for forktest so that you can put ```readline()``` in ```ulib.c```

### Putting ```readline()``` in ```ulib.c```

Now that we have several programs using the ```readline()``` function in doesn't make sense to have multiple copies of this function in multiple files. You will put it into ```ulib.c``` to make it a library function. To do this, simply add a copy of ```readline()``` to the end of the ```user/ulib.c``` file. Next you need to add a prototype for ```readline()``` to the ```user/user.h``` file at the end:
```
int readline(int, char *, int);
```
After you do this, you can remove ```readline()``` from ```cat.c``` and ```sort.c```. You will also be using ```readline()```  in ```head.c``` and ```tail.c```.

### Update ```cat -n```

I have added tests in lab03 that more fully tests ```cat -n```. That is, the tests in lab02 only show 9 lines or less. You may have to update you implementation of ```cat -n``` to handle line numbers greater than 9.

## head 
[**EASY**]

The head command show the first ```count``` lines of a text file or stdin, where ```count``` is 10 by default:
```
$ head README
xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
Version 6 (v6).  xv6 loosely follows the structure and style of v6,
but is implemented for a modern RISC-V multiprocessor using ANSI C.

ACKNOWLEDGMENTS

xv6 is inspired by John Lions's Commentary on UNIX 6th Edition (Peer
to Peer Communications; ISBN: 1-57398-013-7; 1st edition (June 14,
2000)).  See also https://pdos.csail.mit.edu/6.1810/, which provides
pointers to on-line resources for v6.
$
```
This is really just a variation of ```cat``` and you can start with your Lab02 ```cat.c``` source code. The ```head``` has a ```-n count``` to show the first ```count``` lines of a file:
```
$ head -n 3 README
xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
Version 6 (v6).  xv6 loosely follows the structure and style of v6,
but is implemented for a modern RISC-V multiprocessor using ANSI C.
$
```
Just like ```cat```, ```head``` can accept multiple file names as arguments. When more than one file is provided, the output includes a header to indicate the following lines are from the specified file:
```
$ cat > foo.txt
foo
bar
baz
$ head -n 2 README foo.txt
== > README <==
xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
Version 6 (v6).  xv6 loosely follows the structure and style of v6,
== > foo.txt <==
foo
bar
$
```
Also, just like our other commands, if no filenames are provided, ```head``` reads from stdin (0):
```
$ cat README | head -n 3
xv6 is a re-implementation of Dennis Ritchie's and Ken Thompson's Unix
Version 6 (v6).  xv6 loosely follows the structure and style of v6,
but is implemented for a modern RISC-V multiprocessor using ANSI C.
$
```

## tail 
[**MODERATE**]

The ```tail``` command is the opposite of the ```head``` command. It prints the last ```count``` lines of a file or stdin, where ```count``` is 10 by default:
```
$ tail README
(kaashoek,rtm@mit.edu).  The main purpose of xv6 is as a teaching
operating system for MIT's 6.1810, so we are more interested in
simplifications and clarifications than new features.

BUILDING AND RUNNING XV6

You will need a RISC-V "newlib" tool chain from
https://github.com/riscv/riscv-gnu-toolchain, and qemu compiled for
riscv64-softmmu.  Once they are installed, and in your shell
search path, you can run "make qemu".
$
```
Tail can also accept multiple file names and read from stdin and accept the option ```-n count``` to print the last ```count``` lines:
```
$ cat README | tail -n 3
https://github.com/riscv/riscv-gnu-toolchain, and qemu compiled for
riscv64-softmmu.  Once they are installed, and in your shell
search path, you can run "make qemu".
$
```

You are required to use the provided linked list implementation for you ```tail.c``` soluiton. You can start with your Lab02 ```sort.c``` implementation. The basic idea is the following:
- Read each line of the file one at a time
- Create a list of ```count``` lines
  - That is, once you have read ```count```, adding the next line will remove the first line from the list
- At the end of reading all the lines in the file you end up with a list of ```count``` lines or less (in the case where the file has less than ```count``` lines)
- Print the resulting list
- Free all the elements of the list (you want to do this because you may be processing multiple files and you could end up using a lot of memory)

Note that a real implementation of ```tail``` would not need to read every line in the file in order to print the last ```count``` lines. Instead, real Unix/Linux kernels support an ```lseek()``` function that can move the file position to the end and then read backward to find the last ```count``` lines. We will examine how to add support for this to xv6 later in the semester.

## seq
[**MODERATE**]

The ```seq``` command is a useful program that simply generates a list of numbers:
```
$ seq 5
1
2
3
4
5
$
```
The full usage looks like this:
```
$ seq
usage: seq [-p prefix] last
       seq [-p prefix] first last
       seq [-p prefix] first inc last
$
```
When you just provide one number (```last```), ```seq``` will print 1 to ```last```. You can provide two numbers, in which case ```seq``` will print the numbers from ```first``` to ```last```:
```
$ seq 100 105
100
101
102
103
104
105
$
```
Finally, if you provide three numbers ```seq``` will print the numbers starting at ```first```, then skip by ```inc```, until ```last``` is reached. Note that ```first``` can be greater than ```last``` and ```inc``` can be negative:
```
$ seq 30 -2 20
30
28
26
24
22
20
$
```
Note that the ```atoi()``` function in xv6 does not handle negative numbers. So, you will have to handle a negative ```inc``` string value as a special case when parsing the command line arguments.

Finally, you can provide the optional ```-p prefix``` arguments to add a string prefix to each number:
```
$ seq -p foo_ 5
foo_1
foo_2
foo_3
foo_4
foo_5
```
Putting most of our commands together:
```
$ seq -p id_ 100 -1 0 | head -n 20 | tail -n 5 | sort | cat -n
     1  id_81
     2  id_82
     3  id_83
     4  id_84
     5  id_85
$
```
And all of them:
```
$ seq -p id_ 100 -1 0 | head -n 20 | tail -n 5 | sort | cat -n | wc -l
5
$
```
Pretty impressive. The creators UNIX came up with a pretty nice way to allow commands to interact though pipe redirection.
