---
title: Effective读书记录
date: 2020-02-26 21:43:46
tags:
categories:
 - CPP
 - 服务器编程-书籍记录
top: 110
img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/Effective%20C%2B%2B/%E5%B0%81%E9%9D%A2.jpg
---

# 让自己习惯C++
## 条款02 尽量用const, enum, inline 替换#define
前言 我目前自己做的框架中大量用了#define.... 因为用enum涉及到转换才能到int. 来学习下这条

**使用 const 常量来替换 #define**
谨防机号表出错, 特殊情况下 还能减少字量

#define无法限定作用域, 这点我已经感受到了

定义C风格常量字符串
const char* const NAME = "lsmg"; 防止指向和指向内容改变
定义C++风格常量字符串
const std::string NAME = "lsmg";

class专属常量, 使用如下方式. 可以限定作用域
```c++
class Game
{
private:
	static const int MAX_ROOM = 10000; // 常量声明式 - 常量且只有一份
	Gameroom* rooms[MAX_ROOM];
}
```
如果要获取class专属常量的地址, 或者需要定义式. 则需要在`实现文件`而非`头文件`, 如下声明
`const int Game::MAX_ROOM` - 未给定初值
由于class常量已经在声明时获得初值, 所以不用在给定初值

这里我发现一件事情, 这本书 不能只读一遍, 最少两边完整的
前面就已经开始使用后面的条款了, 先着重本条款, 跨条款的等第二遍


## 条款03 尽可能使用const
**总结**
- 将某些东西声明为const可帮助编译器探测到错误用法
const可以用作在任何作用域内的对象, 函数参数, 函数返回类型, 成员函数本体


```c++
char greeting[] = "Hello";
char *p = greeting;
// const 在 * 左边 被指物是常量 在右边自己是常量 两边都是常量
const char *p = greeting; // 指针指向可以变, 指向的值不能变
char* const p = greeting; // 指针指向不可以变, 指向的值可以变
const char* const p = greeting; // 都不可以变
```
**const 参数, 可你帮你检查 == 被写成=的情况**
这本书看来挺有意思的2333333

**const 函数, 这里看不太懂 没有原来如此的感觉**

## 条款04 确定对象使用前已经被初始化
**总结**
- 为内置型对象进行手工初始化.
- 构造函数最好使用成员初值列, 不要在构造函数中赋值. 初值列次数应该与声明次序相同
- 为了免除 跨编译单元的初始化次数问题, 用`local static`对象替换`non-local static`对象

永远在使用对象前, 进行初始化.
在构造函数中, 对所有值进行初始化
构造函数使用`成员初始列`进行`初始化操作`而非`赋值操作`


**减少default构造函数不必要的调用**
```c++
Class A
{
public:
	A(const std::string &name, const std::list<Gameroom> &room_list);
private:
	std::string name_;
	std::list<Gameroom> room_list_;
	int roomnum_;
}

A::A(const std::string &name, const std::list<Gameroom> &room_list)
{
	name_ = name; // 这些都是赋值 不是初始化
	room_list_ = room_list;
	roomnum_ = 0;
}
// C++规定 对象的 成员变量  初始化动作  发生在  进入构造函数本体  之前
// name_  room_list_  两个在构造函数中被赋值, 而初始化在进入构造函数之前
// ---发生在这些成员的default构造函数被调用的时候, (发生在进入构造函数前)
// ---roomnum_例外 int属于内置类型

```
上面的初始化方式, 浪费了default的构造函数 所以推荐使用下面的形式
```c++
A::A(const std::string &name, const std::list<Gameroom> &room_list)
	:name_(name), room_list_(room_list), roomnum_(0)
{
}
```
虽然最终结果相同, 但没有浪费default构造函数. 前两个调用的copy构造函数.

**还可以使用成员初值列 来default构造一个成员变量.**
```c++
A::A()
	:name_(), room_list_(), roomnum_(0)
{ // 前两个全部调用的default构造函数
}
```

