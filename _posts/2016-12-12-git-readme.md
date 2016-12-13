---
layout: post
title:  "Git及常用命令说明"
desc: "Git及常用命令说明"
keywords: "git,git server,kinglyjn,张庆力"
date: 2016-12-12
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 概述
```shell
Git Home: https://git-scm.com/
Git is a version control ststem.  
Git is a free software distributed under the GPL.
Git has a mutable index called stage.
The way that git work:
    Git tracks changes of files.
    work file ---[git add]---> stage(index) ---[commit]---> HEAD-master
    * git add命令实际上就是把要提交的所有修改放到暂存区（Stage），
    * 然后执行git commit就可以一次性把暂存区的所有修改提交到分支。
	  * 每次修改，如果不add到暂存区，那就不会加入到commit中。
Git creating a new branch is quick and simple.
```
<br>


### 修改git用户名称和邮箱
```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
<br>

### 初始化git仓库
```shell
初始化一个Git仓库，使用git init命令。
```
<br>

### 添加文件到git仓库
```shell
添加文件到Git仓库，分两步：
第一步，使用命令git add <file>，注意，可反复多次使用，添加多个文件；
第二步，使用命令git commit，完成。
```
<br>

### 查看git仓库当前状态
```shell
$git status
```
<br>

### 如果git status告诉你有文件被修改过，用git diff可以查看修改内容
```shell
$git diff
$git diff HEAD -- readme.txt
$git diff HEAD^ -- readme.txt
```
<br>

### 显示从最近到最远的提交日志
```shell
$ git log
$ git log --pretty=oneline
$ git log --graph --pretty=oneline --abbrev-commit (查看分支合并图)
```
<br>

### 回退到以前的版本
```shell
$ git reset --hard HEAD^     #回退到上一个版本（只能回到以前的版本）
$ git reset --hard HEAD~100  #回退到上100个版本（只能回到以前的版本）
$ git reset --hard c126a6bd08346e59b95a9cfb5dbea53d6ba2f2a4 #回到特定版本号ID的版本（可以回到以前的版本，也可以回到将来的版本，版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了）
$ git reset HEAD readme.txt  #把暂存区的修改回退到工作区(unstage), 用git status可以看到缓存区是干净的。（这条命令和git checkout -- readme.txt 是相对应的，git checkout -- readme.txt 是将工作区的修改丢弃掉，而git reset HEAD readme.txt 是将缓存区的修改回退到工作区）
[注] --hard：彻底将版本库、暂存区、工作区的文件恢复到指定的版本库
     --mixed：将版本库、工作区的文件恢复到指定的版本库
     --soft：仅仅将已提交的版本库恢复到指定的版本库，一般不用
```
<br>


### 在Git中，总是有后悔药可以吃的，Git提供了一个命令git reflog用来记录你的每一次命令
```shell
$ git reflog
```
<br>

### 让文件回到最近一次git commit或git add时的状态
```shell
$git checkout -- readme.txt (注意：这里没有--，就变成了“切换到另一个分支”的命令)
把readme.txt文件在工作区的修改全部撤销，这里有两种情况：
一种是readme.txt自修改后还没有被放到暂存区，撤销修改就回到和版本库一模一样的状态；
一种是readme.txt已经添加到暂存区，又作了修改，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次git commit或git add时的状态。
git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
```
<br>

### 删除文件
```shell
一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了:
$ rm test.txt

这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了：
$ git status

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit：
$ git rm test.txt
$ git commit -m "remove test.txt"

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本
$ git checkout -- test.txt
```
<br>

### 仅删除暂存区的文件
```shell
$ git rm --cache <file_name>
```
<br>

### 分布式工作：添加github远程仓库
```shell
1. 在本地开发库生成公钥和私钥, 注意秘钥对的名字不要私自修改，默认为id_rsa*
$ ssh-keygen -t rsa -C "youremail@example.com"

2. 登陆GitHub，打开“Account settings”，Add SSH Key
   并在本地库机器测试是否公钥是否设置成功：ssh -vT git@github.com (调试模式下并禁止虚假用户)

3. 登录Github之后，创建在github创建远程仓库，Create a new repo

4. 将本地库的代码推送到远程仓库
   4.1. 关联远程库
   $ git remote add origin git@github.com:kinglyjn/test.git         (git协议)
     或 git remote add origin https://github.com/kinglyjn/test.git （https协议）   

   4.2. 第一次推送master分支的所有内容
   $ git push -u origin master

   4.3. 此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改
   $ git push origin master

   分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，
   而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！
```
<br>

### 从远程仓库克隆
```shell
1. 在本地开发库生成公钥和私钥
    $ ssh-keygen -t rsa -C "youremail@example.com"

2. 登陆GitHub，打开“Account settings”，Add SSH Key

3. 将远程代码库克隆到本地，在本地执行瑞安命令
   $ git clone git@github.com:kinglyjn/test2.git
   如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了
