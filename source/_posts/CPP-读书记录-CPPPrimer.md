---
title: C++Primer学习
tags:
  - null
categories:
  - CPP
  - 服务器编程-书籍记录
date: 2019-08-16 19:13:22
top: 9

img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%B0%81%E9%9D%A2/C%2B%2BPrimer.png
---

# 基础知识
## c++11 的{}初始方式
```c++
// c++11 的初始化方式, 有助于防范类型转换错误.
int a1 = { 24 };
int b1 = {}; // 默认为0

// 没有 = 同样可以
int a2 { 24 };
int b2 {}; // 默认为0

return 0;
```

无符号的变量在 超出范围的时候对应变化
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/C%2B%2B%E5%9F%BA%E7%A1%80%E5%A4%8D%E4%B9%A0/%E6%9C%89%E7%AC%A6%E5%8F%B7%E5%92%8C%E6%97%A0%E7%AC%A6%E5%8F%B7%E7%9A%84%E9%87%8D%E7%BD%AE%E7%82%B9.jpg)


## 转义字符表
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/C%2B%2B%E5%9F%BA%E7%A1%80%E5%A4%8D%E4%B9%A0/c%2B%2B%E8%BD%AC%E4%B9%89.jpg)

**char 在默认情况下, 既不是有符号. 也不是无符号.**
```c++
char a; // 此时a可能是 signed char 也可能是 unsigned char 可能会在不同系统之前产生错误
// 可以显式声明, 来确保不会出现此错误
signed char a;
unsigned char a; 
```

**输入**
```c++

char name[10];
cin.getline(name, 10); // 可以读取换行符, 但不保存
cin.get(name, 10); // 不读取换行符, 可能导致get到换行符

// 空行和超出长度问题 将在后面说明
```
运算符优先级
![](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=59a3e1017d3e6709aa0d4dad5aaef458/63d9f2d3572c11df57c9a205612762d0f703c2f8.jpg)

**基础的数组**
```c++
int a[] = {1, 2, 3, 4};
// 下方由于 new 只是返回一个地址所以用 int* 接收 
int* b = new int[4];
cout << a[0] << endl; // 1
cout << *(a + 1) << endl;// 2
```
**对指针解引用**
第一印象是 解除引用, 然而并不是
```c++
/*
"*"的作用是引用指针指向的变量值，引用其实就是引用该变量的地址，“解”就是把该地址对应的东西解开，解出来，就像打开一个包裹一样，那就是该变量的值了，所以称为“解引用”。也就是说，解引用是返回内存地址中保存的值。
比如int a=10; int *p=&a;
cout<<*p<<endl; 输出a的值，就是解引用操作。
*/
```
## c 风格字符串相关
对字符数组的赋值, 不建议使用 = 赋值, 可能会导致内存覆盖
建议使用 `strncpy(目标位置, 字符串, 长度)` 然后手动在目标字符数组最后一位写入`\0`
这样安全, 不过这是c风格的, 在c++中可以使用**string来代替字符数组**

## 自动变量 静态存储 动态存储
自动变量
```c++
char * getInut() {
	char temp[100]; // 局部变量(自动变量), 函数结束时自动释放 存入栈中
	cin >> temp;
	char* pn = new char[strlen(temp) + 1];
	strcpy(pn, temp);

	return pn;
}
```
动态存储 通过new 和 delete操作内存池(自由存储空间, 堆)

## 宏 异常处理  函数相关
| 定义 | 说明 |
| ----- | ----- |
| `__FILE__` | 存放文件名的字符串字面值 |
| `__LINE__` | 存放当前行号的整形字面值 |
| `__TIME__` | 存放文件编译时间的字符串字面量 |
| `__DATE__` | 存放文件编译日期的字符串字面值 |
| `__VA_ARGS__` | 用来接受函数参数中`...`, 类似printf函数 |

**异常处理**
代码可以使用throw来抛出异常
*大部分*可以指定msg来初始化异常
throw exception_type("msg")
表中exception不能指定msg, 来初始化, 除此外表中其他都需要msg初始

| 错误名称        | 对应原因                       |
| ---------------- | ---------------------------------- |
| exception        | 最常见的问题                       |
| runtime_error    | 只有运行的时候才能查到错误         |
| range_error      | 运行时错误: 超范围                 |
| overflow_error   | 运行时错误: 上溢                   |
| underflow_error  | 运行时错误: 下溢                   |
| logic_error      | 程序逻辑错误                       |
| domain_error     | 程序逻辑错误: 参数对应的结果不存在 |
| invalid_argument | 程序逻辑错误: 无效参数             |
| length_error     |                                    |
| out_of_range     | 程序逻辑错误: 超范围               |

