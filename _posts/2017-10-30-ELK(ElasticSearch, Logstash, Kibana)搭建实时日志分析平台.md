---
layout:     post
title:     	ELK(ElasticSearch, Logstash, Kibana)搭建实时日志分析平台
subtitle:    "\"ELK(ElasticSearch, Logstash, Kibana)搭建实时日志分析平台\""
date:       2017-10-30
author:     Mr Chang
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - ELK
    - ElasticSearch
    - Logstash
    - Kibana

---

# ELK平台介绍

日志主要包括系统日志、应用程序日志和安全日志。系统运维和开发人员可以通过日志了解服务器软硬件信息、检查配置过程中的错误及错误发生的原因。经常分析日志可以了解服务器的负荷，性能安全性，从而及时采取措施纠正错误。

通常，日志被分散的储存不同的设备上。如果你管理数十上百台服务器，你还在使用依次登录每台机器的传统方法查阅日志。这样是不是感觉很繁琐和效率低下。当务之急我们使用集中化的日志管理，例如：开源的syslog，将所有服务器上的日志收集汇总。

集中化管理日志后，日志的统计和检索又成为一件比较麻烦的事情，一般我们使用grep、awk和wc等Linux命令能实现检索和统计，但是对于要求更高的查询、排序和统计等要求和庞大的机器数量依然使用这样的方法难免有点力不从心。


开源实时日志分析ELK平台能够完美的解决我们上述的问题，ELK由ElasticSearch、Logstash和Kiabana三个开源工具组成。**官方网站：https://www.elastic.co/products**


1. Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。
2. Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。
3. Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

![](http://cdn-blog.jetbrains.org.cn/17-10-30/89030989.jpg)

如图：Logstash收集AppServer产生的Log，并存放到ElasticSearch集群中，而Kibana则从ES集群中查询数据生成图表，再返回给Browser。


# ELK平台搭建

### Elasticsearch 安装

1. [点击下载](https://www.elastic.co/cn/downloads/elasticsearch)，
2. 解压zip或者tar包等
3. 在bin/elasticsearch (windows版本是bin\elasticsearch.bat)
4. 请求 `http://localhost:9200/` 能看到带版本的返回值即可

### Logstash 安装

1. [点击下载](https://www.elastic.co/cn/downloads/logstash)
2. 解压zip或者tar包等
3. 在bin目录创建一个 logstash.conf文件 内容如下

		input {
		  tcp {
		   port => 4567 //项目日志配置文件中写的端口
		   type => "logs" 
		 }
		}
		
		filter {
		}
		output {
		        stdout{codec =>rubydebug}
		   elasticsearch {
		   hosts => ["localhost:9200"] // Elasticsearch 的ip 端口
		  }
		}
4. bin/logstash -f logstash.conf 即可

### Kibana 安装

1. [点击下载](https://www.elastic.co/cn/downloads/kibana)
2. 解压zip或者tar包等
3. vi config/kibana.yml 文件  修改 elasticsearch.url = Elasticsearch 的ip 端口 如Elasticsearch 的ip 端口
4. 执行bin/kibana （windows执行bin\kibana.bat）
5. 浏览器打开http://localhost:5601

# 验证

1. logback.xml 配置

![](http://cdn-blog.jetbrains.org.cn/17-10-30/50241995.jpg)

2. 测试类

![](http://cdn-blog.jetbrains.org.cn/17-10-30/5406510.jpg)

3. Logstash 日志

![](http://cdn-blog.jetbrains.org.cn/17-10-30/65094921.jpg)

4. Kibana 显示

![](http://cdn-blog.jetbrains.org.cn/17-10-30/12298904.jpg)


![](http://cdn-blog.jetbrains.org.cn/17-11-1/74960136.jpg)


