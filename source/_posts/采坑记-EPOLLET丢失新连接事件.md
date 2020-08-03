---
title: EPOLLET丢失新连接事件
date: 2020-08-03 23:03:08
tags:
  - 采坑记
categories:
  - 采坑记
---

谨以此文献给我, 两次踩坑.

第一次是根据muduo写我的mongo

第二次则是后来我又根据muduo写higan. ~~框架名字不是重点~~


# 症状描述

1. 最首先的症状就是使用ab压测的时候, 压测会在快结束的时候卡住 而且压测过程中也会出现卡顿
```
PS C:\Users\XXX> ab -n 1000 -c 94 -k  http://10.4.160.96:1022/hello
This is ApacheBench, Version 2.3 <$Revision: 1874286 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.4.160.96 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
apr_pollset_poll: The timeout specified has expired (70007)
Total of 996 requests completed
```
2. 这时候已经建立的连接不受影响 却无法建立新的连接 debug的时候发现会出现建立已经关闭的连接
3. 上面的问题发生与不发生 跟我是否开多线程有关, 如果我不建立额外的EventLoopThread就不会出现这个问题


# 第一波排查 线程之间互相影响

因为问题出现与否 与我是否设置多线程有关, 我就开始排查线程之间的影响.
重点放在了muduo的RunInLoop函数. 因为这个函数我一直不是明白怎么回事.
最终根据muduo的设计思想 一个EventLoop对应一个Thread.
结合RunInLoop就可以达到谁的事件谁处理的效果

举个例子, 连接中断后, 子线程返回相应的事件 要去调用TcpServer的RemoveConnection将自己移除, 如果不使用RunInLoop
当有多个子线程 同时触发同样的事件 同时去对map进行删除操作 必然需要加锁. 况且连接连接的时候也要对map进行插入操作.
使用RunInLoop判断后得知是子线程在调用RunInLoop而非对应的线程, 就将任务存储到了相关的EventLoop中. 而EventLoop中存储的任务只有其所属的Thread才能处理.
所以实现了统一所有子线程的移除连接操作到了主线程中, 而连接的建立也是主线程操作, 最终避免了加锁.

然后那个swap命令处理任务度列减少加锁的时间也很赞

最终我加上了几处重要的RunInLoop, 不过并没有解决问题.............

# 第二波排查 到底返回了多少事件?

我在Acceptor等多个可读事件回调的位置加了计数器, 这时候最先发现的就是连接建立数量不对.

上边996个完成请求缺了4个, 并且正好连接建立了90个连接, 而非设置的并发94, 也是少了4个.

我先怀疑是否是ab压测的处理, 我试了下muduo多线程下94个连接 higan单线程94连接,多线程90连接, 排除了ab的问题

少的链接去了哪里? 我使用netstat命令发现 正好是四个CLOSE_WAIT 我去看了下ab的包正好是114字节 说明了少的四个链接的位置
而且这四个端口号正好是从本该是第91个连接的端口号开始的.
```
[root@fish ~]# netstat -an | grep 1022
tcp        4      0 0.0.0.0:1022            0.0.0.0:*               LISTEN     
tcp      114      0 10.4.160.96:1022        10.4.160.95:8785        CLOSE_WAIT 
tcp      114      0 10.4.160.96:1022        10.4.160.95:8787        CLOSE_WAIT 
tcp      114      0 10.4.160.96:1022        10.4.160.95:8786        CLOSE_WAIT 
tcp      114      0 10.4.160.96:1022        10.4.160.95:8784        CLOSE_WAIT 
```
其实netstat不对劲早就发现了, 然而并不理解netstat命令 没有重视问题.

使用Man命令察看了 第二列是Recv-Q 对于CLOSE_WAIT来说是未接收的字节数 对于LISTEN则是未接收的连接数


# 第三波排查 为什么这四个连接没有被接受

首先我发现了我这行代码写的有问题, 而我的epoll_events_是一个vector 后面的代码有扩容逻辑 难道是因为max_event太小导致的ET模式漏掉事件?
并不是, 未被返回的事件是不会被忽略的 只有你从内核中将事件接收出来, 下次才不会提醒
```c++
// 源代码为 int event_num = epoll_wait(epollfd_, &*epoll_events_.begin(), 一个常数, timeout);
// 修改后为 int event_num = epoll_wait(epollfd_, &*epoll_events_.begin(), static_cast<int>(epoll_events_.size()), timeout);
```

只能继续排查了, 因为知道了Recv-Q的存在 直接搜索几个关键字 找到了这个网页

https://www.cnblogs.com/cxt-janson/p/9273440.html

最后我尝试将EPOLLET模式改为EPOLLLT正常了!!!!!!

# 仍存在的问题

为什么只有我在启用多线程的时候才会发生丢失连接的现象? 这个问题目前还没找到.

# 反思

我最先排查的重点是线程之间的影响, 我对于muduo的多线程处理机制还是不熟悉, 自以为是了某些地方. 

连接建立的数量不对的时候虽然发现了Recv-Q以及CLOSE_WAIT 但由于对Tcp连接状态的理解以及netstat命令的不熟悉, 导致走了弯路.
