
#GDB/CGDB常用命令#

GDB为Unix类系统中默认的`C\C++`调试器，很多其他的调试器都是基于GDB开发，比如CGDB，DDD，以及Eclipse也是封装的GDB来进行`C/C++`。那么学习完GDB后，其他的封装了GDB的调试器用起来也就游刃有余了！

GDB与Windows上的调试器做区别与对比，对程序进行调试也有两种方法联系到程序，一种就是直接用GDB启动程序，另外一种方法就是挂到已经运行的程序中。

```
$ gdb test
GNU gdb (Ubuntu 7.7.1-0ubuntu5~14.04.2) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".

Reading symbols from test...done.
(gdb)

$ gdb
GNU gdb (Ubuntu 7.7.1-0ubuntu5~14.04.2) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb)

(gdb) attach 1290
Attaching to process 1290
Reading symbols from /usr/bin/vim.basic...(no debugging symbols found)...done.

......

(gdb)
```

###GDB基础命令###

**执行程序**

在启动程序后程序并没有执行起来，这时需要运行`run`命令将程序运行起来。从run命令中可以指定程序运行的参数。但是对于挂接进去的程序来说，程序已经执行起来，如果在此时执行`run`命令，那么会得到如下代码块的提示。

```
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) n
```

因此`run`命令可以用于重新执行程序而不用退出gdb，这种方式比较好的地方是不退出GDB会保留之前在GDB中设置的断点的信息。

**单步命令**

`next`执行下一行，然后暂停，它相当于Wingdb的`step over`。

`step`执行下一行，然后暂停，它相当于`step in`。即在遇到函数时进入函数内部。

**恢复执行**

如果程序执行后遇到断点就会停下来，想要再次运行，需要执行`continue`命令。

**断点**

`break`命令用来设置断点，

```
(gdb) b main
Breakpoint 1 at 0x8048426: file watchtest.c, line 6.
```

条件断点设置，可以在设置断点时添加条件，也可以使用`condition`命令为断点添加条件。

```
(gdb) break insert if num_y == 5

// condition添加条件
(gdb) break insert

(gdb) condition 1 num_y == 5

(gdb) info b
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   <MULTIPLE>
	stop only if num_y == 5
```

`tbreak`和`break`命令类似，只是它是临时断点，有效期只到首次到达指定行/位置时为止。


**查看变量**

`print`命令可以用来查看变量值，可以是局部变量，全局变量，数组元素或C的结构体，C++成员变量等。

```
(gdb) print i		// int类型变量 i
$2 = 3
```

**监视点**

另外一类断点其实是监视点，它将断点和变量检查的功能相结合。即当指定的变量的值发生变化时暂停程序执行。

```
(gdb) watch i
Hardware watchpoint 2: i

(gdb) c
Hardware watchpoint 2: i

Old value = 0
New value = 5
main () at watchtest.c:10
10		printf("i is %d.\n", i);
```

类似地还可以使用条件表达式，比如`(gdb) watch (i > 4)`。

**栈帧**

`bachtrace`命令可以显示当前执行位置的栈帧，如下代码块所示。

```
(gdb) run 1 12 5 3 8 2
Starting program: /home/andy/gdb/insert_sort 1 12 5 3 8 2

(gdb) backtrace
#0  insert (new_y=1) at ins.c:29
#1  0x08048562 in process_data () at ins.c:48
#2  0x080485d9 in main (argc=7, argv=0xbffff154) at ins.c:59

(gdb) frame 2
#2  0x080485d9 in main (argc=7, argv=0xbffff154) at ins.c:59
59		process_data();
```

`frame`命令可以用于在栈帧之间切换。默认当前栈帧的编号为0，向下依次排列。如上代码块，执行`frame 2`后将当前调试环境设置为2号帧的内容。

**GUI**

gdb也有GUI调试模式，在启动gdb时添加`-tui`参数，启动后就可以看到源码窗口，调试过程中可以根据源码进行调试。但是这种界面窗口不太好用，容易出现混乱。

![图 ](\image\gdb-using-tui.jpg)

另外一种更好用的基于gdb的GUI调试器是CGDB，它提供的源码窗口更好用一些。

###GDB调试汇编###