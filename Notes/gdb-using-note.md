
# GDB使用笔记 #

### GDB调试器原理 ###

gdb调试器是使用Linux上提供的`ptrace`系统调用来实现的，`ptrace`系统调用的原型如下所示。

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

调试中其他的操作其实就和其他的调试器大同小异了，比如设置断点，单步，源码单步（step/next），指令单步（stepi/nexti），完成当前函数（finish）等，不再一一列举。这块后面可以尝试编写一个调试器，来熟悉一下编程过程。

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
shell command-string  //可以直接执行`command-string`所指定的命令而不需退出gdb。
!command-string

// 比如
(gdb) !ls
Desktop  Downloads
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

gdb命令还是比较随意的，它可以简写，即指使用命令起始的几个字符，只需要不出现歧义即可（二义性）。输入命令的一部分后，可以使用两次`TAB`键来将命令补充完整或显示所有可选的命令。不输入任何内容直接回车则是重复上一条命令，对于`list`和`x`等命令则不是简单重复，gdb会重新组织参数。

gdb命令语法也比较简单，以命令起始，后面跟着命令的参数，命令的长度不受限制。符号`#`后的内容是注释，这对于命令文件比较有用。

两次`Tab`键不但可以不全gdb的命令，还可以不全调试文件中的符号信息，对于输入部分符号，gdb可以搜索符号表，进行不全或显示可选的所有符号。对于`C++`中的重载符号，则需要在符号前输入`'`单引号，提示gdb这是`C++`重载函数，比如`b 'buble(`会提示出`bubble(double, double)和bubble(int, int)`符号。

```
set max-completions limit	// 设置tab补全时，满足条件符号最多显示个数
show max-completions
```

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

gcc编译时，`-g`选项为编译程序保留调试信息，`-g3`选项可以保留宏定义，以便在gdb中调试宏定义。GCC编译的调试信息都是以`DWARF`格式保存，使用最新版本的DWARF格式调试信息可以获得比较好的调试体验，它是GDB目前最具表达力，并且支持最好的调试信息格式。

> `info macro` 查看宏在那些文件被引用，宏定义什么样子，`macro` 可以查看宏展开的样子。

在gdb中执行`r/run`命令可以运行程序，gdb会创建一个inferior进程（老雷起名字为下程，这里总结也用这个名字吧，但是也不太准确）来运行程序。

程序运行时有四类信息会影响到程序，分别是参数，环境变量，工作目录，标准输入输出。

可以使用`run`命令来指定程序运行的参数，`info args`可以查看当前设置的参数值；环境变量通常从GDB的环境变量中继承，`set environment`和`unset environment`命令可以修改程序执行的环境变量；工作目录也从GDB的继承，可以使用`cd`命令在GDB中修改工作目录；标准输入输出也使用GDB相同值，可以在`run`命令行进行重定向，或者使用`tty`命令设置不同设备作为输入输出。

`start`命令是另外一个运行程序的命令，它对于`C/C++`程序比较有用。`C\C++`程序执行时，会以`main`函数作为起点，在gdb中使用`start`命令可以让程序开始运行后停在`main`函数。如果不设置断点，`run`命令执行后直接让程序完整运行起来。但是对于`C/C++`程序中的全局变量或对象初始化，`start`命令则直接执行完毕。

几个有用设置：

```
set disable-randomization on/off  // 将程序的虚拟地址空间随机化关闭，有助于调试
show disable-randomization

set args        // 指定下一次程序执行参数，如果args后没有参数，则run命令执行程序时不传入参数
show args

path directory  // 将directory字符串指定的路径添加到程序执行 PATH中
show paths

show environment [varname]  // 打印环境变量`varname`的值
set environment varname [=value] // 设置环境变量varname值为value
unset environment varname   // 删除环境变量 varname，不同于`set environment varname=`

cd [directory]   // 设置GDB工作路径为directory，如果不指定directory参数，默认为`~`
pwd              // 打印GDB工作目录

info terminal    // 显示GDB记录的终端模式的信息
run > outfile    // 对程序的标准输出重定向到文件 outfile 中

tty /dev/ttyb    // 使用/dev/ttyb作为输入输出的默认终端
```

对于已经运行程序，可以使用`attach pid`命令来挂到`pid`所指定的进程上。`info files`可以显示活动的目标。使用`ps`或`jobs -l`命令行命令可以找到要挂的程序的进程ID。`attach`命令只能执行在支持进程的环境，对于裸板目标，则无法使用该方法。