**const reference 内置类型(初始化与赋值等成本) 一定成员初始值列**

**初始值列总是使用 声明次序进行初始化, 而非写的顺序, 所以最好按声明次序列出**

**不同编译单元内定义之 non-local static对象 的初始化 次序**
static对象: 虚构函数会在main() 结束时被自动调用
local static对象: 函数内的static对象
non-local对象: 其他static对象

编译单元: 产出单一目标文件的那些源码, 基本是单一源码文件和他包含的头文件

```c++
// filesystem.h
class FileSystem
{
public:
	size_t GetNum() const;
};
extern FIieSystem tfs; // 声明

// filesystem.cpp
FIieSystem tfs; //定义

// directory.h
class Directory
{
public:
	Directory(params);	
}
Directory::Directory(params)
{
	size_t num = tfs.GetNum(); // 使用tfs对象
}

// main.cpp
Directory temp_dir(params);
```
只有当 tfs在temp_dir初始化之前被初始化 才能得到正确的结果. 否则会调用未初始化的对象

但是这个次序无法保证, tfs和tempdir是不同的人在不同的时间于不同的源码文件建立起来的
因为C++ 对这种情况没有明确定义

如何解决这个问题呢??
将每个`non-local static`对象搬到自己的专属函数内(该对象在此函数内被声明为static)
函数返回一个reference对象他所包含的对象
用户调用这个函数而不是直接调用对象

解决的原因呢?
C++ 保证函数内的`local static`对象 会在`函数被调用期间`, `首次遇到该对象的定义式`被初始化

```c++
// filesystem.h
class FileSystem
{
public:
	std::size_t GetNum() const;
}
FileSystem& tfs()
{
	static FileSystem fs;
	return fs;
}

// directory.h
class Directory
{
public:
	Directory(params);	
}
Directory::Directory(params)
{
	std::size_t num = tfs().GetNum(); // 使用tfs对象
}
Directory& temp_dir()
{
	static Directory td;
	return td;
}
```
# 构造析构赋值运算
## 条款0506 了解C++默认编写并调用哪些函数 并适当拒绝

**夹带如下私货-public-inline**
- 一个构造函数(如果你没有任何构造函数)
- 一个拷贝构造函数
- 一个析构函数
- 一个拷贝

拷贝构造函数和拷贝运算符 自动生成的只是单纯的将来源对象的每一个`non-static`成员变量
拷贝到目标对象

**遇到const 和 引用成员 默认的拷贝无法工作, 编译器会发出警告**

**将不需要的成员函数声明为private, 并且不实现**

## 条款07 为多态基类声明virtual析构函数
返回指向子类的 父类型指针.
如果delete这个指针, 在父类析构函数不是virtual的情况下, 很可能会导致
父类的成分被销毁, 然而子类多出来的部分不被销毁. 造成局部销毁!!

**防止局部销毁很简单, 将父类的析构函数声明为virtual**

**任何class 只要带有virtual函数 几乎确定应该有一个virtual析构函数**

**无端的声明virtual函数是错误的**
如果class不含virtual函数, 通常表示这个class 不是想做为父类, 这时如果将其析构函数声明为virual
是一个馊主意.............

*class类的大小 = 成员变量占据大小 + vptr(vitual table pointer)指针大小(4~8 字节)*

每一个带有virtual函数的class都有对应的vtbl

当对象调用某一vitual函数的时候, 实际调用的函数取决于
vptr(vitual table pointer)指针指向的vtbl(vitual table)

无端的使用virtual函数 会导致占用空间的增大

## 条款08 别让异常逃离析构函数

**总结**
- 析构函数绝对不要抛出异常, 如果被析构函数调用的函数可能抛出异常, 析构函数应该捕获所有异常
然后吞下它们, 或者结束程序
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应, class应该提供一个普通函数(而非在析构函数中)
执行操作

**析构函数不要抛出异常**
```c++
class Widget
{
	...
	~Widget() {...}
}

void Foo()
{
	std::vector<Widget> v;
}
```
当 `v`被销毁的时候, Widget的析构函数第一次抛出异常C++还能接受?
第二次就会造成不明确行为

