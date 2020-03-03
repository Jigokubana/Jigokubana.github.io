---
title: C++薄书整理
tags:
  - null
categories:
  - CPP
  - CPP基础
date: 2019-10-20 19:13:22
---

# 定义新量

auto 自动根据初始值的类型进行自动类型推导.
decltype 根据表达式的类型定义对象

右值引用, 操纵右值对象
`std::move()`, 可以将一个

**枚举**
不限定作用域的定义
enum color {red, green, blue};
限定在类型内部的作用域
enum class color {red, green, blue};
`color a = color::red`

# 函数
数组传参的长度处理
- 直接传递数组长度
- 使用C风格字符串(默认结尾有标志)
- 使用C++11的新函数begin()和end()同时传递首尾地址

函数指针
`bool (*pf)(int, int)` 可以指向
`bool max(int a,int b)`
调用 `pf(1, 1)`即可
常用于函数的参数是一个函数的返回值

**Lambda**
`[]() -> return type {statements}`
中括号代表lambda引导, 其中的captures子句在同一作用域下lambda主体捕获(访问)哪些对象
以及如何捕获这些对象
- 可以为空, 不会访问外围对象
- [=] 代表用值捕获的方式
- [&] 代表引用捕获

# 类
**辅助函数**
定义辅助函数, 有点像Java里面的重写toString()函数(最常见的代表的例子)
```c++
ostream &print(ostream &os)
{
	os << 1 << endl;
	return os;
}
```
**友元函数**
一个类的辅助函数从概念上可以看成类的接口, 虽然和类的关系很密切, 但并不是类的成员, 所以不能直接访问私有成员
C++ 中可以将该类函数声明为该类的友元.
这样就能访问到类的非公有成员.
在函数的前面加上`friend`关键字
**友元类**
如果A想访问B的私有成员, 可以在B内声明`friend class A`, 这样就可以在B内访问A的私有成员
*友元关系是单向的, 不具有交换性, 同时也不具有传递性*

**构造函数**
默认构造函数没有参数, 或者所有的参数都具有默认值
C++11允许在显示定义构造函数的情况下使用默认构造函数
需要在默认构造函数后面加上`= default`
*使用默认构造函数创建类对象时, 切忌在对象名后I使用圆括号, 成了一个函数声明*

**初始值列表**
T(int a, int b):a_(a), b_(b){}
引用和const修饰的必须用初始值列表初始化, 初始的顺序取决于数据成员在类内的定义顺序,
而非初始值列表的顺序

**简化构造函数**
实际将自己的带参构造函数, 给定默认值, 这样就实现了默认构造函数和带参构造函数的合并

**复制构造函数**
参数为该类的引用

**委托构造函数**
减少构造函数代码量
实际就是一个构造函数后面加上`:`调用另一个构造函数, 同时传入参数
```c++
class A
{
	int bar1_;
	int bar2_;
	A(int b1):A(b1, 2){}
	A(int b1, int b2):bar1_(b1), bar2_(b2){}
}
```
**运算符重载**
T operator /(A, B);
双目运算符两个参数
单目运算符一个参数
对于二元运算符, 左侧运算对象传递给第一个参数, 右侧运算对象传递给第二个参数.

运算符重载的声明和定义的分离
放在类成员中的运算符重载, 需要算入默认的this指针参数
```c++
// test.h
#include <cstdio>
class Test
{
public:
    Test(int a, int b);
    Test& operator+(const Test& right);
	void PrintAB();
private:
	int a_;
	int b_;
};
// test.cpp
#include "test.h"
Test::Test(int a, int b)
{
    a_ = a;
    b_ = b;
}
Test& Test::operator+(const Test& right)
{
    Test test(this->a_ + right.a_, this->b_ + right.b_);
    return test;
}
void Test::PrintAB()
{
    printf("%d %d\n", a_, b_);
}
// main.cpp
int main()
{
    Test test1(1, 1);
    Test test2(2, 2);
    Test test = test1 + test2;
    test.PrintAB();
    return 0;
}

// ++ -- 
Test& operator++(); // 前置版本
Test operator++(int); // 后置版本
```
**类成员指针**
*数据成员指针* (private不能通过指针访问)
其值是数据成员所在地址相对于对象起始地址的偏移值
```c++
A T::*p1 = &T::x
int A::*p1 = &A::value
A a;
// 使用方式
a.*p = ......................;
```
p1指向T类中的 A类型的x数据成员
`A T::`可以用`auto`

