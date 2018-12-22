[TOC]



## 1. 分布式 VS 集中式

#### 1.1 集中式版本控制系统
版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。

#### 1.2 分布式版本控制器
1. 分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。
2. 既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。
3. 和集中式版本控制系统相比，分布式版本控制系统的安全性要高很多，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，随便从其他人那里复制一个就可以了。而集中式版本控制系统的中央服务器要是出了问题，所有人都没法干活了。
4. 在实际使用分布式版本控制系统的时候，其实很少在两人之间的电脑上推送版本库的修改，因为可能你们俩不在一个局域网内，两台电脑互相访问不了，也可能今天你的同事病了，他的电脑压根没有开机。因此，分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。

## 2. 初识 git 
#### 2.1 安装git：

    $ homebrew install git 

#### 2.2 配置git：

    $ git config --global user.name "Your Name"
    $ git config --global user.email "email@example.com"
注意git config命令的--`global`参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

#### 2.3 创建版本库
什么是版本库呢？版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

1. 选择一个合适的地方，创建一个空目录：

        $ mkdir learngit
        $ cd learngit
        $ pwd
        /Users/michael/learngit
    pwd命令用于显示当前目录。在我的Mac上，这个仓库位于/Users/michael/learngit。

2. 通过git init命令把这个目录变成Git可以管理的仓库：

        $ git init
        Initialized empty Git repository in /Users/michael/learngit/.git/
    此时，git就把仓库建好了。

#### 2.4 .git的目录
上边创建的是一个空仓库，里边有个**隐藏的.git的目录，这个目录是Git来跟踪管理版本库**的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。`ls -ah`命令就可以看见

#### 2.5 把文件添加到版本库

1. 编写一个readme.txt文件，放到learngit目录下。内容如下：

>Git is a version control system.
>Git is free software.

2. 用命令git add告诉Git，把文件添加到仓库：
    ​     
        $ git add readme.txt

3. 用命令git commit告诉Git，把文件提交到仓库：

        $ git commit -m "wrote a readme file"
        [master (root-commit) cb926e7] wrote a readme file
        
        1 file changed, 2 insertions(+)  # 1个文件被改动（我们新添加的readme.txt文件），插入了两行内容（readme.txt有两行内容）
        
        create mode 100644 readme.txt

`git commit`命令，-m后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

为什么Git添加文件需要add，commit一共两步呢？
因为c**ommit可以一次提交很多文件，所以你可以多次add不同的文件**，比如：

    $ git add file1.txt
    $ git add file2.txt file3.txt
    $ git commit -m "add 3 files."

#### 2.6 小结  
现在总结一下今天学的两点内容：

- 初始化一个Git仓库，使用**`git init`**命令。
- 添加文件到Git仓库，分两步：
    - 第一步，使用命令**`git add <file>`**，注意，可反复多次使用，添加多个文件；
    - 第二步，使用命令**`git commit`**，完成。 



#### 2.7. 实际做了什么？

1. **工作区：**电脑里能看到的目录，比如我的learngit文件夹就是一个工作区

2. **版本库（Repository）.git：**git的版本库里存了很多东西：

    - 称为`stage`的暂存区
    - git自动创建的第一个分支`master`
    - 指向master的一个指针叫`HEAD`

3. git add把文件添加进去，实际上就是把文件修改添加到暂存区;
4. git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。
5. 需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。

6. add之前，文件有改动，有添加更改，查看状态如下：

        $ git status
        
        # On branch master
        # Changes not staged for commit:
        #   (use "git add <file>..." to update what will be committed)
        #   (use "git checkout -- <file>..." to discard changes in working directory)
        #
        #       modified:   readme.txt//修改文件
        #
        # Untracked files://没踪迹
        #   (use "git add <file>..." to include in what will be committed)
        #
        #       LICENSE
        no changes added to commit (use "git add" and/or "git commit -a")

7. add之后，没有commit：放入暂存区

        $ git status
        # On branch master
        # Changes to be committed:
        #   (use "git reset HEAD <file>..." to unstage)
        #
        #       new file:   LICENSE
        #       modified:   readme.txt

