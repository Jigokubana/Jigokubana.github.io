---
title: 浏览器访问URL到显示
date: 2021-2-1 16:55:22
categories: 
- 基础知识杂烩
tags:
---

# http

![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/base/ip-wireshark.png)


浏览器访问虚拟机创建的nginx默认网页

浏览器根据输入的URL中的IP 进行三次握手建立TCP连接 然后浏览器发送GET请求 服务器回复GET请求 获取各种文件

一般服务器对`GET /`默认返回index.html页面 `https://www.baidu.com/index.html`自己加上index后同样可以访问 但是`https://www.qq.com/index.html`却不行 所以还是跟服务器处理逻辑有关

返回了html页面后浏览器解析其中的内容, 请求css, js, 图片资源等文件 然后进行渲染显示出网页

文件传输完毕后TCP四次挥手关闭连接

如果使用的是域名访问, 则需要访问DNS服务器解析域名获得IP地址, 这样HTTP请求头部还会包含一些特殊的头部如HOST等

HTTP访问的默认端口均为80端口


# https

https进行域名解析后, ip+443端口连接服务器

测试时访问的连接为`https://m.baidu.com/?pu=sz%401321_480` 百度精简版页面, 修改了host防止负载均衡等策略ip不对应

![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/base/https-wireshark.png)


504 505 506 三个包进行TCP三次握手与`110.242.68.9:443`建立连接

507 客户端发送Client Hello 发送随机的key1和自己支持的加密方法

508 ACK应答

509 510 511 服务端发送Server Hello 发送随机的key2和选定的加密方法

512 服务端发送证书

513 ACK应答

514 服务器发送Server Key Exchange

515 服务器发送Server Hello Done

516 ACK应答

518 客户端发送Client Key Exchange, Change Clipher Spec, Finished

519 客户端发送经过加密的GET请求(Wireshark进行了解密)

520 521 进行ACK应答

522 523 524 服务端发送New Session Ticket, Change Clipher Spec, Finished

556 服务器回复GET请求


客户端向服务器发送请求(随机数key1 和支持的加密算法)

服务器回复数字证书(随机数key2 选择的双方支持算法 证书)

客户端验证数字证书可靠性, 使用CA的公钥解密加密后的证书. 验证证书的合法性

客户端生成随机数key3, 根据key1, key2, key3生成会话秘钥, 用服务器发送的证书加密key3发送给服务器

服务器解密key3同样根据key1, key2, key3生成会话秘钥

后续就通过会话密钥进行加密数据和解密数据