## 条款09: 绝不在构造函数和析构过程中调用virtual函数
**总结**
- 在构造和析构期间不要调用virtual函数, 因为这类调用从不下降到`子类`

```c++
class Transaction
{
public:
	Transaction();
	virtual void LogTransaction() const = 0;
	...
}
Transaction::Transaction()
{
	...
	LogTransaction();
}

class BuyTransaction: public Transaction
{
public:
	virtual void LogTransaction() const;
	...
}

class SellTransaction: public Transaction
{
public:
	virtual void LogTransaction() const;
	...
}

BuyTransaction b;
```
BuyTransaction的构造函数会被调用, 但是父类的构造函数会更显被调用.
然后父类构造函数调用`LogTransaction()`的版本是`父类`的版本!!! 不是子类的版本

析构函数也是同样的道理, 当`子类`的析构函数执行后, `子类`中的属性值就成为未定义状态
进入`父类`后对象就成为一个`父类`对象

本例子中既然无法实现使用`virtual`函数从`父类`向下调用, 可以再构造期间, 将
`子类`必要的构造信息向上传给`父类`的构造函数

## 条款10: 另operator= 返回一个reference to *this

注意这只是一个协议, 并无强制性. 如果不遵守代码依然可以通过编译, 然而这份 协议被所有内置类型
和标准程序库提供的类型共同遵守.
因此除非你有一个标新立异的好理由, 不然还是随众吧

## 条款11: 在operator= 中处理"自我赋值"
**总结**
- 确保当对象自我赋值时operator=有良好的的行为. 其中技术包括比较"来源对象"
和"目标对象"的地址, 精心周到的语句顺序, 以及copy-and-swap
- 确认任何函数如果操作一个以上的对象, 而其中多个对象是同一个对象的时候, 进行仍未正确


**自我赋值是什么**

```c++
class Widget {...};
Widget w;

w=w; // 什么这个看起来不可能, 那下面呢?

a[i] = a[j]; // 这个怎么样?  潜在的自我赋值
*px = *py // 这个呢? 潜在的自我赋值
```
**会出现的问题**
```c++
class Gameroom{....};
class Game
{
	...
private:
	Gameroom* room_;
}

// operator=的实现代码
Game& Game::operator=(const Game& ths)
{
	delete room_;
	room_ = new Gameroom(*ths.room_);
	return *this;
}
```
上面的代码 如果 this和ths指向同一个对象就会造成 `room_`构造失败
因为被`delete`的`room_`就是要传入的

如何解决这个问题呢?
*比较来源对象 整同测试*
```c++
Game& Game::operator=(const Game& ths)
{
	if (this == &ths)
	{
		return *this;
	}
	delete room_;
	room_ = new Gameroom(*ths.room_);
	return *this;
}
```
但是如果 `new Gameroom(*ths.room_)`错误, 导致room_指向不安全的内存
使用下面的代码, 可以导出异常安全, 以及自我赋值
```c++
Game& Game::operator=(const Game& ths)
{
	Gameroom* p_room = room_;
	room_ = new Gameroom(*ths.room_);
	delete p_room;
	retutn *this;
}
```
现在如果`new Gameroom`抛出异常, room_还可以保持原状.
同时也能处理自我赋值

*copy and swap*
这一段描述我看完之后, 突然想到了上午看的Jsoncpp里面的代码!!!!! 就有这个
下面的代码更加高效 但是牺牲了清晰性
```c++
class Game
{
	void swap(Game& rhs);
}
Game& Game::operator=(const Game& ths)
{
	Game temp(this);
	swap(temp);
	return *this;
}
```
## 条款12: 复制对象时勿忘其每一个成分

# 资源管理
## 条款13: 以对象管理资源
- 防止资源泄露请使用RALL对象, 他们在构造函数获得资源在析构函数释放资源
- 较常使用的RALL classes是shared_ptr和auto_ptr(在C++17被删掉了) 建议使用前者其复制行为也比较直观

