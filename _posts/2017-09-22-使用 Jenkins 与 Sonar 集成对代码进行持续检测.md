---
layout:     post
title:      使用 Jenkins 与 Sonar 集成对代码进行持续检测
subtitle:    "\"使用 Jenkins 与 Sonar 集成对代码进行持续检测\""
date:       2017-09-22
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Jenkins
    - Sonar
    - Code Review

---

## SonarQube 与 Jenkins 简介

1. SonarQube是 一个开源的代码质量分析平台，便于管理代码的质量，可检查出项目代码的漏洞和潜在的逻辑问题。同时，它提供了丰富的插件，支持多种语言的检测， 如 Java、Python、Groovy、C、C++等几十种编程语言的检测。它主要的核心价值体现在如下几个方面：
	
	* 检查代码是否遵循编程标准：如命名规范，编写的规范等。
	* 检查设计存在的潜在缺陷：SonarQube 通过插件 Findbugs、Checkstyle等 工具检测代码存在的缺陷。
	* 检测代码的重复代码量：SonarQube可以展示项目中存在大量复制粘贴的代码。
	* 检测代码中注释的程度：源码注释过多或者太少都不好，影响程序的可读可理解性。
	* 检测代码中包、类之间的关系：分析类之间的关系是否合理，复杂度情况。


2. SonarQube 平台是由4个部分组成:

	* SonarQube Server
	* SonarQube Database
	* SonarQube Plugins
	* SonarQube Scanner

## Jenkins 与 SonarQube 集成插件的安装与配置

Jenkins 是一个支持自动化框架的服务器，我们这里不做详细介绍。Jenkins 提供了相关的插件，使得 SonarQube 可以很容易地集成 。 

1. 登陆 jenkins，点击"系统管理"，如图。
	![](http://cdn-blog.jetbrains.org.cn/17-9-22/74455252.jpg)
	
2. Jenkins 管理插件

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/68878742.jpg)

3. Jenkins 安装 SonarQube 插件

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/92049540.jpg)
	
4. 进入 Jenkins 系统管理 – 系统设置，配置 SonarQube Server 信息 

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/33406013.jpg)
	
5. 进入 Jenkins 系统管理 - Global Tool Configuration，配置 SonarQube Scanner

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/4000775.jpg)
	
6. 新建 Jenkis 项目

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/87719678.jpg)
	
7. 在 Jenkins 项目构建过程中加入 SonarScanner 进行代码分析

	首先需要在新建的 Jenkins 项目的构建环境标签页中勾选"Prepare SonarQube Scanner evironment"
	
	![](http://cdn-blog.jetbrains.org.cn/17-9-22/58710586.jpg)
	
8. 增加 Execute SonarQube Scanner 构建步骤

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/37443798.jpg)
	
9. 配置 SonarQube Scanner 构建步骤，在 Task to run 输入框中输入 scan，即分析代码；在 JDK 选择框中选择 SonarQube Scanner 使用的 JDK（注意这里必须是 JDK 不能是 JRE）；Path to project properties 是可选择的输入框，这里可以指定一个 sonar-project.properties 文件，如果不指定的话会使用项目默认的 properties 文件；Analysis properties 输入框，这里需要输入一些配置参数用来传递给 SonarQube，这里的参数优先级高于 sonar-project.properties 文件里面的参数，所以可以在这里来配置所有的参数以替代 sonar-project.properties 文件，下面列出了一些参数，sonar.language 指定了要分析的开发语言（特定的开发语言对应了特定的规则），sonar.sources 定义了需要分析的源代码位置（示例中的$WORKSPACE 所指示的是当前 Jenkins 项目的目录），sonar.java.binaries 定义了需要分析代码的编译后 class 文件位置；Additional arguments 输入框中可以输入一些附加的参数，示例中的-X 意思是进入 SonarQube Scanner 的 Debug 模式，这样会输出更多的日志信息；JVM Options 可以输入在执行 SonarQube Scanner 是需要的 JVM 参数

		sonar.projectKey=testSonar 
		sonar.projectName=testSonar 
		sonar.projectVersion=1.0 
		sonar.language=java 
		sonar.java.binaries=$WORKSPACE/testSonar/target/test-classes/ 
	 	sonar.sources=$WORKSPACE/testSonar/src
	 	
10. 配置 Execute SonarQube Scanner 构建步骤

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/53880377.jpg)
	
11. Jenkins 项目构建结果

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/41747847.jpg)
	
12. 分析结果报告

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/25009288.jpg)
	
13. 具体问题展示

	![](http://cdn-blog.jetbrains.org.cn/17-9-22/11613421.jpg)
	
	
	
## 参考资源

[Jenkins 官网](https://jenkins.io/)
[Sonarqube 官网](https://www.sonarqube.org/)
[developerWorks](https://www.ibm.com/developerworks/cn/devops/1612_qusm_jenkins/index.html)


	
	
	
		







