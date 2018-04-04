---
layout:     post
title:     	Docker Compose 配置文件详解
subtitle:    "\"Docker Compose 配置文件详解\""
date:       2017-12-17
author:     Mr Chang
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - mac
    - rsync
    - 备份

---


先来看一份 docker-compose.yml 文件，不用管这是干嘛的，只是有个格式方便后文解说：

	version: '2'
	services:
	  web:
	    image: dockercloud/hello-world
	    ports:
	      - 8080
	    networks:
	      - front-tier
	      - back-tier
	
	  redis:
	    image: redis
	    links:
	      - web
	    networks:
	      - back-tier
	
	  lb:
	    image: dockercloud/haproxy
	    ports:
	      - 80:80
	    links:
	      - web
	    networks:
	      - front-tier
	      - back-tier
	    volumes:
	      - /var/run/docker.sock:/var/run/docker.sock 
	
	networks:
	  front-tier:
	    driver: bridge
	  back-tier:
	driver: bridge
	

可以看到一份标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分，下面先来看 services 的书写规则。

![](http://cdn-blog.jetbrains.org.cn/17-12-17/83995157.jpg)




转载

**http://www.jianshu.com/p/2217cfed29d7**