---
layout:     post
title:     	uptimerobot 监控
subtitle:    "\"多种监控实现\""
date:       2018-03-22
author:     Mr Chang
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - 监控
    - monitor  
---


# 前言

由于搞了多个公共服务于多台vps，需要监控项目稳定性与服务器稳定性，考察了阿里云云监控与uptimerobot，最后选择了uptimerobot


# 教程

1. 访问官网，注册账号 ： https://uptimerobot.com/
2. 登陆，点击`add new monitor`，根据自己的需求添加
3. 配置自定义域名，点击`my setting`,在右下角找到`Public Status Pages`,点击` Add Public Status Page`,里面主要配置的就是自己的域名，即：`Custom Domain`
4. 配置域名解析，添加域名设置CNAME到uptimerobot返回给你的域名即可。


# 效果

 ![](http://cdn-blog.jetbrains.org.cn/18-3-22/83204382.jpg)
 

 ![](http://cdn-blog.jetbrains.org.cn/18-3-22/37499340.jpg)
 
 
# 查看网址

   https://monitor.jetbrains.org.cn/