```
(gdb) info files
Symbols from "/opt/360/browser360-beta/browser360".
Local exec file:
	`/opt/360/browser360-beta/browser360', file type elf64-x86-64.
	Entry point: 0x555557b50000
	0x00005555555542a8 - 0x00005555555542c4 is .interp
	0x00005555555542c4 - 0x00005555555542e4 is .note.ABI-tag
    ......
```

一旦`attach`命令执行挂上目标进程，那么gdb会首先尝试在当前目录查找可执行程序，其次在源文件搜索路径查找。使用`file`命令可以直接加载进程对应的程序。

`detach`可以将已经挂接进程直接释放掉对程序的控制，一旦执行命令，可执行程序就再次独立执行了。如果是使用`run`命令执行的进程，使用`attach`命令则会直接杀死进程。`kill`命令直接杀死正在GDB中调试子进程。

**多进程调试**

在一个gdb会话中可以运行和调试多个程序，此外，一些系统上的gdb可以让你同时运行几个程序。通常情况下，可以从多个可执行程序启动进程，并且多个进程中的每一个进程都可以是多线程进程。

gdb使用一个对象，叫做下程，来表示每个程序执行的状态。通常一个下程对应于一个进程，但是对于没有进程概念的系统也适用。下程一般在进程执行之前已经存在，在进程退出后依然在gdb中保留。gdb中每一个下程都有自己的ID，这个ID不同于进程的ID。通常下程有自己独立的地址空间，尽管对于嵌入式目标来说可能几个下程执行在一个地址空间的不同位置。每一个下程也同样有多个线程在其中执行。

```
info inferiors      // 显示GDB当前管理的所有下程，包括ID，下程标识和运行的可执行程序名称
(gdb) info inferiors
  Num  Description       Executable
  1    <null>            /bin/ls
* 2    <null>            /bin/myls

inferior infno     // 切换当前下程为infno的下程，如上ID=2的下程为当前下程

add-inferios [-copies n] [-exec executable]
    // 为gdb添加管理的下程，-copies n为添加几个副本，-exec指定可执行程序，否则为null

clone-inferior [-copies n] [infno] // 拷贝ID为infno的下程n份，默认为1份。

remove-inferior infno   //删除ID为infno的下程

detach inferior infno   // 从下程infno分离，仅仅是不调试对应进程，但是下程依然存在
                        // Description字段值为 null
kill inferior infno     // 杀死下程，但是下程记录依然存在于gdb中
```

如下为与下程相关的一些设置：

```
set print inferior-events  on/off     // 设置是否打印下程启动与退出或分离的时间信息
show print inferior-events

```

在调试器中对进程调试时，被调试的进程会有继续启动子进程的问题。默认情况下，如果被调试进程使用`fork`等命令创建了子进程时，调试器不会对被调试进程进行调试，但是如果在父进程中设置断点地址被子进程中执行到了，那么子进程会受到`SIGTRAP`信号，这会导致子进程终止。

这种情况下要调试子进程，可以使用一种方法是在子进程中`fork`返回时执行`sleep`函数来暂停子进程，然后可以在gdb中挂入子进程进行调试。其他方法就是可以通过设置gdb的设置来调试子进程。

```
set follow-fork-mode parent/child  // 对于调用fork/vfork的程序调用，设置跟随模式
        // 即调用fork等函数时，是调试子进程还是调试父进程，默认为调试父进程
show follow-fork-mode

set detach-on-fork on/off  // 执行fork时是否自动detach，默认为on
        // 默认值为on，即执行到fork时，是否进行detach操作，具体操作要依赖于follow-fork-mode
        // 如果设置为off，gdb会保持控制所有fork的进程，这样同时调试 父子进程
show detach-on-fork

set follow-exec-mode new/same // 设置调试器对程序调用exec的反应，exec会替换进程的程序映像
```

调试多进程时另外一个好用点为`checkpoint`，它是gdb保存的进程状态的快照，称作检验点，在之后的程序运行时可以回到这点。

```
checkpoint    // 保存一个当前执行程序状态的快照，该命令没参数

info checkpoints // 列举出当前调试会话中已经保存的检查点

restart checkpoint-id // 恢复程序状态到检查点ID `checkpoint-id`上。包括程序变量，寄存器和栈帧。

