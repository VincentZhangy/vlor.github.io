---
layout:     post
title:      idea无法debug
subtitle:    "\"idea无法debug\""
date:       2017-09-13
author:     mrchang
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - idea
    - debug
---

## 起因
1. 今天原公司同事跟我说idea无法debugjava项目，但是可以正常非debug启动
   
## 经过
1. 问了下他java进程是不是卡住没有关闭，然后jmx换个端口，未解决
2. 随即远程到她电脑，观察debug的时候idea右下角提示 `Method breakpoints may dramatically slow down debugging`
    ,英语不好，google翻译结果是 `方法断点可能会大大减慢调试速度` ,随即在 Favorites 中清除了断点，重新debug运行，问题解决

## 结果
1. 水了这篇博客。


## 日常晒猫

   ![](http://cdn-blog.jetbrains.org.cn/17-9-13/11120190.jpg)