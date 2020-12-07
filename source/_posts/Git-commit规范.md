---
title: commit规范
date: 2020-02-22 12:22:02
categories: 
- Git操作
tags:
---
# commit 规范
[阮一峰 Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

Angular 规范.
每个commit message 包括三分部
Header Body 和 Footer
```
<type>(<scope>): <subject> // 必须
// 空一行
<body> // 非必须
// 空一行
<footer> //非必须
```
**Header**
1. type 必需 - 说明commit的类别, 只允许下面七个标识

- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动

2. scope 非必需 - 用于说明 commit影响的范围
比如登录、注册、充值逻辑等等，视项目不同而不同。

3. subject 必需 - commit 目的的简短描述 
- 不超过50字符 
- 第一人称现在时动词开头
- 首字母小写
- 句尾不加句号

**Body**
本次commit的详细描述, 可以分成多行

**Footer**
只用于两种情况 目前用不动 不摘了