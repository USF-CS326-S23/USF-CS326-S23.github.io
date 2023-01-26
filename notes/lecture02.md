---
layout: default
title: Lecture02
nav_order: 2
parent: Notes
permalink: /notes/lecture02
---

# Lecture02 - C Programming

## Why C?
- Good for low-level programming
- Easy mapping between C and RISC-V instructions
- Easy mapping between C types and hardware structures
  - e.g.., set bit flags in hardware registers of a device
- Minimal runtime
  - Easy to port to another hardware platform
  - Direct access to hardware
- Explicit memory management
  - No garbage collector
  - Kernel is in complete control of memory management
- Efficient: compiled (no interpreter)
  - Compiler compiles C  to assembly
- Popular for building kernels, system software, etc.
  - good support for C on almost any platform
- Why not?
  - Easy to write incorrect code
  - Easy to write code that has security vulnerabilities

## Lab01 programs
- hello.c
- sumargs.c

## C Basics
- Data
- Functions
- Primitive data types
- Statements
  - assignment, if/else, for, while
- Expressions
  - arithmetic, relational 
- Preprocessor 
