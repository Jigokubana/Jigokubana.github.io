---
title: 循环判断中使用无符号类型
date: 2020-03-06 17:37:08
tags:
  - CPP踩坑记
  - CPP
categories:
  - CPP
  - CPP踩坑
---

下面的代码 会产生错误.
```c++
vector<int> test;
for (int i = 0; i < test.size() - 1; ++i)
{
}
```
无符号类型的数值和有符号类型的数值进行运算(包括算数逻辑运算等)会将有符号类型的数值转换成无符号类型的数值.

但是对于char short等小于int的数值运算的时候 会转换成int 纯正的int
对于unsigned char在转换成int时，无论最高位是0还是1，都补0
signed char的则高位时1补1 0补0
```c++
// 0*23 1
// 1*23 0
// 1*24 本来是int 但被当作了 无符号int 所以输出
unsigned int a = 1;
int b = -2;
if (a < b) // 逻辑运算也转换了
{
    cout << "a < b" << endl; // 输出
}
cout << a + b << endl; // 4294967295


//   11111111 11111111 11111111 11111111
//   00000000 00000000 00000000 00000010
// 1 00000000 00000000 00000000 00000001
unsigned int c = 2; // 0*22 1 0
int d = -1; // 1*24
if (c < d)
{
    cout << "c < d" << endl; // 输出
}
cout << c + d << endl; // 1   直接把高位的1溢出了 留下了0

// 2*23 1
// 1*23 0
// 1*24 被转换成了int 所以输出-1
unsigned char e = 1;
char f = -2;
if (e < f)
{
    cout << "e < f" << endl; // 没有输出
}
cout << e + f << endl; // -1

unsigned char g = 2;
char h = -1;
if (g < h)
{
    cout << "g < h" << endl; // 未输出
}
cout << g + h << endl; // 1
```