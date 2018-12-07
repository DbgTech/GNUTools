
# GDB使用笔记 #

[TOC]

### GDB调试器原理 ###

gdb调试器是使用Linux上提供的`ptrace`系统调用来实现的，ptrace 系统调用的原型如下所示。

```
long ptrace(enum __ptrace_request request,  // 定义要执行动作
			pid_t pid,                      // Linux进程的ID
            void *addr,                     // 要拷贝地址值
            void *data);                    // 存放要拷贝数据
```

系统调用`ptrace`提供了一种方法来让调用进程可以观察和控制其他进程的执行，检查和改变其内存以及寄存器。它主要用于实现断点调试和系统调用追踪。

当进程被追踪后，被追踪进程接收到除了`SIGKILL`之外的其他任何信号都将停止，跟踪进程会在下一次调用`wait*`函数时收到通知（函数返回值中会有状态值包含被跟踪进程停止原因）。当被跟踪进程停止后，跟踪者进程就可以使用各种`ptrace`请求来查看和修改被跟踪进程。

`reqeust`参数可取的几个典型参数：`PTRACE_TRACEME`表示本进程将被其父进程跟踪，交付给这个进程的所有信号（除`SIGKILL`之外）都将使其停止，父进程将通过`wait()`获知这一情况。`PTRACE_ATTACH`表示挂到一个指定的进程，使其成为当前进程跟踪的子进程，子进程的行为等同于它进行了一次`PTRACE_TRACEME`操作。`PTRACE_CONT`表示继续运行之前停止的子进程，可同时向子进程交付指定的信号。

例如，运行gdb，输入`attach pid`，`gdb`对指定进程执行下述操作：

```
ptrace(PTRACE_ATTACH, pid, 0, 0);
```

如果通过gdb中启动指定的可执行程序，即`gdb`中输入`run`命令，`gdb`执行下述操作：

* 通过`fork()`系统调用创建一个新进程。
* 在新创建的子进程中执行下述操作：`ptrace(PTRACE_TRACEME, 0, 0, 0);`。
* 在子进程中通过`execv()`系统调用加载用户指定的可执行文件。

**信号**

gdb的调试基础其实就是Linux中的信号，整个调试过程都是建立在信号基础上的。

* 在使用参数为`PTRACE_TRACEME`或`PTRACE_ATTACH`的`ptrace`系统调用建立调试关系之后，交付给目标程序的任何信号（除`SIGKILL`之外）都将被gdb先行截获，或在远程调试中被gdbserver截获并通知gdb。
* gdb因此有机会对信号进行相应处理，并根据信号的属性决定在继续目标程序运行时是否将之前截获的信号实际交付给目标程序。
* 信号是实现断点功能的基础。以x86为例，向某个地址打入断点，实际上就是往该地址写入断点指令`INT 3`，即`0xCC`。目标程序运行到这条指令之后就会触发`SIGTRAP`信号，gdb捕获到这个信号，根据目标程序当前停止位置查询 gdb 维护的断点链表，若发现在该地址确实存在断点，则可判定为断点命中。
* gdb暂停目标程序运行的方法是向其发送`SIGSTOP`信号。`kill_lwp(process->head.id, SIGSTOP);`

调试中其他的操作其实就和其他的调试器大同小异了，比如设置断点，单步，源码下一步（step/next），指令下一步（stepi/nexti），完成当前函数（finish）等，不再一一列举。这块后面可以尝试编写一个调试器，来熟悉一下这个编程过程。

###GDB文档笔记###

GDB文档是学习GDB最好的材料，但是它是英文的资料，不太容易看！这里将看过的内容做个笔记，方便以后使用中遇到问题来查看。

**启动gdb**

最简单的形式就是在shell命令中执行`gdb`即启动了。

```
gdb program            // 指定可执行程序
gdb program core       // 指定可执行程序和转储文件
gdb program 1234       // 指定可执行程序和进程ID，调试正在运行程序
```

注意这里对于对第二个参数，gdb优先当作核心转储文件，如果找不到该名字文件时，才将它当作进程ID号处理。其实可执行程序和后面两个参数可以添加额外的选项来明确它们。`-e file/-exec file`来指定可执行文件为file，`-se file`则用来指定可执行文件和符号文件源自同一文件，`-c file/-core file`使用文件file作为转储文件，`-p number/-pid number`将number作为要挂入的进程ID，这样就很明确指出了各个文件，不会出现上面的二义性。

