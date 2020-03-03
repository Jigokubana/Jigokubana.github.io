---
title: 2019年4月13日 FASTJson解析json
categories:
  - Java
date: 2019-04-13 17:22:40
tags:
---
<div class="alert-red"> FASTJson 解析json</div>
<div class="alert-blue"></div>
<div class="alert-green"></div>
<!--more-->

### 调用api部分
```java
String u = "https://route.showapi.com/341-1?showapi_appid=搞了好久A&showapi_sign=搞了好久B";
URL url = new URL(u);
HttpURLConnection connection = (HttpURLConnection)url.openConnection();
connection.connect();

InputStreamReader in = new InputStreamReader(connection.getInputStream());
BufferedReader bf = new BufferedReader(in);
String stringJsonbf=null; //暂时存储
StringBuffer stringBuffer = new StringBuffer();
while ((stringJsonbf= bf.readLine())!=null){
	stringBuffer.append(stringJsonbf);
}
String stringJson=stringBuffer.toString(); //得到json数据转为String
System.out.println(stringJson);
```

### json解析部分
```java
List<类> happies = new ArrayList<类>(); //类为json中所需要对象的对应类
JSONObject jsonObject1 = new JSONObject(stringJson); //string转换为json对象
JSONObject jsonObject2 = jsonObject1.getJSONObject("showapi_res_body");//这个对象中还有个对象用jsonObject1.getJSONObject获取
JSONArray jsonArray =jsonObject2.getJSONArray("contentlist");//获取后的对象里有数组获取数组

for(int i=1;i<jsonArray.length();i++){
	JSONObject jsTempt = jsonArray.getJSONObject(i);
	String title = jsTempt.getString("title");
	String text = jsTempt.getString("text");
	String ct = jsTempt.getString("ct");
	Happy 对象 = new 类(title,text,ct);
	happies.add(对象);//加入list中
}
System.out.println(happies);
```
### 注意问题
#### 搞了好久A和搞了好久B
![](./2019n4y12r/2.png)
这里的appid和sign**不是这个应用的id**=, =而是你账号里的
![](./2019n4y12r/1.png)
好吧..................生成sign的方式官方有

1. 您首先需要设置除了showapi_sign之外的所有必传参数，例如：
`?title=足球&page=1&pag=for_test&showapi_appid=123`

2. 对上述参数key进行排序按照字典序(a-z)，请注意byte[]类型的参数不参与排序和计算签名，比如上传的文件；空值的参数也不参与排序和计算签名。排序后以key+value方式拼装字符串如下：
pagfor_testpage1showapi_appid123title足球
**请注意上述的pag字段排在page字段之前**

3. String str="pagfor_testpage1showapi_appid123title足球"
str=str+secret
也就是str=str+"006513e01bd344fca03610d1fd0145f0" //secret用小写
最后str="pagfor_testpage1showapi_appid123title足球006513e01bd344fca03610d1fd0145f0"
注意在签名计算时,中文依然是中文,并没有被urlencode
String sign=DigestUtils.md5Hex(str.getBytes("utf-8"))
最后得到 sign="030554F4F9375B4DCFEF5ECEC4488737"

不得不说这个代码表示真的好
#### 还有个json解析问题
`JSONObject jsonObject2 = new JSONObject(jsonObject1.getJSONObject("showapi_res_body"))`
好吧开始是这样的结果一直报错/........具体原因先留个坑吧
2019年4月13日00:07:54

###  FASTJson 解析json
文章学习来源 [声明出处](https://blog.csdn.net/xingfei_work/article/details/76572550)
从这里我学会了解析json 自己也点写笔记

`String json1 = "{'id':1,'name':'JAVAEE-1703','stus':[{'id':101,'name':'刘铭','age':16}]}";`
待解析json数据
构建对应的类
### 解析对象
#### 第一层
```java
public class Grade {
	private int id;
	private String name;
	private ArrayList<Student> stus;
	public Grade(int id, String name, ArrayList<Student> stus) {
		super();
		this.id = id;
		this.name = name;
		this.stus = stus;
	}//这个构造方法对应了json数据 由于json数据中有数组 所以写了链表存储数组
	//省略若干get和set以及toString的重写(用于打印对象)
}
```
#### 第二层
```java
public class Student {
private int id;
private String name;
private int age;
//省略若干get和set以及toString的重写(用于打印对象)
}//这里同样对应了json数组中的数据
```
#### 主要部分
```java
String json1 = "{'id':1,'name':'JAVAEE-1703','stus':[{'id':101,'name':'刘铭','age':16}]}";
Grade grade = JSON.parseObject(json1,Grade.class);
System.out.println(grade);
```

不得不说阿里的这个简化了太多了疯狂扣6

### 解析数组
```java
String json2 = "['北京','天津','杭州']";
List<String> list=JSON.parseArray(json2, String.class);
```