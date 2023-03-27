---
layout: default
title: Project02
nav_order: 6
parent: Assignments
permalink: /assignments/project02
---

# Project02 Shell History (10%)

## Deliverables due Tue Mar 28 by 11:59pm in your project04 GitHub repo

- A copy of xv6 with your modified implementation of sh with history support
- Your implemenation should pass all of the Project02 Autograder tests.
- Your source code should conform to xv6 formatting conventions.
- Your Project02 repo should not have any extraneous files or build artifacts
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.

# Contents
1. [Overview](#overview)
2. [History](#history)
3. [Implementation](#implementation)

## Overview

Shell command line history is a very useful feature. The xv6 shell (sh.c) does not currently support command line history. Your job in this project is to add command line history.

### History

You will support the following shell built in commands for history:

```text
history
!<text>
!<num>
```
The ```history``` builtin will list the history of commands:

```text
$ echo hi
hi
$ echo bye
bye
$ history
    1  echo hi
    2  echo bye
    3  history
$ echo foo
foo
$ history
    1  echo hi
    2  echo bye
    3  history
    4  echo foo
    5  history
$ !2
echo bye
bye
$ history
    1  echo hi
    2  echo bye
    3  history
    4  echo foo
    5  history
    6  echo bye
    7  history
$ !5
history
    1  echo hi
    2  echo bye
    3  history
    4  echo foo
    5  history
    6  echo bye
    7  history
    8  history
$
$
$ history
    1  echo hi
    2  echo bye
    3  history
    4  echo foo
    5  history
    6  echo bye
    7  history
    8  history
    9  history
$ !ec
echo bye
bye
$ echo foo
foo
$ wc README
$ grep ACK README
$ history
    5  history
    6  echo bye
    7  history
    8  history
    9  history
   10  echo bye
   11  echo foo
   12  wc README
   13  grep ACK README
   14  history
$ !4
-sh: !4: event not found
$ !baz
-sh: !baz: event not found
$ foobar
exec foobar failed
$ history
    7  history
    8  history
    9  history
   10  echo bye
   11  echo foo
   12  wc README
   13  grep ACK README
   14  history
   15  foobar
   16  history
$
```

### Implementation

For your implementation you should use the linked list implementation to save the history. You should only store the 10 most recent commands. You can add your code directly to ```user/sh.c```, but keep your code together in one location. Project02 will not be interactively graded, but we will review your code for quality.


