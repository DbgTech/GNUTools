
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

调试中其他的操作其实就和其他的调试器大同小异了，比如设置断点，单步，源码下一步（step/next），指令下一步（stepi/nexti），完成当前函数（finish）等，不再一一列举。



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