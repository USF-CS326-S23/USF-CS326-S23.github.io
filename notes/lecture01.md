---
layout: default
title: Lecture01
nav_order: 1
parent: Notes
permalink: /notes/lecture01
---

# Lecture01 - Introduction to Operation Systems

## CS 326 goals
- Understand operating system (OS Kernel) design and implementation
- Hands-on experience writing systems software
- Hands-on experience extending a small OS (xv6)
  
## What is the purpose of an OS?
- Abstract the hardware for convenience and portability
- Multiplex the hardware among many applications
- Isolate applications in order to contain bugs
- Allow sharing among cooperating applications
- Control sharing for security
- Don't get in the way of high performance
- Support a wide range of applications

## Organization: layered picture
- [user/kernel diagram]
- User applications: vi, gcc, chrome, databases
- Kernel services
- Hardware: CPU, RAM/memory, disk, networking
- We care a lot about the interfaces and internal kernel structure

## What services does an OS kernel typically provide?
- Process (a running program)
- Memory allocation
- File contents
- File names, directories
- Access control (security)
- Many others: users, IPC, network, time, terminals

## What's the application/kernel interface?
- "System calls"
- Examples, in C, from UNIX (e.g. Linux, macOS, FreeBSD):

      fd = open("out", 1);
      write(fd, "hello\n", 6);
      close(fd);

- These look like function calls but they aren't 

## Why is OS design+implementation hard and interesting?
- Many design tensions:
  - Efficient vs abstract/portable/general-purpose
  - Powerful vs simple interfaces
  - Flexible vs secure
- Features interact: ```fd = open(); fork()```
- Uses are varied: laptops, smart-phones, cloud, virtual machines, embedded
- Evolving hardware: multi-core, GPUs, fast networks
- Unforgiving environment: quirky h/w, hard to debug

## You'll be glad you took this course if you...
- Want to know what goes on under the hood
- Like infrastructure
- Need to track down bugs or security problems
- Care about high performance
- Want to become a better programmer

## Cool stuff we will do
- Extend a user-level memory allocator
- Extend a Unix shell
- Implement new system calls
- Modify the process scheduler
- Implement kernel threads
- Learn how virtual memory works
- Add I/O multiplexing for processes
- Modify a Unix file system
- Extend xv6 to support contianers

## Class structure
- Online course information:
  - [https://cs326.cs.usfca.edu](https://cs326.cs.usfca.edu)
    - Schedule, assignments (labs and projects)
  - Campuswire: announcements, discussion, help

## Lectures and Lab Sections
- OS ideas
- Case study of xv6, a small OS, via code and xv6 book
- Code examples
- Lab/project background
- OS papers
  - Submit a question about each reading, before lecture.
- 10 minute break in the middle of class
- Class attendance is not required, but...
- Remote attenance is allowed, I will make Zoom links available
   
## Labs and Projects 
- The point: hands-on experience
- Mostly one week each
- Labs are low stakes and less points
- Labs help foundations for the projects
- Projects are more involved and more points
  - Three kinds:
    - Systems programming
    - OS primitives, e.g. thread switching.
    - OS kernel extensions to xv6, e.g. containers
- Use Campuswire to ask/answer lab/project questions.
- Discussion is great, but please do not look at others' solutions!

## Grading:
- 20% labs, based on tests (the same tests you run).
- 80% projects, interactive meetings: we'll ask about your solutions
- No exams, no quizzes.
- Note that most of the grade is from projects. Start them early!
- You can get up to 50% points back for late assignments
  - Up to a week after scores are released on Canvas
- Please read through the course [syllabus](/files/Spring-2023-CS-326-01-Operating-Systems.pdf)
