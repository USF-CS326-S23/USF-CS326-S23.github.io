---
layout: default
title: Debugging
nav_order: 4
parent: Guides
permalink: /guides/debugging
---

# The Art of Debugging

This is focused on Project01.

Some students will write me (and not just in this class) something like  "I've been debugging for hours and I can't find my problem. Can you look at my code?" My job, and the TA's job, is not to find your bugs for you. Our job is to help you figure out how to find your problems. This is ultimately why you are here and the purpose of getting a CS degree: to learn how to become good a problem solver.

What is more helpful for me and the TAs if you have been debugging for a long time is to not just say "I've been debugging for hours," but rather send a message to me and the TAs as a private post (which is better than a direct message) and detail what you have done to debug your code. That is, explain the part of the code or the test you are working on and then walk us through your thought process and your  ```debug_print()``` output to explain why you think your individual code steps are correct, with the output data to backup your understanding. Generally, this process by itself will lead to finding bugs.

Here is what I mean by validating your individual code steps. If you are writing code, then you are manipulating values, either as variables or something in memory through a pointer.  Each statement you write is either reading data, writing data, or making a decision based on data. You need to get into the details of each individual statement or expression you write and validate that the expression is correct, often doing this with concrete values. To do this, you need to have a mental model of the values you are working with. This is why I have spent so much time in lecture illustrating how memory is laid out and how different parts of memory are related to each other. If you don't understand what you are trying to do it will make it difficult to both write correct code and debug it. Often, you may partially understand the concepts and the process of writing code and debugging will help you solidify the concepts.

It is unfortunate that debugging user programs in xv6 in gdb doesn't not work well (single stepping doesn't seem to work). So we are limited to ```printf()``` or ```debug_print()```. Still, this is a common and practical way to debug.

Here is a simple. Let's say we tried to write a simple, but incorrect, implementation of ```free()```:

```
void
myfree(void *p)
{
  struct block_hdr *hp;
  hp = (struct block_hdr *) p;
  hp->used = 0;
  hp->name[0] = '\0';
  return;
}
```

Note, I've added this implementation to the week05 given xv6 code with the implementation of ```malloc_name()``` that can pass the first test.

Now, I compile and run xv6 and then manually execute ```memtest```:

```text
$ memtest m 0 100 A0 p
[USED] name = A0 boff = 0 bsize = 132 doff = 32 dsize = 100
[FREE] name = NONE boff = 132 bsize = 3964 doff = 164 dsize = 3932
```
First, check to see if this output makes sense. The first USED block looks good, ```boff = 0``` means that the first USED block is at the beginning of the heap space (we know this because the beginning of the heap is stored in ```heap_start``` and ```boff``` is calculated as ```boff = ((long) hp) - heap_start```. ```bsize = 132``` makes sense because we know that the total size of the block should be the requested amount (100) plus the size of the block header which is ```sizeof(struct block_hdr) == 32```. The data offset (```doff```) is the offset of the first byte just after the block header, which is ```boff + sizeof(struct block_hdr)```, which is 32. And we know that ```dsize``` is the requested amount, 100.

Now, check the FREE block. It should start, ```boff```, right after the first USED block, which is 0 + 132. or ```boff = 132```. The ```bsize``` should be the remain amount of th heap, that is ```4096 - (used_bsize)```, which is ```4096 - 132 = 3964```, which checks out with the ```bsize``` of the FREE block. The FREE ```doff``` should be ```used_boff + used_bsize = 0 + 132 = 132```. The ```free_doff``` should be ```free_boff + sizeof(struct block_hdr)```, which is ```132 + 32 = 164```. This checks out. The ```free_dsize```should be ```free_bsize - sizeof(struct block_hdr) = 3964 - 32 = 3932```, which also checks out.

Now, let's see if our ```free()``` implementation is working:

```text
$ memtest m 0 100 A0 f 0 p
[USED] name = A0 boff = 0 bsize = 132 doff = 32 dsize = 100
[FREE] name = NONE boff = 132 bsize = 3964 doff = 164 dsize = 3932
```
Note that I haven't implemented merging in ```myfree()```, so I expect to see two free blocks in the block list. However, in this output, there is still a USED block, but this should be a FREE block. So, now we need to go back to the ```myfree()``` implementation to see what's going on. Now, this problem may be simple enough to resolve by carefully inspecting the code and asking yourself if the calculations look right. But, maybe you don't see the problem right away. So, you want to add debug code to make sure the values are what they should be:

```
void
myfree(void *p)
{
  struct block_hdr *hp;
  hp = (struct block_hdr *) p;
  debug_print("hp->used = %d, hp->name = %s\n", hp->used, hp->name);
  hp->used = 0;
  hp->name[0] = '\0';
  return;
}
```
The logic here is I want to verify that I'm freeing the block I think I should be freeing. So, I'm going to check the used value and the name of the used block associated with p.

Note that you have to set ```DEBUG``` to ```1``` in ```user/user.h```:
```
#define DEBUG 1
```

Now, let's run the code with the new ```debug_print()```:
```text
$ memtest m 0 100 A0 f 0 p
user/umalloc.c:225:malloc_name_given(): nbytes = 4104, name =
user/memtest.c:121:mem_refs(): cmd = m, index = 0, size = 100, name = A0
user/umalloc.c:142:malloc_name(): nbytes = 100, name = A0
user/memtest.c:136:mem_refs(): cmd = f, index = 0
user/umalloc.c:130:myfree(): hp->used = 1431655765, hp->name = UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU
[USED] name = A0 boff = 0 bsize = 132 doff = 32 dsize = 100
[FREE] name = NONE boff = 132 bsize = 3964 doff = 164 dsize = 3932
```
The important thing to look at is the output of the ```debug_print()``` we added. The values are strange: ```hp->used``` is super large and ```hp->name``` is garbage. So, now I'm focused on why this the case. I need to see if my ```hp``` pointer is correct. I could go back to notes from class to see a picture of where ```p``` points to relative to the block header for the p region. In this case I just set ```hp``` to ```p``` byte type casting, but this is not correct. So, hopefully through this process you would discover the error in your code.

However, maybe you have a misunderstanding and don't know you need to subtract ```sizeof(struct block_hdr)``` from ```p``` to get ```hp```.  In this case you could come to office hours or post a private message to Campuswire showing us the code you are working on and the debugging code you have added as wells as the output. You give an English description of what you are tying to do and then explain you don't understand your debugging output. With this in hand, myself or the TAs could ask something like "how are you calculating the pointer to the block header for p?" Then you could tell us and we could follow up with a diagram or point to the notes that explains how to get the block header pointer from ```p```.

This is the process we want you to follow and the type of interaction that is going to best lead to your success.

Here are some more takeaways:
- You need to be comfortable executing ```memtest``` manually from the command line to construct your own test cases. You cannot rely on just writing code and running the Autograder.
- You need to fully understand how to interpret the various values we are manipulating, including the output of memtest (e.g., boff, bsize, doff, dsize) and the summary output.
- You need to actively validate your intermediate steps. You can't just write a bunch of code and they try to understand the problem from a trap or garbled output.
- Debugging takes time and energy. It is part of the software development process and something that must be practiced. It is not "an annoyance" that you should rush through.
