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

In order to compile and run xv6 you will need to have access to a RISC-V cross-compiler toolchain and Qemu. You can do all your xv6 work for this class using a dedicated USFCS lab machine we have for CS 326 called griffin.cs.usfca.edu. This lab will explain how to use griffin for our xv6 work. Alternatively, you can install a toolchain and Qemu locally on your own machine. On macOS and Linux this is straightforward. On Windows it is more complicated. See below for more details. My recommentation is to get yourself setup on griffin, then explore setting up a local build environment.

## Console-based Editors

Given this is a systems course, you should dedicate yourself to becoming familiar with a good console-based editor. Such editors include vim, micro, and emacs (not nano). I personally use all three of these editors, but more recently I use micro the most. Knowing a console-based editor is especially important when you are working on a maching remotely using ssh.

## Getting started with xv6 on griffin

Our griffin lab machine 




