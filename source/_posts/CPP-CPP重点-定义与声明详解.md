---
title: 定义与声明详解
date: 2020-02-27 10:38:08
tags:
categories:
 - CPP
 - CPP重点
---
# 变量声明和变量定义

变量声明: 用于向程序表明变量的类型和名字
变量定义: 用于为变量*分配存储空间*, 同时可为变量指定初始值

定义时会自动声明, 变量只能被定义一次, 却可以声明多次
extern声明不是定义

```c++
extern int i; // 声明i 但没有定义i
int i; // 定义且声明i

extern int i = 10; // 定义并声明

// 如果再函数体内部试图初始化一个由extern标记的变量,会引发错误
```

# extern
只有extern声明位于函数外部的时候,才能被初始化

**在C++中 可以用来声明全局变量**
```c++
// file1.cpp
int global_int = 1;

// file2.cpp
extern int global_int;
// 后面可以直接使用 global_int, 因为他在其他地方定义了
```