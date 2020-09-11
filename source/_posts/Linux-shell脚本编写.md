---
title: Shell脚本编写
date: 2020-03-26 17:30:22
categories: 
- Linux
tags:
- Shell
---

由于阿里云服务器github速度异常缓慢, 自己想把一个shell, 改成本地版本发现不是很容易.
所以咕咕咕很久的shell脚本, 算是开始学了. 感觉入门应该不会太难把

我先来我最喜欢的入门网站 菜鸟教程!

既然写的是脚本就需要解释器了
1. `#!/bin/bash` 这个通常见于脚本的第一行, `#!`算是个约定了 说明脚本需要什么解释器
2. `/bin/sh xx.sh` 这种是执行的时候指定, 脚本第一行的那个就失效了

**shell变量-挺正常的规定**
- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

shell变量类型
- 局部变量
- 环境变量
- shell变量-shell内置特殊变量

变量定义 *注意 不能随便加空格 下面的等号两侧不能加空格*
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

单双引号字符串
```shell
# shell 双引号字符串
# # 可以含有转义字符 和变量
xxx1="\"lsmg\""
echo xxx1 # $ "lsmg"


# shell单引号字符串
# # 会原样输出, 无视变量. 即使转义也不能出现单一的单引号 但可以成对出现 用于拼接字符串
xxx2='lsmg' 

# 拼接双引号变量双引号输出 单引号变量单引号输出
your_name="runoob"
# # 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1 # hello, runoob ! hello, runoob !
# # 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3 # hello, runoob ! hello, ${your_name} !
```

字符串操作
```shell
url="https://blog.lsmg.xyz"

echo ${#url} # 输出长度 21

# 注意单引号
echo `expr index "$url" go` # 输出 字符 g o的位置 哪个先出现就输出哪个

# 字符串截取
# # 范围截取
echo #{url:0:5} # 输出 https 第二个数字代表字符个数 不是范围

# # #截取删除左边字符, 保留右侧字符
echo #{url#*//} # blog.lsmg.xyz

```

数组
```shell
# 空格分隔
name=(lsmg Lsmg Jigokubana);

# 你甚至可以 不连续定义
nickname[0]=lsmg
nickname[2]=Lsmg

# 读取
echo ${nickname[0] nickname[1] nickname[2]}
# 输出所有
echo ${name[@]}
# 取得数组元素个数
echo ${#name[@]}
# 取得单个元素长度
echo ${#name[1]}
```

参数
```shell
$0
$1
$2
# .... 为具体某个参数

$# # 参数个数 不包括$0
$@ # 参数数组 解析为一个数组
$* # 参数字符串 解析为一整个字符串

```