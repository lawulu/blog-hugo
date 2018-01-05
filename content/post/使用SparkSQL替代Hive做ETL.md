+++
title = "使用SparkSQL替代Hive做ETL"
draft = true
date = "2017-01-18"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["spark","hive","etl"] 
toc = true

+++

## Spark的优势

1. 性能，比基于Tez的Hive要快
2. 编程友好：支持更多数据源格式的读和写；由于Spark可以做流处理和即席查询，某种程度上可以复用很多逻辑。
3. 可以接入Hive-serde表，与无缝与Hive切换

### 可以忽略错误的输入或者格式
1. `spark.sql.files.ignoreCorruptFiles = true`
2. 处理文本(json/csv)可以有三种模式：`PERMISSIVE DROPMALFORMED  FAILFAST `

PERMISSIVE: 将数据放到某一列里面

DROPMALFORMED：丢弃错误记录

FAILFAST: 抛出异常

### 复杂结构的支持
SELECT EXISTS(values, e -> e > 30) AS v FROM tbl_nested


### 函数
1. 标准函数，UDF，不过UDF对于Spark引擎来说，是一个黑盒，建议不要用
2. agg函数
3. 窗口函数，和hive类似
- rank 
- lag
- lead
4. 其他
- broadcast 利用了broadcast原语做join
```
##貌似不用的话，会优化成用broadcast?
val left = Seq((0, "aa"), (0, "bb")).toDF("id", "token").as[(Int, String)]
val right = Seq(("aa", 0.99), ("bb", 0.57)).toDF("token", "prob").as[(String, Double)]

left.join(broadcast(right), "token").explain(extended = true)
```

### 例子
注意，如果是二维数组来说，从外往内是先列后行。和Java类似`        int[][][] a=new int[2][2][4];  
`对应的是` [[[0, 0, 0, 0], [0, 0, 0, 0]], [[0, 0, 0, 0], [0, 0, 0, 0]]]  
`
```
   val df = Seq(1 -> 2).toDF("i", "j")
   val query = df.groupBy('i)
     .agg(max('j).as("aggOrdering"))
     .orderBy(sum('j))
     .as[(Int, Int)]
   query.collect contains (1, 2) // true
   
   val df = Seq((1, 1), (-1, 1)).toDF("key", "value")
   df.createOrReplaceTempView("src")
   scala> sql("SELECT IF(a > 0, a, 0) FROM (SELECT key a FROM src) temp").show
   +-------------------+
   |(IF((a > 0), a, 0))|
   +-------------------+
   |                  1|
   |                  0|
   +-------------------+
   
   ```
## 迁移中遇到的问题
### 兼容问题
- Hive UDF用到了SimpleDateFormat，Spark中会有问题
- Spark 2.1 不支持MapJoin Hint