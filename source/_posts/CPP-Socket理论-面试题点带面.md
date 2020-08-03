---
title: 面试题点带面
date: 2020-04-11 11:00:20
tags:
categories:
  - CPP
  - Socket理论
---

马上步入四月中旬了, 自己对于socket理论知识知道的还是不多, 于是想了下可以整理下面试题.

单纯整理"正确答案"怎么行. 我相信大部分面试题代表的都是某个知识点 某个知识面 或有他的重要实际应用

部分题目是我自己凭空想出来的, 单独开一篇博客又不是内容足够, 索性放在一起

# 基础api相关
## send发送的时候会自动转换字节序??

作为开篇问题, 我选了这个. 这个问题也算是困扰了一段时间.
网上的解答自然已经有了
https://stackoverflow.com/questions/23106938/do-recv-and-send-convert-the-messages-in-network-order-format-automatically

最先接触到socket字节序转换相信大多是`htons()`
这个说明是将主机字节序转换成网络字节序, 我使用的自然是小端 将小端转换成大端 反转链表就好
而在大端上 头文件自动选择了`htons()`空命令版本

起初我使用send发送全是 类似string的东西 char*保存, 单字节数据自然不会受字节序影响

但是最近学习了moduo后发现了发送数字的实际应用
对于一个 `int32_t`类型的数字需要调用`htobe32()`转换成`大端序`然后`std::copy`到buffer中
接收端读取出四个字节 `std::copy`到`int32_t`中, 需要调用下`be32toh()`转换成`小端序` 这就涉及到了字节序转换

总结下就是单字节数据比如发送一个字符串 是不会收到字节序影响
而如果你直接发送数字比如`int32_t` 就会受到影响

## SIGPIPE信号触发原因和防治

client与server建立连接后 在server发送信息给client的时候, 由于client已经退出,
client方会向server发送RST. 此时server向已经接收到RST的client的socket继续写入数据 会导致SIGPIPE信号
```c++
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstring>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>

int main()
{
    int listen_fd = socket(PF_INET, SOCK_STREAM, 0);

    struct sockaddr_in listen_address{};
    listen_address.sin_family = AF_INET;
    listen_address.sin_port = htons(9123);

    int ret = bind(listen_fd, (struct sockaddr*)&listen_address,
            static_cast<socklen_t>(sizeof listen_address));
    if (ret == -1)
    {
        printf("%s\n", strerror(errno));
        exit(ret);
    }

    ret = listen(listen_fd, 5);

    struct sockaddr_in client_address{};
    socklen_t client_address_len = static_cast<socklen_t>(sizeof client_address);

    int client_fd = accept(listen_fd, (struct sockaddr*)&client_address,
            &client_address_len);

    sleep(5);
    char msg_first[20] = {"first message\n"};
    send(client_fd, msg_first, strlen(msg_first), 0);
    // 第一条信息发送后收到RST

    sleep(2);
    char msg_second[20] = {"second message\n"};
    send(client_fd, msg_second, strlen(msg_second), 0);
    // 第二次信息发送后出现SIGPIPE

    return 0;
}
```

测试代码如上 运行nc 127.0.0.1 9123后立刻按下Ctrl+C杀死client  截获的所有数据包如下

