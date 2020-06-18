---
title: CPP11特性
date: 2020-02-19 17:41:27
tags:
categories:
 - CPP
 - CPP重点
---

# 库新特性

## std::move

用于指示对象可以被移动

- 用于移动字符串 表示不复制字符串而是将内容移动 降低成本 原字符串变量将会未定义
- 从左值带std::move构造 不会进行别名检查 这就意味着不进行自我赋值检查 导致未定义行为

## std::forward<T>

1. 转发左值为左值或右值 依赖于T

```c++
template<class T>
void Wrapper(T&& arg)
{
    Foo(std::forward<T>(arg));
}

```
传递 右值std::string时, 推导T为 std::string
forward将右值引用传递给Foo

传递 左值const std::string时推导T为 const std::string&
将const左值引用传递给Foo

传递 左值非const std::string时推导T为 std::string&
将非const左值引用传递给Foo

2. 转发右值为右值 并禁止右值的转发为左值


```c++
#include <iostream>
#include <memory>
#include <utility>

struct A
{
    A(int&& n) { std::cout << "rvalue overload, n=" << n << "\n"; }
    A(int& n) { std::cout << "lvalue overload, n=" << n << "\n"; }
};

class B
{
public:
    template<class T1, class T2, class T3>
    B(T1&& t1, T2&& t2, T3&& t3) :
        a1_{ std::forward<T1>(t1) },
        a2_{ std::forward<T2>(t2) },
        a3_{ std::forward<T3>(t3) }
    {
    }

private:
    A a1_, a2_, a3_;
};

template<class T, class... U>
std::unique_ptr<T> make_unique1(U&&... u)
{
    return std::unique_ptr<T>(new T(u...));
}

template<class T, class... U>
std::unique_ptr<T> make_unique2(U&&... u)
{
    return std::unique_ptr<T>(new T(std::forward<U>(u)...));
}

int main()
{
    int i = 1;
    make_unique1<B>(2, i, 3);
    make_unique2<B>(2, i, 3);
}
```
lvalue overload, n=2
lvalue overload, n=1
lvalue overload, n=3

rvalue overload, n=2
lvalue overload, n=1
rvalue overload, n=3


# 语言新特性
## Lambda
参考资料
https://zh.cppreference.com/w/cpp/language/lambda
https://blog.csdn.net/qq_34199383/article/details/80469780

```c++
[ 捕获 ] ( 形参 ) -> ret { 函数体 }
[ 捕获 ] ( 形参 ) { 函数体 }
[ 捕获 ] { 函数体 }

& 以引用隐式捕获被使用的自动变量
= 以复制隐式捕获被使用的自动变量

[&]{};          // OK：默认以引用捕获
[&, i]{};       // OK：以引用捕获，但 i 以值捕获

[=]{};          // OK：默认以复制捕获
[=, &i]{};      // OK：以复制捕获，但 i 以引用捕获

[this]{}; // 获取this指针, 如果使用了&和=则会默认包括this
```
**使用场景一**
以sort为代表的函数 需要传入函数的函数
```c++
// 原版
bool compare(int &a, int &b)
{
	return a > b;
}

xxx(compare);

// 新版
xxx([](int a, int b){return a > b;});
```
**使用场景二**
```c++
auto add = [](int a, int b){return a + b;};
int bar = add(1, 2);
```

## 类型特性
https://zh.cppreference.com/w/cpp/types##.E7.B1.BB.E5.9E.8B.E7.89.B9.E6.80.A7.28C.2B.2B11_.E8.B5.B7.29

神仙的头文件<type_traits> 可以判断一个值是否是某个类型

比如判断一个值
是不是 整形
是不是 void
是不是 数组 等等

目前不知道用在什么地方合适..... 留个连接备用吧

## 花括号初始