delete checkpoint checkpoint-id 删除`checkpoint-id`指定的检查点
```

**多线程调试**

在一些系统中，比如Linux，Solaris，单一进程也会有多个执行线程。线程在不同操作系统中有不同的具体语义。gdb提供了调试多线程程序的功能。

`info thread` 查看当前调试的所有进程中所有线程。

`thread <ID>` 切换调试的线程为指定ID的线程。

`thread apply [thread-id-list | all -[ascending]] args`，应用命令到多个线程上。

`set print thread-events` 控制线程启动和退出的消息输出。

`break 'file.c':100 thread all`  在file.c文件100行处为所有的线程设置断点

`set scheduler-locking off|on|step` 用`step`和c`ontinue`命令调试当前被调试线程时，其他线程也是同时执行的，怎么只让被调试程序执行呢？`off`不锁定任何线程，所有线程都执行，默认值为`off`。`on`只有当前被调试程序会执行，`step`在单步时，除了`next`过一个函数情况外，只有当前线程会执行。

在下程中要指定线程可以使用`inferior-num.thread-num`的语法，两个参数分别为下程的ID，以及下程中的线程id。如果gdb中只有一个下程，那么gdb不会显示`inferior-num`。

gdb调试中有两个方便变量`$_thread`和`$_gthread`分别用于表示当前线程的线程标识和全局的线程标识。

**普通断点**

**watch数据变化断点**

**catch事件断点**



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
// x /ni $rip - 30 从rip地址向后30条指令开始反汇编n条指令
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

汇编调试时一个比较有用的命令`x/20i $ip-40`，它可以打印当前执行指令前后二十条指令，用于上下文观察。

### Ubuntu中命令程序源码与符号下载 ###

** 命令所属源码包下载 **

这里以Ubuntu常用的`ls`命令为例。首先查看ls命令本身的信息。

```
~$ which ls
/bin/ls
```

搜索`/bin/ls`程序所属的程序包，用于找到对应源码包名称：

```
$ dpkg -S /bin/ls
coreutils: /bin/ls
```

使用`apt-get`命令获取源码包。首先查看下载源码的源是否已经设置，比如在Ubuntu 16.04版本上，默认的源码源是被注释的（如下），需要将对应的`deb-src`行的注释去掉。

```
$ cat /etc/apt/sources.list
#deb cdrom:[Ubuntu 16.04.4 LTS _Xenial Xerus_ - Release amd64 (20180228)]/ xenial main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted
#deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted

......
```

如果不放开这些源，会造成如下的错误：

```
$ sudo apt-get source coreutils
正在读取软件包列表... 完成
E: 您必须在 sources.list 中指定代码源(deb-src) URI
```

修改后，首先取回更新的软件包列表信息，然后执行获取源码的命令。

```
$ sudo apt-get source coreutils
正在读取软件包列表... 完成
需要下载 5,755 kB 的源代码包。
获取:1 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (dsc) [2,095 B]
获取:2 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (tar) [5,725 kB]
获取:3 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (diff) [28.3 kB]
已下载 5,755 kB，耗时 21秒 (263 kB/s)

gpgv: 于 2017年02月28日 星期二 17时24分15秒 CST 创建的签名，使用 RSA，钥匙号 778FA6F5
gpgv: 无法检查签名：找不到公钥
dpkg-source: 警告: 对 ./coreutils_8.25-2ubuntu3~16.04.dsc 校验签名失败
dpkg-source: info: extracting coreutils in coreutils-8.25
dpkg-source: info: unpacking coreutils_8.25.orig.tar.xz
dpkg-source: info: unpacking coreutils_8.25-2ubuntu3~16.04.debian.tar.xz
dpkg-source: info: applying no_ls_quoting.patch
dpkg-source: info: applying 61_whoips.patch
dpkg-source: info: applying 63_dd-appenderrors.patch
dpkg-source: info: applying 72_id_checkngroups.patch
dpkg-source: info: applying 80_fedora_sysinfo.patch
dpkg-source: info: applying 85_timer_settime.patch
dpkg-source: info: applying 99_kfbsd_fstat_patch.patch
dpkg-source: info: applying 99_hppa_longlong.patch
dpkg-source: info: applying 99_float_endian_detection.patch
dpkg-source: info: applying ppc_sha256_flags.patch
W: 文件'coreutils_8.25-2ubuntu3~16.04.dsc'无法被用户'_apt'访问，无法降低权限以进行下载。 - pkgAcquire::Run (13: 权限不够)
```

搜索`keyring`，查找要用到的公钥。

```
$ aptitude search keyring
p   debian-archive-keyring                     - Debian 档案文件的 GnuPG 存档密钥
v   debian-archive-keyring:i386                -
p   debian-keyring                             - GnuPG keys of Debian Developers and Maintainers
p   debian-ports-archive-keyring               - GnuPG archive keys of the debian-ports archive
......
```

下载公钥，可执行如下命令：

```
$ sudo aptitude install debian-keyring
下列“新”软件包将被安装。
  debian-keyring
