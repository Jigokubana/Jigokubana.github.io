---
title: 头文件互相引用
date: 2020-02-07 16:37:08
tags:
  - CPP踩坑记
  - CPP
categories:
  - CPP
  - CPP踩坑
---
写在前头 本文中的**编译**二字基本都带加粗, 因为目前为止我还没做学到过**编译**器相关的东西, 
姑且将`那个执行过程`称为**编译**, 所以加粗
# 现象
二月六号的时候 我在写那个小游戏, 写头文件`a.h`发现即使引用了一个头文件`b.h`, 也没有办法使用定义在那个头文件之中的结构体.

其实有点我没在意: 当我在`a.h`文件中引用`b.h`的时候`b.h`会变成`invaild`, 这里其实就已经提示了
```c++
// a.h
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#include "b.h"
class A
{
    B *b;
};
#endif //UNTITLED1_A_H

// b.h
#ifndef UNTITLED1_B_H
#define UNTITLED1_B_H
#include "a.h"
class B
{
    A *a;
};

#endif //UNTITLED1_B_H
// main.cpp
#include "a.h" // a.h中有b.h就只include a.h
int main()
{
    A a{};
    B b{};
}

error: ‘A’ does not name a type
error: ‘B’ does not name a type
```
# 原因
## 先说一下 #ifndef #endif
`ifndef`全称`if not defined`
意思是如果`#ifndef`后面的宏没有被定义 就继续**编译**其中的内容
继续**编译**`#ifndef`的下一句就是`#define`这个宏, 这样这个宏就被定义了
第二次**编译**遇到这个头文件的时候, `#ifndef`后面的宏已经被定义了就跳过了if中的内容

## #include的作用
将 #include右边的文件展开到此文件中

## 问题解释
这样当你在main函数中 `#include "a.h"`的时候`a.h`之中的内容就被展开
变成如下
```c++
// main.cpp
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#include "b.h"
class A
{
    B *b;
};
#endif //UNTITLED1_A_H

int main()
{
    A a{};
    B b{};
}
```
还有一个include同样操作得到如下
```c++
// main.cpp
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#ifndef UNTITLED1_B_H
#define UNTITLED1_B_H
#include "a.h"
class B
{
    A *a;
};

#endif //UNTITLED1_B_H
class A
{
    B *b;
};
#endif //UNTITLED1_A_H
int main()
{
    A a{};
    B b{};
}
```
我猜测**编译**的时候是逐行执行的, 至少在头文件的这部分是逐行执行
这样执行完了`行号为2 3 4 5`的四个宏 准备执行`行号为6`的这一行`#include "a.h"`继续展开
得到如下
```c++
// main.cpp
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#ifndef UNTITLED1_B_H
#define UNTITLED1_B_H
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#include "b.h"
class A
{
    B *b;
};
#endif //UNTITLED1_A_H
class B
{
    A *a;
};

#endif //UNTITLED1_B_H
class A
{
    B *b;
};
#endif //UNTITLED1_A_H
int main()
{
    A a{};
    B b{};
}
```
可能有些乱 但仔细理解还是能到这里. 这下继续逐行执行 准备是`行号为6`的这一行(也可能是行号7的部分, 区别就是在include的下一行展开还是本行展开)`#ifndef UNTITLED1_A_H`
这个宏就判断`UNTITLED1_A_H`是不是被定义了, 恩被定义了(第三行代码)跳到对应的`#endif`, 这样就防止了头文件
无穷无尽的调用 我将跳过的部分注释掉方便继续分析

```c++
// main.cpp
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#ifndef UNTITLED1_B_H
#define UNTITLED1_B_H
//#ifndef UNTITLED1_A_H
//#define UNTITLED1_A_H
//#include "b.h"
//class A
//{
//   B *b;
//};
//#endif //UNTITLED1_A_H
class B
{
    A *a;
};

#endif //UNTITLED1_B_H
class A
{
    B *b;
};
#endif //UNTITLED1_A_H
int main()
{
    A a{};
    B b{};
}
```
下一行就是`第十四行`的`class B`的**编译**, 那么问题来了`第16行`的`A`是啥东西??.

这里报错报了两行 我大胆推测一下 `class B`的**编译**的出错后, 依然在继续这个过程, 到了`class A`的时候
发现其中的`B`又是啥呢? 推测不同头文件编译互不影响 所以出现了两行报错

我们回到`class B`的头文件在`class B`的前面加上一行`class A;` 得到如下代码
```c++
// main.cpp
#ifndef UNTITLED1_A_H
#define UNTITLED1_A_H
#ifndef UNTITLED1_B_H
#define UNTITLED1_B_H
//#ifndef UNTITLED1_A_H
//#define UNTITLED1_A_H
//#include "b.h"
//class A
//{
//   B *b;
//};
//#endif //UNTITLED1_A_H
class A;
class B
{
    A *a;
};

#endif //UNTITLED1_B_H
class A
{
    B *b;
};
#endif //UNTITLED1_A_H
int main()
{
    A a{};
    B b{};
}
```

到了这里你会发现A的报错消失了, 因为这次在**编译**`class B`的时候事先知道了`A`是一个类并且你必须把
`class B`之中的这一句`A *a;`写成指针, 因为指针大小确定, 而且没有初始化, 所以**编译**就能通过, 如果你写成
`A a;`依然会报错, 因为不知道A具体是个啥 最起码的内存都没法分配

好的下面继续, 你发现虽然A的报错消失了 但是`B`的报错还在, 我这里依然推测下, 不同头文件之间的**编译**互不影响,
所以依然不知道`B`是什么此时按照上面操作加入`class B;`前置声明, **编译**正确通过.

虽然两个头文件**编译**互不影响, 但是既然是main.cpp把他们引进来的 就同时对main.cpp起作用 自然就知道了A和B是啥


虽然这个问题网上答案很多, 但我对此还是很多不理解, 所以在做某些推测的情况下, 分析出来了这些原因. 如果有误,请直接评论中指出. 这也算是本人的第一篇写的认真的博客