---
title: .*_case
date: 2021-1-14 21:27:27
tags:
categories:
 - CPP
---

https://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used?


# static_cast
1. 普通类型转换 int->float
2. pointer to void*, void* to pointer
3. 向上类型转换不是必须的, 但是向下转换只要不是虚拟继承就能使用


# dynamic_cast

只用来处理多态下的指针和引用

```c++
class Animal
{
public:
	virtual void PrintName()
	{
		std::cout << "Animal\r\n";
	}
};

class Dog : public Animal
{
public:
	void PrintName() override
	{
		std::cout << "Dog\r\n";
	}
};

class Cat : public Animal
{
public:
	void PrintName() override
	{
		std::cout << "Cat\r\n";
	}
};

void TestCast()
{
	// Animal* animal = new Animal(); // not dog not cat
	// Animal* animal = new Dog(); // Dog not cat
	Animal* animal = new Cat(); // not dog Cat

	Dog* dog = dynamic_cast<Dog*>(animal);
	Cat* cat = dynamic_cast<Cat*>(animal);

	if (dog)
	{
		dog->PrintName();
	}
	else
	{
		std::cout << "not dog\r\n";
	}
	if (cat)
	{
		cat->PrintName();
	}
	else
	{
		std::cout << "not cat\r\n";
	}
}
```

# const_cast

可以用来对变量移除和添加const, 其他的cast都无法做到 (static_cast可以添加const但是不能去除)

将const变量移除const是未定义的, 将`非const的变量`的`const引用`是安全的



```c++
int a = 1;
int* ap1 = &a;
const int* ap2 = static_cast<const int*>(ap1); // OK

int* ap3 = static_cast<int*>(ap2); // invalid static_cast from type 'const int*' to type 'int*'

int* ap4 = const_cast<int*>(ap2); // OK
```

- ex1.将const成员函数的this指针去除const来修改成员变量
```c++
// https://www.geeksforgeeks.org/const_cast-in-c-type-casting-operators/
#include <iostream> 
using namespace std; 

class student 
{ 
private: 
	int roll; 
public: 
	// constructor 
	student(int r):roll(r) {} 

	// A const function that changes roll with the help of const_cast 
	void fun() const
	{ 
		( const_cast <student*> (this) )->roll = 5; 
	} 

	int getRoll() { return roll; } 
}; 

int main(void) 
{ 
	student s(3); 
	cout << "Old roll number: " << s.getRoll() << endl; 

	s.fun(); 

	cout << "New roll number: " << s.getRoll() << endl; 

	return 0; 
} 

//$ Old roll number: 3
//$ New roll number: 5
```

- ex2.去除参数的const, 来调用非const参数的函数
```c++
#include <iostream> 
using namespace std; 

int fun(int* ptr) 
{ 
	return (*ptr + 10); 
} 

int main(void) 
{ 
	const int val = 10; 
	const int *ptr = &val; 
	int *ptr1 = const_cast <int *>(ptr); 
	cout << fun(ptr1); 
	return 0; 
}
//$ 20
```

# reinterpret_cast

```c++
typedef unsigned short uint16;

// Read Bytes returns that 2 bytes got read. 

bool ByteBuffer::ReadUInt16(uint16& val)
{
  return ReadBytes(reinterpret_cast<char*>(&val), 2);
}

int b = 1;
char* bp1 = &b; //  cannot convert 'int*' to 'char*' in initialization
char* bp2 = static_cast<char*>(&b); // invalid static_cast from type 'int*' to type 'char*'

char* bp3 = reinterpret_cast<char*>(&b); // ok
```