0 个软件包被升级，新安装 1 个， 0 个将被删除， 同时 44 个将不升级。
需要获取 32.2 MB 的存档。 解包后将要使用 34.4 MB。
读取： 1 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 debian-keyring all 2016.01.20 [32.2 MB]
已下载 32.2 MB，耗时 1分 57秒 (273 kB/s)
正在选中未选择的软件包 debian-keyring。
(正在读取数据库 ... 系统当前共安装有 226574 个文件和目录。)
正准备解包 .../debian-keyring_2016.01.20_all.deb  ...
正在解包 debian-keyring (2016.01.20) ...
正在设置 debian-keyring (2016.01.20) ...
```

再重新下载源码包即可：

```
$ apt-get source coreutils
正在读取软件包列表... 完成
需要下载 5,755 kB 的源代码包。
获取:1 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (dsc) [2,095 B]
获取:2 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (tar) [5,725 kB]
获取:3 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main coreutils 8.25-2ubuntu3~16.04 (diff) [28.3 kB]
已下载 5,755 kB，耗时 21秒 (262 kB/s)
dpkg-source: info: extracting coreutils in coreutils-8.25
dpkg-source: info: unpacking coreutils_8.25.orig.tar.xz
dpkg-source: info: unpacking coreutils_8.25-2ubuntu3~16.04.debian.tar.xz
dpkg-source: info: applying no_ls_quoting.patch
dpkg-source: info: applying 61_whoips.patch
dpkg-source: info: applying 63_dd-appenderrors.patch
dpkg-source: info: applying 72_id_checkngroups.patch
dpkg-source: info: applying 80_fedora_sysinfo.patch
dpkg-source: info: applying 85_timer_settime.patch
dpkg-source: info: applying 99_kfbsd_fstat_patch.patch
dpkg-source: info: applying 99_hppa_longlong.patch
dpkg-source: info: applying 99_float_endian_detection.patch
dpkg-source: info: applying ppc_sha256_flags.patch
```

在当前目录中下载如下的程序包，`coreutils-8.25`即为对应的源码包，其中包含了`/bin/ls`程序的源码。

```
$ ll
总用量 5636
drwxrwxr-x  3 andy andy    4096 12月  9 15:17 ./
drwxrwxr-x  3 andy andy    4096 12月  9 14:53 ../
drwxrwxr-x 14 andy andy    4096 12月  9 15:17 coreutils-8.25/
-rw-r--r--  1 andy andy   28336 3月   3  2017 coreutils_8.25-2ubuntu3~16.04.debian.tar.xz
-rw-r--r--  1 andy andy    2095 3月   3  2017 coreutils_8.25-2ubuntu3~16.04.dsc
-rw-r--r--  1 andy andy 5725008 2月  18  2016 coreutils_8.25.orig.tar.xz
```

** 命令符号下载 **

与程序和源码源不同，符号源一般都不在系统中预配置，这需要手动进行配置。使用如下的步骤一次进行操作。

1. 使用如下命令，创建一个`/etc/apt/sources.list.d/ddebs.list`文件。

    ```
	echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list.d/ddebs.list
    ```

2. 稳定发布版本（并非开发，alpha或beta版本）要求额外的两行信息，这个信息也要添加到1中的文件之中。

	```
    echo -e "deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse\ndeb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list.d/ddebs.list
    ```

	其实就是在`/etc/apt/sources.list.d/`目录下创建`ddebs.list`文件，内容如下：

    ```
    $ cat  /etc/apt/sources.list.d/ddebs.list
    deb http://ddebs.ubuntu.com xenial main restricted universe multiverse
	deb http://ddebs.ubuntu.com xenial-updates main restricted universe multiverse
	deb http://ddebs.ubuntu.com xenial-proposed main restricted universe multiverse
    ```

3. 导入调试符号包的签名信息，在Ubuntu 18.04以及之后系统中使用`sudo apt install ubuntu-dbgsym-keyring`命令，在早些发布的系统中使用`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F2EDC64DC5AEE1F6B9C621F0C8CAB6595FDFF622`

	在Ubuntu 16.04上执行结果如下：
    ```
	$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F2EDC64DC5AEE1F6B9C621F0C8CAB6595FDFF622
	Executing: /tmp/tmp.5ENUbmxwm4/gpg.1.sh --keyserver
	keyserver.ubuntu.com
	--recv-keys
	F2EDC64DC5AEE1F6B9C621F0C8CAB6595FDFF622
	gpg: 下载密钥‘5FDFF622’，从 hkp 服务器 keyserver.ubuntu.com
	gpg: 密钥 5FDFF622：公钥“Ubuntu Debug Symbol Archive Automatic Signing Key (2016) <ubuntu-archive@lists.ubuntu.com>”已导入
	gpg: 合计被处理的数量：1
	gpg:               已导入：1  (RSA: 1)
    ```

4. 执行`sudo apt-get update`更新软件包列表。

5. 调试符号包有`-dbgsym`后缀附加，因此在安装调试符号包之前要先运行：

    ```
    apt-cache policy coretuils
    ```

    这个命令会显示当前安装软件包的版本号，用于找到要下载的符号包的版本号。

    对于要下载的`coreutils`软件包的调试符号包信息如下：

	```
    $ apt-cache policy coreutils
    coreutils:
      已安装：8.25-2ubuntu3~16.04
      候选： 8.25-2ubuntu3~16.04
      版本列表：
     *** 8.25-2ubuntu3~16.04 500
            500 http://cn.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages
            100 /var/lib/dpkg/status
         8.25-2ubuntu2 500
            500 http://cn.archive.ubuntu.com/ubuntu xenial/main amd64 Packages
    ```

6. 安装符号包：

    这里要使用到第五步中所获得的版本信息`8.25-2ubuntu3~16.04`，执行如下的命令获取对应的调试符号包。
    ```
    $ sudo apt-get install coreutils-dbgsym=8.25-2ubuntu3~16.04
    正在读取软件包列表... 完成
    正在分析软件包的依赖关系树
    正在读取状态信息... 完成
    下列【新】软件包将被安装：
      coreutils-dbgsym
    升级了 0 个软件包，新安装了 1 个软件包，要卸载 0 个软件包，有 44 个软件包未被升级。
    需要下载 2,127 kB 的归档。
    解压缩后会消耗 19.9 MB 的额外空间。
    获取:1 http://ddebs.ubuntu.com xenial-updates/main amd64 coreutils-dbgsym amd64 8.25-2ubuntu3~16.04 [2,127 kB]
    已下载 2,127 kB，耗时 29秒 (71.8 kB/s)
    正在选中未选择的软件包 coreutils-dbgsym。
    (正在读取数据库 ... 系统当前共安装有 226584 个文件和目录。)
    正准备解包 .../coreutils-dbgsym_8.25-2ubuntu3~16.04_amd64.ddeb  ...
    正在解包 coreutils-dbgsym (8.25-2ubuntu3~16.04) ...
    正在设置 coreutils-dbgsym (8.25-2ubuntu3~16.04) ...
    ```

至此就将`/bin/ls`程序的符号下载完成了，用gdb调试`/bin/ls`程序有如下的输出。进入gdb调试程序时可以发现调试器已经能够读取到调试符号了。在将之前下载的源码路径配置到gdb的源码搜索路径中，列举源码可以发现已经能够得到对应的源码了。

```
$ gdb -q /bin/ls
Reading symbols from /bin/ls...Reading symbols from /usr/lib/debug/.build-id/d0/bc0fb9b3f60f72bbad3c5a1d24c9e2a1fde775.debug...done.
done.
(gdb) b main
Breakpoint 1 at 0x402a00: file src/ls.c, line 1249.
(gdb) run
Starting program: /bin/ls
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main (argc=1, argv=0x7fffffffdd68) at src/ls.c:1249
1249	src/ls.c: 没有那个文件或目录.
(gdb) bt
#0  main (argc=1, argv=0x7fffffffdd68) at src/ls.c:1249
(gdb) directory /home/XXXX/GDB/coreutils/coreutils-8.25
Source directories searched: /home/XXXX/GDB/coreutils/coreutils-8.25:$cdir:$cwd
(gdb) list
1280	#if ! SA_NOCLDSTOP
1281	  bool caught_sig[nsigs];
1282	#endif
1283
1284	  initialize_main (&argc, &argv);
1285	  set_program_name (argv[0]);
1286	  setlocale (LC_ALL, "");
1287	  bindtextdomain (PACKAGE, LOCALEDIR);
1288	  textdomain (PACKAGE);
```

By Andy@2018-12-04 20:46:23