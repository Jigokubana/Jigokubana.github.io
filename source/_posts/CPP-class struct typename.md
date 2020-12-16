---
title: class struct typename
date: 2020-12-08 19:08:08
tags:
categories:
 - CPP
---

# class 和 struct的区别

1. class默认访问权限是private, struct默认访问权限是public
2. class默认private继承, struct默认public继承
3. class可以使用模板, struct不能使用模板
4. struct在没有构造函数, 可以使用大括号初始化. class则要求全部public下才能大括号初始化

```c++
#include <cstdio>

struct Foo1 {} foo1;
class Foo2 {} foo2;

struct Foo3 {int a; int b;};
class Foo4{int a; int b;};

class Foo44{public: int a; int b;};

int main()
{
    printf("sizeof struct-Foo1: %zu\n", sizeof foo1); // 1
    printf("sizeof class-Foo2: %zu\n", sizeof foo2); // 1

    Foo3 foo3 = {1, 2};
    // Foo4 foo4 = {1, 2}; error: could not convert ‘{1, 2}’ from ‘<brace-enclosed initializer list>’ to ‘Foo4’
    Foo44 foo44 = {1, 2};
    return 0;
}
```

# typename class

https://stackoverflow.com/questions/2023977/difference-of-keywords-typename-and-class-in-templates

