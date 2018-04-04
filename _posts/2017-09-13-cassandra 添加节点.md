---
layout:     post
title:      cassandra 添加节点
subtitle:    "\"cassandra 添加节点\""
date:       2017-09-13
author:     mrchang
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - cassandra
    - nosql
    - 添加节点
---

## 特别说明
1. 无论你是添加新的节点还是替换一个节点，请准备全新的机器，特别是这台机器一定不要曾经安装过cassandra，如果这台机器曾经安装过cassandra（特别是cassandra 安装目录下的data目录还有日志），数据丢失后果自负。

## 安装步骤
增加一个节点和替换一个DOWN掉的节点，步骤都是一样的，只是启动参数不一样

1. 准备一个新机器，cassandra的配置使用和集群中一个普通节点相同的配置。
2. 
    * 然后就可以启动了，增加一个节点，只要bin/cassandra 启动就可以了。
    * 如果是替换一个节点（假设DOWN掉的节点ip=192.168.1.101），启动的时候，可以使用`bin/cassandra -Dcassandra.replace_address=192.168.1.101`来启动（只是第一次这样，以后就直接bin/cassandra启动就可以了）
3. 就是等待数据迁移，当你在其它机器上使用nodetool status看到新节点的状态变成UN状态的时候，就表示迁移完成了。你也可以在新节点上通过nodetool netstats查看数据迁移的进度。
 
如果你的集群数据量很大，这个数据迁移的过程将会给集群带来很大的负载。你需要在启动新节点之前做两件事情：

1. 关闭所有节点的压缩。
    * nodetool disableautocompaction 关闭自动压缩
    * nodetool stop COMPACTION 停止正在执行的压缩。
    * 当新节点启动之后，也要执行nodetool disableautocompaction。
    * 在数据迁移完毕之后，再放开即可nodetool enableautocompaction
2. 限制所有节点数据迁移流量
    * ./nodetool setstreamthroughput 32
        * 限制为32mbps 假设你的集群有10个机器，那么你的新节点的流量大约是32*10mbps。
    你可以根据数据迁移的进度，完成的节点个数，慢慢调大这个值。
    
    
## 日常晒猫

   ![](http://cdn-blog.jetbrains.org.cn/17-9-13/11120190.jpg)