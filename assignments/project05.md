---
layout: default
title: Project05
nav_order: 10
parent: Assignments
permalink: /assignments/project05
---

# Project05 Containers in xv6

## Deliverables due Tue May 16th by 11:59pm in your Project05 GitHub repo

- **IMPORTANT: There is no coding required for Project05**
- You will provide explanations of how you propose to change the code to support container functionality.
- **NOTE: There is no final exam**
- Your first objective will be to understand the given starter container code.
- This code will be explained in class.
- Your second objective is to explain how you will extend the given code in order to support additional container features outlined below.
- Your deliverable will be design document that explains the changes you propose to make to the given kernel code.
- You document should be either and ASCII text file (.txt) or a Markdown (.md) file. You should name this file ```project05-design.txt``` or ```project05-design.md```.
- In your design document you should list all of your team members.
- You can work in groups of up to 3 people to complete this project. Please let me know by Tue May 10th your group. Groups of 1, 2 or 3 are all possible.
- There will be no interactive grading for project05.
- Each member of a group should submit the final design document to your own project05 repo in the top level (the same level as the xv6 directory).

# Contents
1. [Overview](#overview)
2. [Design Document Questions](#design-document-questions)

# Overview

The given project05 starter code contains support for multple consoles in xv6 and base support for containers.

## Multiple consoles

In order to allow for starting containers on their own console, I've added support for multple consoles (sometimes these are called virtual consoles). With multiple consoles the ```init.c``` process now creates multple device files, currently 4: console0, console1, console2, and console3. I've provided a program, ```user/consexec.c``` that demonstrates how to start a process on a new console. To switch between consoles you can use CTRL-T. This will cycle through each console one at a time. For example:

```text
$ consexec console1 -s sh
```
While start a ``sh`` process on console1.

You should see something like:
```
[Console 1 now active]
$
```
You are now in console1 and you can interact with the new shell. You can get back to console0 by typing CTRL-T three times. You will see the new console name at the top of the screen each time you switch. Note that console don't retain a buffer of the output, so you need to presse return in console0 to see the shell prompt again.

While it is not necessary to fully understand the multiple console implementation in the kernel in order to design the container support below, you can check out ```kernel/console.c``` to understand the approach. A new system call was added, ```consw``` to switch the to a designated console.

You will want to check out ```user/consexec.c``` to see how to start a process on a new console. This is used in the ```user/cstart.c``` program described below for starting a new container.

## Base container support

The Project05 starter code also includes base container support. Most of the code lives in ```kernel/proc.h``` and ```kernel/proc.c```. You can find new container code by search for ```Containers``` in the comments. I've added ```// Containers``` to all places in the code that were added or changed to support containers.

First, in ```kernel/proc.h```, we added a container struct ```struct cont```:

```
// Per-container state
struct cont {
  struct spinlock lock;

  // c->lock must be held when using these:
  enum procstate state;        // Container state (UNUSED or USED)
  int nextpid;                 // Per-container PID counter
  struct list_elem elem;       // list_elem for container run list
  struct list proc_list;       // List of process that exist in the container
                               // in any state.
  struct list run_list;        // List of RUNNABLE processes in the container
  char name[16];               // Container name
  char rootpath[128];          // Container root directory path
  struct inode *rootdir;       // inode for root directory
  int max_mem_bytes;           // Max mem bytes to be used by the container
  int max_disk_bytes;          // Max disk bytes to be used by the container
  int used_mem_bytes;          // Keep track of mem bytes used
  int used_disk_bytes;         // Keep track of disk bytes used
  int nticks;                  // Keep track of total ticks used by all processes
                               // in the container.
};
```

A ```struct cont``` is just like a ```struct proc```, but it is used to schedule processes that exist in the container and to keep track of resource usage.

The basic idea is that we start with a root container when the kernel starts. This is called ```cont_root```. The starter code has support for schedule processes that are owned by the root container.

New containers can be created with ```proc.c:alloccont()```. The major work to start a new container lives in ```proc.c:cfork()```. This will create a new container with the the given arguments: ```cname```, ```max_mem_bytes```, ```crootdir```, and ```max_disk_bytes```. You can create a new container from the user-level using the ```cfork()``` system call.

The starter code will create a new container, but it currently cannot schedule processes in new containers. This is part of what you need to design and explain in you Project05 design doc. The starter code also currently ignores the resource limit arguments such as ```max_mem_bytes```, ```crootdir```, and ```max_disk_bytes```. Explaining how to enforce these arguments will also be part of your design document.

I've provided two user programs to work with containers: ```user/cstart.c``` and ```user/cinfo.c```. The ```cstart.c``` program can start a new container:

```
$ cstart
usage: cstart <name> <console> [-s] <max_mem> <root_dir> <max_disk> [argv]
  -s switch to <console> on start
```

For example you can start a new container like this:

```
 cstart foo console1 -s 500000 /foo 1000000 sh
```

In a full solution, this should start a new container called foo on console1. It has a memory limit of 500000 bytes and a disk limit of 1000000. It will start sh in the container. Note that /foo will be an isolated part of the file system, so any programs that you want to run in the container should be copied in the container directory before you start the container. If any the process in a container causes the collective limit of all the container processes to be reached, new memory allocations should fail or new disk block allocations should fail. The container processes should be name-space isolated. Processes in a container should not be able to kill processes in other containers and they should be able to see processes in other containers. Similarly, processes in a container can only access files and directories that are contained in the ```root_dir```.

The ```user/cinfo.c``` command is like ps for containers:

```
$ cstart foo console1 0 / 0 busymany 100 5
$ cinfo
Container: root
  rootpath        : /
  max_mem_bytes   : 0
  used_mem_bytes  : 0
  max_disk_bytes  : 0
  used_mem_bytes  : 0
  nprocs          : 3
  nticks          : 1

Container: foo
  rootpath        : /
  max_mem_bytes   : 0
  used_mem_bytes  : 0
  max_disk_bytes  : 0
  used_mem_bytes  : 0
  nprocs          : 5
  nticks          : 60
$
```

# Design Document Questions

For you Project05 design document you will address the following design questions. The more detail you provide the better, but you are not expected to write complete code. You should, however, provide the locations in the existing project05 starter code where you propose to make changes. The location of your proposed changes is important.

## Question 1 - Scheduling

The given project05 starter code will only schedule processes in the root container. Explain how you would modify the given code to add support for scheduling processing in other containers. Keep in mind that you want to schedule containers themselves in a round-robin fashion.  In this way if Container Foo has 1 process and Container Bar has 4 processes, each container should receive 50% of the CPU time. Another way to say this is that the total ticks for Container Foo should be roughly equal to the number of total ticks for Container Bar if they were both started about the same time and all the processes are CPU-bound.

## Question 2 - Process Namespace Isolation

Explain how process namespace isolation is achieved in the project05 started code. Also, explain we would modify the ```user/ps.c``` program so that if it is executed within a container it will only show the processes running in that container.

## Question 3 - Memory Limiting

One of the parameters to ```proc.c:cfork()``` is ```max_memory_bytes```. Explain how you would modify the kernel to track container memory usage and enforce the ```max_memory_bytes``` limit. Explain any potential limitations in your approach.

## Question 4 - Filesystem Namespace Isolation

One of the parameters to ```proc.c:cfork()``` is ```crootdir``` which is a path to the root directory for the container. Explain how to enforce processes running in a container to only be able to access files and directories within the ```crootdir``` path. Hint: learn how paths are resolved to inode pointers. Noted that each process has a ```cwd``` (current working directory) and paths can both be absolute (e.g. ```/foo/sh```) and relative (e.g., ```../sh```). For absolute paths, they should start at ```crootdir```. For relative paths, they should not be able to "break out" of the ```crootdir``` with ```../```. 

## Question 5 - Disk Space Limiting

One of the parameters to ```proc.c:cfork()``` is ```max_disk_bytes```. Explain how you would modify the kernel to track container disk usage and enforce the ```max_disk_bytes``` limit. When starting a container, the container should now how many bytes of disk space the container is concurrently using. Explain how you would determine the current number of disk bytes used in ```crootdir``` on container start up.
