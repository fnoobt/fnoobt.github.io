---
title: Git使用教程
author: fnoobt
date: 2021-12-20 16:35:00 +0800
categories: [Linux,Git]
tags: [linux,git]
media_subpath: '/assets/img/commons/linux/git/'
---

## 安装

### 理论基础

#### Git 记录的是什么？

![GitVersion](git-version-control.png)

如上，如果每个版本中有文件发生变动，Git 会将整个文件复制并保存起来。这种设计看似会多消耗更多的空间，但在分支管理时却是带来了很多的益处和便利。

#### 三棵树
你的本地仓库有 Git 维护的三棵“树”组成，这是 Git 的核心框架。这三棵树分别是：**工作区域、暂存区域和 Git 仓库**

![GitThreeTrees](git-three-trees.png)

**工作区域（Working Directory）**就是你平时存放项目代码的地方。

**暂存区域（Stage）**用于临时存放你的改动，事实上它只是一个文件，保存即将提交的文件列表信息。

**Git 仓库（Repository）**就是安全存放数据的位置，这里边有你提交的所有版本的数据。其中，HEAD 指向最新放入仓库的版本（这第三棵树，确切的说，应该是 Git 仓库中 HEAD 指向的版本）。

Git 的工作流程一般是：
1. 在工作目录中添加、修改文件；
2. 将需要进行版本管理的文件放入暂存区域；
3. 将暂存区域的文件提交到 Git 仓库。

因此，Git 管理的文件有三种状态：**已修改（modified）**、**已暂存（staged）**和**已提交（committed）**，依次对应上边的每一个流程。

### Git安装

