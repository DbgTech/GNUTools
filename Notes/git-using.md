
#git使用示例

###Git的基本原理###

Git与其他版本控制系统的差别在于Git对待数据的方法，其他的版本控制系统主要存储文件变更信息，保存的信息可以看作一组基本文件和每个文件随着时间逐步累积的差异。如下图1所示，每个版本存储的都是与上一个版本的差异内容。

![图1 Git工作区-暂存区和Git仓库示意图](\image\Git-Store-File-Delta-to-Orgin.jpg)

Git则是将数据看作是小型文件系统的一组快照。提交更新或Git保存项目状态时，对当时的全部文件制作一个快照并保存快照索引。为了高效，如果文件没有修改，git不再重新存储文件而只保留一个链接指向之前存储文件。如图2所示，每一个版本对于没有修改的文件则直接使用虚线表示直线前面文件的索引。

![图2 Git工作区-暂存区和Git仓库示意图](\image\Git-Timing-change-image.jpg)

Git有三种状态，文件会处于其中之一：已提交（committed），已修改（modified）和已暂存（staged）。与三种状态对应引入了Git项目中的三个工作区域的概念：Git仓库，工作目录以及暂存区域。三个工作区域的交互如下图3所示。

![图3 Git工作区-暂存区和Git仓库示意图](\image\Git-WorkingDirectory-StagingArea-GitRepository.jpg)

**Git仓库**用来保存项目的元数据和对象数据库，它是git最重要的部分，从其他计算机克隆仓库时拷贝的就是这里的内容。

**工作目录**是对应项目的某个版本提取出来的内容。即从Git仓库的压缩数据库中提取出来的文件，放在磁盘上供使用或修改。

**暂存区域**是一个文件，保存了下一次将提交的文件列表信息，一般在Git仓库目录中。

基本的Git工作流程：

* 在工作目录中修改文件。
* 暂存文件，将文件快照放入暂存区域。
* 提交更新，找到暂存区域的文件，将快照永久性存储到Git仓库目录。

###Git帮助###

通过如下三种方式可以获取Git命令的使用手册中帮助信息。

```
git help <verb>
git <verb> --help
man git-<verb>		// 仅仅用于Linux类系统
```

比如 `git help config`即打开本地Git文档对应的config命令的文档。

###Git配置###

Git自带一个`git config`工具用来帮助设置Git外观和行为，每台机器上只需配置一次。`git config`配置的变量会存储在三个不同位置：

* `/etc/gitconfig`文件：包含了每一个用户以及他们仓库的通用配置，使用带有`--system`选项的`git config`时会从这个文件读写配置变量。在windows上该文件位于Git安装目录下。
* `～/.gitconfig`或`～/.config/git/config`文件：只针对当前用户，可以传递`git config --global`形式的命令来读写该文件。在Windows系统中，Git会查找`$PATH`目录下的`.gitconfig`文件，用于针对用户的设置。
* 当使用当前仓库的git目录中的config文件（即`.git/config`），则只针对当前仓库生效。

每一级的配置信息会覆盖上一级中相同的配置信息。

**用户信息**：首先要设置你的用户名称和邮件地址，每一个Git的提交都会使用这些信息，会写入每一次提交中。配置个人信息如下方式：

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

使用参数`--global`会将配置信息写入系统登录账户相关的配置文件中，即`～/.gitconfig`中。如果在Windows上使用`--system`选项，配置内容会被写入Git安装目录下`mingw32\etc\gitconfig`文件中，其并不在Git命令行中根目录的`/etc/gitconfig`文件中。

**文本编辑器**： 如果未配置，Git使用系统的默认文本编辑器，通常是Vim。如果要设置为Emacs可以使用`git config --global core.editor emacs`命令。

再比如`git config --global core.autocrlf true`将允许保留纯文本文件中的`CRLF`字符而不强制归一化为`LF`符号。

对于Windows上写文档，将core.autocrlf设置为true，那么在检出代码时换行符LF会被转换成回车换行（CRLF）；而如果在Linux上，则不需要在检出文件时做自动转换，但是如果回车换行（CRLF）此时被当作换行，则会想让git修正，则可以把core.autocrlf设置为input，同时告诉Git在提交时把回车换行（CRLF）换成换行LF，检出时不换；如果是Windows程序员，且开发项目只在Windows上使用，则可以将core.autocrlf设置为false，将回车换行保留在版本库。

```
# 提交时转换为LF，检出时转换为CRLF
git config --global core.autocrlf true

# 提交时转换为LF，检出时不转换
git config --global core.autocrlf input

# 提交检出均不转换
git config --global core.autocrlf false

// 对于提交文档换行符的校验
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true

# 允许提交包含混合换行符的文件
git config --global core.safecrlf false

# 提交包含混合换行符的文件时给出警告
git config --global core.safecrlf warn
```

