---
title: UNP记录
tags:
  - null
categories:
  - 读书记录
date: 2020-10-06 12:17:39
---

由于已经在Linux高性能服务器开发中详细说明过绝大部分系统函数

Linux高性能服务器开发的重叠部分, 我会将新的点补充到原博客中, 这里仅作部分简单的说明

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


https://blog.csdn.net/yangbodong22011/article/details/60399728


实际用Centos(内核为4.18版本)测试后, 实际情况为上方博客的情况. 即backlog是已完成队列大小, 而且会存在+1

看到这里后想到之前遇到的一个BUG, 注册入`epollfd`中的`listenfd`没有设置成`ENONBLOCK`, `epoll_wait不会返回, 进而去调用accept`, 然而客户端`connect`函数返回显示已经建立连接.  --也就是connect返回说明连接建立, 发生在accpet函数调用前.


当客户端发送FIN后服务器read类函数返回0

**@@91 P exec函数之间的关系@@**

**accept函数**
accpet函数的第一个参数必须是 被动套接字 即调用过listen, 否则返回EINVAL错误


# 第五章

正常启动 正常终止时程序的表现与TCP协议的关系

慢系统调用: 适用于那些可能永远阻塞的系统调用, accpet(没有新连接就一直阻塞), read(读取不到一直阻塞)等等.

EINTR: 当阻塞于某个慢系统调用的进程, 捕获某个信号且相应信号处理函数返回时, 慢系统调用`可能`返回一个EINTR错误

诸如read, write, select等来说可以忽略EINTR, 如下方这个常见的代码片段. **但是connect则不行, 否则立即返回错误??**

```c++
ssize_t result = read(fd, buffer, sizeof buffer);
if (errno == EINTR)
{
  continue;
}

```

如果同时有N个SIGCHLD信号到达, 信号处理函数极可能执行不够N次.
- UNIX信号没有排队概念
- 问题导致的结果不确定, 即不确定调用多少次

```c++
void sigchld_handler(int signo)
{
  pid_t pid;
  int stat;
  while ((pid = waitpid(-1, &stat, WNOHANG)) > 0)
  {
    process
  }
  return;
}
```
使用waitpid来获取所有已终止进程状态, 并且指定了`WNOHANG`防止不必要的阻塞.

而wait则不能在这里替换waitpid, 因为wait只能阻塞



**第五章 正常启动 正常终止 accept返回前终止 服务器进程终止 服务主机崩溃 主机崩溃后重启 主机关机**


# 第六章 I/O 复用 select和poll函数

进程需要一种预先告知内核的能力, 使得内核一旦发现进程指定的一个或多个I/O条件就绪, 他就通知进程. --IO复用

UNIX可用的五种IO模型
- 阻塞式I/O
- 非阻塞式I/O
- I/O复用
- 信号驱动式I/O(SIGIO)
- 异步I/O(POSIX的aio_系列函数)

前四种都是同步IO, 因为他们在真正的IO阶段read,readfrom等导致了阻塞
只有异步IO模型才是异步IO

**@@低水位标记 ?@@**


## select函数

尽管timeval允许指定一个微秒级的分辨率, 但是真实的分辨率吗...... 部分实现的粗糙一些. 并且还有`调度延迟(定时器结束后, 内核还需要花时间调度相应进程运行)`

struct fd_set 一个集合,可以存储多个文件描述符
内部是一个32位整数的数组, 数组第一个元素对应0~31描述符 一个二进制位代表一个描述符
数组第二个元素对应32~63 以此类推

可读条件
- socket内核接收缓存区中的字节数大于或等于 其低水位标记
- socket通信的对方关闭连接 读半连接关闭, 对socket的读操作返回0
- 监听socket上有新的连接请求
- socket上有未处理的错误, 可以使用getsockopt的SO_ERROR来读取和清除错误 套接字的读操作不阻塞 并返回-1

可写条件
- socket内核的发送缓冲区的可用字节数大于或等于 其低水位标记, **且套接字已经连接, 或套接字不需要连接UDP**
- socket的写操作被关闭, 对被关闭的socket执行写操作将会触发一个SIGPIPE信号
- socket使用非阻塞connect 连接成功或失败后
- socket上有未处理的错误 套接字的写操作不阻塞 并返回-1

异常条件
- 发送带外数据

当select返回可读事件后 如果不进行处理 则下次select调用会继续返回

## shutdown

1. close会将引用计数-1 等到为0时才关闭套接字. shutdown则可以无视此, 激发TCP的正常连接终止序列
2. close会终止读和写两个方向的数据传送, 但由于TCP全双工有时需要告知对方我方发送已经完毕

# 第七章 套接字选项

# 第八章 基本UDP套接字编程 (未看)

# 第十一章 名字和地址转换


# 