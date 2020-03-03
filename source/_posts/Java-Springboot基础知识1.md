---
title: 纯洁的微笑Gitchat 课程笔记1
tags:
categories:
  - Java
date: 2019-06-07 11:36:10
---
这篇博客主要是想整理下 纯洁的微笑博客的观后感 [传送门](https://www.ityouknow.com/springboot/2016/02/03/spring-boot-web.html). 针对所列的框架, 自己再去丰富内容
<!--more-->

## Web 开发

### 自定义拦截器
Filter在servlet被调用前 截获request. 检查request, 可以进行request的修改(request头和request数据).
在离开response后处理response.

**一种方法**
```java
@Configuration
public class WebConfigure {

    @Bean
    public RemoteIpFilter remoteIpFilter() {
        return new RemoteIpFilter();
    }

    @Bean
    public FilterRegistrationBean testFilter(){

        //新建过滤器注册类
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        //将自己的过滤器添加
        registrationBean.setFilter(new MyFilter());
        //设置过滤器的URL模式
        registrationBean.addUrlPatterns("/*");
        //初始化Filter参数
        registrationBean.addInitParameter("paramName", "paramValue");
        registrationBean.setName("MyFilter");
        registrationBean.setOrder(1);
        return registrationBean;
    }
    //实现Filter的方法
    public class MyFilter implements Filter {
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            System.out.println("拦截器: " + request.getRequestURI());
            filterChain.doFilter(servletRequest, servletResponse);
        }
        //删除部分代码
    }
}
```
**推荐的方式** 不过这种方式虽然简单 但不能定义优先级
```java
//启动类加上 @ServletComponentScan 注解

//这种方式通过filterName来控制优先级......
@WebFilter(filterName = "secondFilter", urlPatterns = "/*")
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        System.out.println("拦截器2: " + request.getRequestURI());
        filterChain.doFilter(servletRequest, servletResponse);
    }
//删除部分代码
}

```
