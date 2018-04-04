---
layout:     post
title:      CodeReview 
subtitle:    "\"Maven SonarQube CodeReview\""
date:       2017-09-21
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Maven
    - SonarQube
    - CodeReview

---


## 介绍

1. SonarQube
	
	* 官网： https://www.sonarqube.org/
	* 介绍：（曾用名Sonar（声纳）是一个开源的代码质量管理系统。 

2. SonarQube 特征

	* 支持超过25种编程语言：Java、C/C++、C#、PHP、Flex、Groovy、JavaScript、Python、PL/SQL、COBOL等。（不过有些是商业软件插件）
	* 可以在Android开发中使用
	* 提供重复代码、编码标准、单元测试、代码覆盖率、代码复杂度、潜在Bug、注释和软件设计报告
	* 提供了指标历史记录、计划图（“时间机器”）和微分查看
	* 提供了完全自动化的分析：与Maven、Ant、Gradle和持续集成工具（Atlassian Bamboo、Jenkins、Hudson等）* 与Eclipse开发环境集成
	* 与JIRA、Mantis、LDAP、Fortify等外部工具集
	* 支持扩展插件
	* 利用SQALE计算技术债务
	* 支持Tomcat。不过计划从SonarQube 4.1起终止对Tomcat的支持。


## 安装

1. 依然使用docker image 部署

2. 如果没有安装docker [请参考这篇博客](https://jetbrains.org.cn/2017/09/13/maven%E7%A7%81%E6%9C%8D-nexus/)

3. 启动

		docker run -d --name sonarqube \
	    -p 9000:9000 -p 9092:9092 \
	    -e SONARQUBE_JDBC_USERNAME=sonar \
	    -e SONARQUBE_JDBC_PASSWORD=sonar \
	    -e SONARQUBE_JDBC_URL=jdbc:mysql://192.168.199.131:3306/tryspread?useUnicode=true&characterEncoding=utf-8 \
	    sonarqube
	    
4. 访问。http://ip:9000

	![](http://cdn-blog.jetbrains.org.cn/17-9-21/39675683.jpg)

## 项目中使用
### 使用方式1 
1. maven setting.xml 设置
		![](http://cdn-blog.jetbrains.org.cn/17-9-21/60017734.jpg)
		
		
2.  pom添加插件
		
		<plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.3.0.603</version>
        </plugin>
	
3. 使用。执行 mvn sonar:sonar 即可

### 使用方式2 
1. pom添加插件
   		
   		<plugin>
                   <groupId>org.sonarsource.scanner.maven</groupId>
                   <artifactId>sonar-maven-plugin</artifactId>
                   <version>3.3.0.603</version>
         </plugin>
2. 使用默认的帐号登录之后,可以:
    * 生成一个代替帐号的`token`
    * 修改一个`admin`的密码
    * 可以在`Administration`=>`System`=>`Update Center`,安装中文插件和其它要分析的语言的插件
    
3. 执行

        mvn clean package sonar:sonar \
          -Dsonar.host.url=http://localhost:9000 \  //此处是sonar控制台访问地址
          -Dsonar.login=token  //token 是登陆到sonar后自己设置的token 
          
        或者
        
        mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent package \ 
        -Dmaven.test.failure.ignore=true \
         deploy \ 
         sonar:sonar -Dsonar.host.url=http://192.168.199.131:9000 -Dsonar.login=2feb1b65a2224c9cb6744f35a7e45988e3443af6
 


## 观察
	
	![](http://cdn-blog.jetbrains.org.cn/17-9-21/98104526.jpg)
	
## 实时code review
    
    * 插件：SonarLint
    
    * 官网：http://www.sonarlint.org/intellij/
    
* 关于SonarQube常用设置，下个博客再讲。




