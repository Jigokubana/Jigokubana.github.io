---
title: C++Primer学习
tags:
  - null
categories:
  - CPP
  - 服务器编程-书籍记录
date: 2019-08-16 19:13:22
top: 80

img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%B0%81%E9%9D%A2/C%2B%2BPrimer.png
---

# TOOD
explicit 参见7.5.4节 265

# 第一部分 基础知识
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
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/CPPPrimer/%E6%9C%89%E7%AC%A6%E5%8F%B7%E5%92%8C%E6%97%A0%E7%AC%A6%E5%8F%B7%E7%9A%84%E9%87%8D%E7%BD%AE%E7%82%B9.jpg)


## 转义字符表
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/CPPPrimer/c%2B%2B%E8%BD%AC%E4%B9%89.jpg)

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
| `__VA_ARGS__` | 用来接受函数参数中`...`, 类似printf函数, 这个宏只能在宏中使用 |

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


# 第二部分 cpp标准库

## 动态内存-自由空间(堆)
![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/CPPPrimer/%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88.png)


## 动态内存与智能指针

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

**使用动态内存的原因**
- 程序不知道自己需要使用多少对象 容器类
- 程序不知道所需对象的准确类型
- 程序需要在多个对象间共享数据

**程序需要在多个对象间共享数据**
一般情况
```c++
vector<string> v1; // empty
{ // 新的作用域
	vector<string> v2 = {"a", "b", "c"};
	v1 = v2;
} // 离开作用域 v2被销毁
// v1中有三个元素, 是原来三个元素的拷贝
```

下面是我们要实现的情况
```c++
vector<string> v1; // empty
{ // 新的作用域
	vector<string> v2 = {"a", "b", "c"};
	v1 = v2; // v1 v2共享相同的元素
} // 离开作用域 v2被销毁 但v2的元素不能被销毁
// v1中有三个元素, 指向原来三个元素
```

创建一个类来管理这个vector, 但是不能直接保存在其中, 因为当这个类的对象销毁的时候内部的vector也就销毁
所以需要保存在动态内存中

所以使用shared_ptr来管理动态分配的vector, 当最后一个vector的使用者被销毁的时候才释放内存

确定下这个类提供的操作
- 修改元素的操作会进行校验 不合法则抛出异常
- 默认构造函数
- 接受单一的initiaizer_list\<string>类型的参数. 这个参数可以接受一个初始器的花括号列表


**直接管理内存**

了解到两个概念 一个`默认初始化`一个`值初始化`

```c++
int i, *pi1 = &i, *pi2 = nullptr;
double *pd = new double(33), *pd2 = pd;
delete i; // 错误 i不是指针
delete pi1; // 未定义 pi1指向一个局部变量 具有潜在性危害
delete pd; // ok
delete pd2; // 未定义 内存已经被释放 具有潜在性危害
delete pi2; // 正确 释放空指针没有错误

const int *pci = new const int(1024);
delete pci // 正确 释放一个const对象

// delete之后指针就变成了`空悬指针` 需要将其置为`nullptr`
```

**shared_ptr和new结合使用**

```c++
// 错误智能指针构造函数是explicit
// 无法将内置指针隐式转换成一个智能指针, 必须使用直接初始化
shared_ptr<int> p1 = new int(1024); 

// 正确  使用了直接初始化
shared_ptr<int> p2(new int(1024));
```

定义和改变方式
```c++
// p管理内置指针q所指向的对象, q必须指向new分配的内存, 且能够转换为T*类型
shared_ptr<T> p(q)

// p从unique_ptr u那里接管了对象的所有权 将U置为空
shared_ptr<T> p(u)

// p 接管了内置指针q所指向的对象的所有权, q必须能转化为T* 类型
// p将使用可调用对象d来代替delete
shared_ptr<T> p(q, d)

// p是shared_ptr p2的拷贝
// p将用可调用对象d来代替delete
shared_ptr<T> p(p2, d)

// 若p是唯一指向其对象的shared_ptr, reset会释放此对象
// 如果传递了可选的参数-内置指针q, 则会令p指向q 否则将p置为空
// 如果还传递了d, 会调用d而不是delete来释放q
p.reset()
p.reset(q)
p.reset(q, d)
```

**注意**

