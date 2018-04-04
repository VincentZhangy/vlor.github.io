---
layout:     post
title:      IdeTalk+Openfire
subtitle:    "\"Idea插件之IdeTalk，使用openfire作服务器\""
date:       2017-09-08
author:     mrchang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Idea
    - IdeTalk
    - openfire
   
---

## 前言
1. 继idea中使用ideTalk插件在日常开发中协同工作续，使用openfire作为通讯服务器，将外网内的远程工作同学添加进来。

## 功能
1. 用户注册，登录
2. 用户群组管理，ideTalk项目组管理
3. 服务端统一推送消息到每个用户
4. 消息日志查看

## 安装

1. 先去[官网](http://www.igniterealtime.org/downloads/index.jsp)下载openfire，这里我们下载的是tar包，方便linux服务器部署

	![](http://cdn-blog.jetbrains.org.cn/17-9-8/4836705.jpg)

2. 将tar包上传到服务器，使用`rz`或者`ftp`等。当然你也可以直接使用`wget`命令直接在服务器上下载

	![](http://cdn-blog.jetbrains.org.cn/17-9-8/96684657.jpg)
	
3. `tar -zxvf openfire_4_1_5.tar.gz `. (根据你所下载的版本调整)

4. openfire 启动在 bin 目录 ，执行 bin/openfire start 

5. 访问 `http://your vps ip :9090/` 根据提示操作

6. 登录用户名为 admin ， 密码为自己设置的密码。

   ![](http://cdn-blog.jetbrains.org.cn/17-9-8/24688095.jpg)
7.ideTalk 直接点击 插件上的小人 注册即可，端口是 5222 
	![](http://cdn-blog.jetbrains.org.cn/17-9-8/62768282.jpg)
	
8.openfire上查看用户
	![](http://cdn-blog.jetbrains.org.cn/17-9-8/8605390.jpg)



## 日常晒猫
	
![](http://cdn-blog.jetbrains.org.cn/17-9-9/97205679.jpg)