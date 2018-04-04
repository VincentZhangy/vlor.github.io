---
layout:     post
title:     	jenkins上传jar，war到阿里云oss
subtitle:    "\"jenkins maven package后上传到私服，并同步到阿里云oss \""
date:       2017-10-19
author:     Mr Chang
header-img: img/post-bg-mma-1.jpg
catalog: true
tags:
    - jenkins
    - oss
    - nexus
    - 私服

---


## 说明

建立统一的artifacts仓库是后续的持续部署的前提。目前，建立artifacts仓库大致有如下三种选择：

FTP服务器：很多用户仍然在用这种方式存储Artifact

专业的Artifacts存储仓库：比如Nexus, Artifactory等。

对象存储服务：比如阿里云OSS，AWS S3等。如果用户的应用系统全部部署在阿里云中，那么使用阿里云OSS来建立artifacts仓库的好处是，a)可靠性、高可用性 b) 上传、下载速度快。

Jenkins是当前最常用的CI服务器，FIT2CLOUD Aliyun-OSS-Plugin for Jenkins的功能是：将构建后的artifact上传到OSS的指定位置上去。

**像我这种需求就是：nexus保存一份，阿里云oss在保存一份**


## 安装说明


插件下载地址：http://files.jetbrains.org.cn/aliyun-oss.hpi 在Jenkins中安装插件, 请到 Manage Jenkins->Advanced -> Upload，上传插件(.hpi文件) 安装完毕后请重新启动Jenkins

## 配置说明

在使用插件之前，必须先在Manage Jenkins | Configure System | 阿里云OSS账户设置]中配置阿里云帐号的Access Key、Secret Key和阿里云EndPoint后缀.
![](http://cdn-blog.jetbrains.org.cn/17-10-19/1133242.jpg)
阿里云后缀：内网填"-internal.aliyuncs.com",外网填".aliyuncs.com",默认外网。如果您的Jenkins也部署在阿里云上面，那么可以使用内网，上传速度更快。

## Post-build actions: 上传Artifact到阿里云OSS


在Jenkins Job的Post-build actions，用户可以设上传Artifact到阿里云OSS。需要填写的信息是：

Bucket名称: artifact要存放的bucket

要上传的artifacts: 文件之间用;隔开。支持通配符描述，比如 text/*.zip

Object前缀设置：可以设置object key的前缀，支持Jenkins环境变量比如: "${JOB_NAME}/${BUILD_ID}/${BUILD_NUMBER}/"

假设一个job的名称是demo，用户的设置如下

bucketName: nexus-backup

要上传的artifacts: *.jar;*.war

Object前缀: ${JOB_NAME}/${BUILD_ID}/${BUILD_NUMBER}
![](http://cdn-blog.jetbrains.org.cn/17-10-19/36125309.jpg)

## 效果图

jenkins 的log 查看 

![](http://cdn-blog.jetbrains.org.cn/17-10-19/12771597.jpg)

因为是走的内网 ，这里可以看到上传时间，是极快的。



## 鸣谢

**github ： https://github.com/fit2cloud/aliyun-oss-plugin**

**七牛 ： https://portal.qiniu.com/signup?code=3lgwdn66m24pe**



![](http://cdn-blog.jetbrains.org.cn/17-10-19/86943307.jpg)




