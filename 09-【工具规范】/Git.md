# Git基本概念

## 工作区、暂存区和版本库

我们先来理解下Git 工作区、暂存区和版本库概念

- **工作区：**就是你在电脑里能看到的目录。
- **暂存区：**英文叫stage, 或index。一般存放在 ".git目录下" 下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- **版本库：**工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

下面这个图展示了工作区、版本库中的暂存区和版本库之间的关系：

![image-20200417105936689](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200417105939-624285.png)  

图中左侧为工作区，右侧为版本库。在版本库中标记为 "index" 的区域是暂存区（stage, index），标记为 "master" 的是 master 分支所代表的目录树。

图中我们可以看出此时 "HEAD" 实际是指向 master 分支的一个"游标"。所以图示的命令中出现 HEAD 的地方可以用 master 来替换。

图中的 objects 标识的区域为 Git 的对象库，实际位于 ".git/objects" 目录下，里面包含了创建的各种对象及内容。

当对工作区修改（或新增）的文件执行 "git add" 命令时，暂存区的目录树被更新，同时工作区修改（或新增）的文件内容被写入到对象库中的一个新的对象中，而该对象的ID被记录在暂存区的文件索引中。

当执行提交操作（git commit）时，暂存区的目录树写到版本库（对象库）中，master 分支会做相应的更新。即 master 指向的目录树就是提交时暂存区的目录树。

当执行 "git reset HEAD" 命令时，暂存区的目录树会被重写，被 master 分支指向的目录树所替换，但是工作区不受影响。

当执行 "git rm --cached <file>" 命令时，会直接从暂存区删除文件，工作区则不做出改变。

当执行 "git checkout ." 或者 "git checkout -- <file>" 命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区的改动。

当执行 "git checkout HEAD ." 或者 "git checkout HEAD <file>" 命令时，会用 HEAD 指向的 master 分支中的全部或者部分文件替换暂存区和以及工作区中的文件。这个命令也是极具危险性的，因为不但会清除工作区中未提交的改动，也会清除暂存区中未提交的改动。

# Git 创建仓库

## git init

Git 使用 **git init** 命令来初始化一个 Git 仓库，Git 的很多命令都需要在 Git 的仓库中运行，所以 **git init** 是使用 Git 的第一个命令。

在执行完成 **git init** 命令后，Git 仓库会生成一个 .git 目录，该目录包含了资源的所有元数据，其他的项目目录保持不变（不像 SVN 会在每个子目录生成 .svn 目录，Git 只在仓库的根目录生成 .git 目录）。

## 使用方法

使用当前目录作为Git仓库，我们只需使它初始化。

```bash
git init
```

该命令执行完后会在当前目录生成一个 .git 目录。

使用我们指定目录作为Git仓库。

```bash
git init newrepo
```

初始化后，会在 newrepo 目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中。

如果当前目录下有几个文件想要纳入版本控制，需要先用 git add 命令告诉 Git 开始对这些文件进行跟踪，然后提交：

```bash
$ git add *.c
$ git add README
$ git commit -m '初始化项目版本'
```

以上命令将目录下以 .c 结尾及 README 文件提交到仓库中。git clone

我们使用 **git clone** 从现有 Git 仓库中拷贝项目（类似 **svn checkout**）。

克隆仓库的命令格式为：

```bash
git clone <repo>
```

如果我们需要克隆到指定的目录，可以使用以下命令格式：

```bash
git clone <repo> <directory>
```

**参数说明：**

- **repo:**Git 仓库。
- **directory:**本地目录。

比如，要克隆 Ruby 语言的 Git 代码仓库 Grit，可以用下面的命令：

```bash
$ git clone git://github.com/schacon/grit.git
```

执行该命令后，会在当前目录下创建一个名为grit的目录，其中包含一个 .git 的目录，用于保存下载下来的所有版本记录。

如果要自己定义要新建的项目目录名称，可以在上面的命令末尾指定新的名字：

```bash
$ git clone git://github.com/schacon/grit.git mygrit
```

# Git 基本操作

Git 的工作就是创建和保存你项目的快照及与之后的快照进行对比。本章将对有关创建与提交你的项目快照的命令作介绍。

## 获取与创建项目命令

### git init

用 git init 在目录中创建新的 Git 仓库。 你可以在任何时候、任何目录中这么做，完全是本地化的。

在目录中执行 git init，就可以创建一个 Git 仓库了。比如我们创建 runoob 项目：

```bash
$ mkdir runoob
$ cd runoob/
$ git init
Initialized empty Git repository in /Users/tianqixin/www/runoob/.git/
# 在 /www/runoob/.git/ 目录初始化空 Git 仓库完毕。
```

现在你可以看到在你的项目中生成了 .git 这个子目录。 这就是你的 Git 仓库了，所有有关你的此项目的快照数据都存放在这里。

