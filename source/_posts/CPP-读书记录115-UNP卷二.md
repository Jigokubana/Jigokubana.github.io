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


**限制**
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

**mqueue创建和删除**
```c++
#include <mqueue.h>

// oflag O_RDONLY O_WRONLY O_RDWR 可以加上O_CREAT和O_EXCL或者O_NONBLOCK
// 创建一个新队列的时候(制定了O_CREAT且消息队列不存在)后两个参数都需要
// 权限位 和 指定某些属性 nullptr则使用默认属性

// 成功返回消息队列 fd 失败 -1
mqd_t mq_open(const char *name, int oflag,
 /*mode_t mode, struct mq_attr *attr*/);
```
注意
消息队列文件描述符不必是(而且很可能不是)像文件描述符或者socket文件描述符那样的短整数

```c++
// 关闭消息队列 引用计数 -1
int mq_close(mqd_t mqdes);
```
类似close函数, 调用进程可以不再使用该文件描述符, 但是其消息队列并不会从系统中删除

当一个进程终止的时候, 所有打开着的消息队列都将关闭, 就像自动调用了mq_close


如果要从系统中删除`mq_open`第一个参数`name` 必须调用下面的函数
```c++
// 引用计数 -1
int mq_unlink(const char *name);
```

每个消息队列有一个保存着当前打开描述符数的引用计数器

当消息队列的引用计数仍大于0时, 其name就能删除
但是队列的析构会在`引用计数为0`的时候自动析构.


第一个demo敲完之后 一直是errno=13.
在`man mq_overview`
```
On Linux, message queues are created in a virtual filesystem.   (Other  implementa‐
tions may also provide such a feature, but the details are likely to differ.)  This
filesystem can be mounted (by the superuser) using the following commands:

# mkdir /dev/mqueue
# mount -t mqueue none /dev/mqueue

The sticky bit is automatically enabled on the mount directory.
```
而且 name参数的格式是`/somename` 这个是相对挂载目录的


**属性设置**
```c++
struct mq_attr
{
    long mq_flags; /* 0, O_NONBLOCK */
    long mq_maxmsg; /* max number */
    long mq_msgsize; /* max size of a msg in bytes */
    long mq_curmsgs; /* number of message currently on queue */
}

int mq_getattr(mqd_t mqdes, struct mq_attr *attr);

// set 的时候 只有mq_attr中的mq_flags起作用 设置取消O_NONBLOCK
// 最大消息数和最大字节数 只能在创建队列的时候设置
// 队列中房钱消息数 只能获取不能设置
// 第三个参数 不为nullptr则返回之前的属性
int mq_setattr(mqd_t mqdes, const struct mq_attr *attr, struct mq_attr *oattr);
```

**发送接收信息**
```c++
// prio 优先级 必须 <= MQ_PRIO_MAX
int mq_send(mqd_t mqdes, const char *ptr, size_t len, unsigned int prio);

// len 必须 >= mq_attr.mq_msgsize 如果小于则 errno = EMSGSIZE
ssize_t mq_receive(mqd_t mqdes, const char *ptr, size_t len, unsigned int *priop);
```
如果不使用优先级 发送的时候可以将 mq_send的 prio 设置为0
接受的时候 mq_receive priop 设置为 nullptr


**消息队列的限制**
- mq_maxmsg 队列中最大消息数
- mq_msgsize单个消息的最大字节
- MQ_OPEN_MAX 一个进程同时打开消息队列的最大数目
- MQ_PRIO_MAX 最大优先级+1

**异步事件通知**
- 产生信号
- 创建一个线程执行一个指定的函数

```c++
// 为指定队列建立或者删除异步事件通知.

// 成功返回 0 失败返回 -1
int mq_notify(mqd_t mqdes, struct sigevent *nofification);

struct sigval
{
    int sival_int;
    void *sival_ptr;
}

struct sigevent
{
    int sigev_notify; // SIGEV_NONE SIGEV_SIGNAL SIGEV_THREAD
    int sigev_signo; // signal number if SIGEV_SIGNAL
    union sigval sigev_value; // passed to signal handler pr thread
    // 下面两个用于 SIGEV_THREAD
    void (*sigev_notify_function)(union sigval);
    pthread_attr_t *sigev_notify_attributes;
}
```
1. 如果 notification != nullptr
当前进程希望 在有一个消息 到达所指定的 先前为空的队列 时得到通知
我们说 该进程被注册为接收该队列的通知

2. 如果 notification == nullptr
如果 该进程被注册为 接受所指定队列通知 则取消它

3. 任何时刻只有一个进程可以被注册为 接收某个队列的通知
4. 当一个消息到达某个空队列, 而且已经注册, 那么只有当
没有因为调用mq_reveive导致的阻塞 的时候才会发出通知
5. 通知发出后 注册立即被撤销.. 需要重新注册???

Unix信号产生后会复位成默认行为.
信号处理程序通常第一个调用signal函数, 用于重新建立处理程序
这时会产生一个空窗时期(信号产生 与 当前进程重建信号处理程序之间)
空窗时期再次产生同一信号 可能终止当前进程

初看起来, mq_notify可能也有这个问题.
不过在消息队列`为空前` `通知不会再次产生`
所以不要在`读出消息后`重新注册
而应该在`读出消息前`重新注册

