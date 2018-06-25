
#GDB常用命令#

GDB为Unix类系统中默认的`C\C++`调试器，很多其他的调试器都是基于GDB开发，比如CGDB，DDD，以及Eclipse也是封装GDB来进行`C/C++`调试。学习完GDB后，其他封装了GDB的调试器用起来也就游刃有余了！

GDB的命令在一行内输入，并没有限制命令行的长度。对于记不住的命令，输入一部分可以使用`TAB`键补充完整命令，类似Shell的功能。

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

另外一种指定调试程序的命令是`file`，如下：

```
$ gdb
GNU gdb (Ubuntu 7.7.1-0ubuntu5~14.04.2) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
......
(gdb) file test

(gdb) run 1 2 3		// run [args]
```

另外一种可以在启动gdb时就指定被调试程序和参数`gdb --args gcc -O2 -c foo.c`，用gdb调试gcc编译`foo.c`文件。

总结一下GDB启动程序或挂接程序有如下的几个方法：

```
$ gdb program

$ gdb program core		// 使用core文件启动程序

$ gdb program 1234		// attach到进程1234，以program作为调试目标

$ gdb program ./1234		// GDB会首先将1234当作core文件名或PID，`./`则直接告诉GDB为文件
```

在启动GDB时有一些选项和参数可用，如下表所列举：

| 选项和参数 | 简写 |            含义        |
|----------|-----|------------------------|
|`-symbols file`| `-s file`| 从file文件读入符号表|
|`-exec file`|`-e file`| 用file作为可执行文件执行|
|`-se file`| 无 | 将file当作可执行程序，且从中读取符号|
|`-core file`|`-c file`| 文件file为core dump文件|
|`-pid number`|`-p number`| 挂接到pid为number的进程上|
|`-command file`|`-x file`| 从文件file中执行GDB命令|
|`-eval-command command`|`-ex command`| 执行GDB命令command，可以写多个`-ex`选项，指定多个命令|
|`-directory directory`|`-d directory`| 添加目录directory到源码和脚本文件搜索路径 |
|`-readnow`|`-r`| 立刻加载所有符号，而非执行时加载 |
|`-nx/-n`| 无 | 在GDB启动时不执行初始化文件里面的命令 |
|`-quiet/-silent`|`-q`| 安静模式，不打印介绍和版权信息 |
|`-nowindows`|`-nw`| 无窗口，以命令行接口运行 |
|`-cd directory`| 无 | 用directory作为工作目录，非当前目录 |
|`-baud bps`|`-b bps`| 远程调试设置串口波特率 |
|`-l timeout`| 无 | 远程调试的连接超时 |
|`-gui`| 无 | 启动时激活文本用户几口，管理文本窗口 |

在GDB启动阶段，按照如下步骤初始化：

* 启动命令行解释器（如果GDB启动指定`-args`，则阻止命令行选项处理）
* 读入home目录下的初始化文件（如果有`.gdbinit`文件），并执行所有命令
* 处理命令行选项和参数
* 读入和执行当前工作目录下的初始化文件（如果有`.gdbinit`文件），只执行和home目录初始文件不同命令
* 读入命令文件，如果使用`-x`选项指定了文件
* 读入记录在历史文件里的命令历史

如果要被调试进程的参数可以使用`show args`命令，同时要修改被调试进程的命令行参数可以使用`set args [args]`来进行修改

**退出GDB**

退出GDB可以输入`quit/q`命令，或者`Ctrl-d`快捷键。

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

`watch`命令可以监视表达式，值变化时中断。表达式可以是简单的变量或者是复杂的表达式，由变量和运算符组成

```
watch a*b + c/d
watch *(*int)0x12345678
watch *global_ptr
```

监视点的实现依赖于系统，它既可以用软件实现也可以用硬件实现。GDB实现软件监视点是通过单步软件，并在每次暂停时检查变量值，这使得监视点设置之后的执行效率比正常执行慢几百倍。一些系统，如PowerPC和X86-CPU的机器，GDB可以支持硬件监视点，硬件断点不会降低程序执行效率。

`rwatch`命令可以监控表达式值被读的点，`awatch`则可以监控表达式被读或写的点。

```
rwatch [-l|-location] expr [thread thread-id] [mask maskvalue]

awatch [-l|-location] expr [thread thread-id] [mask maskvalue]
```

