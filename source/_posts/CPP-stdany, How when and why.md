---
title: void*|function|any|optional|variant, How when and why
date: 2021-2-6 18:56:27
tags:
categories:
 - CPP
---

C++17引入的`std::any`挺想用的, 毕竟要学会使用modern c++.

之前都是通过`void*`来实现相关的功能, 这次看看`std::any`的怎么使用何时使用和为何使用


参考文章 
1. [std::any: How, when, and why](https://devblogs.microsoft.com/cppblog/stdany-how-when-and-why/)
2. [std::optional: How, when, and why](https://devblogs.microsoft.com/cppblog/stdoptional-how-when-and-why/)
3. [std::variant](https://zh.cppreference.com/w/cpp/utility/variant)

# void*

从C而来的`void*`可以用来存储任意参数的地址, 在Linux的系统Api上有相当部分的Api使用了`void*`参数. 

常用之一就是`pthread_create`.

1. 将`void*`转换回原来的类型会由于缺少类型信息来提供安全机制, 一旦转换成了错误的类型 极可能程序直接崩溃
2. 需要自己管理生命周期 (可以通过智能指针解决这个问题)
3. 无法进行高效的拷贝, 你需要将其转换为原始类型才能进行深拷贝 否则只能简单的拷贝下指针的地址 因为你不知道指针指向内存块的长度


# any

相对于void* 解决了其三个缺点. 当然对于将会绑定的类型不确定的话使用any更好,这点消耗是值得的. 如果对于确定类型使用则可能会造成不必要的消耗


```c++
#include <cassert>
#include <any>
#include <iostream>

int main()
{
    std::any a;

    std::any b = 648;

    assert(!a.has_value());
    assert(b.type() == typeid(int));

    try
    {
        std::any_cast<float>(b);
    }
    catch(const std::exception& ex)
    {
        std::cout << ex.what(); // bad any_cast
    }

    auto ptra = std::any_cast<int>(&a); // 为空或者类型不匹配返回nullptr
    assert(!ptra);
    auto ptrb = std::any_cast<int>(&b);
    assert(ptrb);
    *ptrb = 328;

    return 0;
}
```

# function

可用于存储函数, 相对于使用函数指针 function调用成员函数更加方便

# optional

用于解决可选的参数传入和可选的返回值输出

emm 重载下感觉也挺香的吧


# variant

提供一个已知类型的集合类型