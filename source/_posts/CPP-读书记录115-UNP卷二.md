---
title: UNP卷二读书记录
date: 2020-03-15 11:16:20
tags:
categories:
 - CPP
 - 服务器编程-书籍记录
top: 115
img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%B0%81%E9%9D%A2/UNP%E5%8D%B7%E4%BA%8C%E5%B0%81%E9%9D%A2.png
---

# 消息传递
## 管道和有名管道FIFO

**popen pclose**
```c++
#include <stdio.h>
// 成功返回文件指针, 出错为NULL
FILE* popen(const char *command, const char *type);
FILE* fp = popen(command, "r");

// 成功为shell终止状态, 出错则为 -1
int pclose(FILE *stream);
pclose(fp);
```
popen创建一个管道并启动另外一个进程 创建者要么从管道读出标准输入, 要么将标准输出写到管道
返回的文件指针作用
通过type参数控制
type = "r" 创建者 通过文件指针读入command命令产生的标准输出
type = "w" 创建者 通过文件指针为command命令提供标准输入

pclose 关闭这个标准IO流


**FIFO 先进先出(first in, first out)**

又称为有名管道(named pipe) 单向(半双工)数据流, 每个FIFO都有一个路径名与之对应. 
从而允许无亲缘关系的进程访问同一个FIFO

```c++
#include <sys/types.h>
#include <sys/stat.h>

// 成功返回 0 失败返回 -1
int mkfifo(const char *pathname, mode_t mode);

const char FIFO1 = "/tmp/fifo.1";
mkfifo(FIFO1, 0666);

// 创建之后需要打开来进行读或者写 可以使用open也可以使用其他标准IO打开函数
int writefd = open(FIFO1, O_WRONLY, 0); /* 在父进程中打开 父进程写 */
int readfd = open(FIFO1, O_RDONLY, 0); /* 在子进程中打开 子进程读 */

// 只有调用unlink才能从文件系统删除文件名字
unlink(FIFO1);
```
mkfifo 隐含已经指定 O_CREAT | O_EXCL
要么不存在创建一个新的, 要么返回 errno=EEXIST(所指定名字的FIFO已经存在)

如果想要打开已经存在的FIFO可以使用open函数.
可以判断 如果返回-1 并且errno=EEXIST 就调用open函数打开


```c++
// 子进程
readfd = open(FIFO1, O_RDONLY, 0);
writefd = open(FIFO2, O_WRONLY, 0);

// 父进程
writefd = open(FIFO1, O_WRONLY, 0);
readfd = open(FIFO2, O_RDONLY, 0);

// 父进程 这样会阻塞
readfd = open(FIFO2, O_RDONLY, 0);
writefd = open(FIFO1, O_WRONLY, 0);
```
如果调换父进程的两行代码 程序就会进入死锁
因为如果当前没有任何进程`打开某个FIFO来写`, 打开这个`FIFO来读`的进程将会`阻塞`


当对一个管道或者FIFO的最终close发生的时候, 该管道或FIFO中的任何残余数据都将被丢弃


即使在A主机上能够访问到B主机的目录, 即使不同主机上的两个进程都能够通过NFS打开同一个FIFO
他们之间也不能通过FIFO从一个进程到另一个进程发送数据

拒绝服务型攻击Dos
如果是单进程可能会让服务器处于阻塞状态, 多进程也可能会由于一个恶意客户发送大量独立请求
导致服务器子进程数达到上限, 使得后续的fork失败


**其他**
从字节流中获取完整的单个信息
- 带特殊终止序列
许多UNIX应用使用换行分割 因特尔应用程序使用回车符加上一个换行符
- 显式长度
将长度增加在请求中
- 每次连接一个记录
应用通过关闭对端连接指示一个记录结束. HTTP1.0使用的这一技术

标准IO函数fdopen可以将标准IO流与pipe返回的文件描述符相关联'


系统加在管道和FIFO的唯一限制是
- OPEN_MAX
一个进程在任意时刻打开的最大描述符数量 >=16
- PIPE_BUF
可以原子性的写入一个管道或FIFO的最大数据量 >= 512


**习题练习**
1. 父进程终止的时候, 如果子进程的fd[1]仍处于打开状态, 则子进程对fd[0]的read 不会返回文件结束符,
因为fd[1]虽然在父进程中关闭了, 但是在子进程中依然打开. 如果关闭子进程fd[1], 则父进程一旦关闭, 
他的所有文件描述符即关闭, 子进程对fd[0]的read就会返回0.(想到了自己写的服务器, 客户端断开连接就会read出一个0长度的内容)
2. 从 不存在即创建, 存在->打开 变成 存在->打开, 不存在即创建. 可能会在open打开失败后 创建fifo之前
被其他进程抢先创建fifo, 导致本进程的mkfifo调用失败
3. 出错信息写到了标准错误输出
5. 可以把第一个open调用设置成非阻塞, 但是为了避免readline返回错误, 标志必须在readline之前去除
6. 死锁
7. 读进程关闭管道或者FIFO后给写进程一个信号(往关闭的管道写会产生SIGPIPE信号),
写进程关闭管道或者FIFO后将文件结束符发送给读进程(读进程读到长度为0的文件结束符)


## POSIX消息队列
