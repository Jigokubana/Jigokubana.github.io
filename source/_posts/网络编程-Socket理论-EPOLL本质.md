---
title: EPOLL本质
date: 2020-03-25 17:35:20
tags:
categories:
  - 网络编程
  - Socket理论
---

EPOLL本质是什么?  ~~本质当然是复读机~~

参考和图片来源
[如果这篇文章说不清epoll的本质，那就过来掐死我吧！](https://zhuanlan.zhihu.com/p/64138532)

一个socket对应一个端口号, 网络数据包中包含了端口, 内核可以通过端口号找到对应的socket

**select的不足**
select需要遍历两次socket列表, 第一次遍历用于将进程加入到所有socket的等待队列
第二次遍历是每次唤醒都需要将进程从每个socket等待队列移除
由于遍历开销大, 所以才规定了select最大的监视数量

**功能分离**
![](https://pic2.zhimg.com/80/v2-5ce040484bbe61df5b484730c4cf56cd_720w.jpg)
select将 `维护等待队列` 和 `阻塞进程`两个步骤合二为一, 每次select都需要这两部操作, 然而socket并不需要每次都修改
epoll将操作分开, 使用epoll_ctl维护等待队列, 使用epoll_wait阻塞进程

**就绪队列**
select不知道哪些socket接收到了数据, 只能一个个遍历.
如果内核维护一个"就绪队列" 引用存储事件就绪的socket, 进程唤醒后获取"就绪队列"就能够知道
哪些socket接收到数据


**epoll_create创建了什么?**
创建了一次eventpoll对象, epollfd正是他所代表的对象. 由于epollfd也是文件系统一员也存在等待队列

select将进程加入到每个socket的等待队列, 而epoll将进程加入到了eventpoll的等待队列 只加入了一次.

**epoll_ctl修改了什么**
epoll_ctl将 eventpoll添加到sockfd的等待队列中

当socket接收到数据后, 中断程序会操作eventpoll对象, 而不是直接操作进程(socket的等待队列中就是eventpoll)

**接收数据**
socket接收到数据后, 中断程序会给epollpoll的`就绪列表`添加socket的引用.

这样socket就不会直接影响进程, 而是通过改变eventpoll的`就绪列表`来改变进程状态

**epoll_wait**
当程序执行到epoll_wait后如果`就绪队列`已经有socket引用就会返回, 如果为空就阻塞进程

**阻塞和唤醒进程**
进程被添加到了eventpoll的等待队列中, socket接收到数据后, 中断程序一方面修改`就绪队列`一方面唤醒
eventpoll等待队列的进程, 由于`就绪队列` 进程便知道哪些socket发生变化


**实现细节**
eventpoll的数据结构是双向链表, 能够快速的插入和删除的数据结构

就绪队列的数据结构是红黑树, 方便添加删除检索避免重复添加.