```
[root@fish ~]# tcpdump -i lo -Xx tcp port 9123
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
00:26:27.780746 IP localhost.40580 > localhost.grcp: Flags [S], seq 2027132643, win 43690, options [mss 65495,sackOK,TS val 1567772563 ecr 0,nop,wscale 7], length 0
	0x0000:  4500 003c db97 4000 4006 6122 7f00 0001  E..<..@.@.a"....
	0x0010:  7f00 0001 9e84 23a3 78d3 96e3 0000 0000  ......#.x.......
	0x0020:  a002 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
	0x0030:  5d72 4f93 0000 0000 0103 0307            ]rO.........
00:26:27.780802 IP localhost.grcp > localhost.40580: Flags [S.], seq 2570863062, ack 2027132644, win 43690, options [mss 65495,sackOK,TS val 1567772563 ecr 1567772563,nop,wscale 7], length 0
	0x0000:  4500 003c 0000 4000 4006 3cba 7f00 0001  E..<..@.@.<.....
	0x0010:  7f00 0001 23a3 9e84 993c 41d6 78d3 96e4  ....#....<A.x...
	0x0020:  a012 aaaa fe30 0000 0204 ffd7 0402 080a  .....0..........
	0x0030:  5d72 4f93 5d72 4f93 0103 0307            ]rO.]rO.....
00:26:27.780839 IP localhost.40580 > localhost.grcp: Flags [.], ack 1, win 342, options [nop,nop,TS val 1567772563 ecr 1567772563], length 0
	0x0000:  4500 0034 db98 4000 4006 6129 7f00 0001  E..4..@.@.a)....
	0x0010:  7f00 0001 9e84 23a3 78d3 96e4 993c 41d7  ......#.x....<A.
	0x0020:  8010 0156 fe28 0000 0101 080a 5d72 4f93  ...V.(......]rO.
	0x0030:  5d72 4f93                                ]rO.

三次握手结束后 由于立刻按下了Ctrl+C 客户端连接异常结束 发送FIN

00:26:27.969734 IP localhost.40580 > localhost.grcp: Flags [F.], seq 1, ack 1, win 342, options [nop,nop,TS val 1567772752 ecr 1567772563], length 0
	0x0000:  4500 0034 db99 4000 4006 6128 7f00 0001  E..4..@.@.a(....
	0x0010:  7f00 0001 9e84 23a3 78d3 96e4 993c 41d7  ......#.x....<A.
	0x0020:  8011 0156 fe28 0000 0101 080a 5d72 5050  ...V.(......]rPP
	0x0030:  5d72 4f93                                ]rO.

服务端回复收到FIN

00:26:27.970294 IP localhost.grcp > localhost.40580: Flags [.], ack 2, win 342, options [nop,nop,TS val 1567772753 ecr 1567772752], length 0
	0x0000:  4500 0034 8f0d 4000 4006 adb4 7f00 0001  E..4..@.@.......
	0x0010:  7f00 0001 23a3 9e84 993c 41d7 78d3 96e5  ....#....<A.x...
	0x0020:  8010 0156 fe28 0000 0101 080a 5d72 5051  ...V.(......]rPQ
	0x0030:  5d72 5050                                ]rPP

第一次写入信息

00:26:32.781104 IP localhost.grcp > localhost.40580: Flags [P.], seq 1:15, ack 2, win 342, options [nop,nop,TS val 1567777564 ecr 1567772752], length 14
	0x0000:  4500 0042 8f0e 4000 4006 ada5 7f00 0001  E..B..@.@.......
	0x0010:  7f00 0001 23a3 9e84 993c 41d7 78d3 96e5  ....#....<A.x...
	0x0020:  8018 0156 fe36 0000 0101 080a 5d72 631c  ...V.6......]rc.
	0x0030:  5d72 5050 6669 7273 7420 6d65 7373 6167  ]rPPfirst.messag
	0x0040:  650a                                     e.

收到客户端返回的复位RST

00:26:32.781155 IP localhost.40580 > localhost.grcp: Flags [R], seq 2027132645, win 0, length 0
	0x0000:  4500 0028 0000 4000 4006 3cce 7f00 0001  E..(..@.@.<.....
	0x0010:  7f00 0001 9e84 23a3 78d3 96e5 0000 0000  ......#.x.......
	0x0020:  5004 0000 dffd 0000                      P.......

第二次写入信息触发 SIGPIPE
```

方式SIGPIPE中断进程使用`signal(SIGPIPE, SIG_IGN)`忽略这个信号, 这时send会返回-1并设置errno=EPIPE