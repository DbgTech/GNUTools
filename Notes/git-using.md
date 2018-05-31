
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

**添加跟踪与提交更新**:

使用`git add`命令添加要跟踪的文件，例如前面例子中`git add README`，将README文件添加跟踪。

使用`git commit`将暂存的内容提交更新。如果不加参数，那么会启动默认编辑器来提醒输入提交说明信息。前面`git config --global core.editor`命令可以用于修改默认的编辑软件。

如下为一次提交的内容，其中列举出了提交在那个分支（master），本次提交的SHA-1校验和的简略提示（463dc4f），本次提交的日志信息，在下面列举了本次提交涉及的文件修改，两个文件修改，插入两行，创建了README文件节点。

```
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
  2 files changed, 2 insertions(+)
  create mode 100644 README
```

`git commit -a -m "test"`命令将add和commit两个命令合并成一个命令执行。这种方式只能针对跟踪的文件，对于未跟踪的文件还是需要使用`git add`命令将文件先添加追踪。

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

```
# 不追踪 .a 文件
*.a
# 但是lib.a是一个例外，即使前面配置*.a
!lib.a
# 仅仅忽略当前文件夹下的TODO文件，对于子目录下的TODO并不忽略
/TODO
# 忽略build目录的所有文件
build/
# 忽略 doc/notes.txt，而doc/server/arch.txt不会忽略
doc/*.txt
# 忽略doc/目录下的所有pdf，包括更深一级的pdf文件
doc/**/*.pdf
```

**查看修改**：

`git status`命令给出的修改状态是文件级别的，而具体修改的内容通过`git status`命令无法得到，需要使用`git diff`命令。

例如在一个例子中修改了README文件且暂存，修改CONTRIBUTING.md文件没有做处理。得到如下结果：

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   CONTRIBUTING.md

////////////////////////////////////////////////////
$ git diff
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
index a1892e3..37218ed 100644
--- a/CONTRIBUTING.md
+++ b/CONTRIBUTING.md
@@ -1 +1,2 @@
 CONTRIBUTING.md
+Modify
```

可以发现，给出的修改内容是CONTRIBUTING.md的，而README的修改并没有列出来。其实就是对比的工作目录中文件和Git库中HEAD指向提交中该文件内容之间的差异（就是修改之后，没有暂存的内容）。

`git diff --cached`或`git diff --staged`命令用于查看暂存区中待提交内容。其实与`git diff`类似，它也是将暂存区中文件和git库HEAD当前指向提交中文件作对比。

**移除文件/文件改名**:

`git rm`命令用于移除已追踪的文件，如果仅仅执行rm命令，那么文件不会从Git库中移除，并且会提示`Changes not staged for commit`，表示文件有修改，但是还没有放入暂存，和普通修改文件动作类似；而如果使用`git rm`命令则提示为`Changes to be committed`，表示删除文件操作会被提交。如果在使用`rm`删除文件后，再执行`git add`和执行`git rm`等效。

```
$ git rm option-a
  rm 'option-a'
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    option-a
```

如果要删除的文件已经放入暂存区了，执行`git rm`命令会失败。如果彻底删除该文件则需要加强制删除选项`-f`（force），这样既删除暂存区内容，同时从Git库中删除掉该文件。

另外一种情况，只想要把文件从Git库删除，但是还想让它留在当前目录，比如编译生成文件。可是使用`git rm --cached`只删除Git库和暂存区域中该文件，而保留当前目录中文件。还可以使用glob模式，比如`git rm log/\*.log`删除log目录下的所有`.log`文件，`\`符号用于转义，防止shell展开。

```
$ git rm --cached new.txt
rm 'new.txt'

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    new.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        new.txt
```

`git mv`命令用于修改文件的名字，用法类似mv命令。它相当于`mv，git mv，git add`三个命令结合，即先重命名文件，再从Git库删除原有文件，最后将新名字文件加入追踪。

```
$ git mv file1.txt file.txt

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    file1.txt -> file.txt
```

**查看历史**:

`git log`查看Git库的提交记录，它会将当前库所有的提交记录显示出来，如下类似的方式。最近的记录排在前面，包括提交的SHA-1校验和，作者名字，电子邮件地址，提交时间以及提交说明。

```
$ git log
commit f374c9b5397427c220a4eb69d732ec5e707d5abc (HEAD -> master)
Author: Andy Guo <xiao_0429@126.com>
Date:   Thu May 31 09:51:41 2018 +0800

    rm new.txt