```bash
ls -a
.    ..    .git
```

### git clone

使用 git clone 拷贝一个 Git 仓库到本地，让自己能够查看该项目，或者进行修改。

如果你需要与他人合作一个项目，或者想要复制一个项目，看看代码，你就可以克隆那个项目。 执行命令：

```bash
 git clone [url]
```

[url] 为你想要复制的项目，就可以了。

例如我们克隆 Github 上的项目：

```bash
$ git clone git@github.com:schacon/simplegit.git
Cloning into 'simplegit'...
remote: Counting objects: 13, done.
remote: Total 13 (delta 0), reused 0 (delta 0), pack-reused 13
Receiving objects: 100% (13/13), done.
Resolving deltas: 100% (2/2), done.
Checking connectivity... done.
```

克隆完成后，在当前目录下会生成一个 simplegit 目录：

```bash
$ cd simplegit/
$ ls
README   Rakefile lib
```

上述操作将复制该项目的全部记录。

```bash
$ ls -a
.        ..       .git     README   Rakefile lib
$ cd .git
$ ls
HEAD        description info        packed-refs
branches    hooks       logs        refs
config      index       objects
```

默认情况下，Git 会按照你提供的 URL 所指示的项目的名称创建你的本地项目目录。 通常就是该 URL 最后一个 / 之后的项目名称。如果你想要一个不一样的名字， 你可以在该命令后加上你想要的名称。

## 基本快照

Git 的工作就是创建和保存你的项目的快照及与之后的快照进行对比。本章将对有关创建与提交你的项目的快照的命令作介绍。

### git add

git add 命令可将该文件添加到缓存，如我们添加以下两个文件：

```bash
$ touch README
$ touch hello.php
$ ls
README        hello.php
$ git status -s
?? README
?? hello.php
$ 
```

git status 命令用于查看项目的当前状态。

接下来我们执行 git add 命令来添加文件：

```bash
$ git add README hello.php 
```

现在我们再执行 git status，就可以看到这两个文件已经加上去了。

```bash
$ git status -s
A  README
A  hello.php
$ 
```

新项目中，添加所有文件很普遍，我们可以使用 **git add .** 命令来添加当前项目的所有文件。

现在我们修改 README 文件：

```bash
$ vim README
```

在 README 添加以下内容：**# Runoob Git 测试**，然后保存退出。

再执行一下 git status：

```bash
$ git status -s
AM README
A  hello.php
```

"AM" 状态的意思是，这个文件在我们将它添加到缓存之后又有改动。改动后我们再执行 **git add** 命令将其添加到缓存中：

```bash
$ git add .
$ git status -s
A  README
A  hello.php
```

当你要将你的修改包含在即将提交的快照里的时候，需要执行 git add。

### git status

git status 以查看在你上次提交之后是否有修改。

我演示该命令的时候加了 -s 参数，以获得简短的结果输出。如果没加该参数会详细输出内容：

```bash
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

    new file:   README
    new file:   hello.php
```

### git diff

执行 git diff 来查看执行 git status 的结果的详细信息。

git diff 命令显示已写入缓存与已修改但尚未写入缓存的改动的区别。git diff 有两个主要的应用场景。

- 尚未缓存的改动：**git diff**
- 查看已缓存的改动： **git diff --cached**
- 查看已缓存的与未缓存的所有改动：**git diff HEAD**
- 显示摘要而非整个 diff：**git diff --stat**

在 hello.php 文件中输入以下内容：

```bash
<?php
echo '菜鸟教程：www.runoob.com';
?>
$ git status -s
A  README
AM hello.php
$ git diff
diff --git a/hello.php b/hello.php
index e69de29..69b5711 100644
--- a/hello.php
+++ b/hello.php
@@ -0,0 +1,3 @@
+<?php
+echo '菜鸟教程：www.runoob.com';
+?>
```

git status 显示你上次提交更新后的更改或者写入缓存的改动， 而 git diff 一行一行地显示这些改动具体是啥。

接下来我们来查看下 git diff --cached 的执行效果：

```bash
$ git add hello.php 
$ git status -s
A  README
A  hello.php
$ git diff --cached
diff --git a/README b/README
new file mode 100644
index 0000000..8f87495
--- /dev/null
+++ b/README
@@ -0,0 +1 @@
+# Runoob Git 测试
diff --git a/hello.php b/hello.php
new file mode 100644
index 0000000..69b5711
--- /dev/null
+++ b/hello.php
@@ -0,0 +1,3 @@
+<?php
+echo '菜鸟教程：www.runoob.com';
+?>
```

### git commit

使用 git add 命令将想要快照的内容写入缓存区， 而执行 git commit 将缓存区内容添加到仓库中。

Git 为你的每一个提交都记录你的名字与电子邮箱地址，所以第一步需要配置用户名和邮箱地址。

