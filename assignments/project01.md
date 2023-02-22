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
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.

# Contents
1. [Overview](#overview)
2. [Preliminaries](#preliminaries)
3. [Heap Allocator](#heap-allocator)
4. [Grading Rubric](#grading-rubric)
5. [Code and Repo Quality](#code-and-repo-quality)
6. [Autograder Tests](#autograder-tests)

## Overview

To get a better idea of how user-level heap memory is managed and how a user-level process can request more memory from the kernel you will implement your own versions of ```malloc()``` and ```free()``` that conform the the requirements specified below. Note, that xv6 comes with its own implemenations of ```malloc()``` and ```free()``` which come from the book "The C Programming Language", 2nd Edition by Dennis Ritchie and Brian Kernighan. In the project01 starter code I've moved the xv6 implemenation of ```malloc()/free()``` to ```umalloc_xv6.c```. You are welcome to study this implemenation, but we will be doing things a bit differently. For one, the current implemenation uses a singly linked list for keeping track of free blocks, we are going to use the doubly linked list implemenation that we used in lab02 and lab03. We are also going to add code and data to make it easier to visualize used and free blocks.

## Preliminaries

When xv6 starts a user-level process the memory layout of the process looks like this:

```text
HEAP (0)
STACK (4096)
GUARD (4096)
DATA (multiple of 4096)
CODE (multiple of 4096)
```

See Figure 2.3 on page 26 and Figure 3.4 on page 36 in the [xv6 Book]((assignments/book-riscv-rev3.pdf)). The ```CODE``` section starts at virtual address 0 and is a multiple of 4096 bytes. 4096 is the native page size of the RISC-V processor and is a common page size in other computer processors. This means that when mapping virtual addresses, the processor and the kernel do so in multiples of 4096. The ```DATA``` will start on a 4096 page and will also be a multiple of 4096. The ```GUARD``` page is unmapped and will help catch programs that try to use too much stack space (that is more than 4096 bytes). The ```STACK``` section also starts on a multiple of 4096 and is 4096 bytes in size. Initially, there is no actual HEAP memory. If a user process needs heap memory it must request memory from the kernel.

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

This will work, but is very wasteful and a long running program could run out of memory. Instead, we want to request heap memory from the kernel, but manage this heap memory so that memory blocks that are freed can eventually be reused by future calls to ```malloc()```. In order to do this, we need a way to keep track of free memory and used memory. When we need more memory from the kernel, we will call ```sbrk()```. When we free memory, we not only want to mark the block as free, but also see if this free block can be merged with neighboring free blocks to create a larger free block.

### ```sbrksz()```

To support testing of your memory allocator, I've added a new system call to xv6:
```c
long sbrksz(void);
```
This system call will return the current size of the process address space, the "break". This will be used by some of the tests to make sure you are not requesting more memory from the kernel than necessary for a particular sequence of allocations and deallocations. For example, consider the following code snippet:
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

I increased the MAXARG macro from 32 to 64 to allow for more command line arguments to be passed to the ```memtest``` program.

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
- When looking for a free block, you should use a "first fit" approach. That is, you should find the first free block that can accommodate the ```malloc()``` request.
- When splitting a free block, you should convert the free block to a used block, then create a new free block in the remained space of the original free block, then add this to the block list. Another way to say this is that when splitting a free block, the used block should go first and the new free block should come second, right after the the new used block. 

### Memory Blocks

To keep track of memory blocks, we are going to use a memory block header:

```c
struct block_hdr {
  struct list_elem elem;
  char name[8];
  int used;
  int size;
};
```
Every block of memory that we manage will have a block header. Since we are implementing ```malloc()/free()```, we can't use ```malloc()/free()``` for block headers. Instead we will embed the block headers in the heap space. That is, each block of memory will have a header associated with it. So, a block will have a header and then the usable space in the block.

## Grading Rubric

Your Project01 score will come from two parts: functionaly that is determined by passing the autograder tests and repo and code quality.

| Rubric Item | Score Percentage |
|-------------|------------------|
| Tests       | 70%              |
| Quality     | 30%              |


You should plan to complete the project on time. If you don't complete functionality on team, that is, if you don't pass all the tests, push what you have working. You will have one week after scores are posted to Canvas to earn upto 50% back on missing or incomplete functionality. You can get 100% back of points deducted for code and repo quality within one week after scores are posted to Canvas.

## Code and Repo Quality

### Code

Writing consice, consistent, and well structured code takes practice. Here are things we are looking for in terms of code quality:

- Consistent formatting of code
  - In our case, we will follow the xv6 formatting conventions
- Consistent indentation
- Consistent vertical spacing (newlines)
- Consistent spacing around expressions and statemments
- Consistent naming of functions and variables
  - We are using Snake Case (e.g., ```malloc_name()```)
- Consistent comments, such as capitolization and spacing
- No commented out code
- No redundant code
- No excessively long functions
  - If a function becomes long, break up the code in to multiple functions
- Pick easier to understand code over shorter, but more complicated code

### Repo

Your repo should be complete in that if we clone it, it should build and run as exepected. However, your repo should not include an extra files or generated files like object files and executables. Please check that your repo is complete by cloning it from GitHub to a new location and run the Autograder. Also, check that you don't have any extra files.

## Autograder Tests

Here is an overview of each of the tests and the type of functionality that is being tested.

### Test 01 - Simple allocation

```text
memtest m 0 3000 A0 p s c 4096
[USED] name = A0 boff = 0 bsize = 3032 doff = 32 dsize = 3000
[FREE] name = NONE boff = 3032 bsize = 1064 doff = 3064 dsize = 1032
malloc_summary():
used  : 3000
free  : 1032
over  : 64
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test does a single ```malloc(3000)```. You need to be able to request at least 4096 (```BLOCK_MIN_INCR```) amount of heap space using ```sbrk()``` then create a USED block followed by a FREE block. You could do this directly like I have shown in class, but I recommend that you first create a large FREE block, then use your code to walk the block list to find a FREE block. The idea is that you only want one piece of code that looks for a FREE block, then splits a found FREE block into a FREE block and a USED block.

### Test 02 - Two allocations and no room for a free block

```text
memtest m 0 3000 A0 m 1 1000 A1 p s c 4096
[USED] name = A0 boff = 0 bsize = 3032 doff = 32 dsize = 3000
[USED] name = A1 boff = 3032 bsize = 1064 doff = 3064 dsize = 1032
malloc_summary():
used  : 4032
free  : 0
over  : 64
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test is doing two calls to ```malloc()```. However, in the second allocation (A1) there is not enought space to split FREE block into a USED block and a FREE block. That is there is not enough room for ```sizeof(struct block_hdr) + BLOCK_MIN_SIZE```, which is ```32 + 4 = 36``` bytes. In this case you just give the remaining amout to the second allocation by adding this amount to its size field and you do not create a new FREE block.

### Test 03 - Three allocations with a remaining free block

```text
memtest m 0 2000 A0 m 1 1000 A1 m 2 500 A2 p s c 4096
[USED] name = A0 boff = 0 bsize = 2032 doff = 32 dsize = 2000
[USED] name = A1 boff = 2032 bsize = 1032 doff = 2064 dsize = 1000
[USED] name = A2 boff = 3064 bsize = 532 doff = 3096 dsize = 500
[FREE] name = NONE boff = 3596 bsize = 500 doff = 3628 dsize = 468
malloc_summary():
used  : 3500
free  : 468
over  : 128
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test does 3 calls to ```malloc()``` and leaves a FREE block at the end of the block list.

### Test 04 - Free with one merge


```text
name = "04"
input = ["python3", "runxv6.py", "memtest m 0 1000 A0 f 0 p s c 4096"]
expected = """
memtest m 0 1000 A0 f 0 p s c 4096
[FREE] name = NONE boff = 0 bsize = 4096 doff = 32 dsize = 4064
malloc_summary():
used  : 0
free  : 4064
over  : 32
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test does a single allocation following by a ```free()```. In this case the USED block needs to be converted to a FREE block, then the FREE block needs to be merged with the existing FREE block above it. You will need to implement block merging for this test to work.

### Test 05 - Free with no merges

```text
memtest m 0 500 A0 m 1 500 A1 m 2 500 A2 m 3 500 A3 f 2 p s c 4096
[USED] name = A0 boff = 0 bsize = 532 doff = 32 dsize = 500
[USED] name = A1 boff = 532 bsize = 532 doff = 564 dsize = 500
[FREE] name = NONE boff = 1064 bsize = 532 doff = 1096 dsize = 500
[USED] name = A3 boff = 1596 bsize = 532 doff = 1628 dsize = 500
[FREE] name = NONE boff = 2128 bsize = 1968 doff = 2160 dsize = 1936
malloc_summary():
used  : 1500
free  : 2436
over  : 160
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test is actually easier to pass than 04. In this case we are calling ```free()``` on ```A2```, which has two USED neighbors. So, in this case you just need to convert the A2 USED block into a FREE block by setting the used field to 0 and changing name to be the empty string. No merging is necessary.

### Test 06 - Free with two merges

```text
memtest m 0 500 A0 m 1 500 A1 m 2 500 A2 m 3 500 A3 f 0 f 2 f 1 p s c 4096
[FREE] name = NONE boff = 0 bsize = 1596 doff = 32 dsize = 1564
[USED] name = A3 boff = 1596 bsize = 532 doff = 1628 dsize = 500
[FREE] name = NONE boff = 2128 bsize = 1968 doff = 2160 dsize = 1936
malloc_summary():
used  : 500
free  : 3500
over  : 96
u+f+o : 4096
total : 4096
mem_check() passed with heap less than 4096
```

This test create a situation in which a USED block is surrounded by two FREE blocks. That is A1 will have a FREE block below and a FREE block above. So, when we free A1, it will require two merges to make one large FREE block by combining both neighbors.

### Test 07 - Multiple malloc()s and free()s

```text
memtest m 0 1000 A0 m 1 1000 A1 m 2 1000 A2 f 0 f 1 f 2 m 0 1000 A3 m 1 1000 A4 m 2 1000 A5 f 0 f 1 f 2 m 0 1000 A6
 m 1 1000 A7 m 2 1000 A8 p s c 4096
[USED] name = A6 boff = 0 bsize = 1032 doff = 32 dsize = 1000
[USED] name = A7 boff = 1032 bsize = 1032 doff = 1064 dsize = 1000
[USED] name = A8 boff = 2064 bsize = 1032 doff = 2096 dsize = 1000
[FREE] name = NONE boff = 3096 bsize = 1000 doff = 3128 dsize = 968
malloc_summary():
used  : 3000
free  : 968
over  : 128
u+f+o : 4096
total : 4096
```

The goal of this test is to make sure you are properly converting USED blocks into FREE blocks, then reusing these FREE blocks for future ```malloc()s```. The test makes sure you don't grow the heap with ```sbrk()``` when we don't need to.

### Test 08 - Large malloc() greater than 4096

```text
 memtest m 0 10000 A0 p s c 12288
[USED] name = A0 boff = 0 bsize = 10032 doff = 32 dsize = 10000
[FREE] name = NONE boff = 10032 bsize = 2256 doff = 10064 dsize = 2224
malloc_summary():
used  : 10000
free  : 2224
over  : 64
u+f+o : 12288
total : 12288
mem_check() passed with heap less than 12288
```

This test makes sure you can request more than 4096 bytes from the kernel with ```sbrk()```. That is, you need to request a multiple of 4096 that can satisify the ```malloc()``` request.

### Test 09 - Merging last FREE block with sbrk() memory

```text
memtest m 0 3000 A0 m 1 5000 A1 p s c 8192
[USED] name = A0 boff = 0 bsize = 3032 doff = 32 dsize = 3000
[USED] name = A1 boff = 3032 bsize = 5032 doff = 3064 dsize = 5000
[FREE] name = NONE boff = 8064 bsize = 128 doff = 8096 dsize = 96
malloc_summary():
used  : 8000
free  : 96
over  : 96
u+f+o : 8192
total : 8192
mem_check() passed with heap less than 8192
```

In this case, there is not enough space in the heap for the second ```malloc()``. However, you don't just want to increase the heap by a multiple of 4096. You need to only increase the heap by the amount needed if we combine the last FREE block with a new FREE block from extending the heap. That is, this test make sure you don't allocate more heap space than needed for the request.

### Test 10 - Large malloc()s and free()s

```text
memtest m 0 5000 A0 m 1 5000 A1 m 2 5000 A2 m 3 5000 A3 f 2 f 3 p s c 20480
[USED] name = A0 boff = 0 bsize = 5032 doff = 32 dsize = 5000
[USED] name = A1 boff = 5032 bsize = 5032 doff = 5064 dsize = 5000
[FREE] name = NONE boff = 10064 bsize = 10416 doff = 10096 dsize = 10384
malloc_summary():
used  : 10000
free  : 10384
over  : 96
u+f+o : 20480
total : 20480
mem_check() passed with heap less than 20480
```

This test does 4 large (greater than 4096) ```malloc()s``` and two ```free()s``` to make sure you are managing the heap (block list) properly with larger requests.
