---
layout:     post
title:     	spring-boot-maven-plugin 的坑
subtitle:    "\"spring-boot-maven-plugin 的坑\""
date:       2018-02-14
author:     Mr Chang
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - spring-boot-maven-plugin

---


# 作用

spring-boot-maven-plugin插件在打Jar包时会引入依赖包

maven项目的pom.xml中，添加了org.springframework.boot:spring-boot-maven-plugin
插件，当运行“mvn package”进行打包时，会打包成一个可以直接运行的 JAR 文件，使用“Java -jar”命令就可以直接运行。


# 效果

一般的maven项目的打包命令，不会把依赖的jar包也打包进去的，只是会放在jar包的同目录下，能够引用就可以了，但是spring-boot-maven-plugin插件，会将依赖的jar包全部打包进去。比如下面这个jar包的BOOT/INF/lib目录下面就包含了所有依赖的jar包

![](http://cdn-blog.jetbrains.org.cn/18-2-14/76774181.jpg)

spring-boot-maven-plugin插件，在很大程度上简化了应用的部署，只需要安装了 JRE 就可以运行。

但是俺测试发现它的一个缺点，就是它打包成的这个jar包，在被别的项目引用的时候，`会出问题`。如果没有这个插件打包的话，那么它的目录结构是：

![](http://cdn-blog.jetbrains.org.cn/18-2-14/62458663.jpg)



比较这两次打包的区别：在使用spring-boot-maven-plugin插件时，打包后的目录包括三个，BOOT-INF,META-INF,org.springframework.boot.loader，在lib目录里包含了我自己的项目的代码目录；

在没有使用spring-boot-maven-plugin插件时，打包的目录只有两个，META-INF和我自己的项目代码的目录。
　　
　　

