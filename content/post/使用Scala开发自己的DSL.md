+++
title = "使用Scala开发自己的DSL"
draft = false
date = "2017-05-30"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["scala","dsl"] 
toc = true

+++

## 缘起
yinwang写了一篇关于DSL的文章：http://www.yinwang.org/blog-cn/2017/05/25/dsl
，认为大部分情况是不需要DSL的：`绝大部分 DSL 的存在，都是因为设计它的人没有理解问题的本质，没有意识到这问题其实不需要通过设计新的语言来解决。`并且拿出Scala来当做反例，从某种程度上，我很支持yinwang的观点，因为最近看Scala的一些项目，经常会有一些奇怪的操作符，极大的增加了阅读的难度。

yinwang认为因为大家已经熟悉了本语言，所以对库函数的认知会更容易一些。比如，大部分程序员已经习惯了C语言设定的DSL（比如for if while)，所以跨语言学习会减少了很多的难度。

虽然yinwang反对，我们还是要好奇一下，怎么用Scala开发自己的DSL


## 实现DSL


### Scala为什么适合DSL
- 对于一个参数的方法可以省略点/括号
- 可以使用特殊符号做方法
- 函数式编程

### 定义

参考：https://dzone.com/articles/rolling-your-own-dsl-in-scala