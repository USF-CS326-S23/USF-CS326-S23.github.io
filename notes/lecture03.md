---
layout: default
title: Lecture03
nav_order: 3
parent: Notes
permalink: /notes/lecture03
---

# Lecture02 - C Programming in xv6

## C topics
- Memory layout
- Pointers
- Arrays
- Strings
- Lists
- bitwise operators
- not a general intro to C

## Memory layout of a C program in xv6
- Draw figure, see fig 3.4 of text
- Text: code, read-only data
- Data: global C variables
- Stack: function's local variables
- Heap: dynamic memory allocation using sbrk, malloc/free

## Example: compile cat.c
- Makefile defines how
- gcc compiles to .o
- ld links .o files into an executable
  - ulibc.o is xv6 minimal C library
- Executable has a.out format with sections for:
  - text (code), initialized data, symbol table, debug info, and more
  
## C pointers
- A pointer is a memory address
  - Every variable has a memory address (i.e., p = &i)
  - So each variable can be accessed through its pointer (i.e., *i)
  - A pointer can be variable (e.g., int *p)
    - and thus has a memory address, etc.
- Pointer arithmetic
  - char *c;
  - int *i;
  - What is the value of c+1 and i+1?
  - Referencing elements of a struct
  
      struct {
         int a;
         int b;
      } *p;
      p->a = 10
      
- Demo ptr.c
  
## C arrays
- Contiguous memory holding same data type (char, int, etc.)
  - No bound checking, no growing
- Two ways to access arrays:
  - Through index: buf[0], buf[1]
  - Through pointer: *buf, *(buf+1)
- Demo array.c
  
## C strings
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

## Bitwise operators
- char/int/longs/pointers have bits (8, 32, 64 respectively, on RISC-V).
- You can manipulate them with |, &, ~, ^

    10001 & 10000 = 10000
    10001 | 10000 = 10001
    10001 ^ 10000 = 00001
    ~1000 = 0111
- Examples
  - user/usertests.c
  - kernel/fcntl.h
  - kernel/sysfile.c
  - More interesting examples later
 
## Keywords:
- static: to make a variable's visibility limited to the file it is declared
  - but global within the file
- void: "no type", "no value", "no parameters"

## Common C bugs
- Use after free
- Double free
- Uninitialized memory
  - Memory on stack or returned by malloc are not zero
- Buffer overflow
  - Write beyond end of array
- Memory leak
- Type confusion
  - Wrong type cast

References:
  * [https://blog.regehr.org/archives/1393](https://blog.regehr.org/archives/1393)
