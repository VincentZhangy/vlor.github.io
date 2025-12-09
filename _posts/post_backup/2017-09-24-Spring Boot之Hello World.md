---
layout:     post
title:     	Spring Boot之 Hello World
subtitle:    "\"Spring Boot之Hello World\""
date:       2017-09-24
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring Boot

---

## 简介

1. Spring Boot简化了基于Spring的应用开发，你只需要"run"就能创建一个独立的，产品级别的Spring应用。 我们为Spring平台及第三方库提供开箱即用的设置，这样你就可以有条不紊地开始。多数Spring Boot应用只需要很少的Spring配置。

2. 你可以使用Spring Boot创建Java应用，并使用java -jar启动它或采用传统的war部署方式。我们也提供了一个运行"spring脚本"的命令行工具。

3. 我们主要的目标是：
	* 为所有Spring开发提供一个从根本上更快，且随处可得的入门体验。
	* 开箱即用，但通过不采用默认设置可以快速摆脱这种方式。
	* 提供一系列大型项目常用的非功能性特征，比如：内嵌服务器，安全，指标，健康检测，外部化配置。
	* 绝对没有代码生成，也不需要XML配置。

## 快速入门

1. ide这里使用[jetbrains公司的Idea](https://www.jetbrains.com/idea/),新建项目，选择：

	![](http://cdn-blog.jetbrains.org.cn/17-9-24/7442632.jpg)
	
2. 下一步，填写基本信息。

	![](http://cdn-blog.jetbrains.org.cn/17-9-24/68275658.jpg)
	
3. 我们顺便测试下rest接口，所以选择个web模块。

	![](http://cdn-blog.jetbrains.org.cn/17-9-24/70774312.jpg)
	
4. 项目创建完成

	![](http://cdn-blog.jetbrains.org.cn/17-9-24/19246253.jpg)
	
5. 书写test接口,并测试

		`package com.example.demo.controller;
		
		import org.springframework.web.bind.annotation.GetMapping;
		import org.springframework.web.bind.annotation.RestController;
		
		/**
		 * @Author changwenhu
		 * @Date 2017/9/24
		 * @Blog https://jetbrains.org.cn/
		 * @Description ${todo}
		 */
		@RestController
		public class TestController {
		
		    @GetMapping(value = "/")
		    public String  index() {
		        return "Hello World !!!";
		    }
		}`
		

![](http://cdn-blog.jetbrains.org.cn/17-9-24/32505492.jpg)


## 日常晒猫

   ![](http://cdn-blog.jetbrains.org.cn/17-9-24/24426406.jpg)

	

	



