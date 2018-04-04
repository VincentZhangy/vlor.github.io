---
layout:     post
title:      Google BBR 安装
subtitle:    "\"Google BBR 加速\""
date:       2017-09-07
author:     mrchang
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Google BBR
   
---

## 功能
1. 单边加速，效果跟锐速差不多，

## 要求

1. 系统支持：CentOS 6+，Debian 7+，Ubuntu 12+
2. 虚拟技术：OpenVZ 以外的，比如 KVM、Xen、VMware 等
3. 内存要求：≥128M

## 源码

[项目源码](https://github.com/changdaye/shadowsocks)

![](https://ww4.sinaimg.cn/large/a15b4afegy1fjatmtrclkj21k20w411i)

## 安装

 1. 使用`ssh`工具登录到远程主机上，这里推荐使用CRT
 
 ![](https://ww4.sinaimg.cn/large/a15b4afegy1fjau5zr1lbj211o0l6n0r)
 
 2. 执行 `yum -y install git `
 3. 执行 `git clone https://github.com/changdaye/shadowsocks.git `
 4. 执行 `cd shadowsocks`
 5. 执行 `./bbr.sh`
 6. 安装完成后 uname -r 查看系统内核
 7. 含有 4.12 就表示 OK 了 sysctl net.ipv4.tcp_available_congestion_control 
 
 
## 鸣谢
 
 基于91vps脚本更改，前后台取自github相应开源项目，
 使用一键脚本使部署更简单。
 再次感谢相应开源作者的贡献
 
 
## 日常晒猫

![](http://cdn-blog.jetbrains.org.cn/17-9-9/51527777.jpg)
