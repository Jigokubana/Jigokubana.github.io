---
title: Shell脚本编写
date: 2021-1-2 22:00:22
categories: 
- Linux
tags:
- Shell
---

# 第一行 - 解释器

1. `#!/bin/bash` 这个通常见于脚本的第一行, `#!`算是个约定了 说明脚本需要什么解释器
2. `/bin/sh xx.sh` 这种是执行的时候指定, 脚本第一行的那个就失效了


# 变量

规范
- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

shell变量类型
- 局部变量
- 环境变量
- shell变量-shell内置特殊变量

变量定义 注意 不能随便加空格 下面的等号两侧不能加空格
```shell
# 直接赋值
xxx="lsmg"
xxx = "lsmg" # 错误
xxx="Jigokubana" # 随意修改

# 语句赋值
for skill in Ada Coffe Action Java
do
    echo "I am good at ${skill}Script"
done

# 使用变量
echo $xxx
echo ${xxx} # {} 用于标记边界
echo "my name is ${xxx}"

# const
readonly xxx="lsmg" 

# 删除变量
unset xxx; 不能用于删除只读变量
```

# 单引号 双引号 反引号

单引号将剥夺其中的所有字符的特殊含义

双引号中的`$`（参数替换）和`（命令替换）有特殊含义

反引号与$()功能相同会执行其中的命令
```shell
aaa="aaa"

b1="$aaa"
b2=`$aaa` # 会先进行取变量值 $aaa -> aaa
b3='$aaa'

echo $b1 # aaa
echo $b2 # aaa: command not found 
echo $b3 # $aaa
```