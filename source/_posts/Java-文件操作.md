---
title: 4月11日 java文件操作
date: 2019-04-11 21:24:48
tags:
categories:
  - Java
---
<div class="alert-red">java文件操作</div>
<div class="alert-blue">#$%&^*%^&%$</div>
<div class="alert-green"></div>
<!--more-->

[博客来源](https://blog.csdn.net/yhl_jxy/article/details/79272792)

![来源](https://lsmg-img.oss-cn-beijing.aliyuncs.com/%E6%9D%82%E9%A1%B9/Java%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C.png)

Java IO读写文件的IO流分为两大类，字节流和字符流，
基于字节流的读写基类: InputStream和OutputStream
基于字符流的读写基类: Reader和Writer

如果是二进制文件，使用FileInputStream读取；如果是文本文件，使用FileReader读取；
这两个类允许我们从文件开始至文件结尾一个字节或字符的读取文件，或者将读取的文件写入字节数组或字符数组。
如果我们想随机的读取文件内容，可以使用RandomAccessFile。
### 简单的文件读写
**File file = new File("123.txt");**
#### 字节流读取文件
```java
try (InputStream is = new FileInputStream(file)){
	byte[] dataB=new byte[1024];
	is.read(dataB);
	System.out.println("字节文件内容:" + new String(dataB));
} catch (Exception e) {
	e.printStackTrace();
}
```
#### 字节流写文件
```java
try(OutputStream os = new FileOutputStream(file)){
	byte[] dataB = new byte[1024];
	String str = "1234567890abcdef";
	dataB = str.getBytes();
	os.write(dataB);
}catch (Exception e){
	e.printStackTrace();
}
```
#### 字符流读取文件
```java
File file = new File("123.txt");
try (Reader reader = new FileReader(file)){
	char[] dataC = new char[1024];
	reader.read(dataC);
	System.out.println("字符文件内容:"+new String(dataC));
}catch (Exception e){
	e.printStackTrace();
}
```
#### 字符流写文件
```java
try (Writer writer = new FileWriter(file)){
	char[] dataC = new char[1024];
	String str="@@@@1234567890abcdef";
	dataC=str.toCharArray();
	writer.write(dataC);
}catch (Exception e){
	e.printStackTrace();
}
```
### 简单的写文件-指定位置
```java
try (RandomAccessFile raf = new RandomAccessFile(file, "r")) {
	//获取RandomAccessFile对象文件指针的位置，初始位置是0
	System.out.println("RandomAccessFile文件指针的初始位置:"+raf.getFilePointer());
	//移动文件指针位置
	raf.seek(0);
	byte[]  buff=new byte[1024];
	//用于保存实际读取的字节数
	int hasRead=0;
	//循环读取
	while((hasRead=raf.read(buff))>0){
		//打印读取的内容,并将字节转为字符串输入
		System.out.println(new String(buff,0,hasRead));
	}
} catch (Exception e) {
	e.printStackTrace();
}
```

