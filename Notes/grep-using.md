#grep命令#

http://man.linuxde.net/grep

https://linuxstory.org/grep-regular-expressions/

http://www.cnblogs.com/end/archive/2012/02/21/2360965.html

http://blog.51cto.com/haowen/2068456

https://blog.csdn.net/yanlaifan/article/details/52766109

http://man7.org/linux/man-pages/man1/grep.1.html



`grep`命令用于在文件或标准输入中搜索`PATTERN`，`PATTERN`是一个基本正则表达式（BRE）。`grep`命令的一般形式如下所示：

```
grep [OPTION]... PATTERN [FILE]...
```

比如`grep -i 'hello world' menu.h main.c`命令从两个文件中搜索`hello world`字符串。

正则选项和解释如下表：

| 选项 | 完整形式|       含义       |
|-----|--------|-----------------|
|-E   | --extended-regexp | PATTERN是扩展正则表达式(ERE)|
|-F   | --fixed-strings | PATTERN是一组新行分隔符字符串 |
|-G   | --basic-regexp | PATTERN是一个基本正则表达式(BRE)|
|-P   | --perl-regexp | PATTERN是一个Perl正则表达式 |
|-e   | --regexp=PATTERN | 将PATTERN用于匹配 |
|-f   | --file=FILE | 从文件FILE中获取PATTERN表达式 |
|-i   | --ignore-case | 忽略大小写区分     |
|-w   | --word-regexp | 用PATTERN强制匹配整个词 |
|-x   | --line-regexp | 用PATTERN匹配整行 |
|-z   | --null-data | 以0字节结尾的数据行，非新行 |

选项中的杂项：

| 选项 | 完整形式|       含义       |
|-----|--------|------------------|
|-s   | --no-messages | 不输出错误信息 |
|-v   | --invert-match| 选择没有匹配的行，反向选择|
|-V   | --version  | 显示版本信息，退出 |
|无    | --help    | 显示帮助信息，退出 |

输出控制选项:

| 选项 | 完整形式|       含义       |
|-----|--------|------------------|
|-m   | --max-count=NUM | 在NUM次匹配后，则停止|
|-b   | --byte-offset | 打印输出行的字节偏移 |
|-n   | --line-number |  打印输出行的行号 |
|     | --line-buffered| flush输出每一行 |
|-H   | --with-filename| 每一个匹配项输出文件名|
|-h   | --no-filename | 将文件名作为输出的前缀 |
|     | --label=LABEL | 使用LABEL作为标准输入文件名前缀|
|-o   | --only-matching | 只输出匹配PATTERN的部分内容 |
|-q   | --quiet, --silent| 不输出所有正常的输出 |
|     | --binary-files=TYPE | 假设二进制文件是TYPE，Type可以是'binary', 'text', 或'without-match'|
|-a   | --text  | 等价于 --binary-files=text |
|-I   |         | 等价于 --binary-files=without-match |
|-d   | --directories=ACTION| 如何处理目录，ACTION可以是'read', 'recurse', 或'skip'|
|-D   | --devices=ACTION| 如何处理设备，FIFOs和sockets，ACTION是'read'或'skip' |
|-r   | --recursive | 像--directories=recurse |
|-R   | --dereference-recursive | 同样地，但是下面都是 symlinks|
|     | --include=FILE_PATTERN | 只搜索匹配FILE_PATTERN的文件 |
|     | --exclude=FILE_PATTERN | 跳过匹配 FILE_PATTERN的文件和目录 |
|     | --exclude-from=FILE | 跳过任何匹配FILE的规则的文件 |
|     | --exclude-dir=PATTERN | 匹配PATTERN的目录会被跳过 |
|-L   | --files-without-match | 没有匹配项的文件只打印文件名字 |
|-l   | --files-with-matches | 包含匹配条目的文件只打印文件名 |
|-c   | --count   | 每一个文件只打印匹配行的行数 |
|-T   | --initial-tab | make tabs line up (if needed) |
|-Z   | --null  | 在FILE的名字后打印0字节数据 |

上下文控制：

| 选项 | 完整形式|       含义       |
|-----|--------|------------------|
|-B   | --before-context=NUM | 打印上下文中开始的NUM行 |
|-A   | --after-context=NUM | 打印上下文中尾部的NUM行 |
|-C   | --context=NUM | 打印输出上下文的NUM行 |
|     | -NUM | 和--context=NUM选项相同 |
|     | --color[=WHEN],--colour[=WHEN] | 使用markers强调匹配字符串，WHEN可以取'always', 'never', 或'auto' |
|-U   | --binary  | 在EOL（MSDOS/Windows）中不要删除CR字符 |
|-u   | --unix-byte-offsets | 报告偏移，如果CRs不存在时 |


`egrep`和`grep -E`含义相同，`fgrep`和`grep -F`含义相同，但是两个命令已经不再使用。

当FILE是`-`时，则从标准输入中读取数据。如果没有FILE参数，如果命令行选项`-r`指定了，则读取当前目录；否则和`-`相同。
