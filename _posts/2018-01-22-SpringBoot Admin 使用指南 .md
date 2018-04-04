---
layout:     post
title:     	SpringBoot Admin 使用指南
subtitle:    "\"SpringBoot Admin 使用指南 \""
date:       2018-01-22
author:     Mr Chang
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - SpringBoot Admin

---


# 说明

Spring Boot Admin 是一个管理和监控你的 Spring Boot 应用程序的应用程序。 这些应用程序通过 Spring Boot Admin Client（通过 HTTP）注册或者使用 Spring Cloud（例如 Eureka）发现。 UI只是 Spring Boot Actuator 端点上的一个 AngularJs 应用程序。

# 创建服务

1. 创建spring boot 项目，引入依赖

		<dependency>
		    <groupId>de.codecentric</groupId>
		    <artifactId>spring-boot-admin-server</artifactId>
		    <version>1.5.6</version>
		</dependency>
		<dependency>
		    <groupId>de.codecentric</groupId>
		    <artifactId>spring-boot-admin-server-ui</artifactId>
		    <version>1.5.6</version>
		</dependency>
		
2. 启动类中引入注解 `@EnableAdminServer` ，然后运行项目：

![](http://cdn-blog.jetbrains.org.cn/18-1-22/14831048.jpg)

# 创建客户端

1. 随便创建个spring boot 项目，引入依赖。

		<dependency>
		    <groupId>de.codecentric</groupId>
		    <artifactId>spring-boot-admin-starter-client</artifactId>
		    <version>1.5.6</version>
		</dependency>

2. 更改配置文件

		server.port=8888
		spring.boot.admin.client.name=test
		management.security.enabled=false
		
		spring.boot.admin.url=http://localhost:8080
		
		
# 测试

1. 返回服务端查看

	![](http://cdn-blog.jetbrains.org.cn/18-1-22/10881902.jpg)
	
2. test 应用详细信息

	![](http://cdn-blog.jetbrains.org.cn/18-1-22/92688632.jpg)

# 总结

  最后一项可以直接导出文件，并深入分析。