如果在用gdb启动调试程序时就想给程序指定参数，可以使用选项`--args`，如下调试gcc编译程序的过程。

```
gdb --args gcc -O2 -c foo.c
```

```
gdb -q/--quiet
gdb --silent       用于指定gdb不要输出版本和版权信息

gdb -h/-help       显示gdb启动的帮助，不包括gdb内部命令。

-s file/-symbols file 从文件file中读取符号

-x file/-command file 执行文件file中的命令。

-ex command/-eval-command command 执行一条GDB命令，即command。

-ix file/-init-command file 从文件file中执行命令，这个命令执行在加载子进程之前。

-d directory/-directory directory 将directory目录添加到源码和脚本文件搜索路径中。

-r/-readnow 立即读取每一个符号文件的整个符号表

-tui 用于启动GDB的GUI窗口形式
```

gdb在启动时会加载初始化文件，首先会加载系统范围的初始化文件，它的路径使用`--with-system-gdbinit`配置选项指定，如果指定了gdb会先加载它。其次为home目录的`.gdbinit`文件，最后会加载当前工作目录的`.gdbinit`文件。

退出GDB则使用如下的两种命令：

```
q/quit 退出GDB

Ctrl-D 文件结尾符号也可以退出。
```

快捷键`Ctrl-C`并不退出GDB，它是将正在执行的被调试程序中断下来。

在GDB中可以直接执行shell命令而不用退出GDB，对于make是特特利，它可以直接执行，如同在shell中执行一样，而不需额外的修饰。

```
shell command-string/!command-string  可以直接执行`command-string`所指定的命令而不需退出gdb。
```

**gdb日志**

其实gdb的调试调试过程是可以被记录下来的，直接将屏幕上显示的内容记录到文件中。

```
set logging on                    开启日志功能
set logging off                   关闭日志功能
set logging file filename         设置日志记录的文件名称
set logging overwrite [on|off]    设置日志文件覆写的开启与关闭，默认是追加到日志文件
set logging redirect [on|off]     默认重定向，即进入终端也进入日志文件，设置重定向开启后，则只进入文件

show logging                      显示当前的日志功能设置
```

**gdb命令**

gdb命令还是比较随意的，它可以简写，即指使用命令起始的几个字符，只需要不出现歧义即可（二义性）。输入命令的一部分后，可以使用两次`TAB`键来将命令补充完整或显示所有可选的命令。不输入任何内容直接回车则是重复上一条命令，对于`list`和`x`命令则不是简单重复，gdb会重新组织参数。

gdb命令语法也比较简单，以命令起始，后面跟着命令的参数，命令的长度不受限制。符号`#`后的内容是注释，这对于命令文件比较有用。

两次`Tab`键不但可以不全gdb的命令，还可以不全调试文件中的符号信息，对于输入部分符号，gdb可以搜索符号表，进行不全或显示可选的所有符号。对于`C++`中的重载符号，则需要在符号前输入`'`单引号，提示gdb这是`C++`重载函数，比如`b 'buble(`会提示出`bubble(double, double)和bubble(int, int)`符号。

**gdb获取帮助**

`(gdb) h / help`可以显示gdb的帮助内容，显示内容为命令的类别，如下列举一下帮助内容：

```
(gdb) help
List of classes of commands:

aliases -- Aliases of other commands                    // 别名
breakpoints -- Making program stop at certain points    // 断点
data -- Examining data                                  // 查看数据
files -- Specifying and examining files                 // 指定和检查文件
internals -- Maintenance commands                       // gdb维护命令
obscure -- Obscure features                             // 隐晦功能
running -- Running the program                          // 运行程序
stack -- Examining the stack                            // 查看栈内容
status -- Status inquiries                              // 状态查询
support -- Support facilities                           // 支持设施
tracepoints -- Tracing of program execution without stopping the program // 不暂停程序来追踪程序
user-defined -- User-defined commands                   // 用户定义命令

Type "help" followed by a class name for a list of commands in that class.
Type "help all" for the list of all commands.
Type "help" followed by command name for full documentation.
Type "apropos word" to search for commands related to "word".
Command name abbreviations are allowed if unambiguous.
```