## 函数对象
#### std::function  functional
```c++
##include <functional>
##include <iostream>

void PrintA()
{
    std::cout << "A" << std::endl;
}
void PrintB(int bar)
{
    std::cout << "B" << std::endl;
}
int main()
{
    std::function<void()> FPrintA = PrintA;
    FPrintA();

    std::function<void(int)> FPrintB = PrintB;
    FPrintB(1);

// 感觉配合Lambad 挺不错, 在一个函数中经常使用的功能可以这样定义
    std::function<void()> FLambad = [](){std::cout << "Lambad" << std::endl;};
    FLambad();
	
	// 不过使用auto貌似更简单
	auto ALambad = [](){std::cout << "Lambad" << std::endl;};
    ALambad();
}

$ A
$ B
$ Lambad
```

#### std::bind std::ref std::cref
今天在学习
https://github.com/baloonwj/flamingo
的时候, 从中发现的这个函数. 这个函数跟std::function 一起使用.
竟然实现了回调函数调用类的成员函数!!!!

https://zh.cppreference.com/w/cpp/utility/functional/bind

```c++
##include <functional>
##include <cstdio>

void f(int n1, int n2, int n3, int n4)
{
    printf("f function-->n1: %d, n2: %d, n3: %d, n4: %d\n", n1, n2, n3, n4);
}

void ff(int &n1, int& n2, const int& n3)
{
    printf("before ff function-->n1: %d, n2: %d, n3: %d\n", n1, n2, n3);
    n1 = 11;
    n2 = 22;
    // n3 = 33; 编译错误
    printf("after ff function-->n1: %d, n2: %d, n3: %d\n", n1, n2, n3);
}

int main()
{
    auto f1 = std::bind(f, std::placeholders::_2, std::placeholders::_3, 666, std::placeholders::_1);

    f1(1, 2, 3, 4, 5);
    // infunction-->n1: 2, n2: 3, n3: 666, n4: 1
    // 1 绑定_1  2绑定_2  3绑定_3   // 4 5被忽略
    // 按照 f1的顺序传入参数
    // 所以调用为 f(2, 3, 666, 1);

    int n1 = 1, n2 = 2, n3 = 3;
    auto ff1 = std::bind(ff, n1, std::ref(n2), std::cref(n3));
    n1 = -1;
    n2 = -2;
    n3 = -3;
    printf("before ff1 function-->n1: %d, n2: %d, n3: %d\n", n1, n2, n3);
    ff1(n1, n2, n3);
    printf("after ff1 function-->n1: %d, n2: %d, n3: %d\n", n1, n2, n3);
    // before ff1 function-->n1: -1, n2: -2, n3: -3
    // before ff function-- > n1: 1, n2 : -2, n3 : -3 // 这里说明了值传递 参数是绑定时就决定好了 引用参数还是可以改变的
    // after ff function-- > n1: 11, n2 : 22, n3 : -3
    // after ff1 function-- > n1: -1, n2 : 22, n3 : -3 // 引用传入成功改变, 值传入和const引用传入未变

    // std::ref 按引用传入参数 std::cref按const引用传入参数
}
```



#### std::pair utility std::tuple tuple

https://zh.cppreference.com/w/cpp/utility/tuple

pair是 结构体模板, 可在一个单元中存储两个相异对象. pair是tuple拥有两个元素的特殊情况

tuple是固定大小的异类值(啥是异类值??)汇集
```c++
std::tuple<int, double > GetInfoById(int id)
{
    if (id == 0) return std::make_tuple(1, 1.1);
    if (id == 1) return std::make_tuple(2, 2.2);

    throw std::invalid_argument("id");
}

int main()
{
    auto info = GetInfoById(0);
	// 获取指定位置
    std::cout << "1:" << std::get<0>(info)
            << " 2:" << std::get<1>(info) << std::endl;

    int val1;
    double val2;
	// 直接创建引用的tuple
    std::tie(val1, val2) = GetInfoById(1);
    std::cout << "1:" << val1
              << " 2:" << val2 << std::endl;
			  
$ 1:1 2:1.1
$ 1:2 2:2.2
}
```