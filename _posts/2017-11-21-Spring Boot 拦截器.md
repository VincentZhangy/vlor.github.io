---
layout:     post
title:     	Spring Boot 拦截器
subtitle:    "\"Spring Boot 拦截器\""
date:       2017-11-21
author:     Mr Chang
header-img: img/post-bg-mma-2.jpg
catalog: true
tags:
    - Spring Boot 
    - Interceptor
    - 拦截器

---



# 简介

HandlerInterceptor的功能跟过滤器类似，但是提供更精细的的控制能力：在request被响应之前、request被响应之后、视图渲染之前以及request全部结束之后。我们不能通过拦截器修改request内容，但是可以通过抛出异常（或者返回false）来暂停request的执行。


实现 UserRoleAuthorizationInterceptor 的拦截器有： 
ConversionServiceExposingInterceptor |
CorsInterceptor |
LocaleChangeInterceptor |
PathExposingHandlerInterceptor |
ResourceUrlProviderExposingInterceptor |
ThemeChangeInterceptor |
UriTemplateVariablesHandlerInterceptor |
UserRoleAuthorizationInterceptor |

其中 LocaleChangeInterceptor 和 ThemeChangeInterceptor 比较常用。

配置拦截器也很简单，Spring 为什么提供了基础类WebMvcConfigurerAdapter ，我们只需要重写 addInterceptors 方法添加注册拦截器。


# 实现

1. 创建我们自己的拦截器类并实现 HandlerInterceptor 接口
	![](http://cdn-blog.jetbrains.org.cn/17-11-21/6722144.jpg)
	
2. 创建一个Java类继承WebMvcConfigurerAdapter，并重写 addInterceptors 方法
3. 实例化我们自定义的拦截器，然后将对像手动添加到拦截器链中（在addInterceptors方法中添加）

	![](http://cdn-blog.jetbrains.org.cn/17-11-21/24765173.jpg)
	
	
# 查看

然后在浏览器输入地址： http://localhost:8080/index 后，控制台的输出为：

	MyInterceptor1>>>>>>>在请求处理之前进行调用（Controller方法调用之前）
	MyInterceptor1>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
	MyInterceptor1>>>>>>>在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
	
	
![](http://cdn-blog.jetbrains.org.cn/17-11-21/46078190.jpg)