```bash
$ git config --global user.name 'runoob'
$ git config --global user.email test@runoob.com
```

接下来我们写入缓存，并提交对 hello.php 的所有改动。在首个例子中，我们使用 -m 选项以在命令行中提供提交注释。

```bash
$ git add hello.php
$ git status -s
A  README
A  hello.php
$ git commit -m '第一次版本提交'
[master (root-commit) d32cf1f] 第一次版本提交
 2 files changed, 4 insertions(+)
 create mode 100644 README
 create mode 100644 hello.php
 
```

现在我们已经记录了快照。如果我们再执行 git status:

```bash
$ git status
# On branch master
nothing to commit (working directory clean)
```

以上输出说明我们在最近一次提交之后，没有做任何改动，是一个"working directory clean：干净的工作目录"。

如果你没有设置 -m 选项，Git 会尝试为你打开一个编辑器以填写提交信息。 如果 Git 在你对它的配置中找不到相关信息，默认会打开 vim。屏幕会像这样：

```bash
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
# modified:   hello.php
#
~
~
".git/COMMIT_EDITMSG" 9L, 257C
```

如果你觉得 git add 提交缓存的流程太过繁琐，Git 也允许你用 -a 选项跳过这一步。命令格式如下：

```bash
git commit -a
```

我们先修改 hello.php 文件为以下内容：

```bash
<?php
echo '菜鸟教程：www.runoob.com';
echo '菜鸟教程：www.runoob.com';
?>
```

再执行以下命令：

```bash
git commit -am '修改 hello.php 文件'
[master 71ee2cb] 修改 hello.php 文件
 1 file changed, 1 insertion(+)
```

### git reset HEAD

git reset HEAD 命令用于取消已缓存的内容。

我们先改动文件 README 文件，内容如下：

```bash
# Runoob Git 测试
# 菜鸟教程 
```

hello.php 文件修改为：

```bash
<?php
echo '菜鸟教程：www.runoob.com';
echo '菜鸟教程：www.runoob.com';
echo '菜鸟教程：www.runoob.com';
?>
```

现在两个文件修改后，都提交到了缓存区，我们现在要取消其中一个的缓存，操作如下：

```bash
$ git status -s
 M README
 M hello.php
$ git add .
$ git status -s
M  README
M  hello.php
$ git reset HEAD hello.php 
Unstaged changes after reset:
M    hello.php
$ git status -s
M  README
 M hello.php
```

现在你执行 git commit，只会将 README 文件的改动提交，而 hello.php 是没有的。

```bash
$ git commit -m '修改'
[master f50cfda] 修改
 1 file changed, 1 insertion(+)
$ git status -s
 M hello.php
```

可以看到 hello.php 文件的修改并未提交。

这时我们可以使用以下命令将 hello.php 的修改提交：

```bash
$ git commit -am '修改 hello.php 文件'
[master 760f74d] 修改 hello.php 文件
 1 file changed, 1 insertion(+)
$ git status
On branch master
nothing to commit, working directory clean
```

简而言之，执行 git reset HEAD 以取消之前 git add 添加，但不希望包含在下一提交快照中的缓存。

### git rm

如果只是简单地从工作目录中手工删除文件，运行 **git status** 时就会在 **Changes not staged for commit** 的提示。

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除，然后提交。可以用以下命令完成此项工作

```bash
git rm <file>
```

如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 **-f**

```bash
git rm -f <file>
```

如果把文件从暂存区域移除，但仍然希望保留在当前工作目录中，换句话说，仅是从跟踪清单中删除，使用 **--cached** 选项即可

```bash
git rm --cached <file>
```

如我们删除 hello.php文件：

```bash
$ git rm hello.php 
rm 'hello.php'
$ ls
README
```

不从工作区中删除文件：

```bash
$ git rm --cached README 
rm 'README'
$ ls
README
```

可以递归删除，即如果后面跟的是一个目录做为参数，则会递归删除整个目录中的所有子目录和文件：

```bash
git rm –r * 
```

进入某个目录中，执行此语句，会删除该目录下的所有文件和子目录。

### git mv

git mv 命令用于移动或重命名一个文件、目录、软连接。

我们先把刚移除的 README 添加回来：

```bash
$ git add README 
```

然后对其重名:

```bash
$ git mv README  README.md
$ ls
README.md
```

# Git 分支管理

几乎每一种版本控制系统都以某种形式支持分支。使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。

有人把 Git 的分支模型称为**必杀技特性**，而正是因为它，将 **Git** 从版本控制系统家族里区分出来。

创建分支命令：

```bash
git branch (branchname)
```

切换分支命令:

```bash
git checkout (branchname)
```

当你切换分支的时候，Git 会用该分支的最后提交的快照替换你的工作目录的内容， 所以多个分支不需要多个目录。

合并分支命令:

```bash
git merge 
```

你可以多次合并到统一分支， 也可以选择在合并之后直接删除被并入的分支。

