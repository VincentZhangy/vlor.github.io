---
layout:     post
title:     	ArchiSteamFarmV3.0使用服务器steam云挂卡
subtitle:    "\"ArchiSteamFarmV3.0使用服务器steam云挂卡\""
date:       2017-10-29
author:     Mr Chang
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - ArchiSteamFarm
    - ASF
    - 云挂卡
    - Steam

---


# 前言

ASF不再基于.NET Framework 4.6.1（ASF V2系列）编写，如今使用.NET Core 2.0（ASF V系列）。值得庆幸的是，3.0系列的新出了linux版的程序，想在云端挂卡的童鞋，再也不用安装mono了，更不用使用win版的服务器，大大降低了使用内存，提高了运行速度等。

# 安装步骤

首页你要有一个云服务器，阿里云，腾讯云等等，也可以用跟国外免费的vps，随便一个都可以，只使用ASF挂卡，最低的配置都能驾驭。

我使用的是腾讯云的学生主机，早就过了学生年代了，在TB花钱认证的8年学生身份，你没看错是8年。。。每个月12元2核2G1M，除了创建这个blog，还在用来steam挂卡，和开饥荒服务器什么的。

我安装的原版的CentOS 7.0 64位，SSH工具使用的putty-0.65 FTP工具使用的WinSCP，个人认为win上用起来最顺手好用的工具。

### 安装.NET Core

既然ASF3.0 基于.net core开发，我们就要安装它，SSH连接服务器后，输入命令：

`sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc`

回车 继续输入

`sudo sh -c 'echo -e "[packages-microsoft-com-prod]\nname=packages-microsoft-com-prod \nbaseurl=https://packages.microsoft.com/yumrepos/microsoft-rhel7.3-prod\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/dotnetdev.repo'`

进行安装操作

`sudo yum update`

`sudo yum install libunwind libicu`

`sudo yum install dotnet-sdk-2.0.0`

以上命令全部完成，安装.net core完毕，就可以进行使用ASF。


### 安装ArchiSteamFarm

ASF需要到[ArchiSteamFarm的github](https://github.com/JustArchi/ArchiSteamFarm/releases/)的发布页下载，下载最新的发布版即可，这里使用的是ASF-linux-x64，下载好后放在本地。
打开ftp工具，将ArchiSteamFarm拷贝到你服务器的文件中，比如opt文件下：

![](http://cdn-blog.jetbrains.org.cn/17-10-29/8874290.jpg)

打开SSH工具，cd到你的安装目录下，比如我的

`cd /opt/ASF`

对ASF工具提升权限，直接输入命令：

`chmod +x ArchiSteamFarm`

### 使用ArchiSteamFarm

在使用之前我们可以先编辑它的config文件夹，里面是对你的asf的设置，如果以前你使用过2.0的话，你直接将2.0中的config文件覆盖进去即可。
如果是第一次使用，推荐用官方给你网页版bot编辑页面进行设置，[点击进入网页版设置页面](https://justarchi.github.io/ArchiSteamFarm/#/)
设置方法如下：

	取一个Name，这可以是你的昵称或任何其他你想命名你的机器人。请避免空格，可以_用作单词分隔符。
	开Enabled开关。
	果您使用Steam Parental PIN解锁您的帐户，请将其放入Parental PIN。
	写Steam Login您用于登录的Steam帐户名称（可选）。
	写Steam Password您用于登录的Steam密码（可选）。
	现在点击Download按钮，如果你正确地完成了一个新的BotName.json文件将被下载。找到该文件并将其放入config目录（在ASF目录中）。我命名我的机器人Main，所以下载的文件被命名Main.json。
	
![](http://cdn-blog.jetbrains.org.cn/17-10-29/68679737.jpg)

当然你也可以直接编辑它的json文件，也很简单。

asf设置完成后，我们使用screen开启新的窗口，如果你没用安装screen请输入以下命令安装：

`yum -y install screen`

安装后我们开启窗口

//开启一个名为asf的窗口，或者直接输入screen开启窗口
`screen -s asf`

然后启动asf

`./ArchiSteamFarm`

![](http://cdn-blog.jetbrains.org.cn/17-10-29/58347588.jpg)

注意， 如果开启的时候提示没有权限,请查看上面的步骤，使用chmod命令加权。


下面是screen的一些操作：


	开启后不要直接ctrl+c关闭窗口，这会让你ASF直接关掉，请使用ctrl+a+d来讲窗口部署到后台，在关闭SSH
	查看已经启动的窗口screen -ls
	杀掉窗口，我们用ls命令的到窗口进程号，比如9527。我们使用kill 9527来关掉窗口。
	打开使用的窗口 screen -r
	
到此，我们完成所有操作，可以安心的挂卡了。



![](http://cdn-blog.jetbrains.org.cn/17-11-1/38709407.jpg)





