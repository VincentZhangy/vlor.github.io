---
layout:     post
title:      cassandra 安装
subtitle:    "\"cassandra 安装与配置\""
date:       2017-09-10
author:     mrchang
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - cassandra
    - nosql
   
---

## 下载

1. 去[官网](http://cassandra.apache.org/)下载安装包
2. 安装包

	![](http://cdn-blog.jetbrains.org.cn/17-9-10/36454228.jpg)
3. 上传到服务器后tar解压
4. 目录讲解
	![](http://cdn-blog.jetbrains.org.cn/17-9-10/97042215.jpg)
	
## 要求

1. 主机一台
2. jdk 1.7+ 一个

## 安装
1. 安装jdk 并配置环境变量，这里就不多说了
2. 修改cassandra配置
	* 主要修改 /conf/cassandra.yaml，简单安装主要修改四处
		* `rpc_address: 192.168.40.211（本机ip）`
		* `listen_address: 192.168.40.211（本机ip）`
		* `- seeds: "192.168.40.211"（集群seeds节点ip）`
		* `authenticator: PasswordAuthenticator（cassandra 登陆用户名密码）`
	* PS:关闭防火墙 service iptables stop（此命令为临时关闭）或开放7000（cassandra 节点间通信端口） 与9042 端口（外部访问数据库通信端口）
3. 启动cassandra

	`cd bin/`
	
	`./cassandra -f (前台启动 去掉-f 后台启动)`
## 客户端
1. cassandra 客户端 这里推荐使用DevCenter，[下载链接](https://www.devcenter.co/)

	![](http://cdn-blog.jetbrains.org.cn/17-9-10/47114996.jpg)
	
2. 新建链接

	![](http://cdn-blog.jetbrains.org.cn/17-9-10/14531145.jpg)
	
3. 添加ip地址

	![](http://cdn-blog.jetbrains.org.cn/17-9-10/78760046.jpg)

4. 添加账号密码（密码为：cassandra）

	![](http://cdn-blog.jetbrains.org.cn/17-9-10/71788755.jpg)

5. 执行建表语句
	* cassandra简单建表语句与关系型数据库类似 
	* replication_factor' :3 （3表示副本因子为3 需要<=集群节点数）

## 特别强调

1. cassandra 存放集合 是有大小限制的
	* 集合的每一项最大是64K
	* 保持集合内的数据不要太大，免得Cassandra 查询延时过长，只因Cassandra 查询时会读出整个集合内的数据，集合在内部不会进行分页，集合的目的是存储小量数据。
	* 不要向集合插入大于64K的数据，否则只有查询到前64K数据，其它部分会丢失。
  	
2. cassandra数据库的数据一致性问题
	* 要求各个节点中时间一致性，即系统时间，请保证系统时间一致请做ntp同步。ntp同步问题请参考另一篇文档代码解决数据一致性问题 ，保存更新数据库记录时候，设置时间 即 setDefaultTimestamp(System.currentTimeMillis()))
            
3. 删除数据后，数据复活问题      
	* Cassandra通过写一条“tombstone”来标记一个数据被删除了。被标记的数据默认要10天(配置文件中的gc_grace_seconds)后且被compaction或cleanup执行到对应的SSTable时才会被真正从磁盘删除，因为如果当时这个delete操作只在3个节点中的2个执行成功，那么一旦2个有tombstone的节点把数据删了，集群上只剩下没tombstone的那个节点，下次读这个key的时候就又返回对应的数据，从而导致被删除的数据复活。Repair操作可以同步所有节点的数据从而保证tombstone在3个节点中都存在，因此如果想确保删除100%成功不会复活需要以小于gc_grace_seconds的周期定期执行repair操作(所以官方建议”weekly”)。
	
	* 然而在select数据的时候，在每个SSTable遇到的所有符合查询条件的tombstone要放内存中从而在合并每个SSTable文件的数据时判断哪些column数据没被删能最终返回，tombstone太多会导致heap被大量占用需要各种GC从而影响性能，所以cassandra默认读到100000个(可配置)tombstone就强制取消这次读取，因为遇到这种情况一定是你“doing it wrong”。因此如果经常删除一个row key下的column key，同时又有一次select一个row key下多个column key的需求，很容易遇到这种情况，tombstone很多的时候即使不到10w最终成功读取了，这次读取也会很慢，并很快导致触发年轻代GC从而拖慢整个节点的响应速度。
	
	* 最好的解决方案肯定是设计的时候就避免。但假如在不修改表结构的情况下，解决方案有两种：如果不能接受一个column key被删又复活，先把gc_grace_seconds改成0，然后删除的时候用ALL模式，当然只要有1个节点超时/挂掉删除就会失败；如果能接受被删的数据复活，删除只为了减少磁盘空间使用，那就直接把gc_grace_seconds设成0，复活就复活吧。


## 日常晒猫

   ![](http://cdn-blog.jetbrains.org.cn/17-9-10/9896790.jpg)