开始前我们先创建一个测试目录：

```bash
$ mkdir gitdemo
$ cd gitdemo/
$ git init
Initialized empty Git repository...
$ touch README
$ git add README
$ git commit -m '第一次版本提交'
[master (root-commit) 3b58100] 第一次版本提交
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README
```

## 列出分支

列出分支基本命令：

```bash
git branch
```

没有参数时，**git branch** 会列出你在本地的分支。

```bash
$ git branch
* master
```

此例的意思就是，我们有一个叫做 **master** 的分支，并且该分支是当前分支。

当你执行 **git init** 的时候，默认情况下 Git 就会为你创建 **master** 分支。

如果我们要手动创建一个分支。执行 **git branch (branchname)** 即可。

```bash
$ git branch testing
$ git branch
* master
  testing
```

现在我们可以看到，有了一个新分支 **testing**。

当你以此方式在上次提交更新之后创建了新分支，如果后来又有更新提交， 然后又切换到了 **testing** 分支，Git 将还原你的工作目录到你创建分支时候的样子。

接下来我们将演示如何切换分支，我们用 git checkout (branch) 切换到我们要修改的分支。

```bash
$ ls
README
$ echo 'runoob.com' > test.txt
$ git add .
$ git commit -m 'add test.txt'
[master 3e92c19] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
$ ls
README        test.txt
$ git checkout testing
Switched to branch 'testing'
$ ls
README
```

当我们切换到 **testing** 分支的时候，我们添加的新文件 test.txt 被移除了。切换回 **master** 分支的时候，它们有重新出现了。

```bash
$ git checkout master
Switched to branch 'master'
$ ls
README        test.txt
```

我们也可以使用 git checkout -b (branchname) 命令来创建新分支并立即切换到该分支下，从而在该分支中操作。

```bash
$ git checkout -b newtest
Switched to a new branch 'newtest'
$ git rm test.txt 
rm 'test.txt'
$ ls
README
$ touch runoob.php
$ git add .
$ git commit -am 'removed test.txt、add runoob.php'
[newtest c1501a2] removed test.txt、add runoob.php
 2 files changed, 1 deletion(-)
 create mode 100644 runoob.php
 delete mode 100644 test.txt
$ ls
README        runoob.php
$ git checkout master
Switched to branch 'master'
$ ls
README        test.txt
```

如你所见，我们创建了一个分支，在该分支的上移除了一些文件 test.txt，并添加了 runoob.php 文件，然后切换回我们的主分支，删除的 test.txt 文件又回来了，且新增加的 runoob.php 不存在主分支中。

使用分支将工作切分开来，从而让我们能够在不同开发环境中做事，并来回切换。

## 删除分支

删除分支命令：

```bash
git branch -d (branchname)
```

例如我们要删除 testing 分支：

```bash
$ git branch
* master
  testing
$ git branch -d testing
Deleted branch testing (was 85fc7e7).
$ git branch
* master
```

## 分支合并

一旦某分支有了独立内容，你终究会希望将它合并回到你的主分支。 你可以使用以下命令将任何分支合并到当前分支中去：

```bash
git merge
$ git branch
* master
  newtest
$ ls
README        test.txt
$ git merge newtest
Updating 3e92c19..c1501a2
Fast-forward
 runoob.php | 0
 test.txt   | 1 -
 2 files changed, 1 deletion(-)
 create mode 100644 runoob.php
 delete mode 100644 test.txt
$ ls
README        runoob.php
```

以上实例中我们将 newtest 分支合并到主分支去，test.txt 文件被删除。

合并完后就可以删除分支:

```bash
$ git branch -d newtest
Deleted branch newtest (was c1501a2).
```

删除后， 就只剩下 master 分支了：

```bash
$ git branch
* master
```

## 合并冲突

合并并不仅仅是简单的文件添加、移除的操作，Git 也会合并修改。

```bash
$ git branch
* master
$ cat runoob.php
```

首先，我们创建一个叫做 change_site 的分支，切换过去，我们将 runoob.php 内容改为:

```bash
<?php
echo 'runoob';
?>
```

创建 change_site 分支：

```bash
$ git checkout -b change_site
Switched to a new branch 'change_site'
$ vim runoob.php
$ head -3 runoob.php
<?php
echo 'runoob';
?>
$ git commit -am 'changed the runoob.php'
[change_site 7774248] changed the runoob.php
 1 file changed, 3 insertions(+)
 
```

将修改的内容提交到 change_site 分支中。 现在，假如切换回 master 分支我们可以看内容恢复到我们修改前的(空文件，没有代码)，我们再次修改 runoob.php 文件。