8. git commit 之后，
    ​     
        $ git status
        # On branch master
        # nothing to commit (working directory clean)

## 3. 常用命令列表
#### 3.1 状态
**`git status:`**  掌握仓库当前的状态

    $ git status  
    no changes added to commit (use "git add" and/or "git commit -a")
    # readme.txt被修改过了，但还没有准备提交的修改。

**`modified:`**   指出某些文件被修改
​    
    $ modified  readme.txt  //readme.txt被修改
    nothing to commit (working directory clean)  
    # 提交成功，没有新的修改内容


**`git diff:`**   difference,看具体修改了什么内容

**`git log:`**查看提交历史，以便确定要回退到哪个版本。 
​    
    git log --pretty=oneline//信息精简
**`git lg:`**彩色log

**`git reflog:`**查看命令历史，以便确定要回到未来的哪个版本。

**`git reset --hard commit_id: `**
HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。

**`git reset --hard HEAD^(类似于commit_id)`**
上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

**`git diff HEAD -- readme.txt：`**查看区别
命令可以查看工作区和版本库里面最新版本的区别：
​    
    DVS:learngit davisqiao$ git diff HEAD -- readme.txt
    diff --git a/readme.txt b/readme.txt
    index 29f151f..0c57509 100644
    --- a/readme.txt
    +++ b/readme.txt
    @@ -1,4 +1,4 @@
     Git is a distributed version control system.
     Git is free software.
     config git 
    -AAA
    \ No newline at end of file
    +ABC
    \ No newline at end of file


#### 3.2 撤销修改

**`git checkout -- file:`**撤销修改，add之前

    $ git checkout -- readme.txt     

你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。，即退回上一次add或者commit的版本，丢弃工作区。
**`git checkout .`**  本地所有修改的。没有的提交的，都返回到原来的状态

**`git reset HEAD file:`** 撤销修改，add之后
​    
    $ git reset HEAD readme.txt
当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步：

1. 用命令`git reset HEAD file`，就回到了场景1。
2. 按场景1操作。即清除暂存区，退回到上一次commit 的版本，再用`git checkout -- file`撤销工作区
3. 已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。



#### 3.3 删除文件
你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了：

    $ rm test.txt
    Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了：
    
    $ git status
    # On branch master
    # Changes not staged for commit:
    #   (use "git add/rm <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       deleted:    test.txt
    #
    no changes added to commit (use "git add" and/or "git commit -a")

删除文件后，确实是真的删除，则：**`git rm + git commit`**
​    
    $ git rm test.txt
    $ git commit -m "remove test.txt"

若发现删除错了，不想删除了，则：

版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
​    
    $ git checkout -- test.txt
    git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

#### 3.4 分支
查看分支：**`git branch`**
创建分支：**`git branch <name>`**
切换分支：**`git checkout <name>`**
创建+切换分支：**`git checkout -b <name>`**
合并某分支到当前分支：**`git merge <name>`**
删除分支：**`git branch -d <name>`**

删除远程分支：`git push --delete origin davis-pm420-dev`



#### 3.5 冲突
Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。
**`git log --graph`** 此命令可以看到分支合并图。

1. master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
2. 那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
3. 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
4. 合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。

#### 3.6 获取新的更新包 **`git fetch`**

一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到git fetch命令。

    $ git fetch <远程主机名>

上面命令将某个远程主机的更新，全部取回本地。

git fetch命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。

默认情况下，git fetch取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。


    $ git fetch <远程主机名> <分支名>

比如，取回origin主机的master分支。


    $ git fetch origin master

所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如origin主机的master，就要用origin/master读取。

git branch命令的-r选项，可以用来查看远程分支，-a选项查看所有分支。



    $ git branch -r
    origin/master
    
    $ git branch -a
    * master
      remotes/origin/master

上面命令表示，本地主机的当前分支是master，远程分支是origin/master。

取回远程主机的更新以后，可以在它的基础上，使用git checkout命令创建一个新的分支。


    $ git checkout -b newBrach origin/master

上面命令表示，在origin/master的基础上，创建一个新分支。

此外，也可以使用git merge命令或者git rebase命令，在本地分支上合并远程分支。


    $ git merge origin/master
    # 或者
    $ git rebase origin/master

