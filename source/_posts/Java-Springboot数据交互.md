---
title: Springboot数据交互
tags:
categories:
  - Java
date: 2019-05-18 14:34:46
---
<div class="alert-red"></div>
<div class="alert-blue"></div>
<div class="alert-green"></div>
<!--more-->
## 后端向前端传送

这种方式需要注意 编写html的时候 提示根据return的 页面决定 return了对应的页面对应的页面就会有提示 这种方法重定向无法传递

###
```java
@PostMapping("/index")
public String index(Map<String, Object> map){
	map.put("username","123");
	return "index";
}
```
### json报文 [来源](https://blog.csdn.net/chinrui/article/details/70832310)
返回的格式为json格式
```java
@ResponseBody
public Map<String, String> login() {
	Map<String, String> hashMap = new HashMap<>();
	hashMap.put("msg", "登录成功");
	return hashMap;
}
```
对于上面的代码来说，还可以做进一步的优化，由于所有的 Restful 接口都只是返回数据，所以我们可以直接在类级别上添加 @ResponseBody 注解。
而大多数情况下，@Controller 与 @ResponseBody 又会一起使用，所以我们使用 @RestController 注解来替换掉它们，从而更加简洁地实现功能。
```java
@RestController
@RequestMapping("/sys/user")
public class UserController {
	@RequestMapping("login")
	public Map<String, String> login() {
		Map<String, String> hashMap = new HashMap<>();
		hashMap.put("msg", "登录成功");
		return hashMap;
	}
}
```
### 解决重定向传参问题
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

## 前端向后短传送
### ajax
```javascript
$.ajax({
	type: "POST",
	dataType: "json",
	url: "/login",
	data: $('#loginForm').serialize(),

	success: function (result) {
		if(result.info==="登录失败"){
			document.getElementById("info").innerText="账号不存在或密码错误";
		}
	}
});
```
### 模拟表单
```javascript
function toMain(key) {
var input1 = $("<input>");
input1.attr("type", "hidden");
input1.attr("name", "key");
input1.attr("value", key);

var $form = $('<form method="POST"></form>');
$form.attr('action', "/toMain");
$form.appendTo($('body'));
$form.append(input1);
$form.submit();
}
```


### @RequestParam注解
```java
@PostMapping("/index")
public String index1(@RequestParam(value = "username")String name){
	return "index";
}
```

### @PathVariable注解 使用url+额外文本传参数
```java
RequestMapping("user/get/mac/{macAddress}")
public String getByMacAddress(@PathVariable String macAddress){
　　//do something;
}
```

### 对应的直接传递
```java
public String getByMacAddress(String username, String password){
　　对应表单直接赋值
}
```

## 参数校验
```java
@RequestMapping("login")
public Result login(@RequestBody @Valid UserModel userModel) {

}
public class UserModel {

    private String username;
    private String password;

    @NotBlank(message = "用户名不能为空")
    public String getUsername() {
        return username;
    }

    @NotBlank(message = "密码不能为空")
    public String getPassword() {
        return password;
    }
}
```
正则表达式
```java
@NotBlank(message = "用户名不能为空")
@Pattern(
        regexp = "1(([38]\\d)|(5[^4&&\\d])|(4[579])|(7[0135678]))\\d{8}",
        message = "手机号格式不合法"
)
public String getUsername() {
    return username;
}
```