如果core.autocrlf为true或input且core.safecrlf为true，那么Git在入库时会检查行结束符是否满足条件。即在Windows上，如果core.autocrlf为true，core.safecrlf为true，工作区文件中包含了LF，则Git就会拒绝入库。默认的core.autocrlf值为warn，所以在执行`git add`命令时提示warnning。

使用`git config --list`命令可以列举出Git当前能够找到的所有配置信息。命令`git config <key>`可以查看`<key>`对应的配置内容，使用`git config --help`查看文档中config更多的参数含义。

###Git基础###

**获取Git仓库**：

有两种方法，一种是在现有的目录下导入所有文件到Git中，第二种是从一个服务器克隆一个现有Git仓库。

在当前目录初始化仓库，执行`git init`，命令创建一个名为`.git`的子目录，子目录包含了Git仓库所必须的文件。然后要通过`git add`，`git commit`添加文件到仓库中，例如如下例子。

```
git add *.c
git add LICENSE
git commit -m "init project version"
```

另外一种方法是使用`git clone`命令，例如`git clone https://github.com/libgit2/libgit2`将libgit2克隆到本地一份，其中包含一个`.git`文件夹包含了Git仓库内容。如果要克隆为不同的目录名，则可以使用`git clone https://github.com/libgit2/libgit2.git mylibgit`来克隆到本地`mylibgit`目录中。

**检查状态**:

`git status`用于检查Git库当前的状态。如下例子所示。

```
$ git status
On branch master
nothing to commit, working directory clean

///////////////////////////////////////////
$ echo "My Project" > README
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        README

nothing added to commit but untracked files present (use "git add" to track)

///////////////////////////////////////////
$ git add README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   README

///////////////////////////////////////////
$ echo "modify exist" >> LICENSE
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   LICENSE

///////////////////////////////////////////
$ echo "Add ReadMe" >> README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   LICENSE
        modified:   README

$ git status -s
 M LICENSE
AM README

/////////////////////////////////////////////
$ git add LICENSE
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   LICENSE
        new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README

$ git status -s
M  LICENSE
AM README
```

`On branch master`表示当前位于master分支，工作区干净，没有需要提交内容。在创建了README文件之后，出现`Untracked files`提示，表示文件还没有纳入跟踪。执行`git add`命令后，状态提示变为`Changes to be committed`，表示修改将会在下一次提交。当然，如果执行了`git commit`命令则有回到最初的状态。修改一个已跟踪的文件LICENSE，其修改后状态提示为`Changes not staged for commit`，即已修改但是没有暂存。

再次修改README文件，可以发现README也出现在了LICENSE一样的位置，同时也出现在暂存区。使用`-s`选项用简单形式查看当前状态，README文件有两个状态，第一列的状态表示暂存区中的状态，第二列为工作目录状态。`A`表示增加文件，`M`表示文件修改过。将LICENSE执行add命令后，其暂存区状态变为M，工作区状态空白。

|字符  | 对应状态 |
|-----|---------|
| ' ' | unmodified|
| M   | modified |
| A   | added |
| D   | deleted |
| R   | renamed |
| C   | copied |
| U   | updated but unmerged |
| ?   | untracked |

**跟踪新文件**:

使用`git add`命令添加要跟踪的文件，例如前面例子中`git add README`，将README文件添加跟踪。

**忽略文件**:

一般在项目中总会有一些不需要纳入Git库的文件，比如编译的临时文件，日志文件等。可以创建一个`.gitignore`文件列出要忽略的文件模式，比如如下例子。

```
*.[oa]
*~
obj\
```

将忽略后缀名为`.o`和`.a`的文件，第二行表示以波浪线结尾的文件名，vim等编辑器临时文件，第三行表示忽略目录下的obj目录。

`.gitignore`文件的规范如下：

* 所有空行或者以`#`开头的行都被Git忽略
* 可以使用标准glob模式匹配，比如`*.[oa]`
* 模式匹配可以以`/`符号开始，防止递归
* 模式匹配以`/`结尾指定目录
* 对于忽略指定模式以外的文件或目录，可以在模式前加`!`表示取反

glob模式即简化的正则表达式，星号（*）匹配零个或多个任意字符；[abc]匹配一个列在方括号中的字符；问号（?）只匹配一个任意字符；如果方括号中使用中划线分割字符，则表示在两个字符范围内的都可以匹配；两个星号表示匹配任意中间目录，比如`a/**/z`可以匹配`a/z`，`a/b/z`或`a/b/c/z`等。

**查看修改**：

