---
layout:     post
title:      记一次 Linux 下 XordDos 木马的清除
subtitle:    "\"记一次 Linux 下 XordDos 木马的清除\""
date:       2017-08-14
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Linux
    - XordDos

---


## 前言

1. 朋友（小蒋子）的服务器中了木马后，一直向外发送请求，带宽被占满。甚至还引起了阿里云网络异常。

2. 用 iptables 封禁向外请求的 ip 后，网络请求终于没有了，但是服务器资源被木马脚本占用

首先我并不知道服务器中了什么招，因此从以下几个步骤追踪木马：

## 解决

1. 参考之前解决公司生产上种挖矿木马minerd的经验
	* 查看redis中是否有crond脚本，没有
	* 查看crontab -e 中是否有crond 脚本，没发现。

2. 只能面向Google解决问题，因为之前从防火墙看到服务器是一直在`往外发包`的，所以猜测可能是`ddos`一类的病毒。

3. top查看资源占用最高的进程id与名称，并kill掉，发现几秒后又出现了`随机字母`的另外一个名字的进程

4. 重复kill几次，发现名称一直是 `随机字母 固定位数`。

5. Google搜索linuxDdos木马，随机字母进程名，发现[这篇帖子感觉最靠谱](https://sebastianblade.com/linux-xorddos-trojan-removal/)。
6. 详细按照步骤操作后解决。

## 总结

1. 优先查看 redis与crond中是否有定时脚本。有人会问为什么要看redis中，[请参考](http://11019859.blog.51cto.com/11009859/1850771)，[这篇帖子](https://huangjj27.gitlab.io/2017/01/12/redis-minerd/)。

2. 分析病毒木马是否有相应特征。

3. 如果你是像我一样的初级开发，那么找到特征后用关键字去Google吧。

4. 祝好运。


