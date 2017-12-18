+++
title = "基于Dubbo的Batch项目"
draft = false
date = "2017-01-17"
Categories = ["AkiRoss"] 
Description = "" 
Tags = ["dubbo","batch"] 
toc = true

+++

这个Demo完成于去年，今天把一些敏感信息去掉，放到github上

# dubbo batch demo
包含了一个Batch项目和一个Portal。
依赖当当的DubboX

## Batch
Batch使用了Spring batch，Quartz，Dubbo
通过自定义的MetaData来管理所有数据
## Portal
一个实例项目，改造自当当的Elastic-job的Admin Portal，用到lombok和Freemarker

# 项目地址
https://github.com/lawulu/dubbo-batch-demo



