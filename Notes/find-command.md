#Find命令#

find命令的基本形式为`$find -option [path...] [expresion]`。默认的`path`为当前目录，默认的表达式为`-print`。

一种常用的形式如下所示。

```
find path -option [-print] [-exec/-ok command] {} /;
```

命令中path为要查找的目录；`-option`是文件的过滤条件；`-exec`是对每个查找到的文件要执行的操作；`{}`表示查找到的文件全路径；`\;`表示shell语句的分隔符。

表达式可以由运算符，设置项，测试项和动作项等组成。其中选项分为三种，设置项，测试选项和动作项；而运算符包括`()`，`!/-not`，`-a/-and`，`-o/-or`，`,`等，它们用于连接表达式，形成复合过滤条件，其表达意义也容易理解，不再一一表述。

对于要满足多个匹配条件的情况，可以使用“或”来组合，即选项`-o`。如下例子中满足条件之一即可，即查找`jpg／jpeg`文件。

```
find ~ \( -iname '*jpg' -o -iname '*jpeg' \) -type f
```

如果多个选项之间不指定运算符，那么选项之间默认使用`-and`，即并列的关系。

下面逐类说明一下选项的含义。

###设置项###

设置项分为两种，一种是位置项，还有一种普通项。位置项如果用于表达式运算时其值总为`true`。

位置项包括如下三个:

**-daystart**: 从本日开始计算时间

**-follow**: 排除符号连接

**-regextype**:

普通设置项则包括如下几个，它们在表达式中的值也总是`true`。

**--version**: 显示find命令的版本信息。

**--help**: 显示find命令的帮助信息。

**-depth**: 查找进入子目录之前先行查找完本目录。

**-maxdepth LEVELS**: 查找子目录的最大深度。

**-mindepth LEVELS**: 查找子目录的最小深度。

**-mount**: 查找时部跨越文件系统的mount点。

**-noleaf**:

**-xdev**: 将范围局限在先行的文件系统中。

**-ignore_readdir_race**:

**-noignore_readdir_race**:

###测试项###

在测试项中会使用到数值，这里用N表示，N可以是`+N`或`-N`。`+N`表示天数则是大于N天，表示文件大小则是大于N的文件；`-N`表示天数则是N天之内，表示文件大小则是小于N的文件。

**-aX**: 限定文件的访问时间（其中X代表`min`，`newer`，`time`等），其中包括`-amin N`，`-anewer FILE`，`-atime N`，

`-amin N`表示访问时间，单位为分钟，N为负值则表示最近N分钟内访问过，如果为`+N`则表示30分钟以前访问过。

`-anewer FILE`表示比文件FILE访问时间更近的文件。

`-atime`表示访问的时间，以天为单位`+N`表示访问时间超过N天，`-N`表示访问时间在N天之内。

**-cX**: 限定文件的创建时间。

该类命令下包含了`-cmin N`，`-cnewer FILE`，`-ctime N`三类，分别对应于上面文件访问时间。

**-mX**: 限定文件的修改时间。

该类命令包含`-mmin N`，`-mtime N`，`-mnewer FILE`三个。

**-empty**: 查找大小为0的文件或空目录。

**-false**: 将find指令的回传值皆设为False。

**-fstype TYPE**: 查位于某一类型文件系统中的文件，这些文件系统类型通常可 在`/etc/fstab`中找到。

**-gid N**: 列出查找目录内组id为N的文件或目录。

**-group NAME**: 查找在目录中属于组NAME的文件。

选项`-group groupname`按照文件组进行过滤。`-nogroup`和`-nouser`用于查找无有效属组和无有效属主的文件。

**-ilname PATTERN**:

**-iname PATTERN**:

**-inum N**:

**-iwholename PATTERN**:

**-iregex PATTERN**：

**-links N**：查找硬连接数为N的文件或目录，`+N`表示大于N，`-N`表示硬链接数小于N的文件或目录。

命令`find /home -links +2` 查硬连接数大于2的文件或目录。

**-lname PATTERN**: 链接名字满足PATTERN的文件或目录。

**-name PATTERN**: 查找名字满足PATTERN的文件或目录。 **-iname PATTERN**类似前面的选项，但是对于文件或目录名字忽略大小写。

例如`find ~ -iname '*jpg'`查找home目录中的所有jpg结尾的文件，不区分大小写。

