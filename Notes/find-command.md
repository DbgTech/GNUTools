
#Find命令#

基础语法：

```
find path -option [-print] [-exec -ok command] {} /;
```

命令中path为要查找的目录；`-option`是文件的过滤条件；`-exec`是对每个查找到的文件要执行的操作；`{}`表示查找到的文件全路径；`\;`表示shell语句的分隔符。

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

**-type d**： 表示查找文件类型，d表示目录，f表示文件，b表示块设备，c表示字符设备，p表示管道文件，l表示符号链接。

**-mtime -n/+n**：按照修改时间进行过滤，`+n`表示n天以前，`-n`表示n天以内。其他还可以使用`-ctime`，`-mtime`，`-atime`，分别表示根据文件更改时间，修改时间或访问时间来进行过滤。如果要使用更细粒度可以使用`-cmin`，`-mmin`，`-amin`来限定分钟数。可用`-mtime -7`过滤修改时间在一周之内的文件。

**-newer f1 !f2**： 更改时间比f1新，但是比f2旧的文件。`-anewer tmp.txt`即存取时间比文件tmp.txt更近的文件或目录。

**-size**：可以用该选项过滤文件大小。比如`find /var/log -size +1g`用于过滤log目录下大小大于1G的文件。

**-owner**：用于过滤属于特定用户的文件，比如命令`find /data -owner bcotton`用于寻找属于bcotton的文件或目录。

**-perm**：用于过滤具有特定权限的文件，比如`find ~ -perm -o=r`搜索home目录下其他用户具有读权限的文件。

**-depth**: 查找进入子目录之前先行查找完本目录。

**-mount**: 查找时部跨越文件系统的mount点

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

find命令还可以和xargs，grep，awk，sed等程序进一步处理。

**参考文章**:

1. https://mp.weixin.qq.com/s/R2qNDRekP-AIoMzY7Iaoqg
2. http://www.cnblogs.com/lsdb/p/8342379.html
3. https://blog.csdn.net/windone0109/article/details/2817792