下面注释有几行，第一行为输入`help`加上一个类别名字来列举该类别中的命令；`help all`列举所有的命令；`help`加命令名字显示命令的完整文档；`apropos word`用于搜索和`word`相关的命令。

gdb有一个`compelte`命令可以用来将一个命令补充完整，比如`complete i`会列举出如下内容：

```
if
ignore
inferior
info
init-if-undefined
interpreter-exec
interrupt
```

有几个命令需要额外说明一下，`info`，`set`，`show`三个命令。`info`用于描述被调试程序的状态，例如`info args`显示当前执行程序的参数，`info registers`显示当前程序暂停状态的寄存器内容，使用`help info`可以看到info子命令的完整列表。`show`命令与info命令相对应，它用于显示gdb本身的状态，比如`show radix`显示gdb当前显示数字使用的数基，十六进制，还是十进制，直接输入show命令显示gdb中可修改的参数，以及这些参数当前的值。`set`命令既可以用来设置被调试程序状态，比如`set $rax = 10`修改寄存器，还可以修改程序变量的值等，同时还可以修改gdb本身的状态值，比如设置gdb的提示符为`$`可以用`set prompt $`。

如下几个show的杂项子命令是没办法用set命令进行设置，如下：

```
show version         显示GDB的版本信息

show copying
info copying          显示GDB的复制许可

show warranty
info warranty          显示GDB的授权信息

show configuration     GDB的编译配置，报告GDB的bug时有用。
```

**GDB运行程序**

gcc编译时，`-g`选项为编译程序保留调试信息，`-ggdb3`选项可以保留宏定义，以便调试宏。

`info macro` 查看红在那些文件被引用，宏定义什么样子

`macro` 可以查看宏展开的样子




**多线程调试**

`info thread` 查看当前进程的线程

`thread <ID>` 切换调试的线程未指定ID的线程

`break 'file.c':100 thread all`  在file.c文件100行处为所有的线程设置断点

`set scheduler-locking off|on|step` 用step和continue命令调试当前被调试线程时，其他线程也是同时执行的，怎么只让被调试程序执行呢？off不锁定任何线程，所有线程都执行，默认值。on 只有当前被调试程序会执行，step在单步时，除了next过一个函数情况外，只有当前线程会执行。

``

``

``

**源文件**




### 代码反汇编 ###

`disas/disassemble`命令可以对代码进行反汇编，如果由符号可以直接使用符号名进行反汇编；否则如果使用地址就需要两个参数。

```
// disas start,end / disas start,+length两种形式，start为起始地址，end为结束地址，+length为反汇编长度
(gdb) disas 0x32c4, 0x32e4
Dump of assembler code from 0x32c4 to 0x32e4:
   0x32c4 <main+204>:      addil 0,dp
   0x32c8 <main+208>:      ldw 0x22c(sr0,r1),r26
   0x32cc <main+212>:      ldil 0x3000,r31
   0x32d0 <main+216>:      ble 0x3f8(sr4,r31)
   0x32d4 <main+220>:      ldo 0(r31),rp
   0x32d8 <main+224>:      addil -0x800,dp
   0x32dc <main+228>:      ldo 0x588(r1),r26
   0x32e0 <main+232>:      ldil 0x3000,r31

// disas /m XXXXX    反汇编XXXXX函数，同时将源码列出
(dbg) disas /m main
Dump of assembler code for function main:
5       {
   0x08048330 <+0>:    push   %ebp
   0x08048331 <+1>:    mov    %esp,%ebp
   0x08048333 <+3>:    sub    $0x8,%esp
   0x08048336 <+6>:    and    $0xfffffff0,%esp
   0x08048339 <+9>:    sub    $0x10,%esp

6         printf ("Hello.\n");
=> 0x0804833c <+12>:   movl   $0x8048440,(%esp)
   0x08048343 <+19>:   call   0x8048284 <puts@plt>

7         return 0;
8       }
   0x08048348 <+24>:   mov    $0x0,%eax
   0x0804834d <+29>:   leave
   0x0804834e <+30>:   ret
```

