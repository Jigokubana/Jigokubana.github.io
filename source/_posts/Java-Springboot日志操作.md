---
title: Springboot日志操作
tags:
categories:
  - Java
date: 2019-05-18 14:14:29
---
<div class="alert-red">springboot</div>
<div class="alert-blue">日志</div>
<div class="alert-green"></div>
<!--more-->

### 日志
```java
Logger logger = LoggerFactory.getLogger(getClass());
logger.trace();
logger.debug();
logger.info();
logger.warn();
logger.error();

springboot 默认日志级别为info及以上
```

### 日志配置
```java
# com.xyz 下日志级别
logging.level.com.xyz=trace
# 设置root级别 设置默认级别
logging.level.root=debug

#输出到当前项目根路径下的 springboot.log 文件中
#logging.file=springboot.log
#输出到当前项目所在磁盘根路径下的 /springboot/log目录中的 spring.log 文件中,
logging.path=springboot/log
```
### 更改输出格式
```java
# 日志输出格式说明：
# %d 输出日期时间，
# %thread 输出当前线程名，
# %-5level 输出日志级别，左对齐5个字符宽度
# %logger{50} 输出全类名最长50个字符，超过按照句点分割
# %msg 日志信息
# %n 换行符
# 修改控制台输出的日志格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 修改文件中输出的日志格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} >>> [%thread] >>> %-5level >>>
%logger{50} >>> %msg%n
```