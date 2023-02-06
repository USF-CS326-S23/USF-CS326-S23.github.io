---
layout: default
title: xv6 Debugging - User
nav_order: 2
parent: Guides
permalink: /guides/xv6_debug_user
---

# xv6 Debugging user-level programs

# Contents
1. [printf](#debugging-with-printf)
2. [gdb](#debugging-with-gdb)

## Debugging with printf

Adding calls ```printf()``` can be useful to help see values of variables as your program is executing. Of course, for your final submission you want to remove all debuggin calls to ```printf()```. Actually remove them, do not just comment them out as this is not good practice. Instead of using ```printf()``` for debugging, consider using ```fprintf()``` as this allows you to specify the file descriptor you want to use for your output. You can use stderr (2) for your debug messages as these will still be displayed in the case the stdout (1) of the program you are debugging has redirected to a pipe or file. You can use ```fprintf()``` like this:

```c
fprintf(2, "sort() - buf = %s\n", buf);
```

Note we are using 2 as the first argument to mean stderr. Don't forget to add a newline "\n" to your output. Also, a convention I use in such messages is to display the function name in addition to the debug message to give me context for the message.

Finally, you can also open a debug log file to send your output like this:

```c
#include "kernel/fnctl.h"
...
/* Global debug fd */
int debug_fd = 2;
...

debug_fd = open("debug.log", O_CREATE | O_WRONLY);
if (debug_fd < 0) {
    fprintf("2, "open(\"debug.log\") failed\n.);
    exit(-1);
}

...
fprintf(debug_fd, "sort() buf = %s\n", buf);
...
```

After you program runs, you can check the ```debug.log```:
```
$ cat debug.log
```

## Debuggin with gdb

It is possible to use gdb to debug both xv6 user programs and kernel code. We will focus on user code here.

### Setup

The xv6 source includes a ```.gdbinit``` to help get gdb initialized properly to debug xv6 code. In order to allow gdb to use this configuration you need to add a gdbinit file to ```~/.config/gdb```:

```
[benson@griffin ~]$ cd ~/.config
[benson@griffin .config]$ mkdir gdb
[benson@griffin .config]$ cd gdb
[benson@griffin gdb]$ cat > gdbinit
set auto-load safe-path /
[benson@griffin gdb]$
```

When using ```cat``` to create a file, make sure to type a newline (RETURN) at the end of the last line, then type CTRL-D to close the file. You can also use your editor to create this file.

Also to properly debug xv6 user-level programs we need to turn off compiler optimiztions so variables are not optimized out. You can do this by changing the xv6 Makefile. Look for the following line:

```
CFLAGS = -Wall -Werror -O -fno-omit-frame-pointer -ggdb -gdwarf-2
```

Now change the ```-O``` in this line to ```-O0```. That a capital letter O followed by the number 0. It should look like this:

```
CFLAGS = -Wall -Werror -O0 -fno-omit-frame-pointer -ggdb -gdwarf-2
```


### Debugging

In order to use gdb to debug xv6 code you will want to have two terminal windows: one for running qemu and xv6 and one for running gdb.

In the first terminal window do the following:

```
benson@griffin lab02-solution]$ make qemu-gdb
*** Now run 'gdb' in another window.
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -global virtio-mmio.force-legacy=false -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 -S -gdb tcp::26306
```

This will be the terminal in which you are running qemu/xv6.

Now in the second terminal, do the following:

```
[benson@griffin lab02-solution]$ riscv64-unknown-linux-gnu-gdb
GNU gdb (GDB) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "--host=x86_64-pc-linux-gnu --target=riscv64-unknown-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
The target architecture is set to "riscv:rv64".
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x0000000000001000 in ?? ()
(gdb) file user/sort.o
Reading symbols from user/sort.o...
(gdb) b sort
Breakpoint 1 at 0x1c4: file user/sort.c, line 70.
(gdb) c
Continuing.
```

This example assumes you are debugging the sort.c program. After you type "c", xv6 will continue booting in the first terminal. In the first terminal you can now type:

```
$ listtest > listtest.txt
```
To create a file to sort. Then do:
```
$ sort listtest.txt
```
You should see something like the following in the second terminal window:
```
[Switching to Thread 1.2]

Thread 2 hit Breakpoint 1, sort (fd=3) at user/sort.c:70
70	  list_init(&lines);
(gdb)
```

Now you can interact with gdb. Unfortunately, when debugging user-level code with gdb, the next (n) and step (s) command do not work as expected. However, you can set breakpoints, then continue execution to a breakpoint and inspect variable value. Here is an example:

```
(gdb) l
65	  int n;
66	  struct list lines;
67	  struct line *lp;
68	  char buf[STR_LEN];
69
70	  list_init(&lines);
71
72	  while((n = readline(fd, buf, STR_LEN)) > 0){
73	    //printf("buf = %d", buf);
74	    lp = malloc(sizeof(struct line));
(gdb) b 72
Breakpoint 2 at 0x1d2: file user/sort.c, line 72.
(gdb) c
Continuing.

Thread 2 hit Breakpoint 2, sort (fd=3) at user/sort.c:72
72	  while((n = readline(fd, buf, STR_LEN)) > 0){
(gdb) print fd
$1 = 3
(gdb) print buf
$2 = '\000' <repeats 127 times>
(gdb) b 74
Breakpoint 3 at 0x1d4: file user/sort.c, line 74.
(gdb) c
Continuing.

Thread 2 hit Breakpoint 3, sort (fd=3) at user/sort.c:74
74	    lp = malloc(sizeof(struct line));
(gdb) print buf
$3 = "[google.com, 142.251.46.174]\n", '\000' <repeats 98 times>
(gdb) print n
$4 = 29
(gdb)
```

So, you can use the list (l) command to list the source code. You can specific specific files:

```
(gdb) l sort
59	  return i;
60	}
61
62	void
63	sort(int fd)
64	{
65	  int n;
66	  struct list lines;
67	  struct line *lp;
68	  char buf[STR_LEN];
(gdb) l
69
70	  list_init(&lines);
71
72	  while((n = readline(fd, buf, STR_LEN)) > 0){
73	    //printf("buf = %d", buf);
74	    lp = malloc(sizeof(struct line));
75	    if (lp == 0){
76	      printf("sort() - malloc() failed\n");
77	      exit(-1);
78	    }
```
Entering list (l) again will continue the listing which will help you find line numbers to use for setting breakpoints. Just set a break point at each point in the code where you want gdb to stop execution so you can inspect variable values with print (p).

Note that when debugging user-level programs and setting breakpoints, you may encounter gdb stopping a different program than the one you are debugging. This is because, breakpoints are set on virtual addresses (we will learn more about this later). If this happens, just use the continue (c) command in gdb to continue execution until you get to the breakpoint in your target program.