commit 0cf0c746b67358843fd0a2b4f6be3ea925c0fab8
Author: Andy Guo <xiao_0429@126.com>
Date:   Thu May 31 09:50:01 2018 +0800

    remove usless file

commit b7573c6ba3d3bccffe1c36ccd1aa321912f1c5a2
Author: Andy Guo <xiao_0429@126.com>
Date:   Thu May 31 09:40:13 2018 +0800

    add option -a

......

```

`git log`还有很多选项用来过滤或格式化输出。

| 选项 | 说明 |
|-----|-----|
| -p | 显示每次提交内容差异|
| -n | n为数字，表示显示最近几次|
|--stat|列出每次提交的简略统计信息，多少修改文件，修改添加与移除行|
|--pretty|使用不同格式展示提交历史，见下面详述|
|--graph| 以ASCII码图形方式显示提交历史，分支比较明显|
|--name-only| 仅在提交信息后显示已修改文件清单|
|--name-only| 显示新增，修改，删除文件清单|
|--abbrev-commit| 显示SHA-1前几个字符，简要形式|
|--relative-date| 使用较短的相对时间显示|
|--since/--after| 从那天开始，2.weeks|
|--until/--before| 直到那一天为止 |
|-S| 显示添加或删除（文件内容）某个关键字的提交 |
|--grep| 仅显示提交说明中含有特定关键字的提交|
|--author| 指定作者相关的提交|
|--committer| 指定提交者相关的提交 |


`--pretty=`可用的展现方式：

`oneline`在一行显示每一个提交，`short`以简略方式形式显示，`full`则是完整显示，`fuller`额外再添加提交者信息。`format:xx-xx`可以定义输出形式。

比如`git log --pretty=format:"%h - %an, %ar : %s"`，显示哈希值，作者，提交时间以及提交说明。

```
$ git log --pretty=format:"%h - %an, %ar : %s"
f374c9b - Andy Guo, 19 minutes ago : rm new.txt
0cf0c74 - Andy Guo, 21 minutes ago : remove usless file
b7573c6 - Andy Guo, 31 minutes ago : add option -a
```

|选项 | 说明 |
|----|-----|
|%H | 提交对象（commit）的完整哈希字串 |
|%h | 提交对象的简短哈希字串 |
|%T | 树对象（tree）的完整哈希字串 |
|%t | 树对象的简短哈希字串 |
|%P | 父对象（parent）的完整哈希字串 |
|%p | 父对象的简短哈希字串 |
|%an | 作者（author）的名字 |
|%ae | 作者的电子邮件地址 |
|%ad | 作者修订日期（可以用 --date= 选项定制格式）|
|%ar | 作者修订日期，按多久以前的方式显示 |
|%cn | 提交者（committer）的名字 |
|%ce | 提交者的电子邮件地址 |
|%cd | 提交日期 |
|%cr | 提交日期，按多久以前的方式显示 |
|%s  | 提交说明 |


**远程库**:

和远程库关联，有https和SSH两种方式，HTTPS有时一些服务器上会有大小限制，使用SSH上传代码则没有这样的限制。使用SSH需要配置RSA Key，需设置本地SSH配置，参考github上的Key设置。

###.git目录说明###

在目录中执行了`git init`命令后或者`git clone`命令克隆一个仓库到本地时，都会生成一个`.git`目录，其中包含了Git的所有的配置信息。如果想要将当前库脱离Git管理，直接删除`.git`目录即可。

如下是一个`.git`目录中内容的列表。

```
├── HEAD
├── branches
├── config
├── description
├── hooks
│ ├── pre-commit.sample
│ ├── pre-push.sample
│ └── ...
├── info
│ └── exclude
├── objects
│ ├── info
│ └── pack
└── refs
├── heads
└── tags
└── logs
│ ├── HEAD
│ └── refs
│ │ └── heads
│ │ │ └── master
```

**HEAD**: 当前目录对应提交的指针。

**config**: 包含了仓库的设置信息，该文件中设置只对当前仓库有效。信息包括远程仓库URL，email地址，以及用户名等，`git config`不使用`--global`或`--system`修改的就是该文件。

**description**: gitweb用来显示仓库的描述。

**hooks**: Git提供了一系列脚本，可以在git每个有实质意义的阶段让它们自动运行。这些脚本就被称为hooks，可以在`commit/rebase/pull`等命令前后运行，脚本名字表示什么时候运行。

**info/exclude**: 作用类似`.gitignore`文件，就是不想让git处理的文件写到该文件中。

**objects**: 存放文件压缩对象和快照压缩对象的地方。

**refs**: 保存了分支文件，文件内容为某个提交的SHA-1哈希值

**heads**: 

**tags**: 在Git库中建立的tags，每个tag对应一个文件

**logs**: 缓存了日志信息。

**commit内容介绍：**

git对跟踪的文件会进行压缩，并用git自己的数据结构形式存储。压缩的对象有一个唯一的名字即一个哈希值，这个名字会存放在object目录下。commit的结果可以当作一个工作目录的快照，但不仅仅是一种快照。

在提交内容时，git为了创建工作目录的快照做了两件事情：

1. 如果文件没有发生变化，git仅仅只把压缩文件的名字（就是哈希值）放入快照。
2. 如果文件发生了变化，git会压缩它，然后把压缩的文件存入object目录，最后再把压缩文件的名字（哈希值）放入快照。

快照创建好后，本身会被压缩并且以一个哈希值命名，所以压缩对象都放在了object目录了。比如一个对象的Hash（SHA1）值为4cf44f1e3fe4fb7f8aa42138c324f63f5ac85828，那么它会存在4c子目录下以`f44f1e3fe4fb7f8aa42138c324f63f5ac85828`名字命名。

```
├── 4c
│ └── f44f1e3fe4fb7f8aa42138c324f63f5ac85828 // hash
├── 86
│ └── 550c31847e518e1927f95991c949fc14efc711 // hash
├── e6
│ └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391 // hash
├── info // let's ignore that
└── pack // let's ignore that too
```

一次提交中具有2+n个对象生成，n对应于n个修改或加入的文件；2个对象中，一个是提交时所创建的快照，另外一个是本次提交的信息，比如工作目录快照哈希值，提交说明信息，提交者，父提交的哈希值。如下代码块是一次提交几个对象的内容，在这次提交中一共加入两个文件file1.txt，file2.txt，它们的内容对应文件名（容易识别）；提交时注释内容为`add two file`。所以这次提交就有四个对象，分别对应两个文件，快照和提交信息文件；它们如下所列出对应关系。在快照文件中可以发现，每一行四列，第一列为文件类型，第二列为数据类型，第三列为哈希值，第四列为对应文件或目录名字。

```
$ git cat-file -p 5a089aae997b6f247fa66848206e9b5261d3325d
file1.txt

