---
title: Git的基础使用
date: 2020-02-07 18:24:42
categories: 
- 必备技能
- Git操作
tags:
- Git操作
---
# 命令总结
```
# -t Specifies the type of key to create.  The possible values are
#“dsa”, “ecdsa”, “ed25519”, or “rsa”.
# -C 生成注释 ...所以后面的邮箱就是个注释????
ssh-keygen -t rsa -C "youremail@example.com"

git init
git add <file>
git commit -m <message>
git remote add origin  # origin 意为远程库的名字, git的默认叫法

git status
git diff
git log
git log --pretty=oneline # 简单显示

# 注意reset是恢复到上一个版本 即为你上一次commit版本或者pull最近的一次
git checkout --<filename> # 恢复工作区, 可用于恢复修改和恢复勿删文件
git reset HEAD <file> # 恢复暂存区
git reset HEAD^ # 恢复版本库
git reset --hard <hash版本号 即为log中的一串英文字母, 只需要前几个字母即可>
git reflog # 所有版本日志

git rm 从版本库中删除文件

git push -u origin master # 带上 -u 以后提交就不需要加入origin master, 类似记忆功能

`git checkout -b dev` #相当于一下两条命令
`git branch dev` # 分支创建
`git checkout dev` # 分支切换
`git branch -d dev` # 分支删除
`git branch` # 查看当前所有分支
`git merge dev` # 将制定的dev分支合并到当前的分支
```

 # 基本概念
首先把Git的三个区说明一下吧 自己后来慢慢感觉这些理论还是很重要的
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/%E6%9D%82%E9%A1%B9/Git%E4%B8%89%E5%A4%A7%E5%88%86%E5%8C%BA.png)
工作区
顾名思义, 你当前在IDE直接修改的代码全部都是位于工作区的代码
暂存区
暂时存取的区域, 你每次使用add提交的文件全部在这个区域中, 这些文件等待你的commit
版本库
这里存放的是若干个版本, 你每次的commit就是将暂存区的文件作为一个版本提交到版本库, 同时相应的文件被提交后 暂存区的文件被清除

# 基础的操作
**你的第一次提交**
```
git init
git add <file>
git commit -m <message>
```
`git init `
在你命令所执行的文件夹生成版本库
`git add <file>`
将指定文件从工作区添加到暂存区
如果`<file>`用 `.` 代替则为所有相对上一次commit修改过的文件
`git commit -m <message>`
提交暂存区的文件到版本库中成为一个版本 message为这个版本(提交)的描述

**好了你已经成功完成了一次提交, 继续去写代码了**
好你又写完了一堆代码 这时你想知道你工作区的状态是啥-你修改了哪些文件
```
$git status
-------提示如下-----------
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

	修改：     test.txt

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
```
`git status`告诉你 你修改了test.txt文件, 但是你想知道你怎么修改了这些文件, 做了什么改动
这时候就需要`git diff`
```
$git diff
-------提示如下-----------
diff --git a/test.txt b/test.txt
index 0858ae8..8c14912 100644
--- a/test.txt
+++ b/test.txt
@@ -1,3 +1,4 @@
 111111111111111
 222222222222222
 333333333333333
+444444444444444
```
**现在你知道你进行了什么修改, 又进行了一次提交**
```
lsmg@ubuntu:~/temp$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）
	修改：     test.txt
修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
lsmg@ubuntu:~/temp$ git add test.txt 
lsmg@ubuntu:~/temp$ git status
位于分支 master
要提交的变更：
  （使用 "git reset HEAD <文件>..." 以取消暂存）
	修改：     test.txt
lsmg@ubuntu:~/temp$ git commit -m "ver 0.02"
[master ee4400f] ver 0.02
 1 file changed, 1 insertion(+)
lsmg@ubuntu:~/temp$ git status
位于分支 master
无文件要提交，干净的工作区
```

# 时光穿梭机
**在commit之间切换**
Git还可以在不同commit间切换, 当然你必须知道进行过哪些commit, 你不能自己都记在脑子里吧, 所以也存在相关的命令`git log`来查看你的commit记录
```
lsmg@ubuntu:~/temp$ git log
commit ee4400fc2b5694282b866b701f7c21655149e5a2 (HEAD -> master)
Author: ***********
Date:   Sat Dec 7 21:06:42 2019 -0800
    ver 0.02

commit 603260e28ac0cc8fb8e4243120eb00bca83585d2
Author: ***********
Date:   Sat Dec 7 20:57:58 2019 -0800
    ver 0.01

lsmg@ubuntu:~/temp$ git log --pretty=oneline # 简洁显示
ee4400fc2b5694282b866b701f7c21655149e5a2 (HEAD -> master) ver 0.02
603260e28ac0cc8fb8e4243120eb00bca83585d2 ver 0.01
```
上面的`HEAD`代表当前版本, 上一个版本为`HEAD^`, 上上一个版本为`HEAD^^`如此类推
回到前N个版本`HEAD~N`
`git reset --hard HEAD^` 回到上一个版本.
这时你当前版本将会丢失, 使用`git log`也不会查看到原来的版本信息
这时使用`git reflog`来查看你的所有版本日志
```
lsmg@ubuntu:~/temp$ git reset --hard 603260e28ac0cc8fb8e4243120eb00bca83585d2
HEAD 现在位于 603260e ver 0.01
lsmg@ubuntu:~/temp$ git log
commit 603260e28ac0cc8fb8e4243120eb00bca83585d2 (HEAD -> master)
Author: rjd67441 <rjd67441@hotmail.com>
Date:   Sat Dec 7 20:57:58 2019 -0800

    ver 0.01
lsmg@ubuntu:~/temp$ git reflog 
603260e (HEAD -> master) HEAD@{0}: reset: moving to 603260e28ac0cc8fb8e4243120eb00bca83585d2
ee4400f HEAD@{1}: commit: ver 0.02
603260e (HEAD -> master) HEAD@{2}: commit (initial): ver 0.01
lsmg@ubuntu:~/temp$ git reset --hard ee4400f
HEAD 现在位于 ee4400f ver 0.02
```

**撤销修改**
工作区中的撤销
`git checkout -- <filename>`
一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次git commit或git add时的状态。
暂存区中的撤销
`git reset HEAD <file>`
版本库的撤销
`git reset HEAD^` 乖乖回退一次

**删除文件**
删除本地文件后
确实需要从版本库中删除
`git rm`删除然后`git commit`提交即可
误删除需要使用`git chechout -- <filename>`来恢复