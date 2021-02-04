---
title: Docker使用
date: 2021-1-31 23:34:22
categories: 
- Linux
tags:
- docker
---


# 安装Mysql

创建容器

```shell
yum install docker

# https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 配置镜像加速


docker pull mysql # 拉取镜像

docker run -itd --name mysql-live -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456789 mysql # 创建容器

docker ps # 查看容器状态
```

配置远程账号

```shell
docker exec -it mysql-live bash # 进入容器

mysql -uroot -p123456789

CREATE USER 'lsmg'@'%' IDENTIFIED WITH mysql_native_password BY '123456789'; # 创建远程用户

GRANT ALL PRIVILEGES ON *.* TO 'lsmg'@'%';

flush privileges;
```

# 导出导入文件

从容器导出文件

```shell

# 导出mysql容器的sql导出文件

docker cp mysql-live:/live_user.sql /root
```