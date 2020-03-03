---
title: 牛客整理
tags:
  - null
categories:
  - CPP
  - CPP基础
date: 2019-10-19 12:31:37
---

# 运算出结果类
```c++
char a=101;
int sum=200;
a+=27;sum+=a;
printf("%d\n",sum);
```
a为 -128~127 
 127  = 0111 1111
"128"= 1000 0000 (-128的补码)

关于1000 0000代表-128 我偶然发现一个博客有种恍然大悟的感觉
[浅析为什么char类型的范围是 —128~+127](https://blog.csdn.net/daiyutage/article/details/8575248)
这个博客实在是写的太好了

---
```c++
char a,b,c,d;
scanf("%c,%c,%d,%d",&a,&b,&c,&d);
printf("%c,%c,%c,%c",a,b,c,d);
}
```
%c进行输入会进行转化后存入
输入 1 即认为输入 '1' 转换成49存入

---

# 类相关
## 虚函数
纯虚函数可以让类先具有一个操作名称，而没有操作内容，让派生类在继承时再去具体地给出定义。
凡是含有纯虚函数的类叫做 抽象类 。这种类不能声明对象，只是作为基类为派生类服务.
除非在派生类中完全实现基类中所有的的纯虚函数，否则，派生类也变成了抽象类，不能实例化对象。

## 静态
静态成员可以作为默认实参

# 定义类

# 基础中的基础
这部分有些只保留题目, 不做解析 便于自己想象
```c++
unsigned int k = 20;
while (k >= 0)
{
	--k;
}
```

下面三个函数全部错误
```
void test1()
{
    unsigned char array[MAX_CHAR+1],i; // 这个函数在于MAX_CHAR可能会大于255导致i永远无法达到
    for(i=0;i<=MAX_CHAR;i++){
        array[i]=i;
    }
}
char*test2()
{
    char p[] = "hello world";
    return p;
}
char *p =test2();
void test3(){
    char str[10];
    str++;
    *str='0';
}
```
---

# 基础中的Api
虽然自己还没用过, 但还是要积累
```c++
for(i=4;i>1;i--)
{
	for(j=1;j<i;j++)
	{
		putchar('#');
	}
}
```
开始看到这个题目, 为putchar是放到缓冲区中, 结果就是正常的输出
