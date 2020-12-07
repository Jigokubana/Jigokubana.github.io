---
title: Send返回值引出的(非)阻塞IO
date: 2020-03-20 11:14:20
tags:
categories:
  - 网络编程
  - Socket理论
---
# 现存疑问
1. WNOWAIT和O_NONBLOCK 的关系?
2. 当非阻塞send时, 返回值与什么有关? 本地socket发送缓冲区空余还是对端空余?
3. 对端接收缓冲区满了会发生什么?
4. 暂且认为send返回值是拷贝到缓冲区中的字节数, 如果这些数据没有成功发送呢?
5. send函数发送的字节数 超出了recv端的接受能力 会发生什么?

最近这两天在实战一个http服务器的编写.

遇到了一个问题: send的返回值小于传入的len, 因为我仅调用一次send函数 导致页面一直显示加载

然后找了下为什么? 什么时候? send返回值会小于len;

# send
https://stackoverflow.com/questions/14700906/socket-programming-send-return-value

```
In practice, send() in blocking mode sends all the data, regardless of what the documentation says, unless there was an error, in which case nothing is sent.
实战中, send() 处于阻塞状态会发送所有的数据, 除非发生错误 但这会导致数据没有发送

In non-blocking mode, it sends whatever will fit into the socket send buffer and returns that length if > 0. If the socket send buffer is full, it returns -1 with errno = EWOULDBLOCK/EAGAIN.
在非阻塞模式下, 只要能放入socket发送缓冲区 就返回放入的长度 如果缓冲区满了就返回-1 设置EWOULDBLOCK/EAGAIN
```

[非阻塞模式下 send 和 recv 函数的返回值](https://mp.weixin.qq.com/s?__biz=MzU2MTkwMTE4Nw==&mid=2247486643&idx=1&sn=9db678398f390759ff81b354ef056000&chksm=fc70f75fcb077e49363b087ba45609fdfbcba286338e70c2d4c7092e83e2d3feac56455e02a2&scene=27#wechat_redirect)

这篇博客里也说明了这个问题. 下面是原博客的代码稍作修改
```c++
//推荐的方式二：在一个循环里面根据偏移量发送数据
bool SendData(const char* buf , int buf_length)
{
    //已发送的字节数目
    int sent_bytes = 0;
    int ret = 0;
    while (true)
    {
        ret = send(fd, buf + sent_bytes, buf_length - sent_bytes, 0);
        if (ret == -1)
        {
            if (errno == EAGIN)
            {
                //严谨的做法，这里如果发不出去，应该缓存尚未发出去的数据，后面介绍
                break;
            }             
            else if (errno == EINTR)
            {
                continue;
            }
            else
            {
                return false;
            }
        }
        else if (ret == 0)
        {
            //认为对端关闭了连接
            return false;
        }

        sent_bytes += ret;
        if (sent_bytes == buf_length)
        {
            break;
        }
        //稍稍降低 CPU 的使用率
        usleep(1);
    }
    return true;
}
```