**-newer FILE**: 查找更新时间比FILE更新的文件。

`-newer f1 !f2` 更改时间比f1新，但是比f2旧的文件。`-anewer tmp.txt`即存取时间比文件tmp.txt更近的文件或目录。

**-nouser**: 查找在系统中属于作废用户的文件.

**-nogroup**: 查找文件或目录的组不属于本地组的文件或目录。

**-path PATTERN**: 查找文件路径满足特定模式字符串的文件。

**-perm [-/]MODE**:  满足特定权限的文件或目录，MODE前不带符号表示精确匹配。

选项`-perm -MODE`表示最少具有MODE权限的文件或目录；而选项`-perm /MODE`则表示具有任意一组权限的文件或目录。

例如`find ~ -perm -o=r`搜索home目录下其他用户具有读权限的文件。

**-regex PATTERN**: 表示符合`PATTERN`正则字符串的文件。

**-readable**: 查找可读的文件或目录。

**-writable**: 查找可写的文件或目录。

**-executable**: 表示查找具有执行权限的文件或目录。

**-wholename PATTERN**: 

**-size N[bcwkMG]**: 表示限制查找文件的大小，N表示大小，同样有`+N`和`-N`的区别，就是分别对应大于和小于指定大小。后面的单位为

`find . -size +4090c -print`表示查找当前目录大小大于4090字节的文件。

|符号| 含义 |
|---|------|
|b | 块（512字节）|
|c | Char，字符|
|w | Word，两个字|
|k | KByte，千字节|
|M | MByte，兆字节|
|G | GByte，吉字节|

```
find . -type f -size +10k	// 搜索当前目录大于10Kb的文件。

find /var/log -size +1g		// 用于过滤log目录下大小大于1G的文件。
```

**-true**: 将find指令的回传值皆设为True

**-type [bcdpflsD]**: 指定查找文件类型

选项`-type d` 表示查找文件类型为目录。

|符号| 含义 |
|---|------|
|f | 普通文件 |
|d | 目录文件 |
|l | 符号链接文件 |
|b | 块设备文件 |
|c | 字符设备文件 |
|p | 管道文件 |
|s | 套接字文件|

**-uid N**: 查找UID为N的文件或目录

**-used N**: 查找文件或目录被更改之后在指定时间曾被存取过的文件或目录，单位以天计算

**-user NAME**: 查找符和指定的拥有者名称的文件或目录

**-xtype [bcdpfls]**: 此参数的效果和指定“-type”参数类似，差别在于它针对符号连接检查

**-context CONTEXT**:

**-owner**：用于过滤属于特定用户的文件，比如命令`find /data -owner bcotton`用于寻找属于bcotton的文件或目录。

**-follow**：遇到符号链接文件时，就跟踪链接所指向的文件进行过滤。

###动作项###

**-delete**: 将找到的文件删除

**-print**：默认选项，将查找到的文件输出到标准输出。

**-print0**: 查找到的文件或目录不以换行符分隔，而是以NULL字符分隔。它其实用于xargs命令的`-0`选项，即xargs使用NULL作为分隔符而不是空白字符（空格，TAB，换行符）

**-printf FORMAT**: 将文件或目录名称列出到标准输出，格式可以自行指定（即FORMAT格式）.

**-fprint FILE**: 类似`-print`选项，只是将结果输出到文件。

**-fprint0 FILE**: 类似`-print0`，只是将结果输出到文件FILE中。

**-fprintf FILE FORMAT**: 类似`-printf FORMAT`，只是将信息输出到文件FILE中。

**-ls**: 列举查找到目录的内容信息，按照`ls`命令格式。

**-fls FILE**: 将列出查找到的目录的信息，结果写到FILE文件中.

**-prune**: 只查找当前目录，忽略子目录。

**-quit**:

**-exec COMMAND ;**: 选项`-exec command {} /;` 将查找到的文件执行command操作，`{}`和 `/;`之间有空格，`{}`表示找到的文件，`/;`表示结束命令。`-ok`和`-exec`命令相同，区别在于执行前要询问用户。

比如命令`find . -name "*jpg" -type f -exec rm -rf {}`删除查找到的符合`*jpg`的所有文件。

**-exec COMMAND {} + -ok COMMAND ;**:

**-execdir COMMAND ;**:

**-execdir COMMAND {} + -okdir COMMAND ;**:

