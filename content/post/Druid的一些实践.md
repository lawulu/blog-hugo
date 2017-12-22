+++
title = "Druid的一些实践"
draft = false
date = "2017-09-08"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["druid","kafka"] 
toc = true

+++



## 安装

### 文档
参考 https://docs.imply.io/on-premise/cluster

不同之处：
- 使用了多个ZK
- 修改`imply/conf/druid/middleManager/runtime.properties`的`druid.worker.capacity`


### 监控
官方提供两个扩展graphite-emitter和statsd-emitter，跟我们现在用的Zabbix+grafana不能无缝对接。需要调研

### 日志滚动
因为启动脚本是共用的，而不同的模块需要指定不同的路径，修改起来比较麻烦。所以直接使用logrotate
```
cat /etc/logrotate.d/druid

/root/imply/var/sv/historical.log
/root/imply/var/sv/middleManager.log
/root/imply/var/sv/broker.log
/root/imply/var/sv/coordinator.log
/root/imply/var/sv/overlord.log
{
    daily
    rotate 7
    copytruncate
    dateext
    compress
    delaycompress
    missingok
    notifempty
}


```

## 使用


### TopN取代GroupBy

可以参考Imply生成的Json，基本都是用的TopN，默认是`max(1000, threshold)`，一般来说，Top1000的话前900肯定是准确的.
- 为什么TopN会快？
https://groups.google.com/forum/#!topic/druid-development/zUKGEkSAAMo

### 使用KIS而非Tranquility
- Tranquility 时间窗口太死，不适合我们的场景
- 架构/代码复杂
- 测试有丢数据


### 利用hyperHyperLog

将UserId和其他Id拼在一起，完成group by userId, bizId
```
{
  "type": "hyperUnique",
  "name": "uniUser",
  "fieldName": "userId"
},
{
  "type": "hyperUnique",
  "name": "uniAppUser",
  "fieldName": "appUserId"
},
{
   "type": "hyperUnique",
  "name": "uniCampaignUser",
  "fieldName": "campaignUserId"
},


```

## 维护

### 废弃掉Segement
`update ad_druid2.druid_segments  set used = 0 
where id < 'ad_request_2017-09-27T08:00:00.000+08:00_2017-09-28T08:00:00.000+08:00_2017-09-27T00:00:06.479Z'
`

## 迁移

公司重新购买了Hadoop集群，硬盘不足的问题解决了。想把HDFS作为Deep Storage用起来。先把思路记录下来，等折腾完了再补充。


### Mysql到Druid

#### 确定时间点
- 按照UTC时间划分
- 将Druid之前试跑数据废弃
- 将Mysql数据按天生成Json文件，批量导入
- 数据校验


#### 影响
- 查询需要并两个dataSource
- Mysql数据没有Uniq相关的Metric


### NFS based到HDFS based


#### 目标
- 尽可能快的迁移
- 数据完整性和正确


#### 方案一：基于Offset

##### 步骤：

①关掉旧Druid的KIS服务，并记录下消费的offset

②修改Segments的名字

③迁移Segements到HDFS

④重新Index

⑤在新Druid上面启动KIS，Offset设定

##### 问题：

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

#### 方案二：基于时间划分


##### 步骤：

①在新集群上面启动KIS服务，两个集群同时跑一段时间

②根据日期切割

##### 问题：
1. 对kafka集群的压力double
2. 按照日期切割依旧存在合并时候Indexing问题

方案二的技术风险要小一些。
