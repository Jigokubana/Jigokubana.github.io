---
title: Git的基础使用
date: 2020-02-07 18:24:42
categories: 
- Git操作
tags:
---
2020年8月9日23:06:10 删除已经熟记的部分命令 精简博客

# git设置代理

下面设置http代理只对http链接有效, 如果你的库绑定了ssh链接 则无效
```shell
export http_proxy=http://127.0.0.1:1080
export https_proxy=http://127.0.0.1:1080

curl cip.cc # 查询当前的状态

IP	: xxx.xxx.xxx.xxx
地址	: 中国  xx  xx
运营商	: 联通
数据二	: xxx | 联通
数据三	: 中国xxxx | 联通
URL	: http://www.cip.cc/xxx.xxx.xxx.xxx
```


# 命令总结

密钥生成
```
# -t Specifies the type of key to create.  The possible values are
#“dsa”, “ecdsa”, “ed25519”, or “rsa”.
# -C 生成注释 ...所以后面的邮箱就是个注释
ssh-keygen -t rsa -C "youremail@example.com"
```

```
git remote add origin  # origin 意为远程库的名字, git的默认叫法
git log --pretty=oneline # 简单显示
```

```
# 注意reset是恢复到上一个版本 即为你上一次commit版本或者pull最近的一次

git checkout --<filename> # 恢复工作区, 可用于恢复修改和恢复勿删文件
git reset HEAD <file> # 恢复暂存区
git reset HEAD^ # 恢复版本库
git reset --hard <hash版本号 即为log中的一串英文字母, 只需要前几个字母即可>
git reflog # 所有版本日志

git rm 从版本库中删除文件
```
```
git push -u origin master # 带上 -u 以后提交就不需要加入origin master, 类似记忆功能

git checkout -b dev 相当于一下两条命令
git branch dev  分支创建
git checkout dev  分支切换

git branch -d dev   分支删除
git branch   查看当前所有分支
git merge dev  将制定的dev分支合并到当前的分支
```

# 基础的操作

`git status`告诉你 你修改了哪些文件 但是你想知道你怎么修改了这些文件, 做了什么改动

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