---
title: java输入输出流
categories:
  - Java
date: 2019-04-16 12:04:41
tags:
---
<div class="alert-red">java输入输出流</div>
<div class="alert-blue">知识整理</div>
<div class="alert-green">IO流</div>
<!--more-->

## 输入输出的区别
![](https://blog.lsmg.xyz/2019/04/11/4y11r/1.png)
### InputStream、OutputStream
处理字节流的抽象类
InputStream 是字节输入流的所有类的超类,一般我们使用它的子类,如FileInputStream等.
OutputStream是字节输出流的所有类的超类,一般我们使用它的子类,如FileOutputStream等.

### InputStreamReader  OutputStreamWriter
处理字符流的抽象类
InputStreamReader 是字节流通向字符流的桥梁,它将字节流转换为字符流. **同时可以解决乱码问**题
OutputStreamWriter是字符流通向字节流的桥梁，它将字符流转换为字节流.**同时可以解决乱码问题**

### BufferedReader BufferedWriter
BufferedReader 由Reader类扩展而来，提供通用的缓冲方式文本读取，readLine读取一个文本行，
从字符输入流中读取文本，缓冲各个字符，从而提供字符、数组和行的高效读取。
BufferedWriter  由Writer 类扩展而来，提供通用的缓冲方式文本写入， newLine使用平台自己的行分隔符，
将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。

## InputStream
### BufferedInputStream-带有缓冲区域的InputStream [来自](https://blog.csdn.net/niyingxunzong/article/details/33335485)
InputStream
|--FilterInputStream
|----BufferedInputStream
		
1. BufferedInputStream对外提供滑动读取的功能实现，通过预先读入一整段原始输入流数据至缓冲区中。
2. 外界对BufferedInputStream的读取操作实际上是在缓冲区上进行。
3. 如果读取的数据超过了缓冲区的范围，那么BufferedInputStream负责重新从原始输入流中载入下一截数据填充缓冲区，然后外界继续通过缓冲区进行数据读取。
这样的设计的好处是：避免了大量的磁盘网络IO，因为原始的InputStream类实现的read是即时读取的，即每一次读取都会是一次IO操作（哪怕只读取了1个字节的数据），可想而知，如果数据量巨大，这样的磁盘网络消耗非常可怕。而通过缓冲区的实现，读取可以读取缓冲区中的内容，当读取超过缓冲区的内容后再进行一次IO，载入一段数据填充缓冲，那么下一次读取一般情况下就直接可以从缓冲区读取，减少了IO操作。

## OutStream
### flush操作
当写文件需要flush()的效果时，需要 
FileOutputStream fos = new FileOutputStream(“c:\a.txt”); 
BufferedOutputStream bos = new BufferedOutputStream(fos); 
也就是说，需要将FileOutputStream作为BufferedOutputStream构造函数的参数传入，然后对BufferedOutputStream进行写入操作，才能利用缓冲及flush()。

查看BufferedOutputStream的源代码，发现所谓的buffer其实就是一个byte[]。 
**BufferedOutputStream的每一次write其实是将内容写入byte[]，当buffer容量到达上限时，会触发真正的磁盘写入.**
而另一种触发磁盘写入的办法就是调用`flush()`了。

`BufferedOutputStream`在`close()`时会自动`flush ()`
### 输出时创建中间目录
只是创建一个文件对象不会在文件系统上创建相应的文件或目录。
`File.mkdir()`和`File.mkdirs()`之间的区别是，后者将创建任何中间目录，如果它不存在。

## 关闭流 [来自](https://blog.csdn.net/zhaoyanjun6/article/details/54894451)
close（）方法的作用 
1、关闭输入流，并且释放系统资源 
2、BufferedInputStream装饰一个 InputStream 使之具有缓冲功能，is要关闭只需要调用最终被装饰出的对象的 close()方法即可，因为它最终会调用真正数据源对象的 close()方法。因此，可以只调用外层流的close方法关闭其装饰的内层流。
