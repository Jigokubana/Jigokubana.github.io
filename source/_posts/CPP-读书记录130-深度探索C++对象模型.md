---
title: 深度探索C++对象模型
tags:
  - null
categories:
  - CPP
date: 2020-09-15 15:17:39
---

# 第一章 关于对象

加上封装之后的成本增加了吗?

并没有增加成本, C++在布局以及存取时间上主要的额外负担是由virtual引起的
- virtual func机制 用来支持一个有效率的执行期绑定
- virtual base class 用来实现多次出现在继承体系中的base class, 有一个单一而被共享的实例


C++ 有两种成员变量`static`和`nonstatic`
三种成员函数`static`, `nonstatic`和`virtual`