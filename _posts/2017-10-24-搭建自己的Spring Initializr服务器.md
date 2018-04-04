---
layout:     post
title:     	搭建自己的Spring Initializr服务器
subtitle:    "\"搭建自己的Spring Initializr服务器\""
date:       2017-10-24
author:     Mr Chang
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Spring Initializr
    - start.spring.io

---


# 前言

之所以想要搭建自己的Spring Initializr服务器，是因为我的网络环境start.spring.io访问会time out，ping 的话丢包严重，以前公司网络翻墙会好一点，家庭网络是废的，现在基本是全废了。所以决定自己搭建个。

![](http://cdn-blog.jetbrains.org.cn/17-10-24/75742911.jpg)

简单看了下官方文档，发现可以直接自己搭建的，只需要源码打包然后在。。。。即可，详细看下面吧。



# 搭建

1. 访问github clone 源代码。地址：https://github.com/spring-io/initializr
2. 打包安装到本地，后面使用 mvn clean install
3. 创建springboot web 工程，并添加依赖。
![](http://cdn-blog.jetbrains.org.cn/17-10-24/19930134.jpg)
4. 引入配置文件，原始配置文件在service模块中
![](http://cdn-blog.jetbrains.org.cn/17-10-24/36748972.jpg)
	基本需要自己修改的也就是方框中的配置，这些配置在源码的**initializr-generator模块**中的这个类中**InitializrMetadata**
5. 最后springboot mvn package 打包，找地方部署即可。
	
# 效果

![](http://cdn-blog.jetbrains.org.cn/17-10-24/73063798.jpg)


![](http://cdn-blog.jetbrains.org.cn/17-10-24/71521342.jpg)

![](http://cdn-blog.jetbrains.org.cn/17-10-24/59750904.jpg)

# 说明

项目地址： https://github.com/changdaye/spring-initializr ，可直接去release中下载jar包使用

搭建完感觉直接找台可以访问start.spring.io的主机，做转发应该也可以，还能简单点。

不过也无所谓，有跟我情况差不多的小伙伴的话，可以使用这个地址： **http://start.jetbrains.org.cn/** 网页能打开基本就可以使用，不能用的话可以给我发邮件之类的。谢谢。



![](http://cdn-blog.jetbrains.org.cn/17-11-1/19024053.jpg)