`disas`具有三个参数，`\r`表示将原始代码的十六进制打印出来；`/s`和`/m`表示如果有源代码可用，则将汇编对应的源代码也打印出来，`/m`参数不如`/s`打印信息详细，已经被废掉了。

对于打印指定文件内的函数汇编，比如`foo.c`中的`bar()`函数，需要使用`disassemble 'foo.c'::bar`形式，而不能使用`disassemble foo.c:bar`形式。

还有一种查看汇编代码的方法是使用`x`命令，它用于查看内存中数据，也可以将内存数据显示为汇编代码。

```
// x /ni address  反汇编address处的n条汇编指令
(gdb) x /5i 0x0000555559544474
   0x555559544474:	mov    %r12,%rdi
   0x555559544477:	mov    %r14,%rsi
   0x55555954447a:	mov    %rax,%rdx
   0x55555954447d:	callq  0x555557c7e330
   0x555559544482:	mov    %r15,%rdi
// x /ni $rip - 30 从rip地址向后30条指令开始反汇编

```

GDB汇编有几个参数，如下：

```
set disassembler-options option1[,option2…]
show disassembler-options	// 用于定义给反汇编器传入的指定信息，默认为空

set disassembly-flavor instruction-set
show disassembly-flavor		// 通过disas或x/i命令反汇编时，选择指令集，仅对X86家族有效

set disassemble-next-line	// 当执行停止时，是否反汇编以一行源码或指令 ON/OFF/AUTO，默认值是OFF
show disassemble-next-line
```

###GDB调试汇编代码###

gdb是GNU的源码级调试器，它是Linux上的标准调试器。gdb既可用于高级语言，像C和`C++`，的调试，也可以用于汇编语言的调试。

`gdb [program]`，启动程序`program`，如果`program`需要参数可以使用`gdb --args program param1 param2 ...`形式来启动程序。

```
h[elp] [keyword]
	显示帮助信息。

r[un] [args]
	开始执行程序，如果程序需要命令行参数（比如 foo hi 3），你应该在这里指定程序参数（即run hi 3）

b[reak] [address]
	在指定的地址处设置断点，如果不指定地址则是在当前地址处设置断点。`address`参数可以给定符号化地址，比如`main`，或者给定数字化地址，比如`*0x10a38`（这里的`*`是必须的）。

c[ontinue]
	在遇到断点停止后，使用该命令继续执行程序。

i[nfo] b[reak]
	显示当前设置的所有断点，按照序号排列。

d[elete] b[reakpoints] number
	删除指定编号的断点。

p[rint][/format] expr
	使用指定的格式打印表达式的值，默认打印十进制数。表达式可以包括程序变量或寄存器，打印寄存器时要在寄存器前加`$`而不是`%`符号。比较有用的格式有如下几种：
	d decimal	十进制
	x hex       十六进制
	t binary    二进制
 	f floating point 浮点数
	i instruction 指令
 	c character 字符
	例如，要显示寄存器%rdi的十进制数，则输入p/d $rdi。注意需要使用64位格式的寄存器名字，当前指令寄存器值`p/x $rip`。

i[nfo] r[egisters] register
	一种可选方式来打印寄存器值，如果不指定寄存器则打印所有寄存器值。指定寄存器不需要指定`$`或`%`。

x/[count][format] [address]
	检查指定内存地址的内容，如果不指定地址值，则检查当前地址的内容。如果指定了count，则显示指定数量的值。地址可以是符号化（即main）或数字化的形式（即0x10a44）)。格式和print命令了类似，对于打印程序指令特别有用，比如 x/100i foo反汇编并且打印出foo处的100条指令。

disas[semble] address    // 符号化地址
disas[semble] address ,address
disas[semble] address [+number]

	另外一种打印汇编程序指令的方法，在一个地址周边或两个地址之间的指令。

set var = expr
	设置指定寄存器或内存地址的内容为表达式expr的值，例如: set $rdi=0x456789AB or set myVar=myVar*2.

s[tep]i
	执行一条指令，然后回调gdb中停下来。

n[ext]i
	类似stepi，如果指令是一条子函数调用，nexti则会跳过子函数，而stepi则会跳入子函数中执行。

whe[re]
	显示当前的活动栈。

backtrace/bt
	显示当前栈帧。

q[uit]
	退出gdb
```


By Andy@2018-12-04 20:46:23