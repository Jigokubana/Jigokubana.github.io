---
title: Mysql基本使用
tags:
  - Mysql
categories:
  - 数据库
  - Mysql
date: 2020-03-06 15:25:28
---

# 安装和配置
Ubuntu18.04 安装 mysql5.7
```shell
sudo apt update
sudo apt-get install mysql-server
```

安装完毕后
```shell
# 打开shell管理程序
sudo mysql

# 123456 换成你自己的密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

# 刷新一下
FLUSH PRIVILEGES;

# 退出
exit

# 登录测试
mysql -uroot -p # 回车 后输入自己的密码再次回车
```