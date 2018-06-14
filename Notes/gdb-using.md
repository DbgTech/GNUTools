
#GDB/CGDB常用命令#

GDB为Unix类系统中默认的`C\C++`调试器，很多其他的调试器都是基于GDB开发，比如CGDB，DDD，以及Eclipse也是封装的GDB来进行`C/C++`。那么学习完GDB后，其他的封装了GDB的调试器用起来也就游刃有余了！

GDB的命令在一行内输入，并没有限制命令行的长度，对于记不住的命令，输入一部分可以使用`TAB`键补充完整命令，类似Shell的功能。

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

下面依次总结一下GDB调试的基本命令。

###执行程序###

在启动程序后程序并没有执行起来，这时需要运行`run`命令将程序运行起来。从run命令中可以指定程序运行的参数。但是对于挂接进去的程序来说，程序已经执行起来，如果在此时执行`run`命令，那么会得到如下代码块的提示。

```
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) n
```

因此`run`命令可以用于重新执行程序而不用退出gdb，这种方式比较好的地方是不退出GDB会保留之前在GDB中设置的断点的信息。

###单步命令###

`next`执行下一行，然后暂停，它相当于Wingdb的`step over`。`next n`表示单步执行n步，如果遇到函数则跳过。

`step`执行下一行，然后暂停，它相当于`step in`，即在遇到函数时进入函数内部。`step n`表示单步执行n步，遇到函数进入函数中。

###恢复执行###

如果程序执行后遇到断点就会停下来，想要再次运行，需要执行`continue`命令。

有条件的执行还有另外两个命令

`until`命令可以用于处理循环，想让程序执行到循环结束时暂停，可以使用`until`命令。另外达到同样的目的可以在循环结束位置设置断点，然后执行`continue`命令，这样也可以实现在循环结束时停下来。

`finish`命令用于执行程序到当前函数返回之后为止，其实从GDB角度看它是当栈顶函数帧完成时为止。

###断点###

这里断点泛指可以暂停程序的机制，其实包含断点，监视点以及捕获点。捕获点是当特定事件发生时暂停执行。

**1. 断点**

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

break命令有如下几种设置断点的方法：

```
(gdb) break function		// 在特定函数名字上设置断点

(gdb) break filename:function // 在特定源码文件中的function函数名上设置断点

(gdb) break line_number     // 在特定的代码行上设置断点，line_number为行号

(gdb) break filename:line_number     // 在特定的源码文件的指定代码行上设置断点，line_number为行号

(gdb) break +offset / break -offset //  当前栈帧中真该执行的源码行前或后偏移行数上设置断点

(gdb) break *address       // 型号后面的address为设置断点的地址值
```

条件断点的基本语法是`break break-args if (condition)`，`break-args`用于指示设置断点的位置参数。除了前面使用到的变量比较条件`i > 4`外，还有其他的一些可以使用。

* 相等，逻辑和不相等运算符（`<`, `<=`, `==`, `!=`, `>`, `>=`, `&&`, `||`等）
* 按位和移位运算符（`&`, `|`, `^`, `>>`, `<<`等）
* 算术运算符（`+`, `-`, `*`, `/`, `%`）
* 程序中函数调用，比如`break 44 if strlen(mystring) == 0`

`tbreak`和`break`命令类似，只是它是临时断点，有效期只到首次到达指定行/位置时为止。

**2. 监视点**

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

类似地还可以使用条件表达式，比如`(gdb) watch (i > 4)`。在监视点中也可以使用非常复杂的表达式，比如`(gdb) watch (i|j > 12) && i > 24 && strlen(name) > 6`设置的监视点监控的条件很多，很复杂的表达式

**3. 捕获点**

**4. 其他命令**

`info breakpoint/ info b / i b`用于列举当前调试中插入的断点的信息。



`delete`命令用于删除断点。delete命令后面跟的数字是断点的编号，即通过`info breakpoint`命令列出的断点列表前面的编号。如果没有任何参数的`delete`命令会删除所有的断点。

```
(dbg) delete 1
```

`clear`命令用于清除断点，它的使用方法与`break`类似，可以指定行数，函数名以及文件名加行数或文件名等形式。

`disable/enable`命令用开启或关闭断点，如果不想让某个断点生效，可以设置它为disable。`enable`命令有一个`once`参数表示此次使能断点只有一次，当断点断下来一次之后变为`disable`状态。

```
(gdb) disable 1

(gdb) enable 2 3

(gdb) enable once 2 3
```

###查看变量###

`print`命令可以用来查看变量值，可以是局部变量，全局变量，数组元素或C的结构体，C++成员变量等。

```
(gdb) print i		// int类型变量 i
$2 = 3
```

对于结构体指针，可以使用`print *tmp`来打印结构体内容。

`disp *tmp`是另外一种显示变量的方式，每当遇到断点暂停时就会执行`display`命令显示结构体内容。

`disable disp 1`关闭显示列表中1号的显示，`enable disp 1`开启显示。`undisplay 1`可以删除显示命令。

还有一种方式可以显示结构体或类对象数据，就是GDB的`commands`命令，它可以形成一个命令集合指定在某次断点执行。

```
(gdb) b 37

(gdb) commands 1
Type commands for when breakpoint 1 is hit, one per line
End  with a line saying just "end"
>p temp->val
>if(temp->left != 0)
 >p temp->left->val
 >else
 >printf "%s\n", "none"
 >end
>if(temp->right != 0)
 >p temp->right->val
 >else
 >printf "%s\n", "none"
 >end
>end
```

