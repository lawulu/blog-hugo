+++
title = "SparkML-卡方检验"
draft = false
date = "2017-04-18"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["ml","spark"] 
toc = true

+++

## Spark例子

```
//org.apache.spark.examples.ml.ChiSquareTestExample

val data = Seq(
      (0.0, Vectors.dense(0.5, 10.0)),
      (0.0, Vectors.dense(1.5, 20.0)),
      (1.0, Vectors.dense(1.5, 30.0)),
      (0.0, Vectors.dense(3.5, 30.0)),
      (0.0, Vectors.dense(3.5, 40.0)),
      (1.0, Vectors.dense(3.5, 40.0))
    )

val df = data.toDF("label", "features")
val chi = ChiSquareTest.test(df, "features", "label").head
println("pValues = " + chi.getAs[Vector](0))
println("degreesOfFreedom = " + chi.getSeq[Int](1).mkString("[", ",", "]"))
println("statistics = " + chi.getAs[Vector](2))
```
结果：

```
chi: org.apache.spark.sql.Row = [[0.6872892787909721,0.6822703303362126],WrappedArray(2, 3),[0.75,1.5]]

```
因为Vector是两列，所以结果也是两列，即，需要检测每一个维度和label相关性。


## 几个概念
### 卡方检验
卡方检验两个用途：
- 检测一批样本是否符合某种分布(Goodeness of Fit Test)
- 检测两个变量是否独立（本文着重讨论相关性检测）
### 卡方检验计算过程
- 假设变量是独立的，没有相关性
- 计算统计值：按照假设计算理论值，然后计算每一项的sqrt（实际值-理论值）/理论值，把所有的项都加起来
- 自由度，（distinctCount(label)-1）x（distinctCount(feature)-1），可以用(行-1) x(列-1)来理解
- 查表求p值，如果p值非常小（例如0.05），就拒绝假设

## 计算过程

### 举个例子
```
 val data = Seq(
      (0.0, Vectors.dense(0.5)),
      (0.0, Vectors.dense(0.5)),
      (1.0, Vectors.dense(1.5)),
      (0.0, Vectors.dense(3.5)),
      (2.0, Vectors.dense(3.5)),
      (1.0, Vectors.dense(1.5))
    )

```
统计每个维度取值和label的对应关系

label | 0.5 | 1.5 | 3.5 | 总数 
---|--- |--- |--- |---
0 | 2 | 0 | 1 | 3 
1 | 0 | 2 | 0 | 2 
2 | 0 | 0 | 1 | 1 
合计 | 2 | 2 | 2 | 6 

假设该维度和label是独立的，那么无论维度如何取值，取到的label为0的数目都是总数的1/2。

维度取0.5的一共为2个，则理论上，0.5对应的label数目为1. 做出对应的表格如下：

label | 0.5 | 1.5 | 3.5 | 总数 
---|--- |--- |--- |---
0 | 1 | 1 | 1 | 3 
1 | 2/3 | 2/3 | 2/3 | 2 
2 | 1/3 | 1/3 | 1/3 | 1 
合计 | 2 | 2 | 2 | 6 

计算统计值为8


### 验证统计值

用Python验证：

```

from scipy import stats
import fractions
obs = [2, 0, 1, 0, 2, 0, 0, 0, 1]
obs = [2, 0, 1, 0, 2, 0]
# 0, 2, 0, 0, 0, 1]
exp = [1, 1, 1,
       fractions.Fraction(2,3),fractions.Fraction(2,3), fractions.Fraction(2,3),
       fractions.Fraction(1,3),fractions.Fraction(1,3), fractions.Fraction(1,3)]

exp = [1, 1, 1,
       fractions.Fraction(2,3),fractions.Fraction(2,3), fractions.Fraction(2,3)]
       # fractions.Fraction(1,3),fractions.Fraction(1,3), fractions.Fraction(1,3)]

print(stats.chisquare(obs, f_exp = exp ,ddof=4))


obs= [0, 0, 1]
#exp= [fractions.Fraction(1,3),fractions.Fraction(1,3), fractions.Fraction(1,3)]
exp= [0.33333, 0.33333, 0.33333]

print(stats.chisquare(obs, f_exp = exp ))
```
结果正确
### 计算P值
//TODO

### 验证结论

假设数据是这样子的：
```
 val data = Seq(
      (0.0, Vectors.dense(0.5)),
      (0.0, Vectors.dense(0.5)),
      (1.0, Vectors.dense(1.5)),
      (1.0, Vectors.dense(1.5)),
      (2.0, Vectors.dense(3.5)),
      (2.0, Vectors.dense(3.5))
    )

```
肉眼可查，feature和label相关性很高。计算的P值为

```
pValues = [0.017351265236664526]

```
这样理解：假如feature和label独立不相关的，只有1.7%的可能取到像样本这样的数据。所以假设不成立。

参考：
http://www.statisticshowto.com/probability-and-statistics/chi-square/
https://segmentfault.com/a/1190000003719712