#### windows安装  
进入[Git官网](https://git-scm.com)下载安装对应的[Win版本](https://git-scm.com/download/win)  
安装完成后，在开始菜单里找到<kbd>Git</kbd> -> <kbd>Git Bash</kbd>，会弹出一个命令行窗口，后续在此操作Git

#### ubuntu安装
```bash
apt-get install git
```

#### 配置用户信息
安装完后在cmd命令行设置用户信息，最好与注册的github一致
```bash
git config --global user.name "user.name"
git config --global user.email "user.email@example.com"
#检查信息是否写入成功
git config --list 
```

## 连接Github

### 为本机生成 SSH 密钥对
执行如下命令产生 SSH 密钥对：
```bash
ssh-keygen -t rsa -C "本机标识"
```
上面命令中的 `-C` 只是给产生的密钥对加了一个注释，建议填写跟当前机器相关的内容

生成的 SSH 密钥对存储在 `C:\Users\账户名\.ssh`{: .filepath} 目录下，分别为私钥 `id_rsa` 和公钥 `id_rsa.pub`

接着将 id_rsa.pub 公钥里的内容复制到剪贴板，可以执行如下命令去复制，也可以打开 `C:\Users\账户名\.ssh\id_rsa.pub`{: .filepath} 文件去复制：
```bash
clip < ~/.ssh/id_rsa.pub
```

### 将公钥拷贝到 GitHub 上
将剪贴板中ssh key粘贴到Github的中，title建议写本机标识，以后删除才能分辨。  
拷贝位置：**Github** --> <kbd>Settings</kbd> --> <kbd>SSH and GPG keys</kbd> --> <kbd>New SSH key</kbd>中

### SSH 测试

执行如下命令，初次设置需要输入 yes，出现 successfully 表示成功。
```bash
ssh -T git@github.com
```

### Git远程操作
常见命令如下：

|               命令名称                |                              作用                               |
| :-----------------------------------: | :-------------------------------------------------------------: |
|             git remote -v             |                    查看当前所有远程地址别名                     |
|     git remote add 别名 远程地址      |                       给远程Git仓库起别名                       |
|         git push 别名 分支名          |                 推送本地分支上的内容到远程仓库                  |
|          git clone 远程地址           |                   将远程仓库的内容克隆到本地                    |
|  git fetch 远程库地址别名 远程分支名  |            将远程仓库对于分支最新内容拉取到本地仓库             |
|           git merge 分支名            |                合并分支，git fetch后需要使用这个                |
|  git pull 远程库地址别名 远程分支名   |    将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并     |
|         git branch -d 分支名          |                            删除分支                             |
| git push 远程库地址别名 -d 远程分支名 | 删除远端仓库分支:例如删除远端的test分支 git push origin -d test |

## 初始化

### 本地初始化
创建一个空目录，注意路径中不要含有中文字符。打开Git Bash，通过下面的命令把这个目录变成git可以管理的仓库
```bash
git init
```

初始化Git项目，成功后将创建一个.git隐藏文件

系统自动创建了唯一一个master分支

版本控制系统只能跟踪文本文件的改动，且编码方式是utf-8

### 远程克隆
克隆GitHub上别人的代码
```bash
git clone 目标
```

## 基础操作

创建一个test.txt文件，内容如下：
```
Hello World
```

### 添加文件到暂存区
```bash
git add test.txt
```

### 提交文件到仓库
```bash
git commit -m "a new file"
```

`-m` 后面输入的是本次提交的说明，提交成功后会显示：
1. `file changed`：1个文件被改动（我们新添加的test.txt文件）；
2. `insertions`：插入了一行内容（test.txt有一行内容）。

为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件
```bash
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files."
```

如果提交的备注写错了，可以用以下命令修改刚刚提交的备注
```bash
git commit --amend
```

### 一步到位

从工作目录一步添加到仓库：`git commit -am "说明"`
```bash
git commit -am "change the license file"
```

> `-a` 的意思是add。

### 修改文件

将test.txt文件修改如下
```bash
Hello World ABC
```

由于你对工作目录的文件进行了修改，导致这个文件和暂存区域的对应文件不匹配了，此时运行`git status`， Git 将给你提出两条建议：
- 使用 `git add` 命令将工作目录的新版本覆盖暂存区域的旧版本，然后准备提交
- 使用 `git checkout` 命令将暂存区域的旧版本覆盖工作目录的新版本（危险操作：相当于丢弃工作目录的修改）

提交
```bash
git add test.txt
git commit -m "append ABC"
```

每次commit都会生成一个`快照`

### 查看历史记录
```bash
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)
Author: jianglang <my_hb@qq.com>
Date:   Fri Aug 19 21:06:15 2022 +0800
 
    append ABC
 
commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: jianglang <my_hb@qq.com>
Date:   Fri Aug 19 20:59:18 2022 +0800
 
    a new file
```

`git log`显示最近到最远的提交日志，我们可以看到两次提交，最后一次是append ABC

git的版本号是用SHA1计算出来的一个16进制数

如果嫌输出信息太多，可以加上`--pretty=oneline`
```bash
$ git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append ABC
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 a new file
```

## 查看状态

我们怎么知道哪些文件是新添加的，哪些文件已经加入了暂存区域呢？总不能让我们自己拿个本本记下来吧？
当然不，作为世界上最伟大的版本控制系统，你能遇到的囧境，Git 早已有了相应的解决方案。随时随地都可以使用`git status`查看当前状态
```bash
$ git status
On branch master
nothing to commit, working tree clean
```

常见信息：
  - `On branch master`: 我们位于一个叫做“master”的分支里，这是默认的分支
  - `nothing to commit, working directory clean` : 说明你的工作目录目前是“干净的”，没有需要提交的文件（意思就是自上次提交后，工作目录中的内容压根儿就没改动过）。
  - `Untracked files` 说明存在未跟踪的文件（呈现为红色）。所谓的“未跟踪”文件，是指那些新添加的并且未被加入到暂存区域或提交的文件。它们处于一个逍遥法外的状态，但你一旦将它们加入暂存区域或提交到 Git 仓库，它们就开始受到 Git 的“跟踪”。
  - `use "git reset HEAD <file>..." to unstage` 的意思是“如果你反悔了，你可以使用 `git reset HEAD` 命令恢复暂存区域”。如果后面接文件名，表示恢复该文件；如果不接文件名，则表示上一次添加的文件。

## 回退
有关回退的命令有两个：`reset` 和 `checkout`

![GitResetCheckout](git-add-commit-reset-checkout.webp.png)

### 回滚快照

> 注：快照即提交（commit）的版本，每个版本我们称之为一个快照。
{: .prompt-tip }

用 reset 命令回滚上一版本快照
```bash
# git reset --mixed HEAD~ 的缩写， --mixed 选项是默认的
git reset HEAD~
``` 

> 注：HEAD 表示最新提交的快照，而 HEAD~ 表示 HEAD 的上一个快照，HEAD~~表示上上个快照，如果表示上10个快照，则可以用HEAD ~10
{: .prompt-tip }

>**默认**  
git reset HEAD~ 命令其实影响了两棵树：首先是移动 HEAD 的指向，将其指向上一个快照（HEAD~）；然后再将该位置的快照回滚到暂存区域。  
**--soft选项**  
`git reset --soft HEAD~`命令就相当于只移动 HEAD 的指向，但并不会将快照回滚到暂存区域。相当于撤消了上一次的提交（commit）。一不小心提交了，后悔了，那么你就执行 `git reset --soft HEAD~` 命令即可（此时执行 git log 命令，也不会再看到已经撤消了的那个提交）。  
**--hard选项**  
reset 不仅移动 HEAD 的指向，将快照回滚动到暂存区域，它还将暂存区域的文件还原到工作目录。

### 回滚指定快照
reset 不仅可以回滚指定快照，还可以回滚个别文件。
```bash
git reset 快照 文件名/路径
```

这样，它就会将忽略移动 HEAD 的指向这一步（因为你只是回滚快照的部分内容，并不是整个快照，所以 HEAD 的指向不应该发生改变），直接将指定快照的指定文件回滚到暂存区域。

***不仅可以往回滚，还可以往前滚！***

这里需要强调的是：reset 不仅是一个“复古”的命令，它不仅可以回到过去，还可以去到“未来”。

唯一的一个前提条件是：你需要知道指定快照的 ID 号。

那如果不小心把命令窗口关了不记得ID号怎么办？
```bash
git reflog
```
Git记录的每一次操作的版本ID号

## 版本对比

目的：对比版本之间有哪些不同

### 准备工作

创建一个 README.md 文件，写入：
```
测试说明
```

执行 git add README.md 命令将文件添加到暂存区域，接着执行 `git commit -m "add readme"` 提交一个项目的快照  

更改README文件
```
关于
测试说明
```

### 比较暂存区域与工作目录
直接执行 `git diff` 命令是比较暂存区域与工作目录的文件内容：

> 这里可能出现一个问题，直接执行的结果出现中文乱码。在cmd界面的冒号：后面输入q退出，使用Notepad++打开README文件，点击编码→转为UTF-8编码。（旧版本中选择UTF-8（无BOM），新版中的UTF-8默认为无BOM）如果重新执行`git diff`发现还是不行！那就全局定义为utf-8

```bash
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

export LESSCHARSET=utf-8 ## linux bash配置环境变量
set LESSCHARSET=utf-8 #windows配置环境变量
```

```bash
diff --git a/README.md b/README.md  # 表示对比的是存放在暂存区域的 README.md 和工作目录的 README.md
index 7966837..472a180 100644 # 表示对应文件的 ID 分别是 7966837 和 472a180，左边暂存区域，后边当前目录。最后的 100644 是指定文件的类型和权限
--- a/README.md  # --- 表示该文件是旧文件（存放在暂存区域）
+++ b/README.md  # +++ 表示该文件是新文件（存放在工作区域）
@@ -1 +1,2 @@  # 以 @@ 开头和结束，中间的“-”表示旧文件，“+”表示新文件，后边的数字表示“开始行号，显示行数”
+ 关于
 测试说明  # 这两行是将两个文件合并显示的结果，前边有个 + 的绿色的那一行说明是新文件独有的，浅灰色的则是两个文件所共有的内容。所以，+1,2 表示新文件在合并显示中从第 1 行开始，显示 2 行。那为啥 -1 后边没有显示的行数？因为在合并显示的结果中，旧文件已经完全包含在新文件中了（也就是旧文件没有自己“独有的”内容）。
\ No newline at end of file  #这是 Git 出于善意的提醒：文件不是以换行符结束。为什么会有这样多此一举的提醒呢？仔细推敲下你就会明白了：diff 将两个文件合并打印，到达最后一个字符的时候，无论文件中是否存在换行符，diff 都需要打印一个换行符！为啥？为了好看呗！！所以如果你的文件最后一个字符不是以换行符结尾，但 diff 又自行添加了一个换行符，所以它觉得有义务提醒你它这么做了！
```

如果在窗口最后有一个 `:` 这代表窗口太小，没办法显示全部，正在等待命令（Vim编程知识）  
> **移动命令**  
j、k：向下移动一行/向上移动一行  
f、b：向下翻页/向上翻页  
d、u：向下翻半页/向上翻半页  
**跳转命令**  
g、G：跳转到第一行/跳转到最后一行  
先输入数字（如3），再输入g，表示跳转到该行（如第3行）  
**搜索命令**  
输入斜杠（/）或问号（?）,后面接搜索关键字  
区别：斜杠（/）表示从当前位置向下搜索，问号（?）表示从当前位置向上搜索。  
接着输入 n 表示顺着当前的搜索方向快速跳转到下个匹配的位置，大写的 N 则是与当前搜索方向相反。  
**退出和帮助**  
在点点（:）后边输入 q，表示退出 diff；输入 h 表示进入帮助界面，你会看到很多命令和功能，输入 q 可以退出帮助界面。  

### 比较两个历史快照

执行 `git diff ID号1 ID号2` 命令，即可比较 Git 仓库中两个快照的差异：

### 比较 Git 仓库中的快照和当前工作目录内容

执行 `git diff 仓库快照ID号`

### 比较之前版本的快照与当前工作目录内容
输入 `git diff 之前版本快照ID号` 命令即可

### 比较当前版本快照与当前工作目录内容
输入 `git diff HEAD` 命令即可

### 比较Git仓库与暂存区域

如果希望比较最新提交的快照和暂存区域的文件，只需要执行 `git diff --cached` 命令；当然也可以指定其他快照，就是需要多写上一个 ID 值，即`git diff --cached ID号`。

> 总结来说，`git diff`默认代表和当前工作目录比较（可指定和历史快照比较），`git diff --cached`代表和暂存区域比较。
{: .prompt-tip }

## 修改最后一次提交、删除文件和重命名文件

### 修改最后一次提交
情况一：版本刚一提交（commit）到仓库，突然想起漏掉两个文件还没有添加（add）  
情况二：版本刚一提交（commit）到仓库，突然想起版本说明写得不够全面，无法彰显你本次修改的重大意义……

由于使用reset命令过于繁琐，需要提交一个新的版本，这里可以使用带 --amend 选项的 commit 命令（即git commit --amend），Git 会“更正”最近的一次提交。由于这里没有`-m`说明，会进入编辑提交说明界面。如果不需要修改，可以连续按下两个大写Z来退出，或者按下（:）再输入`q!`退出，Git会保留旧的提交说明。如果需要提交说明又不想用这么繁琐的方式，输入`git commit --ammend -m "新的提交说明"` 即可。

### 删除文件
#### 问题一：不小心删除文件怎么办？
现在从工作目录中手动删除 README.md 文件，又希望恢复，用 checkout 命令可以丢弃工作区的修改：
```bash
git checkout -- README.md
```

> `--` 很重要，没有 `--` ，就变成了“切换到另一个分支”的命令
{: .prompt-warning }

该命令意思就是，把README.md文件在工作区的修改全部撤销，这里有两种情况：
1. test.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
2. test.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次 `git commit` 或 `git add` 时的状态。

#### 问题二：那么如何彻底删除一个文件呢？
假如你不小心把小黄图下载到了工作目录，然后又不小心提交到了 Git 仓库：
```bash
git rm yellow.jpg
```

此时工作目录中的小黄图（yellow.jpg）已经被删除……  
但执行 `git status` 命令，你仍然发现 Git 还有记录，在仓库的快照中还是可以发现有个叫 yellow 的东西，但似乎在暂存区域和当前目录不见了！

此时可以执行 `git reset --soft HEAD~` 命令将快照回滚到上一个位置，然后重新提交，Git 就不会再提小黄图的事儿了

> 注意：rm 命令删除的只是工作目录和暂存区域的文件（即取消跟踪，在下次提交时不纳入版本管理）  
当手动删除文件时，然后使用 `git rm <file>` 和 `git add<file>` 效果是一样的。
{: .prompt-tip }

#### 问题三：我在工作目录中增加一个 test.py 文件，然后执行 git add test.py 命令将其添加到暂存区域，此时我修改 test.py 文件的内容，那么暂存区域和工作目录就是两个不同的 test.py 文件了，此时如果我执行 git rm test.py 命令，Git 会下意识地阻止我，这是怎么办？

因为两个不同内容的同名文件，谁知道你是不是搞清楚了都要删掉？还是提醒一下好，别等一下出错了又要赖机器……
根据提示，执行 `git rm -f test.py` 命令就可以把两个都删除。

#### 问题四：我只想删除暂存区域的文件，保留工作目录的，应该怎么操作？
执行 `git rm --cached 文件名` 命令。

### 重命名文件
直接在工作目录重命名文件，执行`git status`出现错误：

正确的姿势应该是：
```bash
git mv 旧文件名 新文件名
```

> 注：Windows 使用 ren 命令修改文件名，Linux 是使用 mv 命令
{: .prompt-tip }

### 忽略文件
#### 如何让Git 识别某些格式的文件，然后自主不跟踪它们？
比如工作目录中有三个文件1.temp、2.temp 和 3.temp，我们不希望后缀名为 temp 的文件被追踪，可是每次执行`git status`都会出现：

解决办法：在工作目录创建一个名为 .gitignore 的文件。

然后你发现 Windows 压根儿不允许你在文件管理器中创建以点（.）开头的文件。windows需要在命令行窗口创建（.）开头的文件。执行 `echo *.temp > .gitignore` 命令，创建一个 .gitignore 文件，并让 Git 忽略所有 .temp 后缀的文件

## 创建和切换分支

### 分支是什么？
假设你的大项目已经上线了（有上百万人在使用），过了一段时间你突然觉得应该添加一些新的功能，但是为了保险起见，你肯定不能在当前项目上直接进行开发，这时候你就有创建分支的需要了。

![GitBranch](git-branch.png)

对于其它版本控制系统而言，创建分支常常需要完全创建一个源代码目录的副本，项目越大，耗费的时间就越多；而 Git 由于每一个结点都已经是一个完整的项目，所以只需要创建多一个“指针”（像 master）指向分支开始的位置即可。

### 创建分支

创建分支
```bash
git branch 分支名
```

没有任何提示说明分支创建成功（一般也不会失败啦，除非创建了同名的分支会提醒你一下），此时可以执行 `git log --decorate` 命令查看：

> 如果希望以“精简版”的方式显示，可以加上一个 --oneline 选项（即 `git log --decorate --oneline`），这样就只用一行来显示一个快照记录。
{: .prompt-tip }

可以看到最新的快照后边多了一个 (HEAD -> master, feature)

它的意思是：目前有两个分支，一个是主分支（master），一个是刚才我们创建的新分支（feature），然后 HEAD 指针仍然指向默认的 master 分支。

### 切换分支
现在我们需要将工作环境切换到新创建的分支（feature）上，使用的就是之前我们欲言又止的 checkout 命令。
 ```bash
 git checkout feature
 ```

这样 HEAD 指针就指向 feature 分支了

此时修改 README.md 文件，提交之后，将 HEAD 指针切回 master 分支，之前对 README.md 文件的修改已经荡然无存了，这是因为我们的工作目录已经回到 master 分支的状态中

查看分支图
```bash
git log --oneline --decorate --graph --all
```

_`--graph` 选项表示让 Git 绘制分支图，`--all` 表示显示所有分支_

## 合并和删除分支

### 合并分支
当一个子分支的使命完结之后，它就应该回归到主分支中去

合并分支我们使用 `merge` 命令，将 feature 分支合并到 HEAD 所在的分支（master）上
```bash
git merge feature
```

如果 Git 提示失败：  
> 这是因为合并 README.md 文件的时候出现冲突。所以自动合并失败；请修改冲突的内容并重新提交快照。

意思是说现在你需要先解决冲突的问题，Git 才能进行合并操作。所谓冲突，无非就是像两个分支中存在同名但内容却不同的文件，Git 不知道你要舍弃哪一个或保留哪一个，所以需要你自己来决定。  
此时执行 `git status` 命令也会显示需要你解决的冲突

然后 Git 会在有冲突的文件中加入一些标记，不信你打开 README.md 文件看看：

以`=======`为界，上到`<<<<<<< HEAD`的内容表示当前分支，下到`>>>>>>> feature`表示待合并的 feature 分支，之间的内容就是冲突的地方。

将 README.md 统一修改（去掉 `<<<<<<< HEAD` 等内容），保存文件，然后提交快照，可通过下面的命令查看是否合并
```bash
git log --decorate --all --graph --oneline
```

当然，如果不存在冲突，可以直接合并
```bash
# 相当于 git branch feature2 和 git checkout feature2 两个命令的合体
git checkout -b feature2
``` 

在工作目录随便创建一个文本文件（feature2.txt）并提交快照。  
此时feature2 分支比 master 分支快了一步。现在我们切换回 master 分支，并将 feature2 分支合并进来。  
这次 Git 就只会显示了 `Fast-forward`（快进）这词儿，这是因为 feature2 这个分支的父结点是 master 分支，所以 Git 只需要简单移动 master 的指向即可。

### 删除分支

删除分支
```bash
git branch -d 分支名
```

由于 Git 的分支原理实际上只是通过一个指针记载，所以创建和删除分支都几乎是瞬间完成。

> 注意：如果试图删除未合并的分支，Git 会提示你“该分支未完全合并，如果你确定要删除，请使用 git branch -D 分支名 命令。”
{: .prompt-tip }

### Git的两种合并方式（Fast-forward 和 Three-way merge）

#### Fast-forward
所谓的 Fast-forward 就是当待合并的分支位于目标分支的直接上游时，Git 只需把目标分支的指针直接移动即可实现合并。

比如 master 分支是位于 feature2 分支的直接上游，将 feature2 分支合并到 master 分支，只需要移动 master 的指针即可：

#### Three-way merge
如果待合并的两个分支不在同一条线上，那么进行合并就需要解决一个根本的问题 —— 冲突！

>**为何两个分支在同一条线上就不会冲突呢？**  
因为 Git 的快照是按时间顺序提交的，所以在同一条线上的两个快照，它们是有先后顺序的，尽管两者可能出现同名文件不同内容，Git 会认为这是“改变”而不是“冲突”。

举个例子：

![GitBranchMerge](git-branch-merge-method-demo.png)

合并 C3 和 C4 得到 C5，但 C5 应该如何处理“冲突”呢？

SVN 会把问题抛给用户，让用户自行解决；Git 则显得更为高明，它会找到第三个快照，然后综合三者特点自动解决冲突。

那第三个快照应该如何决定呢？

没错，应该找两者的共同“祖先”作为参照物，一比较就知道两个分支都干了些什么。

图片中 C3 和 C4 的共同祖先是 C1，可以看到 C3 和 C4 分别增加了 test2.py 和 test1.py 两个文件。因为对比之后发现三者并没有冲突，所以 C5 应该是三者的合体，即同时拥有 common.py、test1.py 和 test2.py 三个文件。

另外，值得一提的是，Git 这种合并方式也适用于**同名文件的不同更改**。

> 当 Git 检测到同名文件的同一行有不同的内容，才会报冲突并让你自行决定谁去谁留的！
{: .prompt-tip }

## 匿名分支和checkout命令
### 匿名分支

如果在没创建分支的情况下执行 `git checkout HEAD~` 命令，Git将提示：

>当前的 HEAD 指针处于分离状态，你可以环顾四周，做一些实验性修改并提交它们，并且你可以在这个状态中丢弃任何你所做的提交，而不影响任何分支，做法是执行 checkout 命令切换回别的分支。  
如果你希望创建一个新分支，并保持你所做创建的提交，你可以（现在或稍后）通过使用带有 -b 选项的 checkout 命令实现。例如：  
git checkout -b <new-branch-name>

你使用了 checkout 命令但没有指定分支名，所以 Git 帮你创建了一个匿名分支，OK，既然是匿名，那么当你切换到别的分支时，这个匿名分支中的所有提交都会被丢弃掉（因为你都没给人家命名，反正以后都找不回了，不丢了干啥？）。因此，你可以利用匿名分支来做做实验什么的，反正不会有负面影响。

### 再论checkout
事实上呢，checkout 命令有两种功能：

1. 从历史快照（或者暂存区域）中拷贝文件到工作目录
2. 切换分支

#### 从历史快照（或者暂存区域）中拷贝文件到工作目录

当给定某个文件名时，Git 会从指定的提交中拷贝文件到暂存区域和工作目录。比如执行 `git checkout HEAD~ README.md` 命令会将上一个快照中的 README.md 文件复制到工作目录和暂存区域中：

如果命令中没有指定具体的快照 ID，则将从暂存区域恢复指定文件到工作目录 `git checkout README.md`

有些朋友可能会问：“上次看你在文件名的前边有加两个横杆（--），这次怎么就没有了呢？”

问得好，Git 提醒你写成 `git checkout -- README.md` 的形式，那是为了预防你恰好有一个分支叫做 README.md，那么它就搞不懂你要恢复文件还是切换分支了，所以约定两个横杆（--）后边跟的是文件名。

反过来说，如果你确保你没有一个叫做 README.md 的分支，你直接写 `git checkout README.md` 也是妥妥的没问题啦

#### 切换分支
首先我们知道 Git 的分支其实就是添加一个指向快照的指针，其次我们还知道切换分支除了修改 HEAD 指针的指向，还会改变暂存区域和工作目录的内容。

那回过头来，如果我们只想恢复指定的文件/路径，那么我们只需要指定具体的文件，Git 就会忽略第一步修改 HEAD 指向的操作，这不正跟之前讲 reset 命令的时候一样吗？

### checkout 命令和 reset 命令的区别
#### 恢复文件

checkout 命令和 reset 命令都可以用于恢复指定快照的指定文件，并且它们都不会改变 HEAD 指针的指向。

下面开始划**重点**：

它们的区别是 reset 命令只将指定文件恢复到暂存区域（--mixed），而 checkout 命令是同时覆盖暂存区域和工作目录。

> 注意：也许你试图使用 `git reset --hard HEAD~ README.md` 命令让 reset 同时覆盖工作目录，但 Git 会告诉你这是徒劳（此时 reset 不允许使用 --soft 或 --hard 选项）。
{: .prompt-tip }

这样看来，在恢复文件方面，reset 命令要比 checkout 命令更安全一些。

#### 恢复快照

reset 命令是用来“回到过去”的，根据选项的不同，reset 命令将移动 HEAD 指针（--soft） -> 覆盖暂存区域（--mixed，默认）-> 覆盖工作目录（--hard）。

checkout 命令虽说是用于切换分支，但前面你也看到了，它事实上也是通过移动 HEAD 指针和覆盖暂存区域、工作目录来实现的。

那问题来了：它们有什么区别呢？

下面开始划**重点**：

第一个区别是，对于 `reset --hard` 命令来说，checkout 命令更安全。因为 checkout 命令在切换分支前会先检查一下当前的工作状态，如果不是"clean"的话，Git 不会允许你这样做；而 `reset --hard` 命令则是直接覆盖所有数据。

另一个区别是如何更新 HEAD 指向，reset 命令会移动 HEAD 所在分支的指向，而 checkout 命令只会移动 HEAD 自身来指向另一个分支。

### 强制三方合并
通常情况下，Git 会尽可能地尝试使用 Fast-forward 方式来合并分支，因为这样效率非常高，只有在万不得已的时候才使用三方合并（Three-way merge）

但是，有时候 Fast-forward 的方式却很容易让我们忽略了分支的存在！

举个例子，比如feature分支是master分支的一部分时，如果我们把feature分支删除，就会直接丢掉分支的信息，根本就不知道有个分支存在过……

可有时候我们确实希望保留分支的信息，应该怎么办呢？

只需要在 merge 的时候使用 `--no-ff` 选项即可告诉 Git 不要使用 Fast-forward 方式合并分支。这样 Git 就会乖乖地使用三方合并（Three-way merge）

这样就算删掉了feature分支，我们仍然可以很容易看出有个分支曾经存在过

****

本文参考

> 1. [Git使用教程](https://www.jianshu.com/p/e57a4a2cf077)  