最后一种方法是使用GDB的`call`命令，GDB中可以调用程序中已经编译的函数，用程序已有的函数输出变量的值。

```
(gdb) commands 2
Type commands for when breakpoint 1 is hit, one per line
End  with a line saying just "end"
>printf "**************current tree****************"
>call printtree(root)
>end
```

对于`commands`命令集要取消时，和设置是类似，只需要在要输入命令时输入`end`即可取消命令集。

对于数组指针要输出整个数组内容，可以使用类似对象的方法，但是有时候使用`*`后只能输出一个元素。这时可以使用GDB的人工数组。比如

```
(gdb) p *x@25		// 即 *pointer@number_of_elemets

(gdb) p (int [25])*x	// 输出整个数组内容
```

查看当前栈帧的全部局部变量的内容可以使用`info locals`命令获取所有局部变量的值。

最后，如果要查看结构体或类对象的类型，可以使用`ptype`命令，它可以列出当前对象对应的类型。

**方便变量**

GDB维护了方便变量，以`$`开头，比如在显示变量时，总会使用`$4`等类似的方式标记内容。可以用`$4`来引用刚刚显示过的它所代表的变量。

`p $`会显示刚刚显示过的变量值，`p $n`显示编号为n的方便变量的值，而`$$n`显示从`$`开始向前第n个显示的值。

`$_`变量被赋值为最近执行的x命令中被检查的地址，`$__`变量则被赋值为最近执行的x命令中检查地址的值。

使用 `set $foo = *object_ptr`命令来设置一个方便变量。



###栈帧###

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

###内存###

print和display命令允许指定显示的格式，比如`(gdb) p /x y`将y变量按照十六进制格式显示。其他的还有`/c`表示按照字符形式显示，`\s`按照字符串显示，`\f`按照浮点格式显示。

###源码调试###

`list/l`用于列举当前位置对应的源代码

###汇编调试###

**汇编单步**

`nexti/stepi`是汇编单步，依次执行一条汇编指令，它与源码的单步执行不同，源码单步是一次执行依据源码。

在汇编下设置断点时GDB有一个bug，在程序起始点设置断点后程序并不能停在起始点，所以设置到起点汇编地址的后五个字节处。

```
break *_start + 5
```

`where`命令可以i显示GDB当前停在了程序什么位置。

汇编调试一个重要的命令是查看汇编指令，`disassemble _start`用于查看标号`_start`处的汇编指令。一般情况下GDB显示的汇编指令为`AT&T`风格，如果要显示为Intel风格汇编指令，可以使用`set dissemble-flavor intel`。

`info registers`命令用于显示当前寄存器中的内容。

`print`命令可以查看不同格式的数据，比如

```
print /d $ecx
print /x $ecx
print /t $ecx
```

同样适用`x`命令也可以查看内存，`x`命令后可以跟参数来给出内存显示格式，`/`符号后又三个域，第一个是显示的数量，第二个表示显示的格式，第三个域表示显示的内存大小，最后给出内存地址。

```
x /12cb &msg    // msg变量所在地址，以字符形式显示12个字节大小数据
x /12db &msg    // msg变量所在地址，以十进制形式显示12个字节大小数据
x /12xh &msg    // msg变量所在地址，以十六进制形式显示12个半字大小数据
x /12xw &msg    // msg变量所在地址，以十六进制形式显示12个字大小数据
```

`display`命令可以在暂停时显示指定变量的内容，比如`display $eax`可以在暂停时显示寄存器`eax`中的内容。

一个技巧是使用display命令来显示汇编指令，`display /i $eip`命令可以在每次暂停时打印停止位置的汇编指令（即下一条即将执行的指令）。

###转储文件###

在gdb调试下，使用`generate-core-file`命令可以转储当前进程的状态信息。

内核转储文件和调试对象，就可以在非当前环境下查看转储文件当时进程的运行状态（寄存器和内存值等）。它和Windows下的dump类似。

###GUI###

gdb也有GUI调试模式，在启动gdb时添加`-tui`参数，启动后就可以看到源码窗口，调试过程中可以根据源码进行调试。但是这种界面窗口不太好用，容易出现混乱。

![图 ](\image\gdb-using-tui.jpg)

另外一种更好用的基于gdb的GUI调试器是CGDB，它提供的源码窗口更好用一些。



另一种界面

>https://github.com/snare/voltron

###GDB/Windbg对比###

WinDbg和GDB常用命令对比：

|WinDbg命令 | GDB命令 | 功能 |
|----------|---------|-----|
|bp | break或b | 设置软件断点 |
|ba | watch | 设置硬件断点、监视点|
|k  | backtrace或bt | 显示函数调用序列（栈回溯） |
|g  | continue或c | 恢复执行 |
|p/t | next/step或n/s | 单步跟踪 |
|d  | x | 观察内存 |
|dv | info locals | 观察局部变量 |
|dt | pt | 观察数据类型（结构）|
|gu | finish | 执行到函数返回 |
|.frame | frame | 切换到当前栈帧 |
|lm | i shared | 列模块 |


|  GDB命令  | 命令别名 | 功能说明 |
|----------|---------|---------|
|info | I | 显示一些信息，如info b，显示设置的断点 |
|contnue| c | 继续执行，直到断点或程序结束 |
|backtrace| bt | 栈回溯 |
|ptype | pt | 显示变量类型  |


Stallman的教程
http://www.unknownroad.com/rtfm/gdbtut/gdbtoc.html