这种报告都是事后报告，即读写动作发生后，在下一条指令时才能暂停下来。

其中的`thread`参数可以限制到指定线程上，也就是指定线程触发了读写动作时才会断下来。这个参数只对硬件实现的监视点才有效，软件实现监视点无效。


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

`whatis`命令也可以查看变量的类型，但是对于结构体或类它不会列出类型的详细信息。

**方便变量**

GDB维护了方便变量，以`$`开头，比如在显示变量时，总会使用`$4`等类似的方式标记内容。可以用`$4`来引用刚刚显示过的它所代表的变量。

`p $`会显示刚刚显示过的变量值，`p $n`显示编号为n的方便变量的值，而`$$`显示从`$`开始的倒数第一个变量的值，`$$n`显示从`$`开始倒数第n个显示的值。

`$_`变量被赋值为最近执行的x命令中被检查的地址，`$__`变量则被赋值为最近执行的x命令中检查地址的值。

使用 `set $foo = *object_ptr`命令来设置一个方便变量。

`$_thread`和`$_gthread`两个变量用在条件中，用于指示当前线程（`info thread`中的编号）。

`$ecx`可以指示寄存器ECX的值，例如`break write if $rsi == 2`当寄存器RSI值为2时则在write函数暂停下来。

在GDB中要显示值时，往往不是单一变量，而是要用表达式的形式来表达。

* `C/C++`中的表达式
* `addr@len`用于表示一个指针指向的数组，数组元素个数为len
* `file::nm`表示在file中定义的nm函数或变量
* `{type}addr` 以type指示的类型格式来读取addr处的数据
* `$`最近显示的值
* `$n` 最近显示的变量（或表达式）中第n个的值
* `$$` 最近显示的值中从`$`向前数的第一个
* `$$n` 最近显示的值中从`$`向前数的第n个
* `$_` x命令最近检查的地址
* `$__` x命令最近检查的地址处的值
* `$val` 方便变量，可以赋值任何值
* show values [n] 显示最近显示的10个值，如果指定n则显示`$n`附近的值
* show conv 显示所有的方便变量


###栈帧###

`bachtrace`命令可以显示当前执行位置的栈帧，如下代码块所示。`backtrace [n]/bt [n]`打印栈上所有帧，或者n帧。

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

`info frame`查看当前选择的栈帧内容，`info frame addr`则查看在addr地址处的栈帧的信息。

`info args`选择的栈帧的参数，`info locals`显示选择的栈帧的本地变量。

`info reg [rn]`显示当前选择栈帧的寄存器值，如果指定rn参数则显示指定寄存器值。`info all-reg [rn]`则显示所有寄存器，包括浮点寄存器的值。

`where`命令可以查看当前的栈帧情况。

###寄存器###

`info reg`可以显示当前寄存器的内容。

`set $<name>=<value>`可以设置寄存器的值，比如`set $ecx=2`将ECX寄存器的值设置为2。

`info all-reg`则显示所有的寄存器值，包括浮点数寄存器。


###内存###

print和display命令允许指定显示的格式，比如`(gdb) p /x y`将y变量按照十六进制格式显示。其他的还有`/c`表示按照字符形式显示，`\s`按照字符串显示，`\f`按照浮点格式显示。

###源码调试###

`list/l`用于列举当前位置对应的源代码。

`dir names` 增加目录names到源码路径前面，`dir dirname/directory dirname`命令可以将dirname路径添加到源码搜索路径中。

`directory /home/ge/eglibc-2.15/libio`将`/home/ge/eglibc-2.15/libio`路径添加到源码搜索路径。

其中`$cdir`指编译目录，`$cwd`指向当前工作目录。

`dir` 清空源码路径

`show dir` 显示当前的源码路径

`list`显示接下来执行的10行源码，`list -`显示当前行的前面10行，`list lines`显示lines附件的源码，可以使用`[file:]num`，`[file:]function`，`+off`，`-off`，`*address`等来指定要显示的位置。

`list start,end` 从start行开始显示到end行结束。

`info line num`显示源码行num的对应编译后的代码的起始和结束地址。

`info source` 显示当前源码文件的名字，`info sources`列举处所有在用的源码文件。

