
#Find命令#

https://www.cnblogs.com/davidwang456/p/3753707.html



find命令的基本形式为`$find -option [path...] [expresion]`。默认的`path`为当前目录，默认的表达式为`-print`。

一种常用的形式如下所示。

```
find path -option [-print] [-exec/-ok command] {} /;
```

命令中path为要查找的目录；`-option`是文件的过滤条件；`-exec`是对每个查找到的文件要执行的操作；`{}`表示查找到的文件全路径；`\;`表示shell语句的分隔符。

表达式可以由运算符，设置项，测试项和动作项等组成。其中选项分为三种，设置项，测试选项和动作项；而运算符包括`()`，`!/-not`，`-a/-and`，`-o/-or`，`,`等，它们用于连接表达式，形成复合过滤条件，其表达意义也容易理解，不再一一表述。

下面逐类说明一下选项的含义。

###设置项###

设置项分为两种，一种是位置项，还有一种普通项。位置项如果用于表达式运算时其值总为`true`。

位置项包括如下三个:

**-daystart**:

**-follow**:

**-regextype**:

普通设置项则包括如下几个，它们在表达式中的值也总是`true`。

**--version**: 显示find命令的版本信息。

**--help**: 显示find命令的帮助信息。

**-depth**: 查找进入子目录之前先行查找完本目录。

**-maxdepth LEVELS**: 查找子目录的最大深度。

**-mindepth LEVELS**: 查找子目录的最小深度

**-mount**: 查找时部跨越文件系统的mount点

**-noleaf**:

**-xdev**:

**-ignore_readdir_race**:

**-noignore_readdir_race**:

###测试项###

在测试项中会使用到数值，这里用N表示，N可以是`+N`或`-N`。

**-aX**: 限定文件的访问时间（其中X代表`min`，`newer`，`time`等），其中包括`-amin N`，`-anewer FILE`，`-atime N`，

`-amin N`表示访问时间，单位为分钟，N为负值则表示最近N分钟内访问过，如果为`+N`则表示30分钟以前访问过。

`-anewer FILE`表示比文件FILE访问时间更近的文件。

`-atime`表示访问的时间，以天为单位`+N`表示访问时间超过N天，`-N`表示访问时间在N天之内。

**-cX**: 限定文件的创建时间。

该类命令下包含了`-cmin N`，`-cnewer FILE`，`-ctime N`三类，分别对应于上面文件访问时间。

**-mX**: 限定文件的修改时间。

该类命令包含`-mmin N`，`-mtime N`，`-mnewer FILE`三个。

**-empty**: 查找大小为0的文件或空目录

**-false**:

**-fstype TYPE**: #查位于某一类型文件系统中的文件，这些文件系统类型通常可 在/etc/fstab中找到

**-gid N**: 列出查找目录内组id为N的文件或目录

**-group NAME**: 查找在目录中属于组NAME的文件

**-ilname PATTERN**:
**-iname PATTERN**:
**-inum N**:
**-iwholename PATTERN**:
**-iregex PATTERN**：

**-links N**：查硬连接数为N的文件或目录，`+N`表示大于N，`-N`表示小于N的文件或目录。

命令`find   /home   -links   +2` 查硬连接数大于2的文件或目录

**-lname PATTERN**: 链接名字满足PATTERN的文件或目录。

**-name PATTERN**: 查找名字满足PATTERN的文件或目录。

**-newer FILE**: 查找更新时间比FILE更新的文件。

**-nouser**: 查找在系统中属于作废用户的文件
**-nogroup**: 查找文件或目录的组不属于本地组的文件或目录。

**-path PATTERN**: 查找文件路径满足特定模式字符串的文件。
**-perm [-/]MODE**:  满足特定权限的文件或目录，MODE前不带符号表示精确匹配。

`-perm -MODE`表示最少具有MODE权限的文件或目录，`-perm /MODE`则表示具有任意一组权限的文件或目录。

