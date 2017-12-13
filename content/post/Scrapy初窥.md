+++
title = "Scrapy"
draft = false
date = "2015-10-15"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["scrapy","爬虫","python"] 
toc = true

+++

最近出于兴趣看了一下Scrapy.第一次仔细研究一个Python框架,先说结论,爬虫分为两种,一种是贪婪爬虫,最常用的就是搜索引擎,看见Url就爬.另外一种是专业爬虫,类似于网络小数定期更新,价格比对.Scrapy适合做第二种.Scrapy专心做爬取逻辑,其优势是框架比较成熟,各种插件比较全.如果真正做一个工业化的产品,Scrapy远远不够,还要攒很多东西进来.其实如果对`requests`和`beautifulsoup`熟悉,直接上这俩就挺好的.


## 简介
Scrapy其实是一个很轻量级或者简单的框架.几个概念：
1. `Request`,`Response`,顾名思义,对应的就是网络的请求和响应.
2. `Item`或者python dict,对应的是爬取到的结果.
3. `Pipeline`对Item的处理.
4. `Selector`对应的是对HTML的处理
5. `Middleware`相当于Filter,分两种,一种是针对Spider,一种针对网络连接
6. `Scheduler`一个深度优先的队列
7. `Feed` 官方提供的json或者json line的输出
8. `meta` 可以在多个Spider之间传输数据

其实把文档看一遍,下一个demo,很快就可以跑起来.
 
 看图其实很清楚：
![Scrapy Arch](/iimg/scrapy.png)

从StartRequest开始,不停的返回Item或者返回Request,如果返回Item,就发到Pipeline去处理,返回Request,经过Callback处理之后,直到所有的Request都变成Item.正所谓,世界是属于Request,归根结底还是属于Item的.
## 简单示例
```
import scrapy
class ZhihuSpider(scrapy.Spider):
    name = "zhihu"
    start_urls = [
        'https://www.zhihu.com/explore',
    ]

    def parse(self, response):
        # self.logger.info("------zzhon中国")

        for feed in response.css('.recommend-feed'):
            link = response.urljoin(feed.css('a::attr(href)').extract_first());
            yield scrapy.Request(link,callback=self.parse_content)
           

##!!!  print response.css('#zh-question-detail > div').extract()[0].encode('utf-8')

    def parse_content(self,response):
        yield {
            'title':response.css('#zh-question-title > h2 > a::text').extract_first(),
            'content':response.css('#zh-question-detail > div::text').extract(),
            'link':response.url
        }

```
当然这个示例跑起来之前,需要改一些配置文件和设置User-agent…
## 吐槽
Java出身的程序员的吐槽… 不用在意😜
### 日志只打印Unicode问题
Debug和监控起来非常不方便,我看官方把这个issue当做wont fix了,强烈推荐直接上Python3.对非英文世界来说,Python2下scrapy shell简直没了存在的必要…

```
2015-10-14 23:25:54 [scrapy] DEBUG: Scraped from <200 https://www.zhihu.com/question/29218955/answer/41797438> 
{'content': [u'\u666e\u6d31\u5728\u8fc7\u53bb\u5e94\u8be5\u662f\u4e0d\u4e0a\u53f0\u9762\u7684\u5427\uff1f\u4e5f\u6ca1\u95ee\u662f\u4e0d\u662f\u5c31\u95ee\u4e3a\u4ec0\u4e48\u4e86\uff0c\u5982\u679c\u95ee\u9898\u6709\u9519\u8bef\uff0c\u8bf7\u6307\u6b63'], 'link': 'https://www.zhihu.com/question/19258955/answer/42797438', 'title': u'\u666e\u6d31\u7531\u539f\u6765\u4e0d\u4e0a\u53f0\u9762\u7684\u8fb9\u9500\u8336\u5230\u5982\u4eca\u53d7\u5230\u5e7f\u5927\u8336\u53cb\u559c\u7231\u7684\u8336\u7c7b\uff0c\u9664\u4e86\u7092\u4f5c\u5916\uff0c\u666e\u6d31\u7684\u5de5\u827a\u6216\u8005\u8d28\u91cf\u6709\u4e86\u5f88\u5927\u8fdb\u6b65\u5417\uff1f'} 

```
### Scheduler太弱
自带的scheduler只能深度优先和广度优先,官方推荐是自家的scrapyd,稍微了解了下,应该是启动时候把任务分给多个进程,进程之间好像没法交互.
### Spider/Request之间很难协同
很多时候我们需要某几个Request应该在一个组里面,同一个组的Request可以共用Cookie,共用代理.这类的需求可以通过meta机制来实现…由于Scrapy的Request是无脑yield出来的,实现出来肯定很丑陋.
### Callback hell
通过Callback来绑定处理方法,入口和可测试性很差,怪不得要加入Contract机制
### 容错
容错机制很少,需要把Task持久化,官方提供的持久化到disk的方案感觉很初级很粗糙.

## Splash
Pyspider是用的`PhantomJS`,Scrapy提供了一个基于Docker和Lua的Splash.
这货的问题是Splash没有保持渲染状态的方法,对于复杂的JS交互,一般的做法是一个lua脚本执行很多次,最后返回所有的结果.这样爬取的颗粒比较大,如果中间出现异常,不好处理,而且调用`splash::wait()`总感觉不靠谱,没有提供Dom加载之后的回调吗？貌似也无法解决共用tab的问题。
### 原理
Splash可以执行Lua脚本,官方封装了Splash类,可以和JS交互.这样,就可可以找到Dom中的Button,直接调用click事件,然后返回某些DOM中的Html或者Json.
### Lua
Lua 很值得一学,Nginx,游戏引擎,Redis都在用……
快速入门：http://tylerneylon.com/a/learn-lua/

Lua稍微复杂并且可以玩出花的内置机制就是Metatable了,可以重载对字典(对象)各种操作符.

### 



