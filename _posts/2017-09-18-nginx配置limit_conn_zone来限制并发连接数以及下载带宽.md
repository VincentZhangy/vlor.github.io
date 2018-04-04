---
layout:     post
title:      nginx配置limit_conn_zone来限制并发连接数以及下载带宽
subtitle:    "\"nginx配置limit_conn_zone来限制并发连接数以及下载带宽\""
date:       2017-09-18
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - nginx
    - limit_conn_zone
    - 并发连接数
    - 下载带宽

---


## 配置方法如下：

1. 在nginx.conf里的http{}里加上如下代码：

		#ip limit
		limit_conn_zone $binary_remote_addr zone=perip:10m;
		limit_conn_zone $server_name zone=perserver:10m;
		
2. 在需要限制并发数和下载带宽的网站配置server{}里加上如下代码

		limit_conn perip 2;
		limit_conn perserver 20;
		limit_rate 100k;
		
补充说明下参数：

		$binary_remote_addr是限制同一客户端ip地址；
		$server_name是限制同一server最大并发数；
		limit_conn为限制并发连接数；
		limit_rate为限制下载速度；
	
	
* 转载：http://blog.csdn.net/plunger2011/article/details/37812843