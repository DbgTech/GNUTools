#grep命令#

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
|-b   | --byte-offset | 打印        print the byte offset with output lines
  -n, --line-number         print line number with output lines
      --line-buffered       flush output on every line
  -H, --with-filename       print the file name for each match
  -h, --no-filename         suppress the file name prefix on output
      --label=LABEL         use LABEL as the standard input file name prefix
  -o, --only-matching       show only the part of a line matching PATTERN
  -q, --quiet, --silent     suppress all normal output
      --binary-files=TYPE   assume that binary files are TYPE;
                            TYPE is 'binary', 'text', or 'without-match'
  -a, --text                equivalent to --binary-files=text
  -I                        equivalent to --binary-files=without-match
  -d, --directories=ACTION  how to handle directories;
                            ACTION is 'read', 'recurse', or 'skip'
  -D, --devices=ACTION      how to handle devices, FIFOs and sockets;
                            ACTION is 'read' or 'skip'
  -r, --recursive           like --directories=recurse
  -R, --dereference-recursive  likewise, but follow all symlinks
      --include=FILE_PATTERN  search only files that match FILE_PATTERN
      --exclude=FILE_PATTERN  skip files and directories matching FILE_PATTERN
      --exclude-from=FILE   skip files matching any file pattern from FILE
      --exclude-dir=PATTERN  directories that match PATTERN will be skipped.
  -L, --files-without-match  print only names of FILEs containing no match
  -l, --files-with-matches  print only names of FILEs containing matches
  -c, --count               print only a count of matching lines per FILE
  -T, --initial-tab         make tabs line up (if needed)
  -Z, --null                print 0 byte after FILE name

Context control:
  -B, --before-context=NUM  print NUM lines of leading context
  -A, --after-context=NUM   print NUM lines of trailing context
  -C, --context=NUM         print NUM lines of output context
  -NUM                      same as --context=NUM
      --color[=WHEN],
      --colour[=WHEN]       use markers to highlight the matching strings;
                            WHEN is 'always', 'never', or 'auto'
  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)
  -u, --unix-byte-offsets   report offsets as if CRs were not there
                            (MSDOS/Windows)

'egrep' means 'grep -E'.  'fgrep' means 'grep -F'.
Direct invocation as either 'egrep' or 'fgrep' is deprecated.
When FILE is -, read standard input.  With no FILE, read . if a command-line
-r is given, - otherwise.  If fewer than two FILEs are given, assume -h.
Exit status is 0 if any line is selected, 1 otherwise;
if any error occurs and -q is not given, the exit status is 2.

Report bugs to: bug-grep@gnu.org
GNU grep home page: <http://www.gnu.org/software/grep/>
General help using GNU software: <http://www.gnu.org/gethelp/>