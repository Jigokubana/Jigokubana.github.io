---
title: TCP状态机
date: 2020-03-25 12:55:20
tags:
categories:
  - CPP
  - Socket理论
---

- 待补充 双向管道的建立和断开细节
- 异常三次握手
- 异常四次挥手


[资料参考TCP 协议状态机](https://blog.csdn.net/q1007729991/article/details/69675752)

[TCP有限状态机分析](https://blog.csdn.net/randyjiawenjie/article/details/6397477)

Tcp状态图, 虚线代表客户端 实线代表服务器
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/CPP/Socket%E7%90%86%E8%AE%BA/TCP%E7%8A%B6%E6%80%81%E6%9C%BA.png)

正常的连接建立还是比较容易理解的


说一下从`CLOSE_WAIT`到`LAST_ACK`
当被动关闭方`B`接收到`FIN`后知道对方`A`要关闭连接, `B`回复一个`ACK`给`A`
1. 如果此时`B`没有额外要发送的数据就给`A`, 就发送`FIN`告知`A`自己也要关闭了, 然后`B`进入`LAST_ACK`
2. 如果此时`B`有额外信息要发送等发送完毕后. 这时`B`发送一个`FIN`告知`A`然后B进入`LAST_ACK`


再说一下从`FIN_WAIT_1`到`TIME_WAIT`
1. 直接从`FIN_WAIT_1`到`TIME_WAIT`, 这个对应上面的第一种情况, 由于`B`没有额外信息发送直接发送了`FIN和ACK`, 这时`A`再发送一个`ACK`, 进入`TIME_WAIT`
2. 经过中转状态`FIN_WAIT2`对应上面第二种情况, 由于`B`有额外信息发送, 只发送了`ACK`, 这时`A`还可以接受数据, 直到`B`发来`FIN`然后`A`发送`ACK`进入`TIME_WAIT`


再说一下`TIME_WAIT`状态
主动关闭方`A`最后发送了`ACK`确认被动关闭方`B`的`FIN`.
但是如果这个`ACK`由于各种原因`B`没有收到, 所以`B`会再次发送`FIN`. 然后`A`会在`2MSL`时间内接受到这个`FYN`, 之后`A`再次回复一个`ACk`.计时器重置`2MSL`时间, 重复上面过程. 直到`2MSL`时间内`A`没有收到`FIN`,说明`B`已经收到了.则结束连接

`2MSL`指的是两个`MSL`时间 单个指的是一个片段在网络中的最大存活时间,
A发送的`ACK`可能消耗一个, B重新发送的`FIN`可能也要消耗一个.最大两个


复位报文段
- 客户端访问不存在的端口， 服务器会发送带RST标志的复位报文段
- 异常终止连接， 发送复位报文段
- 客户端或服务端向半打开状态(对方异常终止连接, 但是本方没有收到结束报文)的连接写入数据, 对方回复会一个复位报文段