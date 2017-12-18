+++
title = "Databricks:Spark knowledge base"
draft = false
date = "2017-02-18"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["spark"] 
toc = true

+++

内容来自：
https://databricks.gitbooks.io/databricks-spark-knowledge-base/

## 最佳实践

### 避免使用GroupByKey
在使用`groupByKey`时候，Spark引擎并不清楚分组之后要做什么，所以无法在数据交换之前在Partition内部提前运算，而`reduceByKey`不同，可以在分区内部提前调用`reduceByKey`的`func`

### 不要将大数据量Copy到Driver中
`myVeryLargeRDD.collect()`这句话会试图将所有数据都Copy到Driver中，导致内存溢出和程序崩溃。为查看数据，可以使用`take`或者`takeSample`，或者将数据写到文件或者数据库。

### 优雅的处理错误数据
使用`map`、`filter`、`flatMap`来过滤错误数据

## 常规故障处理

### Task not serializable
这是因为堆对象只有在被序列化时候，才能在节点之间传递。解决办法:
- 实现序列化
- Declare the instance only within the lambda function passed in map.
- 将实例声明为static,这样在每个节点都会重新初始化
- 使用`rdd.forEachPartition`

```
rdd.forEachPartition(iter -> {
     NotSerializable notSerializable = new NotSerializable();
   
     // ...Now process iter
   });
```
### 缺失依赖
解决办法：在 Maven 打包的时候创建 shaded 或 uber 任务，并将Spark的lib设为provided

https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/troubleshooting/missing_dependencies_in_jar_files.html

### Mac：Error running start-all.sh Connection refused
开启 “远程登录” 功能。进入 系统偏好设置 ---> 共享 勾选打开 远程登录。

### Spark组件的网络连接问题

## 性能和优化
### RDD有多少个分区？
#### 使用 UI 查看在分区上执行的任务数
#### 使用 UI 查看Storage
#### 编程查看 RDD 分区
```
scala> val someRDD = sc.parallelize(1 to 100, 30)
someRDD: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:12

scala> someRDD.partitions.size
res0: Int = 30
```
### 数据本地性
任务应该在离数据尽可能近的地方执行(例如最少的数据传输)。
#### 检查本地性
注意UI中的`Locality Level`
#### 调整本地性
You can adjust how long Spark will wait before it times out on each of the phases of data locality (data local --> process local --> node local --> rack local --> Any)

### Spark Streaming
#### ERROR OneForOneStrategy
If you enable checkpointing in Spark Streaming, then objects used in a function called in forEachRDD should be Serializable. Otherwise, there will be an "ERROR OneForOneStrategy: ... java.io.NotSerializableException:



