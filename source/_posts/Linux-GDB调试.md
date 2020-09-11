---
title: GDB调试
date: 2020-03-14 15:09:20
categories: 
- Linux
tags:
- GDB调试
---

https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html

对多线程的支持
set follow-fork-mode [parent|child]
set detach-on-fork [on|off]

| follow-fork-mode | detach-on-fork | 说明 |
| --- | --- | --- |
| parent | on | 只调试主进程 |
| child | on | 只调试子进程 |
| parent | off | 同时调试父子进程, gdb跟踪父进程, 子进程阻塞在fork位置 |
| child | off | 同上 gdb跟踪子进程, 父进程阻塞 |

进程间切换
| 命令 | 功能 |
| --- | --- |
| info inferiors | 查询正在调试的进程 |
| inferior \<number> | 切换进程 |


| 运行命令 | 缩写 | 功能 |
| --- | --- | --- |
| run | r | 运行程序 |
| continue | c | 继续执行到下有一个断点处 |
| next | n | 单步跟踪, 不进入函数 |
| step | s | 会进入函数 |
| until | | 运行程序直到退出循环体 |
| until + 行号 | | 运行至某行 |
| finish | | 运行程序, 直到当前函数完成返回, 并打印函数返回时的堆栈地址和返回以及参数等信息 |
| call 函数(参数) | | 调试程序中的可见参数, 并传递参数 |
| quit | q | 退出 |

| 设置断点 | 缩写 | 功能 |
| --- | --- |--- |
| break n | b n | 在第n行设置断点, b xx.cpp:500(xx.cpp文件第500行) |
| b fn if a>b | | 在函数f 或者行号n 设置条件断点 |
| break func | b func | 在函数func()的入口处设置断点 |
| delete 断点号n | | 删除第n个断点 |
| disable 断点号n | | 暂停第n个断点 |
| enable 断点号n | | 开始第n的断点 |
| clear 行号n | | 清除第n行的断点 |
| info b | | 显示断点设置情况 |
| delete breakpoints | | 清除所有断点 |

| 查看源代码 | 缩写 | 功能 |
| --- | --- | --- |
| list | l | 默认显示10行 |
| list 行号 | | 以行号为中心的前后十行代码 |
| list 函数名 | | 列出函数名所在函数的代码 |
| list | | 不带参数, 接着上一次的list命令输出下边的内容 |

| 打印表达式 | 功能 |
| --- | --- |
| print p | |
| p a | 显示a的值 |
| p ++a | a的值 |
| p func(22) | 以整数22作为参数调用 后打印 |
| p func(a) | 将变量a作为参数 |
| display 表达式 | 每次单步运行后就打印表达式的值 |
| watch 表达式 | 设置检视点, 一旦被监视的表达式值改变就强行终止正在被调试的程序 |
| whatis | 查询变量或函数 |
| info function | 查询函数 |
| info local | 显示当前堆栈页的所有变量 |


| 查询运行信息 | 功能 |
| --- | --- |
| where/bt | 当前运行的堆栈列表 |
| bt backtrace | 显示当前的调用堆栈 |
| up/down | 改变堆栈的显示深度 |
| set args [args]| 指定程序运行参数 |
| show args| 查看程序参数 |
| info program | 查看程序是否运行, 进程号, 被暂停原因 |


# 安装DebugInfo
```
cat /etc/yum.repos.d/CentOS-Debug.repo

[debug]
name=CentOS-$releasever - DebugInfo
baseurl=http://debuginfo.centos.org/$releasever/$basearch/
gpgcheck=0
enabled=1 # 重点是这个 
protect=1
priority=1


安装 kernel-debuginfo
yum --enablerepo=base-debug install -y kernel-debuginfo-$(uname -r)

安装glibc, 提示缺少其他的也是如此安装
debuginfo-install glibc
```