使用工厂方法得到一个指向资源对象的指针, 使用这个指针操作完毕后需要delete 这个指针
这样这个资源对象的资源就能够被释放

但是, 这是执行了delete的情况, 如果在`指针返回后` 在`delete执行前` 发生了各种情况导致`delete没有被执行`
发生的情况包括但不限于, 提前return, 如果位于循环中还可能是continue乃至goto, 甚至是异常抛出.
所以直接使用指针管理资源十分不安全.

同时我们知道 如果使用`资源管理对象`管理资源 使用资源管理对象的`析构函数来释放这个指向资源的指针`
而且对象的`析构函数`会在资管管理对象`生命周期结束后自动调用`, 这样就能`自动删除资源`, 不用依赖手动在合适的时机delete

使用智能指针是一个非常不错的选择, 当然要注意防止别让多个智能指针指向同一个对象.

auto_ptr在C++17中被废除了, 简单了解下就好了. 书中称这是诡异的复制行为2333,
复加上其底层条件, 受auto_ptr管理的资源绝对没有一个以上的auto_ptr指向他
```c++
// pR1指向CreateResource的返回物
std::auto_ptr<Resource> pR1(CreateResource());
// 现在pR2指向了, pR1被设置为了nullptr
std::auto_ptr<Resource> pR2(pR1);
// 现在pR1指向了, pR2又成了nullptr
pR1 = pR2;
```

引用计数型智慧指针......

*智能指针析构函数使用的是delete 而不是delete []* 所以不要将动态分配的数组保存进去

## 条款14 在资源管理类中小心copying行为




# 设计与声明

## 条款19: 设计class犹如设计type

如何设计高效的classes呢?
- 新class的对象该如何被创建和销毁? 这会影响到构造析构函数内存分配释放函数
- 对象的初始化和对象的赋值有什么样的差别? 决定构造函数和赋值运算符的行为.
搞清初始化和赋值
- 注意class对象被值传递 拷贝构造函数会被调用用来生成临时对象
- 什么是新class的合法值? `setter`函数需要进行的范围检查
- 你的新class需要配合某个继承图系吗? // TODO
- 你的新class需要什么样的转换? 如果需要类型转换需要在class中编写`类型转换函数`
- 什么样的操作运算符和函数对此class而言是合理的? 决定你为class声明哪些函数(条款 23 24 46)
- 将需要驳回的标准函数设置为private
- 你的新type有多么一般化? 或许应该定义一整个class家族. 也许模板能帮你

## 条款20: const 引用 替换掉 值传递

值传递会造成大量额外的构造析构被调用

通过引用传递还能避免对象切割问题. 当一个子类通过值传递并被视为一个基类的时候
子类的特性全部丢失

对于内置类型而言 值传递 比 引用传递 更加高效

## 条款21: 必须返回对象的时候, 别妄想返回其引用
local对象如果被从函数 引用返回 函数结束的时候对象就被销毁 造成未定义行为

如果通过 * 返回一个堆对象 的引用 将会造成诸如内存泄漏等问题

## 条款22: 将成员变量声明为private

成员变量声明为private提供GetSet方法, 看似麻烦... 实则确实麻烦=, =
不过麻烦的相对面就是他的好处

首先就是提供了封装, 一旦你更改了内如的成员变量的实现等, 你只需要改动GetSet方法, 外部浑然不知.
如果你声明为了public, 那么你有极大的可能需要修改每一处直接从外部使用成员的代码.

之外你还可以将进行参数校验, 等等保护.

## 条款23: 宁以non-member, non-friend替换member函数

```c++
class A
{
	DoA();
	DoB();
	DoC();
}
```
现在你需要提供一个函数 `DoD()`实现上面三个函数的功能
你有两个选择
1. 增加一个新的成员函数`DoD()`内部调用三个函数
2. 增加一个普通函数, 增加`A& a`参数调用内部的三个函数

