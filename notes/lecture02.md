---
layout: default
title: Lecture02
nav_order: 2
parent: Notes
permalink: /notes/lecture02
---

# Lecture02 - Programming C (in xv6)


why C?
  * Good for low-level programming
    * Easy mapping between C and RISC-V instructions
    * Easy mapping between C types and hardware structures
    *  e.g.., set bit flags in hardware registers of a device
  * Minimal runtime
    * Easy to port to another hardware platform
    * Direct access to hardware
  * Explicit memory management
    * No garbage collector
    * Kernel is in complete control of memory management
  * Efficient: compiled (no interpreter)
    * Compiler compiles C  to assembly
  * Popular for building kernels, system software, etc.
    * good support for C on almost any platform
  * Why not?
    * Easy to write incorrect code
    * Easy to write code that has security vulnerabilities

Today's lecture: use of C in xv6
  * memory layout
  * pointers
  * arrays
  * strings
  * lists
  * bitwise operators
  * not a general intro to C

Memory layout of a C program in xv6
  * draw figure, see fig 3.4 of text
  * text: code, read-only data
  * data: global C variables
  * stack: function's local variables
  * heap: dynamic memory allocation using sbrk, malloc/free

Example: compile cat.c
  * Makefile defines how
  * gcc compiles to .o
  * ld links .o files into an executable
    * ulibc.o is xv6 minimal C library
  * executable has a.out format with sections for:
    * text (code), initialized data, symbol table, debug info, and more

Basic programs
  * hello.c
  * sumargs.c
  * primitive data types
  * functions
  

C pointers
  * a pointer is a memory address
    * every variable has a memory address (i.e., p = &i)
    * so each variable can be accessed through its pointer (i.e., *i)
    * a pointer can be variable (e.g., int *p)
      * and thus has a memory address, etc.
  * pointer arithmetic
    * char *c;
    * int *i;
    * what is the value of c+1 and i+1?
  * referencing elements of a struct
      struct {
         int a;
         int b;
      } *p;
      p->a = 10
  * demo ptr.c
  
C arrays
  * Contiguous memory holding same data type (char, int, etc.)
    * no bound checking, no growing
  * Two ways to access arrays:
    * through index: buf[0], buf[1]
    * through pointer: *buf, *(buf+1)
  * Demo array.c
  
C strings
  * arrays of characters, ending in 0
    * demo str.c
  * ulib.c has several functions for strings
    * strlen() --- use array access
    * strcmp() --- use pointer access
  * ls.c
    * argv: array of strings
      * each entry has the address of a string
      * xv6's exec puts them on the stack as arguments to main
    * print out argv
      * draw diagram; see fig 3.4 in book

Bitwise operators
  * char/int/longs/pointers have bits (8, 32, 64 respectively, on RISC-V).
  * you can manipulate them with |, &, ~, ^
    10001 & 10000 = 10000
    10001 | 10000 = 10001
    10001 ^ 10000 = 00001
    ~1000 = 0111
  * example:
    * user/usertests.c
    * kernel/fcntl.h
    * kernel/sysfile.c
  * more interesting examples later
 
Keywords:
  * static: to make a variable's visibility limited to the file it is declared
    * but global within the file
  * void: "no type", "no value", "no parameters"

Common C bugs
  * use after free
  * double free
  * uninitialized memory
    * memory on stack or returned by malloc are not zero
  * buffer overflow
    * write beyond end of array
  * memory leak
  * type confusion
    * wrong type cast

References:
  * [https://blog.regehr.org/archives/1393](https://blog.regehr.org/archives/1393)