```
<br>

### 创建分支方式工作
```shell
查看分支：git branch
创建分支：git branch <branch_name>
切换分支：git checkout <branch_name>
创建和切换分支一步完成：git checkout -b <branch_name> 
合并某分支到当前分支：git merge <branch_name>  (快进模式合并分支，当Git无法自动合并分支时，就必须先解决冲突，解决冲突后再提交合并完成。）
删除分支：git branch -d <branch_name>
强行删除分支：git branch -D <branch_name> (如果要丢弃一个没有被合并过的分支就要用到这条命令)
```
<br>

### 分支管理策略
```shell
a. master分支是非常稳定的，仅用来发布新版本，平时不能在上面干活;
b. dev分支是不稳定的，干活都在dev分支上，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
c. 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了;
d. 开发一个新功能，最好在dev分支上再新建一个feature-id分支；
e. 解决程序bug，最好在相应的分支上新建一个issue-id分支；
f. 快进模式分支：通常合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。
   $ git merge <branch_name>
g. 普通模式分支：合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并。
   $ git merge --no-ff -m "message..." <branch_name>  (因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去)
h. master时刻与远程同步；dev是所有成员都在上面工作，需要与远程同步；feature是否推送取决于你是否和小伙伴合作开发；bug分支没必要推送到远程
```
<br>

### BUG分支管理
```shell
a. 修复bug时，我们会通过创建新的bug分支（eg.issue-101）进行修复，然后合并，最后删除；
b. 当手头工作没有完成时，先把工作现场 git stash 一下，然后去修复bug，修复后，再在相应的分支上git stash pop，回到工作现场。
   $ git stash (储存工作现场)
   $ git stash list (查看现有的工作现场列表)
   $ git stash pop <stash@{0}>（恢复并删除工作现场）
   $ git stash apply <stash@{0}> （恢复而不删除工作现场）
```
<br>

### 多人协作
```shell
a. 首先，可以试图 git push origin <branch_name> 推送自己的修改；
   $ git push origin <branch_name>
b. 如果推送失败，则因为远程分支比你的本地库更新，需要先用 git pull 抓取远程的最新提交，
   抓取远程的最新提交之后，初始时，你只能看到本地的master分支，如果需要在dev分支上开发，就必须创建远程库origin的dev分支到本地
   $ git pull origin <branch_name eg.dev>
   $ git branch --set-upstream dev origin/dev （指定本地dev分支与远程origin/dev分支链接，下次直接git pull就可以，相当于git pull origin dev）
   $ git checkout -b dev origin/dev (相当于git branch dev origin/dev; git checkout dev)
c. 合并远程的最新提交，如果合并有冲突，则解决冲突，并在本地提交，没有冲突后，再用git push origin <branch_name> 推送就能推送成功.
[注] 查看远程库信息，使用git remote -v;
     本地新建的分支如果不推送到远程，对其他人就是不可见的;
     git clone 是本地没有repository时，把整个git项目拷贝下来，包括里面的日志信息，git项目里的分支，你也可以直接切换、使用里面的分支等等;
     git pull相当于git fetch和git merge，是本地有repository时，先从远程下载git项目里的文件，然后将文件与本地的分支进行merge。
```
<br>

### Git标签
```shell
发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。
Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针（跟分支很像对不对？但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。
a. 创建标签：git tag <tag_name> (为当前提交HEAD创建标签)
             git tag <tag_name> 6224937 (为commit id为6224937的版本创建标签)
             git tag -a <tag_name> -m "message..." (创建标签并指定标签信息)
             git tag -s <tag_name> -m "message..." (创建PGP签名标签)
b. 删除标签：git tag -d <tag_name> 
c. 查看标签：git tag [--list] （查看标签列表，按字母顺序排序）
             git show <tag_name> (查看标签详细信息)
```
<br>

### Git配置
```shell
配置文件的位置：
repository config：在当前仓库的.git/config
system config：在git安装目录下
global config：在电脑当前用户主目录下.gitconfig

配置生效顺序：
repository config > system config > global config

忽略特殊文件：
a. 在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件
b. 不需要从头写.gitignore文件，GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了
c. 所有配置文件可以直接在线浏览：https://github.com/github/gitignore
d. 忽略的原则：忽略系统自动生成文件、中间编译或可执行文件（eg.*.class）、敏感信息文件（eg.密码口令）
e. 有些时候，你想添加一个文件到Git，但发现添加不了，原因是这个文件被.gitignore忽略了，如果你确实想添加该文件，可以用-f强制添加到Git：
   $ git add -f App.class
   $ git check-ignore -v App.class (找出到底哪个规则忽略了不该忽略的App.class文件，该命令用于检查出错的忽略文件)

常见参数配置：
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
$ git config --global color.ui true （让git的命令输出显示颜色）

配置命令行别名：
$ git config --global alias.st status （告诉Git，以后st就表示status，即git status == git st）
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.unstage 'reset HEAD'
$ git config --global alias.last 'log -1' （git log -1 查看最近一次的提交）
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
<br>

### 搭建内部Git服务器
```shell
a. 搭建内部git服务器非常简单，通常只需要一下几步即可（强烈推荐用Ubuntu或Debian，这样，通过几条简单的apt命令就可以完成安装）：
   1. 安装git
      $ sudo apt-get install git
   2. 创建git用户来运行git服务
      $ sudo adduser git
   3. 创建证书登录
      收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个
      $ cat id_rsa.pub >> ~/.ssh/authorized_keys
   4. 初始化git仓库（假定是在~/gitrepors文件夹中创建sample.git裸仓库），然后把owner改为git
      创建裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，
      并且服务器上的Git仓库通常都以.git结尾
      $ sudo chown -R git:git sample.git
   5. 禁用shell登录
      出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成
      找到类似下面的一行：
      git:x:1001:1001:,,,:/home/git:/bin/bash
      更改为：
      git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
      这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出   
   6. 克隆远程仓库
      $ git clone git@172.16.127.128:~/gitrepors/sample.git
      剩下的推送就简单了
b. 如果团队很小，把每个人的公钥收集起来放到服务器的~/.ssh/authorized_keys文件里就是可行的;
   如果团队有几百号人，就没法这么玩了，这时，可以用Gitosis来管理公钥。
c. 要像SVN那样变态地控制权限，用Gitolite。
```
<br>


