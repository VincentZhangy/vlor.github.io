---
layout:     post
title:      shadowsocksR多用户
subtitle:    "\"一键安装ShadowsocksR多用户管理面板|ss-panel魔改脚本\""
date:       2017-09-07
author:     mrchang
header-img: img/bg_151.png
catalog: true
tags:
    - shadowsocksR
    - ss-panel v3
---

## 功能
1. 包含用户注册模块，邀请模块，节点模块，商城模块，后台管理模块
2. 支持限速、限流、设置访问规则、机器人服务、修改站点外观、BBR加速

## 要求
Centos系统vps一台

vps主机在国外，这里推荐香港

## 源码

[项目源码](https://github.com/changdaye/shadowsocks)

![](https://ww4.sinaimg.cn/large/a15b4afegy1fjatmtrclkj21k20w411i)

## 安装

 1. 使用`ssh`工具登录到远程主机上，这里推荐使用CRT

 ![](https://ww4.sinaimg.cn/large/a15b4afegy1fjau5zr1lbj211o0l6n0r)

 2. 执行 `yum -y install git `
 3. 执行 `git clone https://github.com/changdaye/shadowsocks.git `
 4. 执行 `cd shadowsocks`
 5. 执行 `./ss-panel-v3-mod.sh`
 6. 先安装前台系统，输入 `1` ,并根据提示操作
 7. 经过大约20分钟的等待，前台系统安装完成，默认配置文件在 `/home/wwwroot/default/config/.config.php`

 ![](https://ww4.sinaimg.cn/large/a15b4afegy1fjauqefxe7j21740ku13m)

 * 默认账号：ss@feiyang.li
 * 默认密码：feiyang

 8. 在后台管理中添加一个节点

 ![](https://ww4.sinaimg.cn/large/a15b4afegy1fjavigevjhj21201boafq)

 9. 安装后台节点，执行 `./ss-panel-v3-mod.sh`，输入  `2`

 10. doamin： 域名或者ip都可以，前边要加http或者https。不要弄错，否则可能出现无法推送使用记录的情况。

	 mukey(token)：默认回车，除非你另行设置过。

     node_id：还记得上边那个图里边的ID吗？就是那个了。。。

     配置文件在 `userapiconfig.py`
 11. 在网站节点列表中查看，该节点状态是否变为绿色，即为安装完成
 ![](https://ww4.sinaimg.cn/large/a15b4afegy1fjavmu3zfsj20go0hwjt1)

## 鸣谢

 基于91vps脚本更改，前后台取自github相应开源项目，
 使用一键脚本使部署更简单。
 再次感谢相应开源作者的贡献

## 日常晒猫

![](http://cdn-blog.jetbrains.org.cn/17-9-9/66408733.jpg)
