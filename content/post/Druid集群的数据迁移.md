+++
title = "Druid集群的数据迁移"
draft = false
date = "2017-11-08"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["druid","kafka"] 
toc = true

+++

公司重新购买了Hadoop集群，硬盘不足的问题解决了。想把HDFS作为Deep Storage用起来。先把思路记录下来，等折腾完了再补充。


## Mysql到Druid

### 确定时间点
- 按照UTC时间划分
- 将Druid之前试跑数据废弃
- 将Mysql数据按天生成Json文件，批量导入
- 数据校验


### 影响
- 查询需要并两个dataSource
- Mysql数据没有Uniq相关的Metric


## NFS based到HDFS based


### 目标
- 尽可能快的迁移
- 数据完整性和正确


### 方案一：基于Offset

#### 步骤：

①关掉旧Druid的KIS服务，并记录下消费的offset

②修改Segments的名字

③迁移Segements到HDFS

④重新Index

⑤在新Druid上面启动KIS，Offset设定

#### 问题：

1. KIS的Offset问题

保存在metadata的DruidTask中？
```
"startPartitions" : {
			"partitionOffsetMap" : {
				"11" : 3,
				"2" : 3,
				"5" : 3,
				"8" : 3
			},
			"topic" : "topic1"
			
```

没有找到直接的办法，复制任务风险太大，可能需要修改源代码

2. 同一Segment的数据是两个集群生成的，合并起来的风险？例如segment的Id是递增的

### 方案二：基于时间划分


#### 步骤：

①在新集群上面启动KIS服务，两个集群同时跑一段时间

②根据日期切割

#### 问题：
1. 对kafka集群的压力double
2. 按照日期切割依旧存在合并时候Indexing问题

方案二的技术风险要小一些。
