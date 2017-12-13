+++
title = "CDN流量日志简单分析"
draft = false
date = "2017-03-04"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["cdn","python"] 
toc = true

+++

## 缘起
公司运营的很大一个成本就是CDN耗费。对比Impression和Click
## 数据分析
### 下载并处理CDN原始日志
1. 由于官方提供的合并下载日志工具在Mac下不能用，随机选取一个小时的日志：

```
wget https://cdnlog.cn-hangzhou.oss.aliyun-inc.com/streaming.lawulu.com/2017_03_04/streaming.lawulu.com_2017_03_04_1200_1300.gz?spm=5176.8232200.log.d10.83JI60&OSSAccessKeyId=xxxxx&
```
2. Python处理分析为CSV

```
pat = ( r'\[(.+)\]\s' #datetime
           '(\d+.\d+.\d+.\d+)\s' #IP address
           '-\s' #proxy ip
            '\d+\s' #responseTime
           '".+"\s' #referrer
           '"(GET\s\S+mp4)"\s'  # requested file
            '\d+\s' #status
           '\d+\s' #requestSIze
           '(\d+)\s' #responseSize
           '\D+\s' #HIT OR NOT
           '"(.+)"\s' #user agent
           '"\S+"' #contentType
        )
```
@see https://github.com/richardasaurus/nginx-access-log-parser/blob/master/main.py


3. 导入数据库

```
LOAD DATA INFILE 'cdnlogs.csv' 
INTO TABLE test.cdn_log
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' 
(date_time,ip,url,rsp_size,ua);
```
4. 数据校验

```
SELECT count(*)  FROM test.cdn_log 
'206619'

```

```
cat streaming.lawulu.com_2017_03_04_1200_1300 | wc -l

206618
```

#### 用Sql分析

```

select count(*), sum(rsp_size) from (

SELECT ip,ua,url,sum(rsp_size) as rsp_size FROM test.cdn_log group by ip,ua,url having count(*) =1 )a


'110643','598635187514'


select count(*), sum(rsp_size) from (

SELECT ip,ua,url,sum(rsp_size) as rsp_size FROM test.cdn_log group by ip,ua,url having count(*) >1 )a


'32670', '440010318426'

0.4236
select 440010318426/(598635187514+440010318426) from dual 

select sum(rsp_size) from  cdn_log

1038645505940

```

## 结论
由于IP,UA,URL可以唯一确定一次UV，我们公司有30%的用户占用了40%的流量，对同一个视频会不停的反复下载。
最后查明，是SDK的bug，正在下载视频时候，如果切换到后台，下载会直接停止。所以会反复下载视频，浪费流量。

## 校验
1. 计算ResponseSize之后和Console里面同一时间段的
2. 
```
CURL  -i -X HEAD 'https://streaming.lawulu.com/default/256b9db1-a1ce-4983-8906-4568fa7a9c75-144.mp4'
```

