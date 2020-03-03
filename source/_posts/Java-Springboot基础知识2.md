---
title: 纯洁的微笑Gitchat 课程笔记2
tags:
categories:
  - Java
date: 2019-06-07 16:14:19
---
在Gitchat上 买了一个多月的 纯洁的微笑的SpringBoot讲解 这次来慢慢看看吧一共42讲 [GitChat链接,不妨给一杯咖啡](https://gitbook.cn/gitchat/column/5b86228ce15aa17d68b5b55a)
<!--more-->

**首先是约定优于配置, springboot已经定义好了大部分东西, 只有在不符合约定的时候, 才需要手动去配置相关文件**

## 注解
### @RestController
这个注解等于`@ResponseBody ＋ @Controller`
返回json数据的便捷注解

### @RequestMapping
`@RequestMapping(name="/getUser", method= RequestMethod.POST)`

## 测试
### MockMVC
MockMVC 可以进行POST GET 模拟请求
```java
@SpringBootTest
public class HelloWorldTest {

    private MockMvc mockMvc;

    @Before
    //`@Before`注解的方法 在启动测试后首先执行, 来进行资源的初始化
    public void setUp() throws Exception {
        mockMvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }

    @Test
    public void getHello() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.post("/hello")
          .accept(MediaType.APPLICATION_JSON_UTF8)).andDo(print());
    }

}
```
## 参数传递
[之前的一篇博客, 传送门](https://blog.lsmg.xyz/2019/05/18/dataInteraction/)
### 参数校验
控制器
```java
@RequestMapping(name = "/saveUser", method = RequestMethod.POST)
public void saveUser(@Valid User user, BindingResult result) { //@Valid 代表对这个参数进行校验 BindingResult 用于存储校验结果
    if(result.hasErrors()) {
        List<ObjectError> list = result.getAllErrors();
        for (ObjectError error : list) {
            System.out.println(error.getCode()+ "-" + error.getDefaultMessage());
        }
    }
}
```
实体类
```java
@NotEmpty(message = "姓名不能为空")
private String name;
@Max(value = 100, message = "年龄不能大于100岁")
@Min(value= 0 ,message= "年龄必须大于0岁！" )
private String sex;
private int age;
```
对应的测试类
```java
@Test
public void saveUser() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.post("/saveUser")
            .param("name","")
            .param("age", "101")
            .param("sex", "男")
    );
}
/*Min-年龄必须大于0岁！
  NotEmpty-姓名不能为空
  Max-年龄不能大于100岁*/
```
### 校验的注解汇总
| 注解                | 应用对象         | 检查内容       |
| ------------------- | ---------------- | -------------- |
| @Length(min=, max=) | 用于String对象   | 检查字符串长度 |
| @Max(value=)        |                  | 最大值         |
| @Min(value=)        |                  | 最小值         |
| @NotNull            |                  | 不为空         |
| @Past               | date 或 calendar | 时间是过去吗?  |
| @Future             | date 或 calendar | 时间是将来吗?  |
| @Email              | String           | 格式是邮箱吗?  |

## 配置文件的使用

### 读取配置文件
```java
//application.properties -->neo.title=lsmg

@Value("${neo.title}") //使用注解来获取内容
private String title;
```
## thymeleaf
引入命令空间
`<html xmlns:th="https://www.thymeleaf.org">`
### 基础使用

#### 字符串赋值拼接
```java
"${属性名}"
"'这是固定部分' + ${变化部分属性名}"

简写形式
"|固定部分${变化部分属性名}|" //直接混合
```

#### 条件判断
```java
//if只有内容为真才会显示, unless内容为假才会显示
th:if="${flag == 'yes'}"
th:unless="${flag == 'yes'}"
```
#### 循环
```html
<table>
    <tr  th:each="user,iterStat : ${users}">
        <td th:text="${user.name}">neo</td>
        <td th:text="${iterStat.index}">index</td>
    </tr>
</table>
```
**iterStat 属性值**
index，当前迭代对象的 index（从 0 开始计算）；
count，当前迭代对象的 index（从 1 开始计算）；
size，被迭代对象的大小；
current，当前迭代变量；
even/odd，布尔值，当前循环是否是偶数/奇数（从 0 开始计算）；
first，布尔值，当前循环是否是第一个；
last，布尔值，当前循环是否是最后一个。

#### URL
`th:href="@{https://www.lsmg.xyz/{id}(id=${id})}"`
**如果需要 Thymeleaf 对 URL 进行渲染，那么务必使用 th:href、th:src 等属性**
<div th:style="'background:url(' + @{${img url}} + ');'">
`{id}(id=${id})` 这部分方便了阅读

#### 三目运算
```java
${age gt 30 ? '中年':'年轻'}

gt：great than（大于）
ge：great equal（大于等于）
eq：equal（等于）
lt：less than（小于）
le：less equal（小于等于）
ne：not equal（不等于）
```
#### switch
```html
<div th:switch="${sex}">
    <p th:case="'woman'">女</p>
    <p th:case="'man'">男</p>
    <!-- *: case的默认的选项 -->
    <p th:case="*">蓝</p>
</div>
```
### 高阶使用
#### 内联`[[]]`
如果要使用内联方式 需要在标签或者父标签 甚至是在body中加入 `th:inline="text/javascript/none"`
进行激活

**看了下官方文档[传送门](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)**
`th:inline="none"`代表不显示,其中的`[[]]`不会被thymeleaf识别

`th:inline="javascript"` 代表在js中使用
```javascript
<script th:inline="javascript">
    ...
    var username = [[${session.user.name}]];
    ...
</script>
<script th:inline="javascript">
    ...
    var username = "Sebastian \"Fruity\" Applejuice";
    ...
</script>

<script th:inline="javascript">
    ...
    var username = [(${session.user.name})];  // [()] 类似于 th:utext
    ...
</script>
<script th:inline="javascript">
    ...
    var username = Sebastian "Fruity" Applejuice;
    ...
</script>
```
上面的方式会 让它在静态显示时出现错误。
一般需要加上注释 /**/ 来包裹`[[]]`

thymeleaf支持多种格式

* Strings
* Numbers
* Booleans
* Arrays
* Collections
* Maps
* Beans (objects with getter and setter methods)


```javascript
/*<![CDATA[*/
  //这里说一下经常看到的
  //XHTML解析器会把CDATA中的内容当作纯文本处理，
  //里面的 < & 不会被js翻译而是直接显示
/*]]>*/

//<![CDATA[
  //相同效果
//*]]>*
```

#### 基本对象
```
#ctx：上下文对象
#vars：上下文变量
#locale：区域对象
#request：（仅 Web 环境可用）HttpServletRequest 对象
#response：（仅 Web 环境可用）HttpServletResponse 对象 //常用
#session：（仅 Web 环境可用）HttpSession 对象 //常用
#servletContext：（仅 Web 环境可用）ServletContext 对象
```
#### 内嵌变量
[文档地址, 内容太多, 贴不过来了](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#inlining)
```html
<!--格式化时间-->
<p th:text="${#dates.format(date, 'yyyy-MM-dd HH:mm:ss')}">neo</p>
<!--创建当前时间 精确到天-->
<p th:text="${#dates.createToday()}">neo</p>
<!--创建当前时间 精确到秒-->
<p th:text="${#dates.createNow()}">neo</p>

<!--判断是否为空-->
<p th:text="${#strings.isEmpty(userName)}">userName</p>
<!--判断 list 是否为空-->
<p th:text="${#strings.listIsEmpty(users)}">userName</p>
<!--输出字符串长度-->
<p th:text="${#strings.length(userName)}">userName</p>
<!--拼接字符串-->
<p th:text="${#strings.concat(userName,userName,userName)}"></p>
<!--创建自定长度的字符串-->
<p th:text="${#strings.randomAlphanumeric(count)}">userName</p>
```

## springboot和thymeleaf上传文件
### 配置信息
常用部分
```xml
#支持的最大文件
spring.servlet.multipart.max-file-size=100MB
#文件请求最大限制
spring.servlet.multipart.max-request-size=100MB
```
其他常用设置
```xml
spring.servlet.multipart.enabled=true，是否支持 multipart 上传文件
spring.servlet.multipart.file-size-threshold=0，支持文件写入磁盘
spring.servlet.multipart.location=，上传文件的临时目录
spring.servlet.multipart.max-file-size=10Mb，最大支持文件大小
spring.servlet.multipart.max-request-sizee=10Mb，最大支持请求大小
spring.servlet.multipart.resolve-lazily=false，是否支持 multipart 上传文件时懒加载
```

### 文件上传
解决上传文件大于 10M 出现连接重置的问题
**终于去看了看  表达式**
```java
//Tomcat large file upload connection reset
@Bean
public TomcatServletWebServerFactory tomcatEmbedded() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addConnectorCustomizers((TomcatConnectorCustomizer) connector -> {
        if ((connector.getProtocolHandler() instanceof AbstractHttp11Protocol<?>)) {
            //-1 means unlimited
            ((AbstractHttp11Protocol<?>) connector.getProtocolHandler()).setMaxSwallowSize(-1);
        }
    });
    return tomcat;
}
```
前端网页
```html
<form method="POST" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file" />
    <input type="submit" value="Submit" />
</form>
```
补充 enctype属性
| 值                                | 描述                                                        |
| --------------------------------- | ----------------------------------------------------------- |
| application/x-www-form-urlencoded | 在发送前编码所有字符（默认）                                |
| multipart/form-data               | 不对字符编码 在使用包含文件上传控件的表单时，必须使用该值。 |
| text/plain                        | 空格转换为 "+" 加号，但不对特殊字符编码。                   |

**RedirectAttributes**
RedirectAttributes attr

**attr.addAttribute("param", value);**
attr.addAttribute("name", "user");
attr.addAttribute("success", "ok");
return "redirect:/index";
这种方式相当于系统自动的拼接了url 仍然会暴露信息

**attr.addFlashAttribute("param", value);**
attr.addFlashAttribute("status","999");
attr.addFlashAttribute("message","登录失败");
return "redirect:/toLogin";
这种方式通过session传递, session在跳转到页面后就是马上移除对象, 刷新后即消失

**上传控制器**
```java
@PostMapping("/upload")
public String singleFileUpload(@RequestParam("file") MultipartFile file,
                             RedirectAttributes redirectAttributes) {
  if (file.isEmpty()) {
      redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
      return "redirect:uploadStatus";
  }
  try {
      // Get the file and save it somewhere
      byte[] bytes = file.getBytes();
      // UPLOADED_FOLDER 文件本地存储地址
      Path path = Paths.get(UPLOADED_FOLDER + file.getOriginalFilename());
      Files.write(path, bytes);

      redirectAttributes.addFlashAttribute("message",
              "You successfully uploaded '" + file.getOriginalFilename() + "'");

  } catch (IOException e) {
      e.printStackTrace();
  }
  return "redirect:/uploadStatus";
}
```
### 多文件上传
```html
<form method="POST" action="/uploadMore" enctype="multipart/form-data">
    文件1： <input type="file" name="file" /><br/><br/>
    文件2： <input type="file" name="file" /><br/><br/>
    文件3： <input type="file" name="file" /><br/><br/>
    <input type="submit" value="Submit" />
</form>
```
控制器
```java
@PostMapping("/uploadMore")
public String moreFileUpload(@RequestParam("file") MultipartFile[] files,
                               RedirectAttributes redirectAttributes) {
    if (files.length==0) {
        redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
        return "redirect:uploadStatus";
    }
    for(MultipartFile file:files){
        try {
            byte[] bytes = file.getBytes();
            Path path = Paths.get(UPLOADED_FOLDER + file.getOriginalFilename());
            Files.write(path, bytes);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    redirectAttributes.addFlashAttribute("message", "You successfully uploaded all");
    return "redirect:/uploadStatus";
}
```
## Restful api
使用 swagger2 构建restful api

## 定时任务
### 使用注解方式
启动类 加上注解 `@EnableScheduling` 然后在\
实现类上要有组件的注解@Component
要定时的方法上加上
```java
private int count = 0;
private static final SimpleDateFormat df = new SimpleDateFormat("HH:mm:ss");

@Scheduled(fixedRate = 1000) //单位为秒
public int getCount() {
   System.out.println(df.format(new Date())+" "+count);
   return ++count;
}
```
 @Scheduled(cron = "*/6 * * * * *") 的cron属性
```java
/*
The pattern is a list of six single space-separated fields: representing second, minute, hour, day, month, weekday. Month and weekday names can be given as the first three letters of the English names.
6个由空格间隔的单独的数字  分别代表 xxxxx 月份和工作日可以用英文前三个字母代替
*/
```
**星花和斜杠含义**

*/10 * * * * * = every ten seconds.  */10
这四个字符(由于第一位代表秒 *为通配符任意秒 /为每隔 10与/组合意味每十秒 */10意为 从任意秒开始每10S执行
末尾的 * * * * * 意味任意的 分钟小时......

*可以结合下面这个看*
10 * * * * *
这个意思为 任意的分钟小时...... 当秒为10的时候 触发

**再来一个例子**
0 0/30 8-10 * * *
后三个星花表示任意的 天月和工作日
第一个0表示 0S时
第二个 0/30 表示从0Min开始每隔30Min
第三个 `-` 表示8H和9H和10H
如果第三个 `-` 换为`,` 则为8H和10H