$ git cat-file -p 0ac9638029d5f57f672ab7e1818bcf28812676ad
file2.txt

$ git cat-file -p 67ad13cb129c55ff209597b03bf2af44d6d61d35
100644 blob 5a089aae997b6f247fa66848206e9b5261d3325d    file1.txt
100644 blob 0ac9638029d5f57f672ab7e1818bcf28812676ad    file2.txt
100644 blob fa49b077972391ad58037050f2a75f74e3671e92    new.txt

$ git cat-file -p ff8c1680b1cdfeb9dc297bcc4d87501df4949d64
tree 67ad13cb129c55ff209597b03bf2af44d6d61d35
parent ad61313d949aa844991e86c036b5b7ed3140e3bc
author Andy Guo <xiao_0429@126.com> 1527682270 +0800
committer Andy Guo <xiao_0429@126.com> 1527682270 +0800

add two file
```

**分支/标签/HEAD介绍：**

HEAD对应内容是什么呢？如下所示例子。HEAD并不是一个哈希，它可以看作目前所在分支的指针。那它既然是指针，指向内容是什么呢？看一下master的内容。从master的内容可以清楚看到，它是一个哈希值，并且这个哈希值就是上面我们刚刚进行的一次提交，即提交信息对象的哈希值。

```
$ cat HEAD
ref: refs/heads/master

$ cat refs/heads/master
ff8c1680b1cdfeb9dc297bcc4d87501df4949d64
```

而分支和标签和HEAD相似，也就是一个提交的指针。当删除分支和标签后，它们指向的提交仍然在，只是不容易直接被访问了。

从另外一方面讲，一次提交并非当前工作目录的快照，它是想要提交文件的快照（本次提交中只保存了修改的文件）。

https://linux.cn/article-7639-1.html