```bash
$ git checkout master
Switched to branch 'master'
$ cat runoob.php
$ vim runoob.php    # 修改内容如下
$ cat runoob.php
<?php
echo 1;
?>
$ git diff
diff --git a/runoob.php b/runoob.php
index e69de29..ac60739 100644
--- a/runoob.php
+++ b/runoob.php
@@ -0,0 +1,3 @@
+<?php
+echo 1;
+?>
$ git commit -am '修改代码'
[master c68142b] 修改代码
 1 file changed, 3 insertions(+)
```

现在这些改变已经记录到我的 "master" 分支了。接下来我们将 "change_site" 分支合并过来。

```bash
$ git merge change_site
Auto-merging runoob.php
CONFLICT (content): Merge conflict in runoob.php
Automatic merge failed; fix conflicts and then commit the result.

$ cat runoob.php     # 代开文件，看到冲突内容
<?php
<<<<<<< HEAD
echo 1;
=======
echo 'runoob';
>>>>>>> change_site
?>
```

我们将前一个分支合并到 master 分支，一个合并冲突就出现了，接下来我们需要手动去修改它。

```bash
$ vim runoob.php 
$ cat runoob.php
<?php
echo 1;
echo 'runoob';
?>
$ git diff
diff --cc runoob.php
index ac60739,b63d7d7..0000000
--- a/runoob.php
+++ b/runoob.php
@@@ -1,3 -1,3 +1,4 @@@
  <?php
 +echo 1;
+ echo 'runoob';
  ?>
```

在 Git 中，我们可以用 git add 要告诉 Git 文件冲突已经解决

```bash
$ git status -s
UU runoob.php
$ git add runoob.php
$ git status -s
M  runoob.php
$ git commit
[master 88afe0e] Merge branch 'change_site'
```

现在我们成功解决了合并中的冲突，并提交了结果。

# Git 查看提交历史

在使用 Git 提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，我们可以使用 git log 命令查看。

针对我们前一章节的操作，使用 git log 命令列出历史提交记录如下：

```bash
$ git log
commit d5e9fc2c811e0ca2b2d28506ef7dc14171a207d9 (HEAD -> master)
Merge: c68142b 7774248
Author: runoob <test@runoob.com>
Date:   Fri May 3 15:55:58 2019 +0800

    Merge branch 'change_site'

commit c68142b562c260c3071754623b08e2657b4c6d5b
Author: runoob <test@runoob.com>
Date:   Fri May 3 15:52:12 2019 +0800

    修改代码

commit 777424832e714cf65d3be79b50a4717aea51ab69 (change_site)
Author: runoob <test@runoob.com>
Date:   Fri May 3 15:49:26 2019 +0800

    changed the runoob.php

commit c1501a244676ff55e7cccac1ecac0e18cbf6cb00
Author: runoob <test@runoob.com>
Date:   Fri May 3 15:35:32 2019 +0800
```

我们可以用 --oneline 选项来查看历史记录的简洁的版本。

```bash
$ git log --oneline
$ git log --oneline
d5e9fc2 (HEAD -> master) Merge branch 'change_site'
c68142b 修改代码
7774248 (change_site) changed the runoob.php
c1501a2 removed test.txt、add runoob.php
3e92c19 add test.txt
3b58100 第一次版本提交
```

这告诉我们的是，此项目的开发历史。

我们还可以用 --graph 选项，查看历史中什么时候出现了分支、合并。以下为相同的命令，开启了拓扑图选项：

```bash
*   d5e9fc2 (HEAD -> master) Merge branch 'change_site'
|\  
| * 7774248 (change_site) changed the runoob.php
* | c68142b 修改代码
|/  
* c1501a2 removed test.txt、add runoob.php
* 3e92c19 add test.txt
* 3b58100 第一次版本提交
```

现在我们可以更清楚明了地看到何时工作分叉、又何时归并。

你也可以用 **--reverse** 参数来逆向显示所有日志。

```bash
$ git log --reverse --oneline
3b58100 第一次版本提交
3e92c19 add test.txt
c1501a2 removed test.txt、add runoob.php
7774248 (change_site) changed the runoob.php
c68142b 修改代码
d5e9fc2 (HEAD -> master) Merge branch 'change_site'
```

如果只想查找指定用户的提交日志可以使用命令：git log --author , 例如，比方说我们要找 Git 源码中 Linus 提交的部分：

```bash
$ git log --author=Linus --oneline -5
81b50f3 Move 'builtin-*' into a 'builtin/' subdirectory
3bb7256 make "index-pack" a built-in
377d027 make "git pack-redundant" a built-in
b532581 make "git unpack-file" a built-in
112dd51 make "mktag" a built-in
```

如果你要指定日期，可以执行几个选项：--since 和 --before，但是你也可以用 --until 和 --after。

例如，如果我要看 Git 项目中三周前且在四月十八日之后的所有提交，我可以执行这个（我还用了 --no-merges 选项以隐藏合并提交）：

