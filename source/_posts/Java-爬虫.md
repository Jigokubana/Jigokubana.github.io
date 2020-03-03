---
title: 2019年4月15日 java爬虫入门
tags:
categories:
  - Java
date: 2019-04-16 13:02:32
---
<div class="alert-red">java爬虫</div>
<div class="alert-blue">maven项目 下载和文件流</div>
<div class="alert-green">IDEA配置</div>
<!--more-->

这篇博客不知道为啥被覆盖了 哎还要从新写一次
### maven项目
通过maven项目 不需要像java项目一样去下载第三方jar包了
```xml
<dependencies>
<!--        所用到的httpclient包-->
	<dependency>
		<groupId>org.apache.httpcomponents</groupId>
		<artifactId>httpclient</artifactId>
		<version>4.5.3</version>
	</dependency>

	<dependency>
		<groupId>com.google.guava</groupId>
		<artifactId>guava</artifactId>
		<version>r05</version>
	</dependency>
<!--        Jsoup是一款Java的HTML解析器，可以直接解析某个URL地址，也可以解析HTML内容。其主要的功能包括解析HTM-->
<!--        L页面，通过DOM或者CSS选择器来查找、提取数据，可以更改HTML内容-->
	<dependency>
		<groupId>org.jsoup</groupId>
		<artifactId>jsoup</artifactId>
		<version>1.10.3</version>
	</dependency>
</dependencies>
```
### 下载类
```java
/**
* @ClassName Download
* @Description TODO
* Author lsmg
* Date 2019/4/15 18:03
* @Version 1.0
* 来自 https://www.cnblogs.com/lichenwei/p/4610298.html
**/
public class Download {
/**
 * @Author lsmg
 * @Description //TODO
 * @Date 23:36 2019/4/15
 * @param imgUrl 图片url地址
 * @param extension 图片的后缀名
 * @return void
 **/
synchronized public void imgDownload(String imgUrl, String extension){
	try{
		//获取输入流
		BufferedInputStream in = new BufferedInputStream((new URL(imgUrl).openStream()));

		//文件名
		SimpleDateFormat df = new SimpleDateFormat("HH时mm分ss秒SSS毫秒");
		//创建文件流
		File file = new File("D:\\Pictures\\miku\\"+df.format(new Date())+"."+extension);
		System.out.println(file.getAbsolutePath());

		FileOutputStream fileOutputStream = new FileOutputStream(file);
		BufferedOutputStream out = new BufferedOutputStream(fileOutputStream);

		//缓冲字节流
		byte[] data = new byte[1024];
		int length = 0;
		while ((length = in.read(data)) != -1){
			out.write(data,0,length); //这里划重点 这是修改后的代码没啥问题了 每次写入的长度不一定为1024 而应该是真实长度

		}
		System.out.println("正在下载图片 :"+imgUrl);
		in.close();
		out.close();

	}catch (Exception e){
		e.printStackTrace();
	}
}
```
### 启动类
使用了多线程传入参数i 作用在后边说明
```java
public static void main(String[] args) {
	for(int i=0;i<=10;i++){
		Thread t =new FindImg(i);
		t.start();
	}
}
```
### java爬虫
这个网页带有分页 每次切换页数url只有页数发生了变化 干脆加入了多线程 一个线程负责一页
```java
/**
 * @Author lsmg
 * @Description //TODO 找到select的更好表示方式
 * @Date 23:42 2019/4/15
 * @return void
 **/
public void Find(){

	//建立一个请求客户端
	CloseableHttpClient httpClient = HttpClients.createDefault();

	String url="https://pic.netbian.com/e/search/result/index.php?page="+index+"&searchid=122";

	//使用HttpGet方式请求网址
	HttpGet httpGet = new HttpGet(url);

	//获取网址返回结果
	CloseableHttpResponse response = null;
	try {                     //....执行
		response = httpClient.execute(httpGet);
	}catch (Exception e){
		e.printStackTrace();
	}

	//获取返回结构中的实体
	HttpEntity entity = response.getEntity();

	String html = null;
	//将返回实体输出
	try {
		html= EntityUtils.toString(entity);
		//System.out.println(html);
		EntityUtils.consume(entity);
	}catch (Exception e){
		e.printStackTrace();
	}

	//解析html到一个document
	Document document = Jsoup.parse(html);

	//提取内容
	Elements imgJpg = document.select("img[src$=.jpg]"); //select选择器 下边说明 由于返回多个结果使用Elements存储

	//下载对象
	Download download = new Download();

	for (Element element : imgJpg){
		download.imgDownload("https://pic.netbian.com"+element.attr("src"),"jpg");
	}
}
```
### Jsoup-Java的HTML解析器
这个功能强大记录两条常用的
**[来自](https://www.voidcn.com/article/p-qdzdjxky-p.html)**

[attr^=value] 利用匹配属性值开头
[attr$=value] 利用匹配属性值结尾
[attr*=value]包含属性值来查找元素，比如：[href*=/path/]
还有
a[href] 带有herf内=内容的a标签
a[class=xxx] 带有xxxclass的a标签

element.attr("src") 找到src的内容

### 自己的方法注释
https://blog.csdn.net/qq_34533072/article/details/80830738