+++
title = "Databricks:The Data Engineers Guide to Apache Spark"
draft = false
date = "2017-02-18"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["spark"] 
toc = true

+++

1 DataFrames/SQL并没有提供关于partition的API
2 narrow/wide transformation. When we perform a shuffle, Spark will write the results to disk
3 action分三种：
• actions to view data in the console;
• actions to collect data to native objects in the respective language; 
• and actions to write to output data sources.

4 spark.conf.set("spark.sql.shuffle.partitions", "5")

5 Scala has some particular semantics around the use of == and ===. In Spark, if you wish to filter by equality you should use === (equal) or =!= (not equal). You can also use not function and the equalTo method.
