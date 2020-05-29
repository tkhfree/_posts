---
title: hadoop的介绍一
date: 2019-10-25 03:42:21
tags: Hadoop
---

apache社区里面的分布式存储和分布式计算的框架
- hadoop common
- hdfs
- hadoop yarn
- hadoop mapreduce
借助于hadoop可以把文件和程序自动的分布存储和分布计算

## hdfs:
将文件切割成指定大小数据块，多副本存储在多个机器上
![htfs](hadoop的介绍一/hdfs.png)

## yarn:
资源的管理和调度
多框架&容错性&扩展性
一些组件跑在yarn上，具体如：
![yarn](hadoop的介绍一/yarn.png)

##MapReduce
分布计算的框架
扩展性&容错性&海量数据离线处理
![MapReduce](hadoop的介绍一/mapreduce.png)
map对每一个输入建立key-value值
shuffling重新洗牌，把相同key发到同一个机器
reduce统计相同key的value值

# hadoop生态系统
![hadoop生态](hadoop的介绍一/hadoop.png)
![hadoop生态](hadoop的介绍一/hadoop1.png)
zookeeper分布式协调框架
flume采集器
pig脚本语言
Oozie工作流
Hive是sql查询
Hbase链式存储，hadoop里面的数据库
