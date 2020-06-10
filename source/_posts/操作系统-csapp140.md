---
title: csapp笔记
date: 2020-06-07 9:09:20
categories: 
- 操作系统
tags:
- csapp

top: 140
---

# 第一章 计算机系统漫游

![](http://lsmg-img.oss-cn-beijing.aliyuncs.com/csapp/1-3%E7%BC%96%E8%AF%91%E7%B3%BB%E7%BB%9F.png)

预处理->编译->链接->汇编

```c++
// hello.cpp
#include <cstdio>
#define PI 3.14

int main()
{
    // 1111
    //
    printf("%f\n", PI);
    printf("hello, world!\n");
    return 0;
}
```
预处理
`g++ -E hello.cpp > hello.i`
`tail -9 hello.i`
```c++
# 5 "hello.cpp"
int main()
{


    printf("%f\n", 3.14);
    printf("hello, world!\n");
    return 0;
}
```
宏替换和注释消失 但是空行还在

编译 生成汇编
`g++ -S hello.i`
```s

        .file   "hello.cpp"
        .text
        .section        .rodata
.LC1:
        .string "%f\n"
.LC2:
        .string "hello, world!"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $16, %rsp
        movq    .LC0(%rip), %rax
        movq    %rax, -8(%rbp)
        movsd   -8(%rbp), %xmm0
        leaq    .LC1(%rip), %rdi
        movl    $1, %eax
        call    printf@PLT
        leaq    .LC2(%rip), %rdi
        call    puts@PLT
        movl    $0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .section        .rodata
        .align 8
.LC0:
        .long   1374389535
        .long   1074339512
        .ident  "GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0"
        .section        .note.GNU-stack,"",@progbits

```
汇编 生成 可重定位二进制目标程序
`g++ -c hello.s`

链接 生成执行文件
`g++ hello.o -o hello`

在不同进程间切换 交错执行的执行-上下文切换
操作系统保持进程运行所需的所有状态信息 这种状态-上下文

线程共享代码和全局数据


# 第一部分 程序结构和执行
## 第二章 信息存储

孤立地讲 单个位不是非常有用. 然而, 当把位组合在一起, 再加上某种解释,即赋予不同的可能位组合不同的含义, 我们就能表示任何有限集合的元素.




## 第五章 优化程序性能

编写高效的程序
- 选择一组适当的算法和数据结构
- 编写出编译器能够有效优化以转变为高效的可执行程序的源代码
- 大项目的并行计算


## 第六章 存储器层次结构