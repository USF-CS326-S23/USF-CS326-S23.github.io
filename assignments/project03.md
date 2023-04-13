---
layout: default
title: Project03
nav_order: 8
parent: Assignments
permalink: /assignments/project03
---

# Project03 Queue-based Scheduling in xv6 (15%)

## Deliverables due Wed Apr 19 by 11:59pm in your Project03 GitHub repo

Interactive grading for Project03 will take place on Thu Apr 20.

- A copy of xv6 with your modified kernel that implements queue-based scheduling.
- Your implemenation should pass all of the Project03 Autograder tests.
- Your source code should conform to xv6 formatting conventions.
- Your Project03 repo should not have any extraneous files or build artifacts
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.
- You will demonstrate that your solution works during interactive grading. Be prepared to explain the changes you made in your own words.

# Contents
1. [Overview](#overview)
2. [Reading](#reading)
3. [Queue-based Scheduling](#queue-based-scheduling )
4. [Implementation](#implementation)

## Reading

[xv6 book Chapters 6 and 7](/assignments/book-riscv-rev3.pdf)

## Overview

We are going to modify xv6's array-based scheduler with a Queue-based scheduler using the ```list.c``` implementation we have been using throughout this semester. This will make some operations like picking the next process to run and finding an ```UNUSED``` process more efficient. As a bonus we will be able to optimize the scheduler to prevent excessive CPU utilization when xv6 is idle.

### Queue-based Scheduling

The xv6 kernel implements a simple form of round-robin scheduling by having the scheduler (```proc.c:scheduler()```) running on each processor (CPU) iterate over the proc table to find a process in the ```RUNNABLE``` state. If a process is found, the process's state is marked as ```RUNNING``` and the scheduler switches to the context of the process. If no ```RUNNABLE``` processes are found, the scheduler simply continues to iterate over the proc table. That is the scheduler is an infinite loop with a nested loop that iterates over all 64 (```NPROC```) entries in the proc table (array). This implementation is simple, but wasteful and is one of the reasons that running xv6 in Qemu configured with the default of 3 processors consumes approximately 300% CPU when xv6 is idle. That is, even though the only thing running is the init process and the shell (sh) process, which is just blocked waiting for keyboard input, the xv6 schedulers, one for each CPU, is busy iterating over the proc table. Another minor point is that the current scheduler implementation is not strickly fair in the sense that it is possible that a process that should run next according to round robin order, may in fact be delayed slightly depending on the state of each of the schedulers. That is, if a process goes from ```SLEEPING``` to ```RUNNABLE``` and there are other ```RUNNABLE``` processes in the proc table, the current index of one or more of the schedulers could allow the newly woken up process to run before other processes, who where effectively waiting. This is likely a minor effect, but it shows that there could be a sequencing of events that could give a process more than it's fair share of CPU time. 

Rather than a simple array, many kernels use more sophisticated data structures like queues (lists) or trees for scheduling processes. These data structures are used to make scheduling more efficient and also to enforce different types of scheduling behaviors. We are going to modify the xv6 kernel to support queue based scheduling. In addition, we will also support a free process list to make it efficient to find an ```UNUSED``` proc struct. Our objective is to elminate some of the code in the xv6 kernel that relies on iterating over the entire proc table. When ```NPROC=64``` iterating over the enter proc array is not a big deal, but if ```NPROC=1000``` or ```NPROC=10000```, which is possible in production kernels, simple iteration won't work well. Another benefit of queue-based scheduling is that we can easily add the RISC-V instruction ```WFI``` (Wait for Interrupt) to the scheduler loop in the case that there are no ```RUNNABLE``` processes. This will cause a CPU to go into a low-power state waiting for an interrupt (like console or timer) and only wake up if one of these interrupts occur. This will greatly reduce the about of CPU utilization Qemu and xv6 consume when idle. It will prevent your laptop fan(s) from running fast and loud :-)

### Implementation

We will be developing the implementation to Project03 together in class. Here is an overview of the steps we will take.

First, we will add a queue-based ```unused_list``` linked list so that we can easily find an ```UNUSED``` proc struct with out iterating over the entire proc table array.

1. Copy ```list.h```, ```list.c```, ```assert.h```, from the ```user``` directory to the ```kernel``` directory.
2. Add ```kernel/debug.h```:
   ```c
   #define DEBUG 1
   #define debug_print(fmt, ...) \
    do { if (DEBUG) printf("%s:%d:%s(): " fmt, __FILE__, \
         __LINE__, __func__, __VA_ARGS__); } while (0)
   ```
3. In ```Makefile``` add ```$K/list.o``` to the ```OBJS``` variable.
4. Modify ```kernel/assert.h``` to comment out ```UNUSED``` macro because it conflicts with the ```UNUSED``` enum in ```proc.h```.
5. Modify ```kernel/assert.h``` to replace ```exit(-1)``` with ```panic("list")```.
6. In ```kernel/proc.h``` add ```#include "kernel/list.h"``` at the top of the file (first line). Also add ```#include "kernel/debug.h```.
7. In ```kernel/proc.h``` add ```struct list_elem elem;``` to ```struct proc```.
8. Near the definition of ```struct proc proc[NPROC]``` add a global ```unused_list``` and a ```unused_lock```. This will be the list that holds ```UNUSED``` proc stucts. When we need a new proc struct we will just remove the proc struct at the head of the list. When we free a proc struct we will put it at the end of the list.
9. In ```kernel/proc.c``` Add a new function, ```void proc_print_list(struct list *lp)``` that will print the given proc list (i.e., the ```unused_list``` or the ```run_list```). You cant use code from the ```procdump()``` function. For each process on the given list, this function should print ```SLOT=```, ```PID=```, ```NAME=```, and ```STATE=```. This function will be used for debugging purposes.
10. In ```kernel/proc.c:procinit()``` add initialization of the ```unused_lock``` and the ```unused_list```. You can modify the for loop to added each proc entry to the back of the ```unused_list```.
11. In ```kernel/proc.c:allocproc()``` remove the first element from the ```unused_list``` if one exists. If not, return 0. Make sure that you protect access ot the ```unused_list``` with the ```unused_lock```.
12. In ```kernel/proc.c:freeproc()``` put the newly ```UNUSED``` proc on the ```unused_list```. Protect access with the ```unused_lock```.

Second, we will add queue-based schedule using a ```run_list``` linked list.

1. In ```kernel/proc.c``` near the definition of ```struct proc proc[NPROC]``` add a global ```run_list``` and a ```run_lock```. This will be the list that holds ```RUNNABLE``` processes and the scheduler will pick the next process to run in Round Robin order by picking the process at the head of the ```run_list```.
2. In ```kernel/proc.c:procinit()``` add initialization of the ```run_lock``` and the ```run_list```.
3. At the end of ```kernel/proc.c:userinit()``` add the user init proc to the ```run_list```. Be sure to protect the update to the ```run_list``` with the ```run_lock```. Generally, we are looking for places in ```proc.c``` where we update the state of a proc to ```RUNNABLE``` and making sure we put the proc on the ```run_list```.
4. In ```kernel/proc.c:fork()``` add the newly forked process to the ```run_list``` after the state is set to ```RUNNABLE```.
5. In ```kernel/proc.c:scheduler()``` modify the scheduler loop (code inside ```for(;;)```) so that you look at the head of the ```run_list``` to find the next process to run. That is you want to remove the inner for loop that iterates over all the proc the proc table. Instead look to see if there is a process in the ```run_list```. If there is a process, then remove it from the ```run_list``` and then follow the same steps that the original scheduler takes in order to switch to the selected process. Pay attention to the order of exising locking code. You can incrementally develop the new scheduler by adding debug code and see if you can switch to the initcode properly. Then work your way up to scheduling any ```RUNNABLE``` process. If there is no process on the ```run_list``` you can release the ```run_lock``` and just continue the scheduler loop. That is, you can just keep checking for processes on the ```run_list``` by acquiring the ```run_lock``` checking to see if there is a process, then releasing the ```run_lock```. 
6. Find remaining places in ```kernel/proc.c``` where a process is set to ```RUNNABLE``` and make sure to also put it on the ```run_list```

Third, make the scheduler more energy efficient by adding the WFI (Wait for Interrupt) instruction in the scheduler loop.

1. In ```kernel/proc.c:scheduler()``` if you find there are no processes on the ```run_list```, issue the WFI instruction like this: ```asm volitile("wfi")```. This will put the CPU into a low-power state just waiting for an interrupt. When the WFI instruction returns, you can just continue the scheduler loop.
2. Check to see that you now have near zero CPU utilization of Qemu/xv6 when xv6 is idle.

Time permitting we may explore adding queue support to channels to support fair ordering of access to resources like pipes.

