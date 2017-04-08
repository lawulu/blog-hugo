+++
title = "Tracking以及FuseClick的一些胡言乱语"
draft = false
date = "2016-12-01"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["fuseclick","ad"] 
toc = true

+++

_**广告新人的一些感慨**_

## FuseClick要解决的痛点
在移动互联网烧钱大潮中，（传统来说）有流量变现能力只有广告了。当每一次点击都与钱相关时候，机会和随机会而来的混乱就出现了。对于移动强CPA广告来说，最大的痛点就是转化的核算和归因。各种第三方平台（友盟，TalkingData，热云……），各种不同的规范……，FuseClick就是对标Affiliate，成立的一家Tracking平台，帮助AdNetwork来统一追踪。更有野心的是，他家还要帮忙做优化。类似想统一混乱的公司还有Appcoach,mediation。。回头写一下。
## Tracking流程
### 传统Tracking
对AdNetwork来说，由于无法知道是否安装完成，所以要通过第三方的平台来完成计费。所以，Tracking一般分为Click上报和Install的Postback。即当用户要下载一个App时候，AdNetwork马上告诉广告主（或者代表广告主的检测平台），快看，我这有个用户点击了，他要安装了！！然后就开始默默的等待..广告主收到一个安装，就会根据安装时候的信息反过来去找是谁上报的，一般会根据设备的信息（混乱之源之一)来匹配。所以上报时候一定要把设备信息给带上。上报分302让客户端上报和S2S对接两种，按下不表。
### 宏替换
由于AdNetwork要对接的第三方平台太多，而各家Tracking时候需要的参数是不一样的。（为什么没有一个广告协会之类规范一下？！），所以就会有宏替换这么一个奇特而又奇妙的东西。举一个例子：

假如，大家需要一个参数DeviceId，A家需要的上报是http://a.com/tracking?did=XX,而B家需要的上报是http://b.com/tracking?device=XX，这时候客户需要让AdNetwork知道应该怎么上报，AdNetwork提供一批可供替换的变量，让客户自己填。例如DeviceId的变量是shebeihao，那么上面两家分别填写：http://a.com/tracking?did={shebeihao}，和http://a.com/tracking?device={shebeihao}，等上报时候，将这些信息都替换正确。

### 渠道包
因为（国内）Android并没有一个强势如AppStore的（混乱之源之一）应用商店，所以就会有渠道包这种东西。

## Fuseclick的做法
1. 为每一个AdNetwork建立一个Tracking Link，在Dashboard中配置对应的宏和Postback Url。
2. 提供报表做到小时级别的统一，方便了不同时区的用户。

## 是不是可以这样？
（以下大部分都是猜测）
不提供设备信息，只关心一个ClickId，然后把ClickId发给广告主，广告主使用渠道包来判断是否是来自于本渠道的激活，有的话就直接回调AdNetwork，当然这个要建立在三方互相信任的阶段。

FC公众号上面的这篇对广告行业的介绍写的挺好的。http://www.wtoutiao.com/p/N56UUz.html
