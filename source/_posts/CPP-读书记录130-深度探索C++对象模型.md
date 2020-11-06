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


# 第三章Data语义学

```c++

class Foo
{
};

Foo foo;
```

`sizeof foo = ?`答案是1， 这是为了防止不同的对象却有相同的地址

同时class也存在`字节对齐`现象


在针对虚拟继承问题的时候

```
D对象的内存结构  从低地址开始

D对象数据部分  《- D类对象指针 dptr 指向这里 可以从vptr指向的表中获取A对象数据部分的偏移量 offset
C对象数据部分
B对象数据部分
A对象数据部分  《- dptr + offset 则实现了 子类指针向父类指针的转化
```

A 派生出 B C。 B C派生出D  如何保证在BC对象中都有A对象， 而在D对象中只有一个A对象？

一种方法是 

先安排派生类的部分 再后接基类的部分， 这样派生类D的对象指针就是指向自己数据部分的指针

如果想要实现子类向父类转换呢？

在vtpr指向的表中的 -1 位置放置偏移量， 由于D类对象的指针是内存开始位置 如果要转换成A类指针

则从D类对象的vptr指向的表中获取A类数据部分的偏移量 这样就实现了子类向父类转换


# 第四章 Function语义学

name mangling