###常用例子###

```
find ~ -name "*.txt" -print        # 查找HOME目录下.txt文件并显示

find . -name "[A-Z]*" -print       # 查找当前文件夹下以大写字母开头文件

find . -name "[a-z][a-z][0-9][0-9].txt" -print
# 查找当前目录下，以两个小写字母和两个数字开头的txt文件

find . -size 100c -print      # 查找文件长度大于100字符的文件

find . -name "passwd*" -exec grep "cnscn" {} /;  # 是否存在cnscn用户

find . -name "yao*" | xargs chmod o-w  # 将当前目录yao开头的文件other的写权限去掉。

find /mnt -name tom.txt -ftype vfat # 在mnt下查找tom.txt，且文件系统类型不是vfat

find /home -mtime -2  # home目录下查找最近两天内改动过的文件

find /home -name temp.txt -maxdept 4 # home目录内temp.txt文件，查找深度最多为3层。第四层为限制。
```

find命令还可以和xargs，grep，awk，sed等程序结合做进一步处理。

###find中用到概念###

`xargs`命令和find命令可以说出于同门，都源自于`findutils`。下面连带着记录一下`xargs`命令的用法。这里在说它的用法之前，先说一下命令的参数和标准输入的区别，以及管道在命令中的作用。

**参数、标准输入和管道**

这三者是在命令中会同时出现的概念，比如`ls -la | grep abc`，这条命令中就同时包含了这三者。其中`-la`，`abc`就是命令参数；`|`当然是管道了；而在管道中重定向的就是`ls`命令的标准输出，管道将它重定向为`grep`命令的标准输入。

标准输入可认为是数据流，通常标准输入的流数据来源于终端输入（当然了，为了方便就出现了重定向，那么来源就可以是文件，其他命令的输出等）。Linux命令中，有些命令是可以接受标准输入的，而有一些命令是无法接受标准输入的。比如`ls`命令不能接受标准输入内容，只能使用参数；而`cat`等命令既可以使用标准输入，也可以使用参数。

如何区分一个命令是否可以接受标准输入呢？在Shell中执行命令，单独执行不带任何参数，接受标准输入的命令会在终端等待接受你的输入。例如`cat`命令，执行完之后如下会等待输入字符，输入完一些字符后，按`Ctrl-C`发送终止符，这时`cat`命令开始执行，将结果输出到标准输出上（屏幕）。

```
$ cat
abc
abc
```

管道作用其实就是重定向，将前面命令的标准输出作为后面命令的标准输入。这里管道送给后面命令的内容是标准输入，也就是不接收标准输入的命令无法利用管道重定来的内容，比如`find . -type d | ls`命令执行结果就是列举当前目录内容，并没有将找到的目录内容列举出来。

为了将管道重定向到后面命令的标准输入转换为参数，`xargs`命令的作用就凸现出来了。简单来说`xargs`命令就是将标准输入转换成各种格式化的参数，`[command 1] | xargs [command 2]`就是将命令1的标准输出结果通过管道变为`xargs`的标准输入，`xargs`再将标准输入变成参数传给命令2。这样管道后面就可以通过xargs转换使用不接收标准输入的命令了，比如``find . -type d | xargs ls``

###xargs命令###

`xargs`命令可以将输入内容（通过命令行管道传递）转成后续命令的参数，一般用于：

* 命令组合：尤其用于一些不支持管道输入的命令，比如 `ls`
* 避免参数过长：xargs通过`-nx`来将参数分组，避免参数过长

基本用法如下：

```
Usage: xargs [OPTION]... COMMAND [INITIAL-ARGS]...
Run COMMAND with arguments INITIAL-ARGS and more arguments read from input.
```

命令选项含义：

