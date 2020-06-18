---
title: Linux的使用
date: 2020-03-17 12:29:20
categories: 
- 操作系统
tags:
- Linux
---

| 命令 | 功能 |
| --- | --- |
| cp -r | 复制目录 |
| cd - | 切换到上一个工作目录 |


| 查找 | 功能 |
| --- | --- |
| find ./ -name "core*" | xargs file | 搜寻目录或文件 |
| find ./ -name '*.o' | 查找目标文件夹是否有obj文件 |
| find ./ -name "*.o" \| xargs rm -f | 递归删除当前目录所有obj文件 |


| 查看文件内容 | 功能 | 一般用法|
| --- | --- | --- |
| cat -n | 显示的同时显示行号 | 使用管道 ls \| cat -n |
| head -10 filename | 查看前十行 | |
| tail -10 failname | 查看后十行 ||
| diff file1 file2 | 查看文件差别 ||
| tail -f filename | 动态显示文本的最新信息 ||


| 给文件增加别名 | 功能 |
| ln a b | 创建硬链接, 删除一个另一个仍能使用 |
| ln -s a b | 创建符号链接(软连接) 删除源后另一个无法使用 |


命令行快捷键
Ctl-U   删除光标到行首的所有字符 `$ 1111 222  --> $ `
Ctl-W   删除光标到前边最近一个空格之间的字符 `$ 1111 222 --> $ 1111`


