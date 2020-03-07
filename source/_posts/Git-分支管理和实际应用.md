---
title: 分支管理和实际应用
date: 2020-03-06 19:34:25
categories: 
- 必备技能
- Git操作
tags:
- Git操作
---
# 分支管理
`git checkout -b dev`相当于一下两条命令
`git branch dev` 分支创建
`git checkout dev` 分支切换

`git branch -d dev` 分支删除

`git branch` 查看当前所有分支

`git merge dev` 将制定的dev分支合并到当前的分支

**冲突解决**
`git merge dev` 将制定的dev分支合并到当前的分支
```
  1 111111111111111
  2 222222222222222
  3 333333333333333
  4 444444444444444
  5 <<<<<<< HEAD  # 我在master分支下添加了6666666 并提交
  6 6666666
  7 =======
  8 7777777
  9 >>>>>>> deb # 我在deb分支下添加了7777777 并提交
  ```
  最后需要我手动修改这个文件为自己需要的内容 然后提交即可
  
  **分支管理**
  通常Git会使用`Fast forward`模式 这样删除分支后会丢失分支的信息
  可以再merge的时候加入`--no-ff`这样就能解决问题
  `git merge --no-ff -m "merge with no-ff" dev`
  由于禁用`Fast forward`后
  会生成新的commit所以需要加入` -m "merge with no-ff"`
  
  `git stash` 可以储存当前的工作区 继续其他的工作
  `git stash list` 查看储存的工作区列表
  `git stash apply stash@{0}` 恢复指定的储存
  `git stash pop` 恢复并drop最近的存储
  
  ```
$ cat test.txt # stash前
111111
$ vim test.txt 
$ cat test.txt # 进行了修改
111111
222222
333333
$ git status
位于分支 master
	修改：     test.txt
$ git stash # stash
保存工作目录和索引状态 WIP on master: 854b710 1
$ cat test.txt 
111111
$ git status
位于分支 master
无文件要提交，干净的工作区
$ git stash pop
位于分支 master
	修改：     test.txt
$ cat test.txt 
111111
222222
333333
  ```
  
# 实际工作中分支的应用
[主要参考](https://zhuanlan.zhihu.com/p/38772378)
**主分支**
- **master**: 这个分支最稳定, 相当于放的可发布版本
- **develop**: 开发分支, 平行于master分支, 负责合并各种`用于开发子功能`的分支

**支持分支**:解决某个问题, 结束后合并回`master`或`develop`分支
- **feature**功能分支, 用于开发一个个子功能, 来自`develop`合并到`develop`去
- **release**:发布分支, 用于修改版本号等小修改, 来自`develop`分支合并到`master`分支
- **hotfixes**:紧急修复bug分支, 从`master`创建, 合并回`develop`分支和`master`分支
![](https://pic4.zhimg.com/80/v2-aef704a4c112eaaf5e8637587ee17df3_hd.jpg)


[Git 分支管理规范](https://juejin.im/post/5d82e1f3e51d4561d044cd88#heading-14)

[Git 基础 - 打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)

**develop分支完成了操作, 准备发布新的版本**
```shell
# 从 develop 分支上创建 release 分支:
# 命名规则如下 release-事件-版本
git checkout –b release-20190919-v1.0.0 develop

# 修复完release的bug后再次提交修改:
git checkout release-20190919-v1.0.0
# 提交本地修改, 如果没有修改bug 可以跳过
git add .
git commit –m “提交日志”
# 推送 release 分支
git push origin release-20190919-v1.0.0

# 发布新版本
# 合并 release 分支到 master 分支:
git checkout master
git merge --no-ff release-20190919-v1.0.0
# 合并 release 分支到 develop 分支:
git checkout develop
git merge --no-ff release-20190919-v1.0.0
# 在 master 分支上创建标签:
git tag tag-20190919-v1.0.0
# 删除本地 release 分支:
git branch –d release-20190919-v1.0.0
# 删除远程 release 分支:
git push origin :release-20190919-v1.0.0
```

新版本完成之后
```shell
# 推送master分支
git push origin master

# 注意上边打的tag需要手动提交 默认push不会提交tag
git push origin [tag name]
# 也可以给push增加参数
git push origin --tags
```