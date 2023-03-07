---
layout: default
title: Lab04
nav_order: 5
parent: Assignments
permalink: /assignments/lab04
---

# Lab04 Fun with Processes and Pipes

## Deliverables due Tue Mar 21 by 11:59pm in your lab04 GitHub repo

- A copy of xv6 with your new implementations of ```rdir```, ```rpipe2```, and ```rpipe3```.
- Your implemenation should pass all of the Lab04 Autograder tests.
- Your source code should conform to xv6 formatting conventions.
- Your Project01 repo should not have any extraneous files or build artifacts
- Make sure to test your repo by cloning from GitHub to a different location and run the Autograder. This will catch problems like missing files.

# Contents
1. [Overview](#overview)
2. [rdir](#rdir)
3. [rpipe2](#rpipe2)
4. [rpipe3](#rpipe3)

## Overview

In class we have been learning about file descriptors, processes, and pipes. These lab problems will give you more experience in working with the system calls for file descriptors, processes, and pipes.

### rdir

Usage:
```text
rdir <input_filename> <command> <output_filename>
```

The rdir command will ```fork()``` and ```exec()``` the ```<commnand>```. In addition it will redirect the ```stdin``` of the ```<command>``` to come from ```<input_filename>``` and redirect the ```stdout``` of the ```<command>``` to the ```<output_filename>```. You will need to open ```<input_filename>``` for reading and you will need to create ```<output_filename>``` and open for writing. The ```rdir``` command should only fork one process. The parent should ```wait()``` for the child to exit.

Here is an example:
```
$ rdir README wc wc.out
$ cat wc.out
49 325 2305
```

### rpipe2

Usage:
```text
rpipe2 <command1> <command2>
```

The rpipe2 command will ```fork()``` two processes. In the first process it will ```exec()``` ```<command1>``` and in the second process it will ```exec()``` ```<command2>```. In addition, ```rpipe2``` will create a pipe and redirect the output of ```<command1>``` to the input of ```<command2>```. The parent should wait for both children to exit.

Here is an example:

```text
$ rpipe2 ls wc
29 116 726
```

The output of ls is redirected to the input of wc via a pipe.

### rpipe3

Usage:
```text
rpipe3 <command1> <command2> <command3>
```

The ```rpipe3``` command is like ```rpipe2``` except it will create three processes instead of two. The output of the first process running ```<command1>``` will be piped to the input of the second process running ```<command2>```. The output of the second process will pipe its output to the input of the third process running ```<command3>```. You will need to create three processes and two pipes. The parent should wait for all three processes to exit.

Here is an example:
```text
$ rpipe3 ls wc cat
29 116 726
```

And another:
```text
$ rpipe3 hello wc cat
1 2 14
```
