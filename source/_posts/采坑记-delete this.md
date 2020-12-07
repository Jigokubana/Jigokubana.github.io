---
title: 由muduo tie引出的 delete this
date: 2020-11-26 17:37:08
tags:
  - 采坑记
categories:
  - 采坑记
---

在半个多月以前 看完了深度探索C++对象模型, 然而只是粗略的过了一遍. 并没有结合具体的例子去深刻的理解下


如果delete this 不会遇到 那这样呢?
```c++
#include <cstdio>

void Delete();

class Foo
{
public:

    Foo()
    {
        bar_ = 111111;
    }
    ~Foo()
    {
        printf("~Foo()\n");
    }

    void Bar1()
    {
        printf("Bar1()\n");
    }
    void Bar()
    {
        Delete();
        printf("bar: %d\n", bar_);
        Bar1();
    }

    int bar_;
};

Foo* foo = nullptr;
bool del = false;
void Delete()
{
    if (!del)
    {
        del = true;
        delete foo;
    }
}

int main()
{
    foo = new Foo;
    foo->Bar();
    printf("\n");
    foo->Bar();

    return 0;
}  
```

程序安然的结束了, 但是发现 成员变量bar的值已经被初始化了, 由于delete释放了内存.

```
~Foo()
bar: 0
Bar1()

bar: 0
Bar1()
```

同时delete是两步操作
1. 调用对象的析构函数 (析构函数中调用 会导致递归)
2. 释放相关空间

[官方解释](https://isocpp.org/wiki/faq/freestore-mgmt#delete-this)


**成员变量指针类型**
```c++
#include <cstdio>
#include <functional>

class Foo
{
public:
    
    void Bar1()
    {
        if (bar_)
        {
            bar_();
        }
    }

    void Bar2()
    {
        bar_();
    }
    std::function<void()> bar_;
};

Foo* foo = nullptr;

void PrintfBar()
{
    printf("Bar\n");
}

int main()
{
    foo = new Foo;
    foo->bar_ = PrintfBar;

    foo->Bar1(); // Bar
    foo->Bar2(); // Bar

    delete foo;

    foo->Bar1(); // Segmentation fault (core dumped)

    foo->Bar2();

    return 0;
}
```

测试后所有对指针型成员变量的 解引用操作都会导致`Segmentation fault (core dumped)` 所以下面的代码也有了解释

一个困扰我很久的BUG 即为程序会在HandleEvent触发`Segmentation fault (core dumped)`的原因找到了
```c++
void Channel::HandleEvent()
{
	/**
	 * lock增加TcpConnectionPtr的引用计数 防止从TcpServer中erase后 直接销毁TcpConnection
	 *
	 * 否则如果不增加引用计数 当TcpConnection被销毁后, 所管理的Channel也将会被销毁
	 * 在这之后 不能再使用TcpConnection和Channel的任何 成员函数和成员变量
	 *
	 * 有点类似于在一个 new出来的对象的成员函数中 delete自己
	 * 详见 https://stackoverflow.com/questions/7039597/what-will-happen-if-you-do-delete-this-in-a-member-function
	 *
	 * 经过查阅后自己对深度探索C++对象模型 有了更深得理解
	 */
	std::shared_ptr<void> guard = tie_.lock();
	if (!guard)
	{
		LOG_WARN("connection: %s is closed", connection_name.c_str());
	}

	if (event_ & XEPOLLIN)
	{
        if (readable_callback_)
        {
            readable_callback_();
        }
	}
}
```