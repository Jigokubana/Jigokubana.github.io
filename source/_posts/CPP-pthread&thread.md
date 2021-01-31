---
title: pthread&thread
date: 2021-1-3 11:27:27
tags:
categories:
 - CPP
---

# 线程创建
## pthread

```c++

void* Func(void* arg)
{
	return nullptr;
}
void PthreadLinux()
{
	pthread_t thread1; // 存储线程标识

	pthread_attr_t thread1_attr;
	pthread_attr_init(&thread1_attr);

	pthread_create(&thread1, &thread1_attr, Func, nullptr); // 使用指定属性初始化线程
    // pthread_create(&thread1, nullptr, Func, nullptr); 使用默认属性初始化线程
	printf("thread1-%d\n", thread1); // 1

	pthread_join(1, nullptr);
}
```

## thread
```c++
void* Func(void* arg)
{
	return nullptr;
}

class Foo
{
public:
	void Bar(){printf("bar\n");};
};

void PthreadCpp()
{
	std::thread thread1(Func, nullptr);
	thread1.join();

	Foo foo;
	std::thread thread2(std::bind(&Foo::Bar, &foo));
	thread2.join();

	std::thread thread3([&foo](){
		foo.Bar();
	});
	thread3.join();
}
```

# 互斥锁

## pthread

```c++
pthread_mutexattr_t mutex_attr;
pthread_mutex_init(&mutex, &mutex_attr);
pthread_mutex_lock(&mutex);
pthread_mutex_unlock(&mutex);
pthread_mutex_destroy(&mutex);
```

## thread

```c++
void Foo()
{
	{
		std::mutex mutex1;
		std::lock_guard<std::mutex> lock_guard(mutex1);
	}
}
```

# 信号量

## pthread

## thread

```c++
std::mutex condition_mutex;
std::condition_variable var;

void ConditionCpp(int id)
{
	std::unique_lock<std::mutex> unique_lock(condition_mutex);
	var.wait(unique_lock);
	sleep(2);
	printf("ConditionCpp id : %d\n", id);

	unique_lock.unlock();
	var.notify_one();
}

void ConditionCpp1()
{
	std::vector<std::thread> threads;
	threads.reserve(5);

	for (int i = 0; i < 5; ++i)
	{
		threads.emplace_back([i]()
		{
			ConditionCpp(i);
		});
	}

	var.notify_one();

	int i = 0;
	while (i < 5)
	{
		for (auto& thread : threads)
		{
			if (thread.joinable())
			{
				thread.join();
				i++;
			}
		}
	}
}

// ConditionCpp id : 0
// ConditionCpp id : 1
// ConditionCpp id : 2
// ConditionCpp id : 3
// ConditionCpp id : 4

// Process finished with exit code 0
```