```bash
$ git log --oneline --before={3.weeks.ago} --after={2010-04-18} --no-merges
5469e2d Git 1.7.1-rc2
d43427d Documentation/remote-helpers: Fix typos and improve language
272a36b Fixup: Second argument may be any arbitrary string
b6c8d2d Documentation/remote-helpers: Add invocation section
5ce4f4e Documentation/urls: Rewrite to accomodate transport::address
00b84e9 Documentation/remote-helpers: Rewrite description
03aa87e Documentation: Describe other situations where -z affects git diff
77bc694 rebase-interactive: silence warning when no commits rewritten
636db2c t3301: add tests to use --format="%N"
```

# Git 标签

如果你达到一个重要的阶段，并希望永远记住那个特别的提交快照，你可以使用 git tag 给它打上标签。

比如说，我们想为我们的 runoob 项目发布一个"1.0"版本。 我们可以用 git tag -a v1.0 命令给最新一次提交打上（HEAD）"v1.0"的标签。

-a 选项意为"创建一个带注解的标签"。 不用 -a 选项也可以执行的，但它不会记录这标签是啥时候打的，谁打的，也不会让你添加个标签的注解。 我推荐一直创建带注解的标签。

```bash
$ git tag -a v1.0 
```

当你执行 git tag -a 命令时，Git 会打开你的编辑器，让你写一句标签注解，就像你给提交写注解一样。

现在，注意当我们执行 git log --decorate 时，我们可以看到我们的标签了：

```bash
*   d5e9fc2 (HEAD -> master) Merge branch 'change_site'
|\  
| * 7774248 (change_site) changed the runoob.php
* | c68142b 修改代码
|/  
* c1501a2 removed test.txt、add runoob.php
* 3e92c19 add test.txt
* 3b58100 第一次版本提交
```

如果我们忘了给某个提交打标签，又将它发布了，我们可以给它追加标签。

例如，假设我们发布了提交 85fc7e7(上面实例最后一行)，但是那时候忘了给它打标签。 我们现在也可以：

```bash
$ git tag -a v0.9 85fc7e7
$ git log --oneline --decorate --graph
*   d5e9fc2 (HEAD -> master) Merge branch 'change_site'
|\  
| * 7774248 (change_site) changed the runoob.php
* | c68142b 修改代码
|/  
* c1501a2 removed test.txt、add runoob.php
* 3e92c19 add test.txt
* 3b58100 (tag: v0.9) 第一次版本提交
```

如果我们要查看所有标签可以使用以下命令：

```bash
$ git tag
v0.9
v1.0
```

指定标签信息命令：

```bash
git tag -a <tagname> -m "runoob.com标签"
```

PGP签名标签命令：

```bash
git tag -s <tagname> -m "runoob.com标签"
```

# Git 多用户配置

## 引言

一般来说，安装好 git 后，我们都会配置一个全局的 config 信息，就像这样：

```bash
git config --global user.name "githubname" // 配置全局用户名，如 Github 上注册的用户名
git config --global user.email "githubemail" // 配置全局邮箱，如 Github 上配置的邮箱
```

但是你可能会碰到需要在一台电脑上配置多个用户信息的需求。此时就不能够用一个全局配置搞定一切了。

比如因为我的个人电脑出了问题，我想要提交我的个人项目时，只能用公司配的电脑去提交。而公司的电脑配置的是私有的 gitlab 仓库，而我自己的项目存储在 github 上。这两个仓库不仅仓库地址不一样，仓库的用户名和邮箱都不一样。

## 配置多用户

本文将配置分别是 github 以及 gitlab 上的两个用户，并分别在它们所属的项目上进行 git 操作，这差不多就是配置多用户的大部分操作了。

|        | GITHUB      | GITLAB      |
| ------ | ----------- | ----------- |
| 用户名 | githubname  | gitlabname  |
| 邮箱   | githubemail | gitlabemail |

### 清除全局配置

在正式配置之前，我们先得把全局配置给清除掉（如果你配置过的话），执行命令：

```bash
git config --global --list
```

这会列出所有已经配置的全局配置，如果你发现其中有 user.name 和 user.email 信息，请执行以下命令将其清除掉：

```bash
git config --global --unset user.name
git config --global --unset user.email
```

### 生成钥对

钥对的保存位置默认在 ~/.ssh 目录下，我们先清理下这个目录中已存在的钥对信息，即删除其中的 id_rsa、id_rsa.pub 之类的公钥和密钥文件。

首先我们开始生成 github 上的仓库钥对，通过 -C 参数填写 github 的邮箱：

```bash
ssh-keygen -t rsa -C “githubemail”
```

按下 ENTER 键后，会有如下提示：

```bash
Generatingpublic/privatersa key pair.Enter fileinwhich to save the key (/Users/jitwxs/.ssh/id_rsa):
```

在这里输入公钥的名字，默认情况是叫 id_rsa，为了和后面的 gitlab 配置区分，这里输入 id_rsa_github。输入完毕后，一路回车，钥对就生成完毕了。