**默认参数**
函数可以设定默认参数.

在调用设置有默认参数的函数时只能省略右边的带默认值参数
如果一个参数设置了默认值, 则其右边的参数都需要设置默认值

如果一个函数在头文件中已经声明了默认参数, 实现的时候不能更该已经设定的默认参数

但可以将未设置默认参数的函数设置默认参数

# 动态内存-自由空间(堆)
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/C%2B%2B%E5%9F%BA%E7%A1%80%E5%A4%8D%E4%B9%A02/%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.png)

## 动态内存与智能指针
| 共同操作 | 描述 |
| --- | --- |
| shared_ptr\<T\> sp | 空智能指针, 指向类型为T的对象 |
| unique_ptr\<T\> up | |
| p | 将p作为一个条件判断 若p指向对象则为true |
| \*p | 解引用, 获得指向的对象 |
| p->mem | 等价于 (\*p).mem |
| p.get() | 返回p中保存的指针, 若智能指针释放了对象, 这个函数返回的指针就成了垂悬指针|
| swap(p, q) p.swap(q) | 交换p q中的指针|

| shared_ptr 独有操作 | 描述 |
| -- | --- |
| make_shared\<T\>(args) | 返回一个shared_ptr 指向一个动态分配的T类型对象, 使用args初始化对象 |
| shared_ptr\<T\> p (q) | p是shared_ptr的拷贝, 此操作会递增q中的计数器. q中的指针必须能转换为T* |
| p = q | 两者均为shared_ptr 所保存的智能指针必须能相互转换, 此操作会递减p的引用计数, 增加q的引用计数, 当p的引用计数为0则其管理的原内存释放 |
| p.unique() | return p.use_count() == 1 |

**make_shared**
最安全的分配和使用动态内存的方法
```c++
// p指向 "pppppppppp"的string
shared_ptr<string> p = make_shared<string>(10, 'p');
// 一般使用auto
auto p = make_shared<string>(10, 'p');
```
**shared_ptr的拷贝和赋值**
当进行拷贝或者赋值操作时候 shared_ptr都会记录有多少个其他的shared_ptr
指向相同的对象
引用计数增加的情况
- 拷贝shared_ptr 
- 初始化其他shared_ptr指针
- 作为参数传递给一个函数
- 作为函数的返回值

引用计数减少
- 赋予新值
- 被销毁(例如离开作用域)

当引用计数为0的时候, 就会释放自己管理的对象
```c++
auto r = make_shared<int>(42);
r = q; // 给r赋值, 令他指向另一个地址
		// q原来指向对象的引用计数 递增
		// r 原来指向对象的引用计数 递减
		// r的引用计数 为0 则自动释放
```

**使用动态内存的原因**


使用unique_ptr的时候要注意
1. 不要再函数调用传参的括号中 使用临时变量 这样一旦函数调用完成就会被销毁
```c++
void process(shared_ptr<int> ptr)

shared_ptr<int> p(new int(42 )) ; // 引用 = 1
process(p); // 拷贝 会递增它的引用计数 ;在 process 中引用计数位为2
int i = *p; // 正确 引用计数为1

int *x(new int(1024)); // 危险 这是一个普通指针，不是一个智能指针
process(x) ; // 错误 不能将 int* 转换为 一个 shared_ptr<int> 
process(shared_ptr<int> (x)); // 合法的，但内存会被释放! 因为临时对象会被销毁
int j =*x //未定义的 是一个空悬指针!
```
2. 如果使用unique_ptr不要使用get函数初始化另一个智能指针或为其赋值
因为一旦新生成的智能指针离开作用域或者被释放, 会影响到原来的智能指针
```c++
std::shared_ptr<int> p = new int(8); // 不能将一个int* 赋值给shared_ptr<int>

std::shared_ptr<int> p1;
p1.reset(new int(8)); // 可以

std::shared_ptr<int> p2(new int(8)); // OK

// 使用make_share来创建shared_ptr指针
std::shared_ptr<int> p3 = std::make_shared<int>(20);
auto p4 = std::make_shared<int>(20);

// make_share不能用来创建unique_ptr
std::unique_ptr<int> d;
std::unique_ptr<int> d1 (new int(8)); 
```
unique_ptr不支持普通的赋值和拷贝操作, 因为unique_ptr要独占所指向的对象

```c++
std::unique_ptr<int> d1 (new int(1)); 
std::unique_ptr<int> d2 (new int(8)); 

d1.release(); // 释放原来所指向的对象
d1.reset(d2.release()); // d2释放后由d1获取
```