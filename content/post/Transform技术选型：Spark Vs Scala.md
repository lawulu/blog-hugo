+++
title = "Transform技术选型：Spark Structured Streaming Vs Pure Scala"
draft = false
date = "2017-07-18"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["spark","scala","streaming"] 
toc = true

+++

## 需求

公司要上实时，需要一个应用对原始日志进行转换，简单来说：

1. 从Kafka读取数据，然后写回到Kafka
2. 对日志和数据进行合并
3. 完成归因、去重、真实点击等其他需求
4. 完成计费

有这么几个技术需求：

1. Json的处理，FileBeat产生的In和Druid需要的Out都是Json形式。
2. Kafka读写
3. 外部数据源，Mysql，Redis和MongoDB的读取
4. 逻辑处理

## Spark or Scala
所谓的Spark，即Spark Structured Streaming，所谓的Scala是Scala based on Play Framework
优先考虑的是Spark（Spark Structured Streaming），因为Spark可以做流处理和即席查询，某种程度上可以复用很多逻辑，并且可以接入Hive-serde表，与无缝与Hive切换。

### 技术需求
- Json
 - Spark：问题不大，但需要调研，Scala貌似没什么好用的Json库
 - Scala：Play自带Json库
- Kafka
 - Spark：官方提供有类库，但是屏蔽的细节太多了。现在项目内没人清楚DStream/Spark.readStream是怎么个机制
 - Scala：Kafka官方的Client是Java的，Akka Stream有对应的封装。如果不能用，解决起来也比较方便
- 外部DB
 - Spark：问题不大，需要调研如果精细的控制Partiton与连接之间的关系，还有Lookup问题 
 - Scala：Scala有各种Reactive的驱动

### 非技术需求
- 监控&日志
 - 用Spark的话，现有公司/阿里云的各种基础设施很难用的上
- 补数，关键是对Kafka的Offset做精细的控制 
 - 搜了一下，Spark这块放到Checkpoint了
- TroubleShooting 
 - Spark：？？？公司现在并没有用Spark跑大规模的应用
 - Scala：Akka这块

## 结论


所谓Spark的优势：

1. 以RDD/DataFrame/SQL形式提供的对数据处理的DSL，虽然比Hive效率更高，可以和Scala程序同时工作
2. 横向扩展以及容错
3. Broadcast/Accumulator等原语
4. 可以有效利用HDFS对应的计算资源，并且做为一个Unified的平台，理论上可以重用很多逻辑


Spark的问题：

1. Structured Streaming API 不稳定，我自己测试发现有Job莫名奇妙不运行的情况。
2. 任务调度和执行都是黑盒



Spark并不是银弹，更倾向于用纯Java应用来做。Spark的优势在一个纯Transform的应用里面发挥不出来（上面三点：1显然不如直接写程序，对于2瓶颈在Kafka，3引入Redis可以解决类似问题。
）。后续可以研究一下Kafka Streams。