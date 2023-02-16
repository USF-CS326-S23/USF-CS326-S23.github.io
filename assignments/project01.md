---
layout: default
title: Project01
nav_order: 4
parent: Assignments
permalink: /assignments/project1
---

# Project01 Heap Allocator

## Deliverables due Mon Feb 27 by 11:59pm in your Project01 GitHub repo

- A copy of xv6 with your new implementation of ```malloc()/free()```.
- Your implemenation should pass all of the ```memtest``` tests from the Project01 Autograder tests.
- You source code should conform to xv6 formatting conventions.
- Your Project01 repo should not have any extraneous files or build artifacts

# Contents
1. [Overview](#overview)
2. [Preliminaries](#preliminaries)
3. [Heap Allocator](#heap-allocator)


## Overview

To get a better idea of how user-level heap memory is managed and how a user-level process can request more memory from the kernel you will implement your own versions of ```malloc()``` and ```free()``` that conform the the requirements specified below. Note, that xv6 comes with its own implemenations of ```malloc()``` and ```free()``` which come from the book "The C Programming Language", 2nd Edition by Dennis Ritchie and Brian Kernighan. In the project01 starter code I've moved the xv6 implemenation of ```malloc()/free()``` to ```umalloc_xv6.c```. You are welcome to study this implemenation, but we will be doing things a bit differently. For one, the current implemenation uses a singly linked list for keeping track of free blocks, we are going to use our doubly linked list implemenation that we used in lab02 and lab03. We are also going to add code and data to make it easier to visualize used and free blocks.

## Preliminaries

When xv6 starts a user-level process the memory layout of the process looks like this:

```text
HEAP (0)
STACK (4096)
GUARD (4096)
DATA (multiple of 4096)
CODE (multiple of 4096)
```

See Figure 2.3 on page 26 and Figure 3.4 on page 36 in the [xv6 Book]((assignments/book-riscv-rev3.pdf)). The ```CODE``` section starts at virtual address 0 and is a multiple of 4096 bytes. 4096 is the native page size of the RISC-V processor and is a common page size in other computer processors. This means that when mapping virtual addresses, the processor and the kernel do so in multiples of 4096. The ```DATA``` will start on a 4096 page and will also be a multiple of 4096. The ```GUARD``` page is unmapped and will help catch programs that try to use too much stack space (that is more that 4096 bytes). The ```STACK``` section also starts on a multiple of 4096 and is 4096 bytes in size. Initially, there is no actual HEAP memory. If a user process needs heap memory it must request memory from the kernel.

### ```sbrk()```

The xv6 kernel, as well as other Unix kernel, provides the ```sbrk()``` system call to request more memory for a process and this memory is generally used for heap memory. Here is the prototype for ```sbrk()```:
```c
char* sbrk(int nbytes);
```
The ```nbytes``` argument is the number additional bytes of memory you want the kernel to extend the process's address space with usable memory. The name ```sbrk()``` comes from "set break" where the "break" is the end of the processes address space. Interestly, ```nbytes``` can both be a positive value (grow the process address space) and a negative value (shrink the process address space). In practice, it is difficult to shrink the address space of a process because once a C program has a pointer to memory, the memory must exist as long as the code thinks it should be allocated. That is, in C, we do not have any form of garbage collection.  The return value of ```p = sbrk(nbytes)``` is a pointer (address) to the beginning of the allocated region of memory, and all addresses from p to p + nbytes are valid.

Given this description of ```sbrk()``` we can create a very simple implementation of ```malloc()/free()``` that uses ```sbrk()``` for new ```malloc()``` requests, but free does nothing. That is, ```free()``` won't actually free up any used memory:

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/param.h"

void
free(void *ap)
{
  /* This allocator does not support freeing allocated memory */
  return;
}

void*
malloc(uint nbytes)
{
  char *p;
  p = sbrk(nbytes);
  return p;
}
```

This will work, but is very wasteful and a long running program could run out of memory. Instead, we want to request heap memory from the kernel, but manage this heap memory so that memory blocks that are freed can eventually be reused by future calls to ```malloc()```. So, in order to do this, we need a way to keep track of free memory and used memory. When we need more memory from the kernel, we will call ```sbrk()```. When we free memory, we not only want to mark the block as free, but also see if this free block can be merged with neighboring free blocks to create a larger free block.

### ```sbrksz()```

To support testing of your memory allocator, I've added a new system call to xv6:
```c
long sbrksz(void);
```
This system call will return the current size of the process address space, the "break". This will be used by some of the test to make sure you are not requesting more memory from the kernel than necessary for a particular sequence of allocations and deallocations. For example, consider the following code snippet:
```c
int s1, s2, diff;
s1 = sbrksz();
sbrk(16)
s2 = sbrksz();
diff = s2 - s1;
```
In this case diff should be 16.

### ```sh.c Modifcations```

The started code for Project01 inludes a modified ```sh.c``` to allow more than the default of 10 arguments passed to a commond. This has been increased to 512 to support the heap allocator test program (```memtest.c```) described below.

### ```param.h Modifcation```

I increased the MAXARG macro from 32 to 64 to allow for more command ling arguments to be passed to the ```memtest``` program.

### ```user.h Modification```

I added a ```print_debug()``` macro to help with debug messages:

```c
#define DEBUG 0
#define debug_print(fmt, ...) \
    do { if (DEBUG) fprintf(2, "%s:%d:%s(): " fmt, __FILE__, \
         __LINE__, __func__, __VA_ARGS__); } while (0)
```
In order to use this, just add something like:
```c
debug_print("heap size request = %d\n", heap_req_size);
```
Then change ```DEBUG``` to 1:
```c
#define DEBUG 1
```

## Heap Allocator

The basic idea behind our heap allocator is that we will request memory from the kernel (via ```sbrk()```) as needed to extend the heap space. We will manage all the heap space that we allocate from the kernel. That is, we will keep track of used memory and free memory. We will try to minimize calls to ```sbrk()``` by looking for free blocks in the existing heap space and only extend the heap if we cannot find a free block. As blocks are freed we need to see if we can combine neighboring free blocks to create a larger free block.
Here are some basic parameters we will work with:

- The minimum size of a free block will be 4 bytes.
- The minimum size for a request to ```sbrk()``` will be 4096.
- If we need more than 4096 bytes, they you need to request a multple of 4096 bytes from ```sbrk()```
- That is, the process size as returned by ```sbrksz()``` should always be a multiple of 4096 (this is called an invariant).
- When a ```free(p)``` call is made, you should merge the new free block with neighboring free blocks.
- If a call to ```malloc(n)``` requires a call to ```sbrk()``` you should factor the space in a free block that is at the end of the current heap space.

### Memory Blocks

To keep track of memory blocks, we are going to use a memory block header:

```c
struct block_hdr {
  struct list_elem elem;
  char name[32];
  int used;
  int size;
};
```
Every block of memory that we manage will have a block header. Since we are implementing ```malloc()/free()```, we can't use ```malloc()/free()``` for block headers. Instead we will embed the block headers in the heap space. That is, each block of memory will have a header associated with it. So, a block will have a header and then the usable space in the block.

