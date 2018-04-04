---
layout:     post
title:      cassandra 数据迁移
subtitle:    "\"cassandra 数据迁移\""
date:       2017-09-13
author:     mrchang
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - cassandra
    - nosql
    - 数据迁移
---

## 特别说明
1. cassandra 数据迁移简单分成2种情况，修改表结构与没有修改表结构

## 不修改表结构
1. 请准备全新服务器，并安装cassandra后，删除cassandra 目录下的data目录。将原服务器中的data目录下的文件，copy到新服务器的同目录下，启动新服务器数据库即可。

## 已修改表结构

1. 可行方案，分拼查询出数据，然后插入到新数据库中，并删除原数据库环境中相应数据。缺点：数据量大消耗时间太长。

2. 可行方案，解决上面消耗时间问题。
    *  命令 `./cqlsh ip -u cassandra -p cassandra `
    *  导出  `contract_info=tablename  contract_serial_number=列明`
    
 *  `copy contract_info (contract_serial_number,biz_id,biz_key_values,contract_id,contract_num,create_time,delete_flag,file_id,file_path,file_title,file_type,status) to'/root/shuju/contract_info.csv';
          ` 导入
 *   `copy contract_info (contract_serial_number,biz_id,biz_key_values,contract_id,contract_num,create_time,delete_flag,file_id,file_path,file_title,file_type,status) from '/root/shuju/contract_info.csv';`
 
## 注意
1. 列名保持导入导出一致，数量一致，导入是序列化写入到cvs的每一列，导入也是序列化，所以列个数要一致，你可以换列的位置（如有需要调换列的数据时候），
            如果你修改了表结构，某一列又不需要了，请建表时创建这一列，数据迁移结束后，在删除这一列。
            ALTER TABLE contract_mgt.contract_info
            drop biz_key_values;