`forw regex` 从当前行先前搜索满足regex的源码行（接下来要执行），`rev regex`向后搜索满足特征的源码行。

ubuntu的源码也是可以根据当前调试的程序下载，比如`ls`命令的源码。

```
[~/src]$ apt-get source coreutils
[~/src]$ sudo apt-get install coreutils-dbgsym
[~/src]$ gdb /bin/ls
GNU gdb (GDB) 7.1-ubuntu
(gdb) list main
1192 ls.c: No such file or directory.
in ls.c
(gdb) directory ~/src/coreutils-7.4/src/
Source directories searched: /home/nelhage/src/coreutils-7.4:$cdir:$cwd
(gdb) list main
1192 }
1193 }
```

如下可以将内核的源码文件下载下来。

```
[~/src]$ apt-get source linux-image-2.6.32-25-generic
[~/src]$ sudo apt-get install linux-image-2.6.32-25-generic-dbgsym
[~/src]$ gdb /usr/lib/debug/boot/vmlinux-2.6.32-25-generic
(gdb) list schedule
5519 /build/buildd/linux-2.6.32/kernel/sched.c: No such file or directory.
in /build/buildd/linux-2.6.32/kernel/sched.c
(gdb) set substitute-path /build/buildd/linux-2.6.32
/home/nelhage/src/linux-2.6.32/
(gdb) list schedule
5519
5520 static void put_prev_task(struct rq *rq, struct task_struct *p)
5521 {
```

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

`display /3i $pc`在每次断点断下来或单步执行后，输出当前位置的三条指令。

###info###

`info`通常用于显示被调试程序的信息

###调试符号###

GCC编译时要使用`-g/--gen-debug`参数来保留符号，这样在调试时才会有符号。

Ubuntu的符号服务器:`http://ddebs.ubuntu.com/pool/main/l/linux/`，如果要使用Ubuntu的符号文件，需要手动下载下来，参考`http://askubuntu.com/questions/197016/how-to-install-a-package-that-contains-ubuntu-kernel-debug-symbols`页面可以将符号下载并安装。

![图 3](\image\gdb-using-download-ubuntu-symbols.jpg)

`file [filename]`或`symbol-file [filename]`可以用于从filename文件中读取符号表，`PATH`环境变量会当作搜索路径。`file`命令用于加载符号和程序在一个文件的情况，比如本地编译的程序。


* info address s #show where symbol s is stored
* info func [regex] #show names, types of defined functions (all, or matching regex)
* info var [regex] #show names, types of global variables (all, or
* matching regex)
* whatis [expr] #show data type of expr [or $] without evaluating;
* ptype [expr] #ptype gives more detail
* ptype type #describe type, struct, union, or enum

符号和地址互查，可以使用如下的命令：

`info address symbol`命令用于查找符号symbol存储的地址，对于寄存器变量则显示变量存储在那个寄存器中；非寄存器变量则打印变量存储的栈帧偏移。

`info symbol addr` 打印存储在地址addr处的符号名字，如果没有符号存储在指定的地址处，GDB会打印最近的符号，并显示从addr处的偏移。


![图 4](\image\gdb-using-check-virtual-func-table.jpg)

###多线程调试###

`info threads`列举当前进程的所有线程，`*`代表当前线程。

`thread thread-id`用于切换到线程`thread-id`，`thread 2`用于切换到2号线程。

对多个线程执行命令，`thread apply all bt`打印所有线程的堆栈。`thread apply [thread-id-list | all [-ascending]] command`则可以在特定的几个线程或所有线程上执行命令。

`thread name [name]`显示线程名字。

###信号处理###

`info signals`命令可以列举处当前进程的所有信号。

`info handle`命令则列举出当前进程信号的处理规则，`handle signal act`可以用于设置信号的处理规则，signal用于指定要设置的信号名字，act为处理动作，包括如下行为。

* print 打印信号通知
* noprint 信号触发时静默
* stop 信号触发时暂停执行
* nostop 信号触发时不暂停
* pass 允许程序处理信号
* nopass 程序将接收不到信号

`handle SIGPIPE nostop print` 设置`SIGPIPE`信号不暂停只输出。



###GUI###

gdb也有GUI调试模式，在启动gdb时添加`-tui`参数，启动后就可以看到源码窗口，调试过程中可以根据源码进行调试。但是这种界面窗口不太好用，容易出现混乱。