| 选项 | 完整写法 |  选项含义  |
|-----|---------|-----------|
|-0  |  --null | 以null作为分隔符，而不是空白符（空格，TAB，换行），同时不处理引号，反斜线和结束符EOF|
|-a | --arg-file=FILE | 从文件FILE中读取数据，而不是标准输入 |
|-d | --delimiter=CHARACTER| 输入流中以字符CHARACTER分隔，而不是使用空白符，同时不处理引号，反斜线和结束符EOF |
|-E END| 无 | 设置逻辑结束符EOF串，如果END出现在一行，则换行的输入内容被忽略。如果指定-0或-d，该选项则被忽略|
|-e | --eof[=END] | 如果END指定数据，则等价于`-E END`，否则没有文件结束字符串 |
|-I R | 无 | 和`--replace=R`选项含义相同 |
|-i  | --replace[=R] | 用从标准输入读取的数据替代INITIAL-ARGS中的R，如果没有指定R，则默认{} |
|-L  | --max-lines=MAX-LINES | 每个命令行使用最多MAX-LINES行 |
|-l[MAX-LINES] | 无 | 和-L类似 |
|-n  | --max-args=MAX-ARGS | 每个命令行最多使用MAX-ARGS个参数 |
|-P  | --max-procs=MAX-PROCS| 一次最多执行 MAX-PROCS个进程  |
|-p  | --interactive | 在执行命令前提示 |
|无  |--process-slot-var=VAR | 设置子进程中的环境变量VAR  |
|-r  | --no-run-if-empty  | 如果没有参数，就不执行命令。如果不指定这个参数，则COMMAND最少执行一次 |
|-s  | --max-chars=MAX-CHARS | 限制命令行长度为MAX-CHARS |
|无  | --show-limits |  显示命令行长度限制 |
|-t  | --verbose   | 在执行命令前，先打印执行的命令|
|-x  | --exit  | 如果命令长度超过限制，则退出 参考`-s`选项|
|无  | --help  | 显示帮助信息  |
|无  | --version | 输出版本信息并退出 |

使用`-i`参数默认的前面输出用`{}`代替，`-I`参数可以指定其他代替字符。其实就是管道导过来的每一条数据用`{}`符号默认代替，即在后面命令中使用`{}`就代表一条结果。

```
find . -name "*.log" | xargs -p -i mv {} {}.a

find . -name "file" | xargs -I [] cp [] [].a
```

使用`-i`参数默认的前面输出用`{}`代替，`-I`参数可以指定其他代替字符，如例子中的`[]`。


如下为基础的例子。

```
$ touch a.js b.js c.js		// 例如 生成三个文件

$ ls *.js | xargs ls -la	// 将三个文件 详细信息打印出来，ls的输出通过管道送给 xargs
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 a.js
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 b.js
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 c.js


$ ls *.js | xargs -t ls -la	// -t 选项在执行命令前打印命令
ls -la a.js b.js c.js
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 a.js
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 b.js
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 c.js
```

使用参数替换：

如下例子，将所有的`.js`结尾文件都加上`.c`后缀。`-I '{}'`表示将后面命令中的`{}`替换为前面解析出来的参数。

```
$ls *.js | xargs -t -I '{}' mv {} {}.c
mv a.js a.js.c
mv b.js b.js.c
mv c.js c.js.c
```

参数分组：

```
$ ls *.c | xargs -t -n2 ls -la
ls -la a.js.c b.js.c
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 a.js.c
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 b.js.c
ls -la c.js.c d.js.c
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:18 c.js.c
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:24 d.js.c
```

特殊文件名称:

例如对于`hello 01.h`和`hello 02.h`两个文件执行如下的命令，会发现出现了执行错误。这个时候`-print0`就派上用场了。

```
$ find . -name '*.h' | xargs -t ls -la
ls -la ./hello 01.h ./hello 02.h
ls: cannot access './hello': No such file or directory
ls: cannot access '01.h': No such file or directory
ls: cannot access './hello': No such file or directory
ls: cannot access '02.h': No such file or directory

$ find . -name '*.h' -print0 | xargs -t -0 ls -la
ls -la ./hello 01.h ./hello 02.h
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:26 './hello 01.h'
-rw-r--r-- 1 Administrator 197121 0 六月 22 10:26 './hello 02.h'
```

xargs是这样解决这个问题的。

* -print0：告诉find命令，在输出文件名之后，跟上NULL字符，而不是换行符；
* -0：告诉xargs，以NULL作为参数分隔符；

这样xargs就不会将两个文件名中间的空格作为分隔符，而将两个文件名指派为四个文件名了。

**参考文章**:

1. https://mp.weixin.qq.com/s/R2qNDRekP-AIoMzY7Iaoqg
2. http://www.cnblogs.com/lsdb/p/8342379.html
3. https://blog.csdn.net/windone0109/article/details/2817792
4. http://fatmouse.xyz/2016/05/10/2016-05-10-find-grep-xargs-and-pipe/