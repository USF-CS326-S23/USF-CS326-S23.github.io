---
layout: default
title: Lab02
nav_order: 2
parent: Assignments
permalink: /assignments/lab02
---

# Lab02 - Extending xv6 Commands

## Deliverables due Mon Feb 6 by 11:59pm in your Lab02 GitHub repo

- A copy of xv6 with modified wc, modified cat, and a sort command.
- You source code should conform to xv6 formatting conventions.

## Overview

Currently, xv6 only provides a limited number of common Unix commands the commands that are provided are simple implementations. In this lab we are going to extend two existing commands (wc and cat) and add a new command (sort). In order to support these commands you will need to understand the existing source code, basic usage of command line arguments, file descriptors, and a linked list implemenation.

## Line count only for wc (EASY)

In Linux and xv6, by default the wc command shows the line count, word count, and character count of list of files:

```text
$ wc README
  49  325 2305 README
```

However, in Linux, you can provide the ```-l``` command line option to restrict the output to just the number of lines:

```text
$ wc -l README
49 README
```

For this part of the lab you need to modify the existing implemenation of wc in xv6 to support the ```-l``` option.

Note that wc can also take multiple files as arguments:

```text
$ wc README listtest.txt
49 325 2305 README
9 17 220 listtest.txt
```

The ```-l``` option you add should still work with multple files:

```text
$ wc -l README listtest.txt
49 README
9 listtest.txt
```

Finally, the wc command can also take input from stdin, your modified wc with ```-l``` should also be able to work with stdin:

```text
$ cat README | wc -l
49
```

## Line numbers for cat (MODERATE)

The cat command, by default, simply outputs the contents of the files provided as arguments:

```text
$ cat listtest.txt
[google.com, 142.251.46.174]
[usfca.edu, 23.185.0.2]
[mit.edu, 104.90.21.210]
[openai.com, 13.107.238.57]
Sorted:
[google.com, 142.251.46.174]
[mit.edu, 104.90.21.210]
[openai.com, 13.107.238.57]
[usfca.edu, 23.185.0.2]
```

However, cat on Linux allows for the ```-n``` option which prepends line number to each line in the output:

```
$ cat -n listtest.txt
     1  [google.com, 142.251.46.174]
     2  [usfca.edu, 23.185.0.2]
     3  [mit.edu, 104.90.21.210]
     4  [openai.com, 13.107.238.57]
     5  Sorted:
     6  [google.com, 142.251.46.174]
     7  [mit.edu, 104.90.21.210]
     8  [openai.com, 13.107.238.57]
     9  [usfca.edu, 23.185.0.2]
```

Note: The line number column is 8 characters. 6 for the line number and 2 spaces after the line number.

For this part of the lab you need to modify the existing xv6 implementation of cat to  support for the ```-n``` option.

Just like wc, cat can accept multiple filenames as arguments and it can also accept input from stdin. Your support for ```-n``` should work both with multiple files and stdin. For example:

```text
$ listtest | cat -n
     1  [google.com, 142.251.46.174]
     2  [usfca.edu, 23.185.0.2]
     3  [mit.edu, 104.90.21.210]
     4  [openai.com, 13.107.238.57]
     5  Sorted:
     6  [google.com, 142.251.46.174]
     7  [mit.edu, 104.90.21.210]
     8  [openai.com, 13.107.238.57]
     9  [usfca.edu, 23.185.0.2]
```

Hint: the xv6 implemenation of cat simply reads in chunks of bytes from the input and then writes them to stdout (1). However, we need to read in a line at a time. To help you out, here is an implemenation of ```readline()``` which uses the ```read()``` system call to read an entire line from a file or stdin:

```c
int
readline(int fd, char *buf, int maxlen)
{
  int n;
  char c;
  int i = 0;

  /* Read one character at a time from fd */
  while((n = read(fd, &c, 1)) > 0){
    buf[i] = c;
    /* Look for the newline character */
    if (c == '\n'){
      /* We are at the end of the line, so stop reading */
      break;
    }
    i += 1;
    /* We don't want to read more characters than we have room */
    if (i >= (maxlen - 1)){
      /* We can't recover, so just print a message and exit */
      fprintf(2, "readline() - line too long\n");
      exit(-1);
    }
  }
  /* This is a little tricky. If read() returns 0 AND we didn't
     read previous characters for this line, then we want to return 0.
     Also, if read returns a value less than 0, we want to return this
     error condition. */
  if(((n == 0) && (i == 0)) || (n < 0))
    return n;

  /* Add the null terminator to the end for the string buffer */
  i += 1;
  buf[i] = '\0';
  return i;
}
```

## Adding the sort command to xv6 (HARD)

The sort command in Linux allows you to sort the lines of a text file:

```text
$ cat states.txt
Nebraska
Alabama
California
Oregon
Georgia
```
```text
$ sort states.txt
Alabama
California
Georgia
Nebraska
Oregon
```

The sort command can also accept input from stdin:

```text
$ cat states.txt | sort
Alabama
California
Georgia
Nebraska
Oregon
```

There are lots of different ways to go about implementing sort, but for your implementation you are going to use the provided linked list implementation in the lab02 starter code. In the user directory, you will find list.h, list.c, and listtest.c. The listtest.c program shows the basic usage of the linked list functions and macros.

The basic idea is that you are going to create a linked list of lines (strings) and then you will use the ```list_sort()``` function to sort the lines, then you will iterate of the list sorted list and print each line. You can use the ```readline()``` function given above to read each line in the input to be sorted.

Here is the given listtest.c code with comments:

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/list.h"
#include "user/user.h"

#define STR_LEN 128

/* This struct contains a pair of strings */
struct pair {
  /* To allow this struct to be put into a list, we embed
    a list_elem struct */
  struct list_elem elem;
  char name[STR_LEN];
  char value[STR_LEN];
};

void
pair_list_print(struct list *lp)
{
  /* Print a list of struct par nodes */
  struct list_elem *e;

  /* The list.c function help us walk the list. */
  for (e = list_begin(lp); e != list_end(lp); e = list_next(e)) {
    /* The list.c functions only know about struct list and
       struct list_elem types. In order to get acces to the struct
       that has the list_elem embedded in it, we need to use the
       list_entry macro to get the pointer to the outer struct pair. */
    struct pair *p = list_entry(e, struct pair, elem);
    printf("[%s, %s]\n", p->name, p->value);
  }
  return;
}

bool
pair_list_less(const struct list_elem *a, const struct list_elem *b, void *aux)
{
  /* This is a comparison function used by list_sort() */
  struct pair *ap;
  struct pair *bp;

  /* We need to get pointers to the pair structs for the two pairs
     we are comparing. */
  ap = list_entry(a, struct pair, elem);
  bp = list_entry(b, struct pair, elem);  
  
  return (strcmp(ap->name, bp->name) < 0);
}

int
main(int argc, char *argv[])
{
  struct list dmap;
  struct pair p1;
  struct pair p2;
  struct pair p3;
  struct pair p4;

  /* Don't forget ot initialize the list */
  list_init(&dmap);

  /* We will add each of the structs we defined as local variables. */
  /* Note that it will more common to use malloc() to dynamically
     allocation structs to be added to a list, like in sort. */
  
  strcpy(p1.name, "google.com");
  strcpy(p1.value, "142.251.46.174");
  list_push_back(&dmap, &p1.elem);

  strcpy(p2.name, "usfca.edu");
  strcpy(p2.value, "23.185.0.2");
  list_push_back(&dmap, &p2.elem);

  strcpy(p3.name, "mit.edu");
  strcpy(p3.value, "104.90.21.210");
  list_push_back(&dmap, &p3.elem);

  strcpy(p4.name, "openai.com");
  strcpy(p4.value, "13.107.238.57");
  list_push_back(&dmap, &p4.elem);

  pair_list_print(&dmap);

  /* Sort list */
  list_sort(&dmap, pair_list_less, (void *) 0);
  printf("Sorted:\n");
  pair_list_print(&dmap);
  
  exit(0);
}
```

My solution is about 100 lines of code.
