---
title: CMAKE
date: 2020-02-06 17:07:45
tags:
categories:
  - CPP
  - CPP重点
---
CMAKE并不打算专门去学, 所以就用到哪里学到哪里

今天clion写项目 遇到一个问题 明明写了aux_source_directory 却依然提示没有加入. 后来查了下, 并不推荐使用那个命令.
可能会存在一些问题
https://cmake.org/cmake/help/latest/command/aux_source_directory.html


```cmake
# 必须片段
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo1)
# 指定生成目标
add_executable(Demo main.cc)

# 多文件
# 如果一味地在add_executable中添加源文件, 会导致太长了
# 将dir目录中所有源文件保存在变量中
aux_source_directory(. DIR_SOURCE)
# 将变量赋值给Demo
add_executable(Demo ${DIR_SOURCE})

# 多文件多目录
# 需要在主目录和子文件夹中都编写CMakeLists.txt文件

# 主文件添加子目录
add_subdirectory(dir1)
# 添加链接库
target_link_libraries(Demo Foo)

# dir1目录中
aux_source_directory(. DIR1_SOURCE)
# 生成链接库 Foo 在主文件中添加即可
add_library(Foo ${DIR1_SOURCE})
```

# CMAKE和MAKE之间的区别
[博客原文](https://my.oschina.net/xunxun/blog/86781)

自己的理解
通过为cmake编写CMakeList文件, cmake即可按照规则生成相应的Makefile文件
然后make读取Makefile文件就可以按照规则将源代码编译

总的来说cmake就是为make生成Makefile文件, (Makefile文件可以自己编写, 也可以用Cmake生成)
应该是编写CMakeList的代码量少于或者简单于Makefile, 简化了操作