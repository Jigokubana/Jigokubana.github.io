---
title: UNP记录
tags:
  - null
categories:
  - CPP
date: 2020-10-06 12:17:39
---

由于已经在Linux高性能服务器开发中详细说明过绝大部分系统函数 这里略去

# 第一章

通过定义`包裹函数`可以缩短程序. 在muduo框架中就存在包裹函数 将系统函数进行包装

函数中进行错误处理, 这样外部使用`包裹函数`包装好的系统函数的时候可以简化错误处理


**daytimecpclient-domain.cpp**
第一个样例程序稍作修改 可以解析域名  inet_addr没有请求网络 速度快 而gethostbyname会请求域名
服务器速度可能慢
```c++
sockaddr_in server_addr{};
if ((server_addr.sin_addr.s_addr = inet_addr(argv[1])) == INADDR_NONE)
{
    struct hostent* hostptr = gethostbyname(argv[1]);
    if (!hostptr)
    {
        printf("invalid input %s\n", argv[1]);
        exit(1);
    }
    server_addr.sin_addr.s_addr = *((unsigned long*)hostptr->h_addr);
}
```

迭代服务器: 对于每个客户连接都执行迭代一次
并发服务器: 同时处理多个客户连接

#  第二章 传输层

客户端 主动调用close之后 发送了FIN包

服务端 接收到FIN包后read返回0

服务器 read返回0之后调用close函数 发送FIN包

客户端 接收服务器的FIN包

**@@35 P TCP状态转换图@@**


**@@36 P TCP状态转换和Socket函数间的联系@@**


多宿主计算机  可能插有多块网卡

通过bind到INADDR_ANY来监听多个接口的同一个端口


# 第三章 套接字编程简介

大多数套接字函数 都需要一个指向`套接字地址结构`的指针作为参数. 每个协议簇都定义了他自己的`套接字地址结构`. 这些结构的名字均`以sockaddr_开头`


**@@61 P 不同套接字地址结构的比较@@**


# 第四章 基本TCP套接字编程

**connect函数**

ETIMEDOUT: 客户端没有收到SYN的回应 尝试几次后如果依然没有收到回应则返回错误

ECONNREFUSED: 服务器在指定端口上没有进程等待与之连接导致收到了RST回复  之后立即返回错误

connect失败之后 此套接字无法继续使用 需要close当前套接字后重新调用socket

**bind函数**

如果调用listen或者connect前没有调用bind则操作系统会分配一个临时端口. 对于客户端来说可以接受临时端口, 然而服务器一般都是需要指定端口 供外接连接. 

如果指定的端口为0 则认为没有指定端口 会分配临时端口. 如果地址指定为0(INADDR_ANY) 则系统选择IP地址

RPC服务器可以使用临时端口, 但必须将临时端口注册到端口映射器, 这样客户端才能得到对应的临时端口, 之后进行连接

可以使用getsockname获取到系统临时分配的地址和端口

**listen函数**

socket函数创建的套接字默认被认为是主动套接字(准备调用connect) 使用listen函数将一个`未连接`的套接字转换成一个被动套接字

内核为一个监听的系统套接字维护两个队列
- 未完成队列 已经收到了SYN但还未完成三次握手
- 已完成队列 已经进行了三次握手`等待accept从队列中取走` 已建立的连接 如果`此队列为空, accept阻塞`

未完成队列+已完成队列的总长度 即为listen函数的第二个参数backlog

connect函数调用 -> 服务器收到了SYN包 -> 在未完成队列创建相关内容 -> 创建完毕后返回SYN和ACK -> 客户端返回ACK -> 从未完成队列移动到已完成队列队尾


看到这里后想到之前遇到的一个BUG, 注册入`epollfd`中的`listenfd`没有设置成`ENONBLOCK`, `epoll_wait不会返回, 进而去调用accept`, 然而客户端`connect`函数返回显示已经建立连接.  --也就是connect返回说明连接建立, 发生在accpet函数调用前.



**@@91 P exec函数之间的关系@@**

**accept函数**
accpet函数的第一个参数必须是 被动套接字 即调用过listen, 否则返回EINVAL错误