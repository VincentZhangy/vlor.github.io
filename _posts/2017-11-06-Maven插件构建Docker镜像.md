---
layout:     post
title:     	Maven插件构建Docker镜像
subtitle:    "\"Maven插件构建Docker镜像\""
date:       2017-11-06
author:     Mr Chang
header-img: img/post-bg-mma-0.jpg
catalog: true
tags:
    - Maven插件
    - Docker镜像
    - Docker Image

---


# 背景

微服务架构下，微服务在带来良好的设计和架构理念的同时，也带来了运维上的额外复杂性，尤其是在服务部署和服务监控上。单体应用是集中式的，就一个单体跑在一起，部署和管理的时候非常简单，而微服务是一个网状分布的，有很多服务需要维护和管理，对它进行部署和维护的时候则比较复杂。

下面从Dev的角度来看一下Ops的工作。从Dev提交代码，到完成集成测试的一系列步骤如下：

1. 首先是开发人员把程序代码更新后上传到Git，然后其他的事情都将由Jenkins自动完成。
2. 然后Git在接收到用户更新的代码后，会把消息和任务传递给Jenkins，然后Jenkins会自动构建一个任务，下载Maven相关的软件包。下载完成后，就开始利用Maven Build新的项目包，然后重建Maven容器，构建新的Image并Push到Docker私有库中。
3. 最后删除正在运行的Docker容器，再基于新的镜像重新把Docker容器启动，自动完成集成测试。

整个过程都是自动的，这样就简化了原本复杂的集成工作，一天可以集成一次，甚至是多次。

![](http://cdn-blog.jetbrains.org.cn/17-11-6/9447498.jpg)

本文主要关注的第二步，作为Dev使用Maven插件构建Docker镜像。



# 过程步骤

### 环境

笔者的电脑系统是MacOS，在进行下面的步骤之前，先具备一下条件：

* Docker Registry
* Maven（3.5.0）
* JDK(1.8.0_131)
* Docker for Mac (17.09.0-ce-mac35)


Maven 和JDK 就不用过多多了，必须具有的。Docker Registry是私有的hub，mac上装好docker之后，配置一下Docker Registry的地址，配置如下：

![](http://cdn-blog.jetbrains.org.cn/17-11-6/270945.jpg)

因为docker默认需要私服做https支持，我这边之前有个私服做了https支持，所以我这里就不需要配置了

### pom 配置 


pom文件中需要引入相应的插件。docker-maven-plugin有三款：spotify、fabric8io和bibryam。其中第一款最为流行，资料也多，所以毫不犹豫选择第一款。
插件有两种使用方式，一种是在直接在pom配置中指定baseImage和entryPoint。另一种适合于复杂的构建，使用dockerfile，只需要在配置中指定dockerfile的位置。前一种比较简单，此处略过，主要讲下第二种的配置


	<plugin>
	             <groupId>com.spotify</groupId>
	             <artifactId>docker-maven-plugin</artifactId>
	             <version>${maven.docker.version}</version>
	             <!--插件绑定到phase-->
	             <executions>
	                 <execution>
	                     <phase>install</phase>
	                     <goals>
	                         <goal>build</goal>
	                     </goals>
	                 </execution>
	             </executions>
	             <configuration>
	             <!--配置变量，包括是否build、imageName、imageTag，非常灵活-->
	                 <skipDocker>${docker.skip.build}</skipDocker>
	                 <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
	                 <!--最后镜像产生了两个tag，版本和和最新的-->
	                 <imageTags>
	                     <imageTag>${project.version}</imageTag>
	                     <imageTag>latest</imageTag>
	                 </imageTags>
	                 <forceTags>true</forceTags>                 
	                 <env>
	                     <TZ>Asia/Shanghai</TZ>
	                 </env>
	                 <!--时区配置-->
	                 <runs>
	                     <run>ln -snf /usr/share/zoneinfo/$TZ /etc/localtime</run>
	                     <run>echo $TZ > /etc/timezone</run>                      
	                 </runs>
	                 <dockerDirectory>${project.basedir}</dockerDirectory>
	                 <resources>
	                     <resource>
	                         <targetPath>/</targetPath>
	                         <directory>${project.build.directory}</directory>
	                         <include>${project.build.finalName}.jar</include>
	                     </resource>
	                 </resources>
	                 <!--push到私有的hub-->
	                 <serverId>docker-registry</serverId>
	             </configuration>

	</plugin>
	
	

`${maven.docker.version}`、`${docker.skip.build}`、`${docker.image.prefix}`都是可配置的变量。`${project.basedir}`、`${project.build.directory}`、`${project.build.finalName}`、`${project.version}`分别对应项目根目录、构建目录、打包后生成的结果名称、项目版本号。
上面的pom插件配置，指定了dockerfile的位置和镜像的命名规则。并将docker的build目标，绑定在install这个phase上。

![](http://cdn-blog.jetbrains.org.cn/17-11-6/87342576.jpg)

### dockerfile

	FROM java:8
	
	COPY target/maven-docker-image-0.0.1-SNAPSHOT.jar /app.jar
	EXPOSE 8080
	ENTRYPOINT ["java","-jar" ,"/app.jar"]
	
### setting.xml

在pom插件中，还有一个serverId的配置。这个配置是必要的，对于需要将image上传到私有hub上，在如上配置之后，只需要加上-DpushImage即可实现。serverId是与maven的配置文件setting.xml相对应，setting.xml中增加的配置：

	<server>
	  <id>docker-registry</id>
	  <username>用户名</username>
	  <password>密码</password>
	  <configuration>
	    <email>邮箱</email>
	  </configuration>
	</server>
	
### 结果

![](http://cdn-blog.jetbrains.org.cn/17-11-6/52931533.jpg)


![](http://cdn-blog.jetbrains.org.cn/17-11-6/14752412.jpg)


# 鸣谢

**http://blueskykong.com/2017/11/02/dockermaven/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io**

**https://github.com/spotify/docker-maven-plugin**

**https://github.com/changdaye/maven-docker-image**