![图 ](\image\gdb-using-tui.jpg)

`layout asm` 可以打开汇编窗口，`focus asm`将焦点切换到ASM窗口中。

另外一种更好用的基于gdb的GUI调试器是CGDB，它提供的源码窗口更好用一些。



另一种界面

>https://github.com/snare/voltron

###GDB下程（inferior）###

当前GDB曾经调试过的程序的列表，可以在调试过的可执行程序之间切换。

`info inferiors`

###Shell命令###

在GDB中是可以直接执行Shell命令的，可以使用`!shellcmd`形式来执行Shell的命令`shellcmd`。

另外一种执行shell命令的方式是`shell command string`，这样就会调用shell来执行`command string`命令。

在GDB中执行make是个特例，make也是shell中执行的程序，但是在GDB中不需要以shell命令形式执行，可以直接运行make命令，`gdb> make make-args`就会直接运行make命令了。


###GDB设置###

`show`用于显示调试器GDB自身的信息（主要是GDB的一些设置信息）；如果要设置GDB的配置信息可以使用`set`命令。

`show args`可以显示为调试程序设置的命令行参数

`set args`为被调试进程设置命令行参数。

`show path`显示执行路径。

`show environment [varname]`显示环境变量，如果指定了varname，则只显示它的特定环境变量的值。

`cd [directory]`可以将GDB的当前目录切换到directory目录，pwd显示gdb当前的工作目录。


###GDB命令文件###

`https://sourceware.org/gdb/onlinedocs/gdb/Command-Files.html`


###转储文件###

在gdb调试下，使用`generate-core-file`命令可以转储当前进程的状态信息。

内核转储文件和调试对象，就可以在非当前环境下查看转储文件当时进程的运行状态（寄存器和内存值等）。它和Windows下的dump类似。

https://blog.csdn.net/xuzhina/article/category/1322964/3


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

###GDB命令简写###

|  GDB命令  | 命令别名 | 功能说明 |
|----------|---------|---------|
|info | I | 显示一些信息，如info b，显示设置的断点 |
|contnue| c | 继续执行，直到断点或程序结束 |
|backtrace| bt | 栈回溯 |
|ptype | pt | 显示变量类型  |


Stallman的教程
http://www.unknownroad.com/rtfm/gdbtut/gdbtoc.html


###GDB-Refcard翻译###

如下是GDB命令参考卡的内容，后面逐条翻译一下命令解释：

![图2 ](\image\gdb-using-gdbrefcard-1.jpg)
![图3 ](\image\gdb-using-gdbrefcard-2.jpg)

GDB命令快速参考（版本5）

`[ ]` 包含的内容表示可选的参数；`. . .`显示一个或更多的参数。

**必要命令**

`gdb program [core]`： 调试程序，[使用内核dump文件core]
`b [file:]function`：在函数function[在文件file中]上设置断点
`run [arglist]`：启动程序[使用参数arglist]
`bt/backtrace`: 显示程序函数栈
`p expr`：显示表达式expr的值
`c/continue`：继续执行程序
`n/next`: 执行下一行源码，跳过函数调用
`s/step`: 执行下一行远嘛，跳进函数调用中

**启动GDB**

`gdb` 启动GDB，没有制定调试文件
`gdb program` 开始调试program
`gdb program core` 调试又program生成的内核dump文件core
`gdb --help` 描述命令行选项，帮助信息

**暂停GDB**

`quit` 退出GDB，也可以使用`q`或EOF(即`Ctrl-d`)
`INTERRUPT` (即`Ctrl-c`) 终止当前的命令或发送终止命令给运行的进程

**获取帮助**

`help` 列举命令类别信息
`help class` 每一行描述一个在class类别中的命令
`help command` 描述命令command的信息

**执行程序**

`run arglist` 启动程序，并给它传参数arglist
`run` 使用当前的参数列表启动程序（默认设置的参数，可能没有）
`run . . . <inf >outf` 使用输入，输出重定向启动程序
`kill` 杀死正在运行的程序
`tty dev` 使用dev作为下一次运行中的stdin和stdout
`set args arglist` 指定arglist作为下一次运行中的参数
`set args` 设置参数列表为空
`show args` 显示当前程序运行使用的参数
`show env` 显示所有的环境变量
`show env var` 显示环境变量var的值
`set env var string` 设置环境变量var的值为string
`unset env var` 从环境变量列表中删除变量var的定义