**-regex PATTERN**: 

**-readable**: 
**-writable**: 
**-executable**: 

**-wholename PATTERN**:

**-size N[bcwkMG]**:  
**-true**:  

**-type [bcdpflsD]**:

**-uid N**: 

**-used N**: 

**-user NAME**:

**-xtype [bcdpfls]**:

**-context CONTEXT**:

###动作项###
actions:
`-delete` `-print0` `-printf FORMAT` `-fprintf FILE FORMAT` `-print`
      `-fprint0 FILE` `-fprint FILE` `-ls` `-fls FILE` `-prune` `-quit`
      `-exec COMMAND ;` `-exec COMMAND {} + -ok COMMAND ;`
      `-execdir COMMAND ;` `-execdir COMMAND {} + -okdir COMMAND ;`


**-print**：默认选项，将查找到的文件输出到标准输出。

**-exec command {} /;**: 将查找到的文件执行command操作，`{}`和 `/;`之间有空格，`{}`表示找到的文件，`/;`表示结束命令。`-ok`和`-exec`命令相同，区别在于执行前要询问用户。

比如命令`find . -name "*jpg" -type f -exec rm -rf {}`删除查找到的符合`*jpg`的所有文件。

**-name**: 匹配文件／路径名，区分大小写。
**-iname**：匹配文件／路径名字，不区分大小写。

例如`find ~ -iname '*jpg'`查找home目录中的所有jpg结尾的文件，不区分大小写。

对于要满足多个匹配条件的情况，可以使用“或”来组合，即选项`-o`。比如如下例子，满足条件之一即可，即超着`jpg／jpeg`文件。

```
find ~ \( -iname '*jpg' -o -iname '*jpeg' \) -type f
```

**-user username**: 按照文件属主进行过滤。

**-group groupname**：按照文件组进行过滤。`-nogroup`和`-nouser`用于查找无有效属组和无有效属主的文件。

**-type d**： 表示查找文件类型。

|符号| 含义 |
|---|------|
|f | 普通文件 |
|d | 目录文件 |
|l | 符号链接文件 |
|b | 块设备文件 |
|c | 字符设备文件 |
|p | 管道文件 |
|s | 套接字文件|


**-mtime -n/+n**：按照修改时间进行过滤，`+n`表示n天以前，`-n`表示n天以内。其他还可以使用`-ctime`，`-mtime`，`-atime`，分别表示根据文件更改时间，修改时间或访问时间来进行过滤。如果要使用更细粒度可以使用`-cmin`，`-mmin`，`-amin`来限定分钟数。可用`-mtime -7`过滤修改时间在一周之内的文件。

**-newer f1 !f2**： 更改时间比f1新，但是比f2旧的文件。`-anewer tmp.txt`即存取时间比文件tmp.txt更近的文件或目录。

**-size**：可以用该选项过滤文件大小。比如`find /var/log -size +1g`用于过滤log目录下大小大于1G的文件。

**-owner**：用于过滤属于特定用户的文件，比如命令`find /data -owner bcotton`用于寻找属于bcotton的文件或目录。

**-perm**：用于过滤具有特定权限的文件，比如`find ~ -perm -o=r`搜索home目录下其他用户具有读权限的文件。




**-ftype**: 查找位于某一类型文件系统中的文件，通常可在`/etc/fstab`中查找。

**-follow**：遇到符号链接文件时，就跟踪链接所指向的文件进行过滤。

**-prune**: 忽略某个目录不查找。

**-maxdept/-mindepth**: 搜索时目录的最大深度／最小深度。

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



**参考文章**:

1. https://mp.weixin.qq.com/s/R2qNDRekP-AIoMzY7Iaoqg
2. http://www.cnblogs.com/lsdb/p/8342379.html
3. https://blog.csdn.net/windone0109/article/details/2817792
4. http://fatmouse.xyz/2016/05/10/2016-05-10-find-grep-xargs-and-pipe/