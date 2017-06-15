+++
title = "使用HBase协处理器实现group by"
draft = false
date = "2014-11-20"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["hbase","protocolbuffer"] 
toc = true

+++

老印催着我们上新技术，一会研究Shark(Spark的Sql引擎)，一会研究Storm，现在终于上了HBase，其实不太适合我们的业务。现在一个group by的需求，就很麻烦。搜了一下，只能用协处理器来实现，比较坑的是不同版本协处理器的接口都在变化，自己摸索着来吧。

Google写了BigTable的论文，然后有了HBase，Jeff Dean演讲了一下，然后就有了协处理器。需求驱动的功能总是比邮件胡扯要来的早。
这个特性为用户提供类库和运行时环境，使得他们的代码能够在HBase region server和master上处理。


1 定义Proto，约定数据交换格式。我们需要key1,key2,result以及对应的getResult方法。本质上来说类似于JavaBean，这个比较简单。话说Hive之类在用Thrift，Hadoop有自己的Avro..真够乱的。


2 根据proto编译出Java代码

3 实现

```
@Override
    public void getResult(RpcController controller, GroupByRequest request, RpcCallback done) {
        Scan scan = new Scan();
        scan.addFamily(Bytes.toBytes(request.getFamily()));
        scan.addColumn(Bytes.toBytes(request.getFamily()), Bytes.toBytes(request.getColumn()));
        SumResponse response = null;
        InternalScanner scanner = null;
        try {
            scanner = env.getRegion().getScanner(scan);
            List results = new ArrayList();
            boolean hasMore = false;
                        long sum = 0L;
                do {
                        hasMore = scanner.next(results);
                        for (Cell cell : results) {
                            sum = sum + Bytes.toLong(CellUtil.cloneValue(cell));
                     }
                        results.clear();
                } while (hasMore);
 
                response = GroupByResponse.newBuilder().setSum(sum).build();
             
        } catch (IOException ioe) {
            ResponseConverter.setControllerException(controller, ioe);
        } finally {
            if (scanner != null) {
                try {
                    scanner.close();
                } catch (IOException ignored) {}
            }
        }
        done.run(response);
    }
```

4 打包放到Hbase集群,并修改hbase-site.xml
```
<property>
   <name>hbase.coprocessor.user.region.classes</name>
   <value>org.apache.hadoop.hbase.coprocessor.GroupByProcessor</value>
 </property>
```