**Shell命令**

`cd dir` 切换当前的工作目录到`dir`
`pwd` 打印当前的工作目录
`make . . .` 调用`make`命令
`shell cmd` 执行任意的Shell命令字符串`cmd`

Breakpoints and Watchpoints
break [file:]line
b [file:]line
set breakpoint at line number [in file]
eg: break main.c:37
break [file:]func set breakpoint at func [in file]
break +offset
break -offset
set break at offset lines from current stop
break *addr set breakpoint at address addr
break set breakpoint at next instruction
break . . . if expr break conditionally on nonzero expr
cond n [expr] new conditional expression on breakpoint
n; make unconditional if no expr
tbreak . . . temporary break; disable when reached
rbreak [file:]regex break on all functions matching regex [in
file]
watch expr set a watchpoint for expression expr
catch event break at event, which may be catch,
throw, exec, fork, vfork, load, or
unload.
info break show defined breakpoints
info watch show defined watchpoints
clear delete breakpoints at next instruction
clear [file:]fun delete breakpoints at entry to fun()
clear [file:]line delete breakpoints on source line
delete [n] delete breakpoints [or breakpoint n]
disable [n] disable breakpoints [or breakpoint n]
enable [n] enable breakpoints [or breakpoint n]
enable once [n] enable breakpoints [or breakpoint n];
disable again when reached
enable del [n] enable breakpoints [or breakpoint n];
delete when reached
ignore n count ignore breakpoint n, count times
commands n
[silent]
command-list
execute GDB command-list every time
breakpoint n is reached. [silent
suppresses default display]
end end of command-list
Program Stack
backtrace [n]
bt [n]
print trace of all frames in stack; or of n
frames—innermost if n>0, outermost if
n<0
frame [n] select frame number n or frame at address
n; if no n, display current frame
up n select frame n frames up
down n select frame n frames down
info frame [addr] describe selected frame, or frame at addr
info args arguments of selected frame
info locals local variables of selected frame
info reg [rn]. . .
info all-reg [rn]
register values [for regs rn] in selected
frame; all-reg includes floating point
Execution Control
continue [count]
c [count]
continue running; if count specified, ignore
this breakpoint next count times
step [count]
s [count]
execute until another line reached; repeat
count times if specified
stepi [count]
si [count]
step by machine instructions rather than
source lines
next [count]
n [count]
execute next line, including any function
calls
nexti [count]
ni [count]
next machine instruction rather than
source line
until [location] run until next instruction (or location)
finish run until selected stack frame returns
return [expr] pop selected stack frame without
executing [setting return value]
signal num resume execution with signal s (none if 0)
jump line
jump *address
resume execution at specified line number
or address
set var=expr evaluate expr without displaying it; use
for altering program variables
Display
print [/f ] [expr]
p [/f ] [expr]
show value of expr [or last value $]
according to format f:
x hexadecimal
d signed decimal
u unsigned decimal
o octal
t binary
a address, absolute and relative
c character
f floating point
call [/f ] expr like print but does not display void
x [/Nuf ] expr examine memory at address expr; optional
format spec follows slash
N count of how many units to display
u unit size; one of
b individual bytes
h halfwords (two bytes)
w words (four bytes)
g giant words (eight bytes)
f printing format. Any print format, or
s null-terminated string
i machine instructions
disassem [addr] display memory as machine instructions
Automatic Display
display [/f ] expr show value of expr each time program
stops [according to format f ]
display display all enabled expressions on list
undisplay n remove number(s) n from list of
automatically displayed expressions
disable disp n disable display for expression(s) number n
enable disp n enable display for expression(s) number n
info display numbered list of display expressions
Expressions
expr an expression in C, C++, or Modula-2
(including function calls), or:
addr@len an array of len elements beginning at
addr
file::nm a variable or function nm defined in file
{type}addr read memory at addr as specified type
$ most recent displayed value
$n nth displayed value
$$ displayed value previous to $
$$n nth displayed value back from $
$ last address examined with x
$ value at address $
$var convenience variable; assign any value
show values [n] show last 10 values [or surrounding $n]
show conv display all convenience variables
Symbol Table
info address s show where symbol s is stored
info func [regex] show names, types of defined functions
(all, or matching regex)
info var [regex] show names, types of global variables (all,
or matching regex)
whatis [expr]
ptype [expr]
show data type of expr [or $] without
evaluating; ptype gives more detail
ptype type describe type, struct, union, or enum
GDB Scripts
source script read, execute GDB commands from file
script
define cmd
command-list
create new GDB command cmd; execute
script defined by command-list
end end of command-list
document cmd
help-text
create online documentation for new GDB
command cmd
end end of help-text
Signals
handle signal act specify GDB actions for signal:
print announce signal
noprint be silent for signal
stop halt execution on signal
nostop do not halt execution
pass allow your program to handle signal
nopass do not allow your program to see signal
info signals show table of signals, GDB action for each
Debugging Targets
target type param connect to target machine, process, or file
help target display available targets
attach param connect to another process
detach release target from GDB control
Controlling GDB
set param value set one of GDB’s internal parameters
show param display current setting of parameter
Parameters understood by set and show:
complaint limit number of messages on unusual symbols
confirm on/off enable or disable cautionary queries
editing on/off control readline command-line editing
height lpp number of lines before pause in display
language lang Language for GDB expressions (auto, c or
modula-2)
listsize n number of lines shown by list
prompt str use str as GDB prompt
radix base octal, decimal, or hex number
representation
verbose on/off control messages when loading symbols
width cpl number of characters before line folded
write on/off Allow or forbid patching binary, core files
(when reopened with exec or core)
history . . .
h . . .
groups with the following options:
h exp off/on disable/enable readline history expansion
h file filename file for recording GDB command history
h size size number of commands kept in history list
h save off/on control use of external file for command
history
print . . .
p . . .
groups with the following options:
p address on/off print memory addresses in stacks, values
p array off/on compact or attractive format for arrays
p demangl on/off source (demangled) or internal form for
C++ symbols
p asm-dem on/off demangle C++ symbols in machineinstruction output
p elements limit number of array elements to display
p object on/off print C++ derived types for objects
p pretty off/on struct display: compact or indented
p union on/off display of union members
p vtbl off/on display of C++ virtual function tables
show commands show last 10 commands
show commands n show 10 commands around number n
show commands + show next 10 commands
Working Files
file [file] use file for both symbols and executable;
with no arg, discard both
core [file] read file as coredump; or discard
exec [file] use file as executable only; or discard
symbol [file] use symbol table from file; or discard
load file dynamically link file and add its symbols
add-sym file addr read additional symbols from file,
dynamically loaded at addr
info files display working files and targets in use
path dirs add dirs to front of path searched for
executable and symbol files
show path display executable and symbol file path
info share list names of shared libraries currently
loaded
Source Files
dir names add directory names to front of source
path
dir clear source path
show dir show current source path
list show next ten lines of source
list - show previous ten lines
list lines display source surrounding lines, specified
as:
[file:]num line number [in named file]
[file:]function beginning of function [in named file]
+off off lines after last printed
-off off lines previous to last printed
*address line containing address
list f,l from line f to line l
info line num show starting, ending addresses of
compiled code for source line num
info source show name of current source file
info sources list all source files in use
forw regex search following source lines for regex
rev regex search preceding source lines for regex
GDB under GNU Emacs
M-x gdb run GDB under Emacs
C-h m describe GDB mode
M-s step one line (step)
M-n next line (next)
M-i step one instruction (stepi)
C-c C-f finish current stack frame (finish)
M-c continue (cont)
M-u up arg frames (up)
M-d down arg frames (down)
C-x & copy number from point, insert at end
C-x SPC (in source file) set break at point
GDB License
show copying Display GNU General Public License
show warranty There is NO WARRANTY for GDB.
Display full no-warranty statement.
Copyright 
 c 1991-2014 Free Software Foundation, Inc. Author:
Roland H. Pesch
The author assumes no responsibility for any errors on this card.
This card may be freely distributed under the terms of the GNU
General Public License.
Please contribute to development of this card by annotating it.
Improvements can be sent to bug-gdb@gnu.org.
GDB itself is free software; you are welcome to distribute copies of
it under the terms of the GNU General Public License. There is
absolutely no warranty for GDB.