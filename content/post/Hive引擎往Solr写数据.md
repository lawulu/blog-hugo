+++
title = "Hive引擎往Solr写数据"
draft = false
date = "2014-03-15"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["hive","solr"] 
toc = true

+++
公司有一批结果是通过Hive跑SQL跑出来的，但是需要最终写到Solr里面，方面别的服务使用。
灵感来自
https://chimpler.wordpress.com/2013/03/20/playing-with-apache-hive-and-solr/

Fork之后，做了一些修改：主要是针对Hive0.12和Solr新版本做的一些升级。https://github.com/lawulu/hive-solr/