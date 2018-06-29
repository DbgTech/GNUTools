#grep命令#

`grep`命令用于在文件或标准输入中搜索`PATTERN`，`PATTERN`是一个基本正则表达式（BRE）。`grep`命令的一般形式如下所示：

```
grep [OPTION]... PATTERN [FILE]...
```

比如`grep -i 'hello world' menu.h main.c`命令从两个文件中搜索`hello world`字符串。默认情况下grep会打印匹配的整行内容。

正则选项和解释如下表：

| 选项 | 完整形式|       含义       |
|-----|--------|-----------------|
|-E   | --extended-regexp | PATTERN是扩展正则表达式(ERE)|
|-F   | --fixed-strings | PATTERN是一组新行分隔符字符串 |
|-G   | --basic-regexp | PATTERN是一个基本正则表达式(BRE)|
|-P   | --perl-regexp | PATTERN是一个Perl正则表达式 |
|-e   | --regexp=PATTERN | 将PATTERN用于匹配，一个命令中可以包含多个-e选项，表示同搜索多个串 |
|-f   | --file=FILE | 从文件FILE中获取PATTERN表达式 |
|-i   | --ignore-case | 搜索时忽略大小写区分     |
|-w   | --word-regexp | 用PATTERN强制匹配整个词，这个词构成了整行 |
|-x   | --line-regexp | 用PATTERN匹配整行，与-w不兼容 |
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
|-n   | --line-number |  在输出内容前，打印出匹配行的行号 |
|     | --line-buffered| flush输出每一行 |
|-H   | --with-filename| 每一个匹配项输出文件名，搜索多个文件时这是默认选项|
|-h   | --no-filename | 不将文件名作为输出行 前缀，这个是搜索一个文件时的默认选项 |
|     | --label=LABEL | 使用LABEL作为标准输入文件名前缀|
|-o   | --only-matching | 只输出匹配PATTERN的部分内容 |
|-q   | --quiet, --silent| 不输出所有正常的输出 |
|     | --binary-files=TYPE | 假设二进制文件是TYPE，Type可以是'binary', 'text', 或'without-match'|
|-a   | --text  | 等价于 --binary-files=text ，将二进制文件当作文本文件处理|
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
|-c   | --count   | 每一个文件只打印匹配行的行数，只显示匹配字符串的行数 |
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

`egrep`和`grep -E`含义相同，`fgrep`和`grep -F`含义相同，但是两个命令已经不再使用。当`FILE`是`-`时，则从标准输入中读取数据。如果没有`FILE`参数，如果命令行选项`-r`指定了，则读取当前目录；否则和`-`相同。

例如如下的一些例子：

```
$ grep "Navigat" -c Test.cpp	// -c 只打印文件内符合条件的行数
276

$ echo gun is not unix | grep -b -o "not"	// -b和-o一般同时使用，-b为匹配字符串距离搜索内容起始处的偏移字节，-o则只输出匹配内容
7:not

$ grep "Navigat" -l Test.cpp Test.h		// -l 只打印包含匹配条目的文件名
Test.cpp
Test.h

$ grep "Navigate" . -r -n			// 递归搜索当前目录中所有文件， -r表示递归，-n表示输出行号 . 表示当前目录

$ echo "hello world" | grep -i -o "HELLO"	// -i 表示不区分大小写
hello

$ echo aaa bbb ccc ddd eee | grep -f patfile  -o	// -f 表示使用文件patfile作为匹配字符串输入
aaa
bbb

$grep "Navigate" . -r --include=*.h	// 从当前目录递归地搜索 *.h 文件中包含Navigate字符串的行
									// --exclude=*.{h,cpp}表示排除*.h 和 *.cpp
                                   	// --exclude-from=filelist 	从文件列表中排除要搜索的文件

$grep "aaa" file* -lZ | xargs -0 rm	// -l 表示列出文件名，-Z表示列出数据以0值作为结尾，
									// xargs -0表示处理以0值结尾字符串 rm 则表示删除

$grep -q "test" filename			// -q 表示静默，不输出信息，如果成功返回0，失败则返回非0，一般用于条件测试

$ seq 10 | grep "5" -A 3		// -A 3选项表示输出匹配行以及其后的3行，-B表示输出之前的 3行，-C则是前后三行
5
6
7
8
```

###正则表达式###

正则表达式是一个描述了一组字符串的模式。正则表达式以近似于算术表达式的形式构成，通过使用各种操作符组成小的表达式。

`grep`可以解析三种不同的正则表达式语法：`basic`（BRE），`extended`（ERE）和`perl`（PCRE）。在GNU的`grep`中，在基础语法与扩展语法之间没有功能上的不同。其他的实现中，基本正则表达式没有那么强大。下面的描述适用于扩展正则表达式，它与基础正则表达式的不同在后面总结。兼容Perl的正则表达式实现了一些额外的功能，在`pcresyntax(3)`和`pcrepattern(3)`文档中进行说明，但是这个功能仅在PCRE可用的系统中才可以正常工作。

基本的构建块是正则表达式，它们仅仅匹配一个字符。大部分字符通常都是匹配它们自己的正则表达式，包括所有字母和数字。任何有特殊含义的元字符要在字符串中当作字符表达时都需要斜线进行转义。句号`.`用来匹配任意单个字符，它是否匹配编码错误并未进行明确。

**字符类别和括号表达式**

括号表达式是一个由`[]`括起来的字符列表。它会匹配列表中的任意一个单个字符。如果列表的第一个字符是插入字符`^`，那么它会匹配任意一个不在列表中的字符。它是否匹配编码错误并没有明确指出，正则表达式`[0123456789]`匹配任意一个单个数字。