*成员函数指针*
```c++
int (A::*pf)();
pf = &A::GetValue;
// 当然可以使用auto 来简化
auto pf2 = &A::GetValue;
// 使用方式
(a.*pf)();

```

----
统一初始化
`X x1 = {0} 和 X x2 {0}` 这两种是不同的实现!!, 大多数情况下这两个是相通的, 然而当
`explicit构造函数`存在的时候`前者的初始化是错误的`

`int x{0} int x= 0 int x(0)` 最后一个是错误的可能会和函数声明冲突
```c++
class A
{
	typedef int x;
	int z(x);
}
```

另外还存在一个
```c++
std::atomic<int> a1{0}; // OK
std::atomic<int> a2(0); // OK
std::atomic<int> a3 = 0; // Error
```
这个错误的原因是copy-initialization引起的
这个copy-initialization发生在`T x = a`的声明, 下面我把原博客的部分重要英文替换成中文或代码描述
`std::atomic<int> a3 = 0` 从`int类型`赋值到`可能是 cv-qualified`的`class type`
[cv-qualified](https://stackoverflow.com/questions/27527642/what-does-cv-qualified-mean)
```c++
// ***cv-qualified***
// non cv_qualified
int first; 
char *second; 

// cv-qualified 
const int third; 
volatile char * fourth; 
```
`std::atomic<int> a3 = 0` 满足了这个条件属于`copy-initialization`
[copy-initialization](https://www.quora.com/What-is-the-difference-between-copy-initialization-and-direct-initialization-of-objects-in-c++)
[直接初始化和复制初始化的区别](https://blog.csdn.net/ljianhui/article/details/9245661)
直接初始化会直接用参数生成对象, 而复制初始化会`用参数生成临时对象, 然后将这个对象复制到正要创建的对象`
```c++
class A
{
public:
        A () { } //直接初始化会调用这个构造函数
        A (const A& a) { } //复制初始化会调用这个
}
```
接下来需要通过`std::atomic<int>(int)`把0转换成成一个纯右值(prvalue)的临时对象
[纯右值](https://zh.cppreference.com/w/cpp/language/value_category)
[纯右值](https://www.cnblogs.com/zpcdbky/p/5275959.html)
然后再将这个临时对象用直接初始化(调用复制构造函数), 然而`std::atomic<int>`把拷贝
构造函数给禁用了, 就会出错
`std::atomic<int> a3 {0}`这个会直接调用接收int的构造函数

回到`X x1 = {0}`这里, 这个表达式是一个复制构造, 用一个braced-init-list(指的{0}). 
这里有两个阶段
首先是会把单个参数的构造函数汇总起来(得到A-list)供选择, 如果其中没有合适的构造函数, 再将候选列表
转换为所有构造函数

如果如果A-list没有构造函数, 并且class T存在默认构造函数, 第一阶段就会被忽略
如果在一个使用`{}`的初始化, 并且选择到了`explicit 构造函数头上`就会报错

`explicit`可以防止在调用某个成员函数的时候, 实参类型(通过构造函数之一)自动转换成相应参数,
但此时这个函数需要其他的成员变量参与到函数运行, 恰巧这时所需要的其他成员变量没有得到合适的初始化, 就会发生错误(没有得到合适初始化的原因就是, 自动调用了不合适的构造函数生成了这个函数所需要的参数(一般是对象))

@2019年10月26日21:25:58@算是第一次解决一个环环相扣的问题, 精力有点不足了
https://zhuanlan.zhihu.com/p/21102748 后续会继续进行理解
[这里还发现一个](https://blog.csdn.net/spaceyqy/article/details/22730939)

# 模板, 泛型 动态内存, 数据结构
```c++
template <typename T>
const T& GetMax(const T& a, const T& b)
{
    return a > b ? a : b;
}
int main()
{
    std::cout << GetMax(1, 2);
	std::cout << GetMax<int>(1, 2); // 显示指定模板类型
    return 0; 
}
```
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/C%2B%2B%E5%9F%BA%E7%A1%80%E5%A4%8D%E4%B9%A02/C%2B%2B%E5%86%85%E5%AD%98.png)
C++堆的内存需要手动new和delete, 如果一个对象new出来没有在能被释放的时候释放就会造成--**内存泄漏**(需要及时delete不需要的对象)
一个指向动态内存的指针, 在动态内存被释放后, 指针依然指向原来的地址--**空悬指针**(释放内存后将相应的指针设置为nullptr)

![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/C%2B%2B%E5%9F%BA%E7%A1%80%E5%A4%8D%E4%B9%A02/%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.png)