下面开始生成 gitlab 上的仓库钥对，步骤和上面一样：

```bash
ssh-keygen -t rsa -C “gitlabemail”
```

生成的公钥名就叫做：id_rsa_gitlab。

### 添加 SSH Keys

我相信你既然都看到这篇文章了，你一定掌握了如何将公钥添加到 SSH Keys 中。请将 id_rsa_github.pub 和 id_rsa_gitlab.pub 内容分别添加到 github 和 gitlab 的 SSH Keys 中，这里就不啰嗦了。 

### 添加私钥

在上一步中，我们已经将公钥添加到了 github 或者 gitlab 服务器上，我们还需要将私钥添加到本地中，不然无法使用。切换的时候需要使用ssh-add ~/.ssh/id_rsa_name这个命令，可能出现Could not open a connection to your authentication agent.

这时可以使用：ssh-agent bash 命令，然后再次使用ssh-add ~/.ssh/id_rsa_name这个命令就没问题了。添加命令也十分简单，如下：

```bash
ssh-agent bash
ssh-add ~/.ssh/id_rsa_github // 将 GitHub 私钥添加到本地
ssh-add ~/.ssh/id_rsa_gitlab // 将 GitLab 私钥添加到本地
```

添加完毕后，可以通过执行 ssh-add -l 验证下，如果都能显示出来和下面一样，就 OK 了。

```bash
ssh-add -l
```

### 管理密钥

通过以上步骤，公钥、密钥分别被添加到 git 服务器和本地了。下面我们需要在本地创建一个密钥配置文件，通过该文件，实现根据仓库的 remote 链接地址自动选择合适的私钥。

编辑 ~/.ssh 目录下的 config 文件，如果没有，请创建。

```bash
vim ~/.ssh/config
```

配置内容如下：

```bash
Host github
HostName github.com
User githubname
IdentityFile ~/.ssh/id_rsa_github

Host gitlab
HostName gitlab.mygitlab.com
User gitlabname
IdentityFile ~/.ssh/id_rsa_gitlab
```


该文件分为多个用户配置，每个用户配置包含以下几个配置项：

> Host：仓库网站的别名，随意取
> HostName：仓库网站的域名（PS：IP 地址应该也可以）
> User：仓库网站上的用户名
> IdentityFile：私钥的绝对路径
> 注： Host 就是可以替代 HostName 来使用的别名

可以用 ssh -T 命令检测下配置的 Host 是否是连通的：

```bash
ssh -T git@github
```

当然不用 Host 用 HostName 也是一样的：

```bash
ssh -T git@github.com
ssh -T git@gitlab.mygitlab.com
```

### 仓库配置

恭喜你！完成以上配置后，其实你已经基本完成了所有配置。分别进入附属于 github 和 gitlab 的仓库，此时都可以进行 git 操作了。但是别急，如果你此时提交仓库修改后，你会发现提交的用户名变成了你的系统主机名。

这是因为 git 的配置分为三级别，System —> Global —>Local。System 即系统级别，Global 为配置的全局，Local 为仓库级别，优先级是 Local > Global > System。

因为我们并没有给仓库配置用户名，又在一开始清除了全局的用户名，因此此时你提交的话，就会使用 System 级别的用户名，也就是你的系统主机名了。

因此我们需要为每个仓库单独配置用户名信息，假设我们要配置 github 的某个仓库，进入该仓库后，执行：

```bash
git config --local user.name "githubname"
git config --local user.email "githubemail"
```
执行完毕后，通过以下命令查看本仓库的所有配置信息：
```bash
git config --local --list
```

至此你已经配置好了 Local 级别的配置了，此时提交该仓库的代码，提交用户名就是你设置的 Local 级别的用户名了。

# SVN迁移GitLab

```bash
# 获取到SVN使用的作者名字列表
svn log https://10.0.0.24/svn/DevelopmentVersions/ZLERP/提交测试/自动部署/ERP_Test -q | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2">"}' | sort -u > users.txt
```

```bash
# SVN仓库转变成Git仓库:-r30000从哪个版本号开始
git svn clone -r 30000:HEAD https://10.0.0.24/svn/DevelopmentVersions/ZLERP/提交测试/自动部署/ERP_Test --trunk="Trunk" --tags="Tag" --branches="Branch" --authors-file=./users.txt --no-metadata
```

```shell
# 将标签变为合适的 Git 标签
cp -Rf .git/refs/remotes/origin/tags/* .git/refs/tags/
rm -Rf .git/refs/remotes/origin/tags
# 剩余的引用移动为本地分支
cp -Rf .git/refs/remotes/origin/* .git/refs/heads/
rm -Rf .git/refs/remotes
```

