+++
title = "都是DNS惹的祸"
draft = false
date = "2015-08-01"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["linux","dns","ssh","mysql"] 
toc = true

+++


最近因为工位调整,内部用的测试机换了个地方,结果发现上面部署的应用变得都反应奇慢,每次SSH上去也要卡顿很久.

## 是不是网络问题?
### Ping比较正常
```
PING 192.168.214.227 (192.168.214.227): 56 data bytes
64 bytes from 192.168.214.227: icmp_seq=0 ttl=64 time=0.073 ms
64 bytes from 192.168.214.227: icmp_seq=1 ttl=64 time=0.100 ms
64 bytes from 192.168.214.227: icmp_seq=2 ttl=64 time=0.092 ms
64 bytes from 192.168.214.227: icmp_seq=3 ttl=64 time=0.074 ms
64 bytes from 192.168.214.227: icmp_seq=4 ttl=64 time=0.057 ms
```
没有丢包,traceroute貌似也没有问题
### 测速
生成一个大文件:
```
#*-*coding:utf-8 *-*
'''''
Created on 2012-6-14

'''
def gen_file(path,size):
    #首先以路径path新建一个文件，并设置模式为写
    file = open(path,'w')
    #根据文件大小，偏移文件读取位置
    file.seek(1024*1024*1024*size)#姑且以GB为单位吧
    #然后在当前位置写入任何内容，必须要写入，不然文件不会那么大哦
    file.write('\x00')
    file.close()

gen_file('./test.dat',1)
```
scp上去,速度在兆级别..

## SSH
用`ssh -v` 找到Hang的地方,搜一下相关信息,很快就能找到需要改哪里.
```
vim /etc/ssh/sshd_config
UseDNS=no
```
重启`sudo service ssh restart `解决问题

## Mysql
mysql 客户端也有`--verbose`的选项,但是设置了貌似没什么结果.搜了一下,很多说也是DNS的原因
```
show variables
| skip_name_resolve                                 | OFF                                                                                                                     |

```

修改 `vim /etc/mysql/my.cnf`
增加一行 `skip-name-resolve`