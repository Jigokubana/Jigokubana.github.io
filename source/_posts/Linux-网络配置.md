---
title: 网络配置
date: 2020-07-18 17:39:20
categories: 
- Linux
tags:
- 网络配置
---

# 服务器双网卡配置
内网访问外网

外网接在了校园网上, 不过我的路由器不支持Ipv6, 有没有多余的lan口 只能接在服务器的lan口上

需要设置内网网卡开机启动 固定的Ip地址10.5.1.2 以及子网掩码255.255.255.0
然后路由器设置固定Ip地址10.5.1.3 子网掩码255.255.255.0 与内网网卡同网段

然后使用`firewall-cmd --add-masquerade --permanent`

Masquerading is useful if the machine is a router and machines connected over an interface in another zone should be able to use the first connection.
通过另一个区域(zone)的接口 使用默认zone的接口上网

https://www.server-world.info/en/note?os=CentOS_7&p=firewalld&f=2
配置完才发现的神网址.... 好吧 写得非常清楚