上面命令表示在当前分支上，合并origin/master。

## 4. 工作提交位置

提交位置：

    git push origin HEAD:refs/for/master

若是第二次是第一次的补充，两合二为一

    git push origin HEAD:refs/changes/47


## 5. 已经提交的代码修改注释or代码

有些时候提交改动后,后悔刚刚的改动,比如提交描述写错了,代码有不合适的地方

这个时候就想到如何修改已经提交并且push到服务器的commit,  **能不能修改commit的代码 并且不建立新提交记录呢**

**操作过程:**

1. `git rebase -i master~1`   变基最后一次提交
2. 出来提示信息,修改 `pick` 为 `edit` ，并 :wq 保存退出(VIM命令修改..不多说)
3. 使用 `git commit --amend` 进行修改，完成后 :wq 退出 (这个是修改注释)
4. `修改XX文件`  (这个是修改文件)
5. `git add XX` 刚刚修改的文件
6. 使用 `git rebase --continue` 完成操作
7. :wq 保存退出
8. `git push -f 强制push当前master`
9. commit还是刚刚的commit但是 有些不合适的内容都变啦..不留痕迹




## 6. Git 部分提交

#### 场景

修改了四个文件，分别涉及到两个功能，需要把其中两个文件的修改当做一个功能的commit，另外两个文件的修改当做另一个功能的commit。
但如果要用git add 的命令，会让当前所有的修改都加入commit。怎么办呢？
**用git stash 命令**

#### 步骤

1. 用git add 命令添加第一个commit需要的文件

   ```shell
   git add file1
   git add file2
   ```

2. 隐藏其他修改，git stash 的参数中 -k 开关告诉仓库保持文件的完整  -u 开关告诉仓库包括无路径的文件(那些新的和未添加到git的)。

   这时git status就只能看到file1 和file2，并且当你切换到实际的文件目录，file3 和file4的修改也随之不见。

   ```shell
   git stash -k -u
   ```

3. 提交第一个commit

   ```shell
   git commit -m 'submit function1'
   ```

4. 恢复之前隐藏的修改,这时再git status，file3和file4的修改又回来。

   ```shell
   git stash pop
   ```


5. 提交第二个commit

   ```shell
   git commit -m 'submit function2'
   ```

   ​

## 7.删除本地所有的更改

`git checkout . && git clean -xdf`



## 8. git stash 保存修改现场

用途：当你正在分支上做一个项目的时候，突然必须停下来去做别的事情，但因为没有此项目还没改好，所以不想commit 就可以保留现场，等忙完后再回复现场继续修改。

bug处理保存开发现场
`$ git stash`	当前工作现场“储藏”起来，因此可以放心地创建分支来修复bug

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：$ git checkout master ​$ git checkout -b issue-101

修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

```shell
$ git checkout master 

$ git merge --no-ff -m "merged bug fix 101" issue-101 

$ git branch -d issue-101
```

 

**恢复：**

```shell
$ git stash apply    	 # 恢复后，stash内容并不删除需要用`git stash drop`来删除；

$ git stash pop  		# 恢复的同时把stash内容也删了
```

```shell
$ git stash list 		# 多次stash后，查看 存储的工作现场
$ git stash apply stash@{0} # 先用git stash list查看，然后恢复指定的stash
```

```shell
git branch -D name  # 强行删除 一个未被合并的分支
```





git commit --amend

git rebase  + 最近一次的CommitID（All —> Merged）



返回到某次特定版本的地方。

git reset + 特定某次的CommitID







#### Git:代码冲突常见解决方法



如果系统中有一些配置文件在服务器上做了配置修改,然后后续开发又新添加一些配置项的时候,

在发布这个配置文件的时候,会发生代码冲突:

error: Your local changes to the following files would be overwritten by merge:
​        protected/config/main.php
Please, commit your changes or stash them before you can merge.

如果希望保留生产服务器上所做的改动,仅仅并入新配置项, 处理方法如下:

```
git stash
git pull
git stash pop
```

然后可以使用git diff -w +文件名 来确认代码自动合并的情况.

