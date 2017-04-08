+++
title = "Phantomjs和Casperjs初窥"
draft = false
date = "2016-10-23"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["爬虫","phantomjs","casperjs"] 
toc = true
+++

在这个浏览器最大的时代，JS越来越重的时代，做爬虫其实最好的办法是找一个Headless浏览器，必须要支持执行JS，这样做爬虫就不用去分析HTTP和JS了。我大Java有HtmlUnit，最近尝试的是传说中的PhantomJs。话说为什么没有基于Chrome dev tools的爬虫框架呢，或许可以尝试下油猴子。
## 简介
PhantomJs提供了最原始的API，Casperjs是对PhantomJs的包装，最方便的是不用不停的写if/else（成功了执行下一步），而是用then/wait等方法增加代码的可读性和可写性。不过，Casperjs官方文档直接给出，遇到问题，应该使用PhantomJs的方式先去调用，有问题也要去PhantomJs那里去提…所以，看文档还是直接上PhantomJs吧。
## 示例
casperjs提供了`echo`、`waitfor`、`click`等原语，所以代码大概是这样：
```
casper.thenClick('li.pg_next');

casper.then(function () {
    this.waitForSelector('li.pg_next',function () {
         this.echo(this.getHTML('.post-list'));
    });
})
```
比较苦逼的是，只有通过文件和console来跟外界交互…casperjs提供了一个server示例，直接标注说有内存溢出风险，phantomjs直接把webserver标注为不可在生产上用。

## Debug
```
casperjs --ignore-ssl-errors=yes --ssl-protocol=any  sample.js  --remote-debugger-port=9000 
```
这时候，可以在Safari里面打开，在Scripts里面设置断点，就可以Debug了。看似简单，但是很多坑在里面。

首先，要在console里面执行__run()，但是如果`sample.js`如果太复杂的话，会直接把PhantomJs搞Crash。所以可以在代码里面加上`Debugger`，这样直接能跳过Crash的地方。
另外，PhantomJs是基于Webkit的，Chrome虽然有Webkit血统，但是只使用了起渲染HTML和CSS这一层，自己搞了一套V8去执行JS，所以，最新版本的Chrome是无法Debug PhantomJs的，最好使用血缘更近的Safari。另外再吐槽一下，PhantomJs的Bug真多，动不动就Crash。

而且Console实在是不方便，所以，我觉得，最好的Debug还是用capture截图和多用log输出，例如监听资源。


## 监听资源
```
 var utils = require('utils');

casper.options.onResourceReceived = function(casperObject, res) {
     var contentType = res.contentType;
     if(contentType.indexOf("json")!=-1){
        console.log(utils.dump(res));

     }
}
```
这里可以打印出所有header为`application/json`的请求，然而没什么卵用，因为一般来说，body都被Gzip了，这里比较适合的是打印出url。

另外一个坑是，PhantomJs退出时候，可能有很多Ajax请求还没有完成，所以这里要手动Wait……,不然很多请求抓不到，再所以，PhantomJs抓起来就比较慢，适合一些临时的小任务。