使用unique_ptr的时候要注意
1. 不要在函数调用传参的括号中 使用临时变量 这样一旦函数调用完成就会被销毁
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

2. 如果使用智能指针不要使用get函数初始化另一个智能指针或为其赋值
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


3. unique_ptr不支持普通的赋值和拷贝操作, 因为unique_ptr要独占所指向的对象
```c++
std::unique_ptr<int> d1 (new int(1)); 
std::unique_ptr<int> d2 (new int(8)); 

d1.release(); // 释放原来所指向的对象
d1.reset(d2.release()); // d2释放后由d1获取
```

**智能指针陷阱**
1. 不使用相同的内置指针值初始化(或reset)多个智能指针
2. 不使用delete get返回的指针
3. 不使用get初始化或者reset另一个智能指针
4. 如果你使用get返回指针, 当最后一个对应的智能指针销毁 这个指针就变成了垂悬指针
5. 使用智能指针管理非new分配的内存, 需要传递一个删除器.

**weak_ptr**
不控制所指向对象生存周期, 指向一个shared_ptr管理的对象.



由于weak_ptr不参与对应的shared_ptr的引用计数, vector可能已经被释放了
释放后 lock将返回一个空指针


# 第三部分 类设计者的工具
## 模板与泛型编程

`template <typename T>`

编译器用推断出的模板参数来`实例化`一个特定版本的函数.
不同的模板参数类型`实例化`出不同的函数 然后进行调用.

**模板类型参数**

如果有多个模板参数
```c++
// 错误
template <typename T, U>

// 正确
template <typename T, typename U>

// 正确
template <typename T, class U>
```

**非类型模板参数**

```c++
template <unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M])
{
	return strcmp(p1, p2);
}

compare("hi", "mmm");
int compare(const char (&p1)[3], const char (&p2)[4]) // 注意空字符
```
编译器会使用字面常量的大小来代替N和M, 从而实例化模板

非类型参数可以使`整形`或者是一个指向对象或者函数类型的`指针`或者`(左值)引用`
绑定到`非类型整数参数`的实参必须是一个常量表达式.
绑定到`指针`或者`引用非类型模板参数`的实参必须具有静态的生存期

静态内存用来保存`局部static对象`, `类static数据成员`和`定义于任何函数之外的变量`
栈内存用来保存定义在函数内的非static对象.
分配在静态内存和栈内存的由编译器自动创建和销毁

堆 用来存储动态分配的对象


编译器遇到一个模板类型定义的时候并不会生成代码, 而是实例化出一个特定版本的时候才会实例化


**使用类的类型成员**

```c++
template <typename T>

T::size_type* p;
```
默认情况下, C++语言假定通过域运算符访问名字而不是类型.
如果想要使用模板类型参数(T)的类型成员(size_type) 必须使用`typename`而不是`class`


**默认模板实参**

```c++
template <typename T, typename F = less<T>>
int compare(const T& v1, const T& v2, F f = f())
{
	if (f(v1, v2)) return -1;
	if (f(v2, v1)) return 1;
	return 0;
}
```

# 第四部分 高级主题

tuple这里看了, 例子写了总感觉实用性不是很大? 或许可以用在便捷处理输入参数?

biset感觉还是有点用
```c++
std::bitset<32> b;

// 以下位不足 高位都补0

unsigned long long u = ULLONG_MAX;
std::bitset<32> bu(u);

std::string s1 = "10101010101010";
std::bitset<32> bs1(s1);
std::string s2 = "ababaabbababb";
// std::bitset<32> bs2(s2); std::invalid_argument
// a instand 0 b instand 1
std::bitset<32> bs2(s2, 0, 32, 'a', 'b');

bu.any(); // 存在 置位(1) 的二进制位吗
bu.all(); // 所有位都置位了吗
bu.none(); // b中不存在置位的二进制位吗
bu.count(); // 置位数目
bu.size(); // 数目

bu.test(1); // 位置1是置位的返回true 否则返回false
bu.set(1, true); // 将1位 置 true->1 false->0
bu.reset(1); // 将1位复位 置0
bu.reset(); // 将所有位复位 置0

bu.flip(1); // 1位取反
bu.flip(); // 全部取反
bu[1].flip();

bu[1]; // 1位 1->true 0->false

bu.to_ulong();
bu.to_ullong();

bu.to_string('a', 'b'); // 0->a 1->b
```

