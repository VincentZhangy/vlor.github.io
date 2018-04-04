---
layout:     post
title:     	springboot 使用随机数
subtitle:    "\"springboot 使用随机数\""
date:       2017-12-04
author:     Mr Chang
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - springboot
    - 使用随机数

---



# 参数间的引用

	com.blog.name=mrchang
	com.blog.title=Spring Boot教程
	com.blog.desc=${com.blog.name}正在努力写《${com.blog.title}》

# 使用随机数

在一些情况下，有些参数我们需要希望它不是一个固定的值，比如密钥、服务端口等。Spring Boot的属性配置文件中可以通过${random}来产生int值、long值或者string字符串，来支持属性的随机值。


	# 随机字符串
	com.blog.value=${random.value}
	# 随机int
	com.blog.number=${random.int}
	# 随机long
	com.blog.bignumber=${random.long}
	# 10以内的随机数
	com.blog.test1=${random.int(10)}
	# 10-20的随机数
	com.blog.test2=${random.int[10,20]}