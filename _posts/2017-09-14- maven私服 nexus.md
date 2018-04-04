---
layout:     post
title:      maven私服 nexus
subtitle:    "\"maven私服 nexus 的安装与使用\""
date:       2017-09-13
author:     mrchang
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - maven
    - nexus
    - 私服
    
---

## 简介
1. 私服不是Maven的核心概念，它仅仅是一种衍生出来的特殊的Maven仓库。通过建立自己的私服，就可以降低中央仓库负荷、节省外网带宽、加速Maven构建、自己部署构建等，从而高效地使用Maven。Nexus也是当前最流行的Maven仓库管理软件。

## 要求

1. vps一台
2. 系统centos7
3. 内存大于1g

## 安装

1. 这里我们使用docker进行安装，crt连接到vps

2. ` yum -y install docker `

    ![](http://cdn-blog.jetbrains.org.cn/17-9-14/22748323.jpg)

3. 启动docker: `service docker start `

4. 查找nexus镜像 ： `docker search nexus`

   ![](http://cdn-blog.jetbrains.org.cn/17-9-14/93572570.jpg)
   
5. 一般情况下，我们都是用stars最高的。`docker pull docker.io/sonatype/nexus`

6. 启动nexus容器，对于以后的容器启动，[不清楚如何启动可以去docker hub 查看][https://hub.docker.com/]，

    ![](http://cdn-blog.jetbrains.org.cn/17-9-14/58578914.jpg)
    
7. 一般直接搜索`run `就可以找到如何启动

    ![](http://cdn-blog.jetbrains.org.cn/17-9-14/87774408.jpg)
    
8. 访问查看，管理员账户密码 `admin admin123`

    ![](http://cdn-blog.jetbrains.org.cn/17-9-14/45145189.jpg)
    
9. maven设置

    * setting.xml设置
        ![](http://cdn-blog.jetbrains.org.cn/17-9-14/65102103.jpg)
        
    * pom文件设置
    
        ![](http://cdn-blog.jetbrains.org.cn/17-9-14/12953066.jpg)
        
          <repositories>
                <repository>
                    <id>nexus</id>
                    <name>Team Nexus Repository</name>
                    <url>http://nexus.jetbrains.org.cn/nexus/content/groups/public</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>nexus</id>
                    <name>Team Nexus Repository</name>
                    <url>http://nexus.jetbrains.org.cn/nexus/content/groups/public</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        
        ![](http://cdn-blog.jetbrains.org.cn/17-9-14/81818372.jpg)
        
            <distributionManagement>
                <repository>
                    <id>releases</id>
                    <name>Nexus Release Repository</name>
                    <url>http://nexus.jetbrains.org.cn/nexus/content/repositories/releases/</url>
                </repository>
                <snapshotRepository>
                    <id>snapshots</id>
                    <name>Nexus Snapshot Repository</name>
                    <url>http://nexus.jetbrains.org.cn/nexus/content/repositories/snapshots/</url>
                </snapshotRepository>
            </distributionManagement>

    备注： pom中上传的设置id要与maven setting中的id保持一致。

10. 项目执行mvn delepoy 即可上传到私服。

11. 至于releases库与snaoshots库的区别
    * 简单去说就是relesses库是稳定版本或者生产版本，snaoshots库是不稳定版本或开发版本，项目版本号后面带
`-SNAPSHOT` 的都会上传到snaoshots库。

## 备注

如果觉得中央仓库慢的话，可以用我的私服 只需要配置 `repositories` 标签中的内容,`下载` 依赖， `distributionManagement` 标签中的内容为 `上传`， 无需配置，也不对外开放上传权限。
