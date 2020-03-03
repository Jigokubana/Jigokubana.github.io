---
title: Java设计模式
tags:
categories:
  - Java
date: 2019-05-21 13:41:48
---
<div class="alert-red">Java设计模式</div>
<div class="alert-blue"></div>
<div class="alert-green"></div>
<!--more-->
## 大神文章
[自己是看的作者的书](https://blog.csdn.net/lovelion/article/details/17517213)

## 单例模式
使用单例模式可以确保一个类只有一个实例化的对象

### 饿汉式单例模式
 饿汉式单例模式 无延迟加载 不需要解决多线程问题
```java
/**
 * @ClassName singletonpattern.Connect
 * @Description
 * 饿汉式单例模式 无延迟加载 不需要解决多线程问题
 * Author lsmg
 * Date 2019/5/21 13:25
 * @Version 1.0
 **/
public class Connect {
    private static final Connect instance = new Connect();

    private Connect(){

    }

    public static Connect getInstance(){
        return instance;
    }
}
```
### 懒汉式单例模式
```java
package singletonpattern;
懒汉式单例模式 实现了延迟加载,但需要解决多线程问题
/**
 * @ClassName singletonpattern.Connect1_2
 * @Description TODO
 * 懒汉式单例模式 实现了延迟加载,但需要解决多线程问题
 * Author lsmg
 * Date 2019/5/21 13:30
 * @Version 1.0
 **/
public class Connect1_2 {
    private static Connect1_2  connect1_2;

    private Connect1_2(){}

    //这种方式会造成多线程访问的时候实例化多个对象
//    public  static singletonpattern.Connect1_2 getInstance(){
//        if(connect1_2 == null){
//            connect1_2 = new singletonpattern.Connect1_2();
//        }
//
//        return connect1_2;
//    }

    //这种方式虽然确保了只有一个线程进入, 但是降低了多线程的性能
//    public synchronized static singletonpattern.Connect1_2 getInstance(){
//        if(connect1_2 == null){
//            connect1_2 = new singletonpattern.Connect1_2();
//        }
//
//        return connect1_2;
//    }


    //这种方式需要在private "volatile" static singletonpattern.Connect1_2 connect1_2 这样同样降低效率
//    public synchronized static singletonpattern.Connect1_2 getInstance(){
//        if(connect1_2 == null){
//
//            synchronized (singletonpattern.Connect1_2.class){
//                if(connect1_2 == null){
//                    connect1_2 = new singletonpattern.Connect1_2();
//                }
//            }
//
//        }
//
//        return connect1_2;
//    }

    //使用IoDH 方法
    //建立一个静态内部类
    private static class HoldeClass{
        private final static Connect1_2 instance = new Connect1_2();
    }

    private static Connect1_2 getInstance(){
        return HoldeClass.instance;
    }
}

```