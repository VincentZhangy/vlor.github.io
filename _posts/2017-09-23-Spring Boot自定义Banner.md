---
layout:     post
title:     	Spring Boot自定义Banner
subtitle:    "\"Spring Boot自定义Banner\""
date:       2017-09-23
author:     mrchang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring Boot
    - Banner

---

## 启动效果

![](http://cdn-blog.jetbrains.org.cn/17-9-23/83251554.jpg)

## 步骤

1. 新建Spring Boot工程
2. 在/src/main/resources目录下创建一个banner.txt文件
3. 将ASCII字符画复制进去，就能替换默认的banner了

		${AnsiColor.BRIGHT_YELLOW}
		////////////////////////////////////////////////////////////////////
		//                          _ooOoo_                               //
		//                         o8888888o                              //
		//                         88" . "88                              //
		//                         (| ^_^ |)                              //
		//                         O\  =  /O                              //
		//                      ____/`---'\____                           //
		//                    .'  \\|     |//  `.                         //
		//                   /  \\|||  :  |||//  \                        //
		//                  /  _||||| -:- |||||-  \                       //
		//                  |   | \\\  -  /// |   |                       //
		//                  | \_|  ''\---/''  |   |                       //
		//                  \  .-\__  `-`  ___/-. /                       //
		//                ___`. .'  /--.--\  `. . ___                     //
		//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
		//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
		//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
		//      ========`-.____`-.___\_____/___.-`____.-'========         //
		//                           `=---='                              //
		//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
		//            佛祖保佑       永不宕机     永无BUG                  //
		////////////////////////////////////////////////////////////////////
		
		
4. 从上面的内容中可以看到，还使用了一些属性设置：
	* `${AnsiColor.BRIGHT_RED}`：设置控制台中输出内容的颜色
	* `${application.version}`：用来获取MANIFEST.MF文件中的版本号
	* `${application.formatted-version}`：格式化后的`${application.version}`版本信息
	* `${spring-boot.version}`：Spring Boot的版本号
	* `${spring-boot.formatted-version}`：格式化后的`${spring-boot.version}`版本信息

## 生成ASCII字符网站

1. http://patorjk.com/software/taag
2. http://www.degraeve.com/img2txt.php
3. http://www.network-science.de/ascii/
