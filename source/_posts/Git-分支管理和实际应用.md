---
title: 分支管理和实际应用
date: 2020-03-06 19:34:25
categories: 
- Git操作
tags:
- Git操作
---

2020年3月12日15:42:41 更新 实操了一次hotfix 惊心动魄... 来更新下
2020年7月29日10:53:32 更新 删除一些无意义的话....

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

**暂存代码**

git stash 可以储存当前的工作区 恢复到上一次commit后
git stash list 查看储存的工作区列表
git stash apply stash@{0} 恢复指定的储存
git stash pop 恢复并drop最近的存储 慎用这个 建议使用上边的

  
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

# 本地和远程分支和tag的删除
```shell
# 本地
git tag -d xxx
git branch -d xxx

# 远程
git push origin :refs/tags/xxx
git push origin :xxx
```