```bash
# 关联远程仓库，提交Branch和Tag
git remote add origin http://gitlab01.womaiapp.com/xuli/Test.git
git push origin --all
git push --tags 
```

# Merge/Rebase

假设最开始只有一个origin分支
 然后你在本地新建了一个分支来开发自己的工作叫mywork,然后我们做一些修改添加，并提交两个commit

```ruby
$ git checkout -b mywork origin
$ vi file.java
$ git add file.java
$ git commit 
$ vi otherfile.java
$ git add otherfile.java
$ git commit 
```

![image-20200415165000774](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415165003-196811.png) 

这时有人在origin中做了一些修改并提交，推送到origin了

![image-20200415165020564](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415165021-67720.png) 

这时你可以"pull"命令把"origin"分支上的修改拉下来并且和你的修改合并； 结果看起来就像一个新的"合并的提交"，合并时可以用rebase和merge
***git merage\***

![image-20200415165044520](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415165103-177983.png)

```undefined
git checkout origin
git pull
git checkout mywork
git merge origin
```

这时Git 会用两个分支的末端（C4 和 C6）以及它们的共同祖先（C2）进行一次简单的三方合并计算
 对三方合并后的结果重新做一个新的快照，并自动创建一个指向它的提交对象（C7），此时mywork快进到c7也指向c7。这就是merge他会保留每一次提交的历史。
 ***git rebase\***
 但是，如果你想让"mywork"分支历史看起来像没有经过任何合并一样，你也许可以用 git rebase:

```cpp
git checkout mywork
git rebase origin
// git rebase --onto master server client
```

上面的命令会把你的"mywork"分支里的每个提交(commit)取消掉，并且把它们临时保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把"mywork"分支更新到最新的"origin"分支，最后把保存的这些补丁应用到"mywork"分支上生成c5' 和 c6‘

![image-20200415165139744](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200415165140-432684.png) 

这时如果想把mywork合并到origin上，可以运行如下命令

```undefined
git checkout origin
git merge mywork 
```

# Git工作流的最佳实践

## 简介

今天来总结一下在实际项目开发中使用Git工作流的最佳实践，项目使用gitlab作为版本控制。比较适合小型或者中型的多人合作，快速迭代开发的项目。

## 分支

首先来介绍一下分支的设计。一般分为这么几类分支:

- master分支
- develop分支
- feature分支
- bug分支

master为主分支，为protected分支，只允许其他分支的merge，不允许直接push代码。

develop为开发的主分支，feature和bug分支提交merge request到develop。

在开发中，针对每个功能开一个feature分支进行开发，命名可以为 [feature/xxx]，开发完成后提交merge request，合并到develop分支。

针对开发中遇到的bug，新开一个bug分支，修复后，也提交merge request到develop分支。

## 开发流程

注意点：

- 规范的commit message格式
- 拉取develop分支代码的时候使用rebase而不是merge
- 使用rebase整理个人feature分支的commit
- 开merge request进行code review

首先应该制定一个commit的message的规范和格式，要求每次提交的commit message清晰。具体的格式可以根据团队的习惯制定，只需要能表述清晰即可。

对于个人的feature分支，在准备提交MR合并进develop之前，应该用git rebase来整理自己的commit，使得逻辑清楚。同时可以避免产生[merge branch develop ***]这样的commit。

具体操作如下：

```text
// 在feature/xxx分支上进行开发，并进行了一些提交，此时要合并到develop分支

// 首先用rebase的方式拉取develop分支代码
git pull origin develop --rebase

// 如果有conflict，修复conflict，然后
git rebase --continue

// 如果有多个conflict，不同于直接merge，使用rebase可能需要多次解决conflict，并多次执行git rebase --continue

// 如果有需要使用
git rebase -i
// 来整理自己的commit，可以用来将一些小的commit合并，以及修改优化自己的commit message

// 然后push到feature/xxx分支，由于rebase，可能会导致本地与远端的history不一致，使用force强制重写远端分支
git push --force

// 最后在gitlab上开一个merge request到develop分支

// 团队其他成员review这个MR，并提出修改意见，然后根据修改意见，优化原来的代码，把存在问题的地方改掉，知道最后没有问题，将feature/xxx分支合并到develop
```

## 部署

master分支永远是稳定的版本，在master分支上，可以使用CI工具进行持续集成和自动部署。develop分支每次完成部分功能就可以merge到master，使得项目可以快速高效的迭代开发。

## rebase or merge

在个人分支上，推荐只使用rebase而不是merge，因为rebase可以让你的提交看起来意义明确，并且没有无意义的merge其他分支的commit。

而在主分支上， 只允许merge操作，从而保证主分支的commit历史的完整性。在多人开发的主分支上进行rebase操作是一件很危险的行为，可能会导致其他成员的history与主分支不一致。

## 总结

个人认为，现在这样的git工作流可以很好的适应小团队的项目快速迭代开发，使得项目网络清晰明了。