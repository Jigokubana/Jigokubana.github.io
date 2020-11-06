---
title: HTTP框架底层原理实现
tags:
  - null
categories:
  - 分享会
date: 2020-11-1 22:45:39
---

# 框架

后端组使用各种框架
- Java的Springboot
- Python的Django

框架的底层原理是什么? 当然涉及原理性的东西太多了, 我这里指的是为什么客户端的几次简单点击 最终会转化成了后端的函数调用?

Springboot针对某一个URL能Get Post等等对应的方法. 还能得到指定的参数?

客户端的几下简单的点击就能够将数据传送到后端

后端也只要在相关的函数后处理接收到的数据 并返回即可

# 基础

要解释上面的问题需要用到计算机学院好几门专业课的只是 <操作系统> <计算机组成原理 跳过> <计算机网络>

基础知识我要不要学? HTTP协议? DNS怎么发挥的作用?

GET POST 有什么区别?

无法访问网页怎么办? 浏览器访问流程

校园网断网跳转怎么实现的? DNS污染又是什么?

favicon? 如何实现的自动加载

# 浏览器访问流程

当你输入一个网址, 按下回车后 首先会向DNS服务器发送DNS请求将域名转换成IP. *DNS污染*

然后向 IP+端口号 **建立连接 强调!** 发送HTTP请求 (HTTP默认是80端口)

然而收发HTTP并不是 操作系统提供的功能 那是怎么来的?

# HTTP协议

协议是什么?

文本协议

```
GET / HTTP/1.1
Host: 10.2.5.251
Connection: keep-alive
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,ja;q=0.8


GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
DNT: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,ja;q=0.8
Cookie: BIDUPSID=275AD930CBDC20F9C9E074230710B72E; PSTM=1602485007; BAIDUID=93E8E77E51A5F22CA01131559BA8666A:FG=1; BD_UPN=12314753; BDUSS=BjRVJYdFQ5TEJKWWNPck1ZcWdOVklMT0JHMHhQMUx3UGxDZjhNMVVhRmdqcXRmSVFBQUFBJCQAAAAAAAAAAAEAAADZQdlRcmpkNjc0NDEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGABhF9gAYRfM3; BDUSS_BFESS=BjRVJYdFQ5TEJKWWNPck1ZcWdOVklMT0JHMHhQMUx3UGxDZjhNMVVhRmdqcXRmSVFBQUFBJCQAAAAAAAAAAAEAAADZQdlRcmpkNjc0NDEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGABhF9gAYRfM3; BAIDUID_BFESS=F500721F63FBCA4EDC7DA170AE314FEE:FG=1; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; COOKIE_SESSION=332523_0_8_0_3_6_1_0_7_4_42_0_332523_0_6_0_1604291277_0_1604291271%7C8%230_0_1604291271%7C1; sug=3; sugstore=0; ORIGIN=0; bdime=0; ZD_ENTRY=google


POST http://202.119.196.6:8080/Self/login/verify HTTP/1.1
Connection: keep-alive
Cache-Control: max-age=0
Origin: http://202.119.196.6:8080
Upgrade-Insecure-Requests: 1
DNT: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://202.119.196.6:8080/Self/login/?302=LI
Accept-Language: zh-CN,zh;q=0.9,ja;q=0.8
Host: 202.119.196.6:8080
Accept-Encoding: gzip, deflate, br
Content-Length: 60
Cookie: JSESSIONID=E56420ED06E0ABE922C5A904DBB9C944

foo=&bar=&checkcode=1234&account=08180000&password=md5&code=
```

根据这个格式 请求方法 请求路径 HTTP版本

HEADER部分 需要什么HEADER加入什么

BODY部分 放入数据

所根据的这个格式 这个规定就叫做协议 HTTP的规定就是HTTP协议

*Fiddler包装好的东西*

所以说你在手机上查询电费也好 电脑上浏览网页也罢 前端调用后端也是如此 都会将数据按照这个格式进行打包

流程就是建立连接后 发送打包的数据 所以可以抓包后模拟请求这就这么来的

# TCP ; 框架什么?

![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/CPP%E5%88%86%E4%BA%AB%E5%AD%A6%E4%B9%A0%E4%BC%9A/%E4%B8%83%E5%B1%82%E5%92%8C%E5%9B%9B%E5%B1%82%E5%8D%8F%E8%AE%AE.png)

*将根据这张图说一说, 经常让了解的 七层和四层协议.  (网络访问(链接)层)*

上边面说的建立连接建立的实际是TCP连接

TCP连接是通过 IP和端口号建立的 正好使用的上面的IP和端口号

TCP连接干什么?  连接就是传输数据 这就涉及到了读写操作  全双工

两个套接字之间的通信 客户端持有一个套接字 服务器持有一个套接字 然后客户端的数据发送和服务器端的数据接收就是转化为了针对套接字的读写

套接字的建立和管理都比较复杂, 但是流程极为固定 干脆将这些代码包装起来 对外界提供一个简单的函数即可  框架内部帮你管理

客户端指的就是你的浏览器也好, 你的应用也好 套接字这个核心的东西被框架包装了起来

从上边知道输入网址也好, 使用form标签也罢. 最终都是根据HTTP协议包装了. 然后将字节流发送 (流, 流水 按顺序流入)

*nc 命令*



所以你可以没有域名 照样可以使用HTTP协议 照样访问网页.

*域名 区分作用*

# nginx?



# 自学能力
- 大二转LinuxC++
- 百度百度百度

开阔眼界 StackOverflow 看书


- 视频?
- 总结?
- 看书?
- 文档?

# 大学生
- 社交 我用大二上的社交换到了Linux入门
- 大三上减肥(身高), 参加活动, 锻炼身体, 找对象? 规律作息