在一个括号表达式内，用连字符链接的两个字符构成了区间表达式，它会匹配两个字符之间任意一个字符，包含边界。例如，在默认的`C`语言区域中，`[a-d]`等价于`[abcd]`。许多场景下按照字典顺序区分字符，在这种场景下`[a-d]`不等价于`[abcd]`；例如，它可能等价于`aBbCcDd`。为了得到传统括号表达式的含义，可以通过设置`LC_ALL`环境变量设置为`C`来设置使用C语言区域。

最后，括号表达式内部的特定命名类的字符都是预定义的，如下所述。它们的名字都是一目了然的，包括`[:alnum:]`，`[:alpha:]`，`[:cntrl:]`，`[:digit:]`，`[:graph:]`，`[:lower:]`，`[:print:]`，`[:punct:]`，`[:space:]`，`[:upper:]`，和`[:xdigit:]`。例如，`[:alnum:]`在当前环境下的含义是数字和字母的字符类别。在C语言环境和`ASCII`字符集编码中，这个表达式和`[0-9A-Za-z]`表达的意思相同。（注意在这些类名中括号是符号名字的一部分，必须被包含在括号表达式内部）。在括号表达式内部的大部分元字符会失去它们所表达的特殊含义，要包含一个文字含义字符`]`，则将它放到字符列表的第一个位置。同样的，要包含`^`，则将它放到除了第一个字符的任何位置，最后，要包含一个连字符`-`，将它放到最后。

**锚点**

脱字字符`^`和美元字符`$`都是元字符，它们分别用于匹配一行的开始或结尾的空字符。


**反斜线和特殊表达式**

符号`\<`和`\>`分别匹配一个词的开始和结束的空字符串。符号`\b`匹配次边界的空字符串，`\B`匹配空字符串，如果它不是词的边界的话。符号`\w`是表达式`[_[:alnum:]]`的同义表达式，`\W`是`[^_[:alnum:]]`的同义表达式。

**重复**

一个正则表达式可能跟着几个重复操作符：

|表达式  |   含义   |
|-------|---------|
| ?     | 之前的项目是可选项或者只匹配一次 |
| *     | 前面的项目会匹配0次或多次 |
| +     | 前面的项目会匹配一次或多次 |
| {n}   | 之前的项目会匹配准确的N此 |
| {n,}  | 前面的项目会匹配n次，或更多次 |
| {,m}  | 前面的项目会匹配最多m次，这是GNU的扩展 |
| {n,m} | 前面的项目会匹配最少n次，但是不会多于m次|

**连接**

两个正则表达式可能会被连接，结果正则表达式会匹配任何由两个子串形成的字符串，其中每一个字串分别各自匹配链接成正则表达式的两个子正则表达式。

**二选一**

两个正则表达式可以使用插入字符`|`连接；结果正则表达式则用于匹配两个正则表达式任意一个可选表达式。

**优先**

重复表达式要比连接表达式优先级高，连接表达式又比二选一表达式有更高优先级。整个表达式可能被放到一个括号中，这样就覆盖了游仙界规则，形成了一个子表达式。

**逆向引用和子表达式**

逆引用`\n`用于使用前面第n个括弧子正则表达式匹配子串。

**基本与扩展正则表达式**

在基本的正则表达式中，元字符`？`，`{`，`|`，`(`和`)`会丢失它们自己特殊的含义，要使用`\？`，`\{`，`\|`，`\(`和`\)`代替。

如下给出一些例子：

```
$ grep -n 't[ea]st' express.txt	// []表示选括号内某一个字符，进行匹配
8:I can't finish the test.
9:Oh! The soup taste good.

$ grep -n '[^g]oo' express.txt	// 在[]中的 ^表示除[]之外的字符
2:apple is my favorite food.
3:Football game is not use feet only.
18:google is the best tools for search keyword.
19:goooooogle yes!

// []用来表示范围，比如[a-z]表示小写字母，[0-9]表示0-9的数字，[A-Z]表示所有大写字母任选一个

$ grep -n '^the' express.txt			// ^ 表示以它之后的字符串或表达式作为行的开始
12:the symbol '*' is represented as start.

$ grep -n 'start.$' express.txt			// $ 表示以start. 结尾，$表示行的结尾( 不是字符，是位置）
12:the symbol '*' is represented as start.

// ‘^$’ 就表示空行，$ 字符表示以 $之前的字符或模式匹配行结尾

$ grep -n '^[^a-zA-Z]' express.txt		// 表示不以小写字母 大写字母开始的行
1:"Open Source" is a good mechanism to develop programs.
21:#I am VBird

$ grep -n 'g..d' express.txt			// . 表示任意一个字符，而 * 则匹配0个或多个字符，bash中匹配任意个字符
1:"Open Source" is a good mechanism to develop programs.
10:Oh! The soup taste good.
18:The world <Happy> is the same with "glad".

$ grep -n 'goo*g' express.txt			// * 表示匹配它前面的一个字符或表达式，表示0个或多个，所以goog被匹配出来
20:google is the best tools for search keyword.
21:goooooogle yes!

// .*  则表示0个或多个任意字符。如果要表示确切的字符数量，则需要使用{}
// 由于{} 在Shell中有特殊含义，所以要用 / 进行转义，比如 'o/{2/}'表示匹配2个字母o

// 这里在Windows上有一些不同，需要使用 \进行转义
$ grep -n 'o\{2\}' express.txt			// Windows上转义使用 \
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
10:Oh! The soup taste good.
20:google is the best tools for search keyword.
21:goooooogle yes!

$ grep -E -n '(oo)+' express.txt		// + 是扩展正则，需要使用 -E选项，表示前面的表达式要出现一个或多个。
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
10:Oh! The soup taste good.
20:google is the best tools for search keyword.
21:goooooogle yes!
```


By Andy @2018-06-29 09:03:23