反过来,如果希望用代码库中的文件完全覆盖本地工作版本. 方法如下:

```
git reset --hard
git pull
```

其中git reset是针对版本,如果想针对文件回退本地修改,使用

```
git checkout HEAD file/to/restore  
```



#### 新建分支

git checkout -b workrecord

git fetch

 git rebase origin/master





若有冲突：处理后，

 git add -A

git rebase --continue



#### 忽略本地文件

把文件加到 .git/info/exclude 中





- 现在的开发方式使用rebase也没有问题，rebase在本地完成之后正常push到远端即可，但是要记住，和别人公用的远程分支不应该做rebase操作

  



#### commit --amend 到远端

`commit --amend`也是可以push到远端的，但是push的时候需要加上 `git push --force-with-lease`



#### Git 合并多次commit

 [合并多个commit](https://segmentfault.com/a/1190000007748862)

1. `git log`

2. `git rebase -i 3a4226b` 

   指名要合并的版本之前的版本号  其中，`3a4226b` 不参与合并

3. 将第一个之外的`pick改为s`

4. 若有冲突，修改，然后调用

   `git add .`  

   `git rebase --continue`  

5. 若想放弃，输入 

   `git rebase --abort`

6. 完成后，强制推送到远程

   `git push --force`




#### 强制Pull远程代码到本地

放弃本地更改



```shell
git fetch --all  					#git fetch 只是下载远程的库的内容，不做任何的合并 
git reset --hard origin/branch 	 	# git reset 把HEAD指向刚刚下载的最新的版本
git pull
```







#### 命令：git stash 

1.使用git stash 保存当前的工作现场， 那么就可以切换到其他分支进行工作，或者在当前分支上完成其他紧急的工作，比如修订一个bug测试提交。 

2.如果一个使用了一个git stash，切换到一个分支，且在该分支上的工作未完成也需要保存它的工作现场。再使用git stash。那么stash 队列中 就有了两个工作现场。 

3.可以使用git stash list。查看stash队列。 

4.如果在一个分支上想要恢复某一个工作现场怎么办：先用git stash list查看stash队列。确定要 恢复哪个工作现场 到当前分支。然后用git stash pop stash@{num}。num 就是你要恢复的工作现场的编号。 

5.如果想要清空stash队列则使用git stash clear。 

6.同时注意使用git stash pop命令是恢复stash队列中的stash@{0}即最上层的那个工作现场。而且使用pop命令恢复的工作现场，其对应的stash 在队列中删除。 
使用git stash apply stash@{num}方法 除了不在stash队列删除外其他和git stash pop 完全一样。 



#### 现有代码切换分支

使用Git stash 将A分支暂存起来，然后在某一个分支（如master分支）新建一个分支B，然后在B分支上使用git stash pop 将修改弹出到B分支上，然后这些修改就在B分支上了。然后我们又可以愉快的玩耍了～



#### adb install bug:  INSTALL_FAILED_INSUFFICIENT_STORAGE

所以，很简单，在安装之前先清除 `/data/app/` 下对应包名的文件就好了。解决方法如下：[详情](https://www.jianshu.com/p/4eb025c13a48)

```
adb shell pm uninstall <full.packge.name>
adb shell rm -rf /data/app/<full.package.name>-*
```





#### Alias   Command

g       git
gst     git status
ga      git add
gp      git push
gl      git pull
gb      git branch
gco     git checkout
gm      git merge
grb     git rebase
gd      git diff
gf      git fetch


[详情](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git#aliases)

查看当前theme
echo $RANDOM_THEME
/Users/davisqiao/.oh-my-zsh/themes/gallifrey.zsh-theme



#### reflog

会记录所有HEAD的历史，也就是说当你做 reset，checkout等操作的时候，这些操作会被记录在reflog中。

```bash
$ git reflog

b7057a9 HEAD@{0}: reset: moving to b7057a9
98abc5a HEAD@{1}: commit: more stuff added to foo
b7057a9 HEAD@{2}: commit (initial): initial commit
```

我们要找回我们第二commit，只需要做如下操作：

```shell
$ git reset --hard 98abc5a
```



