+++
title = "Idea在Ubuntu下面的使用"
draft = false
date = "2014-05-07"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["linux","idea"] 
toc = true

+++
## 快捷键 
### 快捷键设定
之前试图采用过Eclipse的快捷键,但是赶紧两种IDE的哲学相差还是比较大的,决定直接采用Idea默认的快捷键,然后再按习惯修改,例如Debug时候的F5F6之类.
一开始用Gnome方案,因为Unbutu下系统快捷键跟Idea冲突比较多,但是按键实在是别扭,例如Eclipse里面的Alt+Left/Right
在Default方案是Ctrl+Alt+Left/Right.这个和Unbuntu的切换Workspace冲突,Gnome方案是Shift+Alt+Left/Right,按起来好别扭..
最后打算还是使用Default方案,直接在Keyboard里面把这几个快捷键给禁用了.
还有一些输入法快捷键,我感觉其实只留Shift和ctrl+,就够了.Ctrl+Space,Ctrl+Alt+B(小语种软键盘)都要统统干掉.
更换的快捷键:
Alt+Left/Right和Ctrol+Alt+Left/Right互换. 一般打开Tab页比较多
Ctrl+W 关闭 比Ctrl+F4好按一些
Ctrl+D Duplication复制的机会比较少

### 一些常用快捷键
Ctrl+alt+v
单词跳 ctrl+left/right

ctrl+F12 outline
Alt+insert 生成getter,setter
修改了快捷键,Ctrl+shift+i
列模式: alt+shift+insert

### 几个trick
1. 不输入参数,移动光标到行尾.ctrl+shift+enter

2. 用惯了Eclipse快捷键的人可能会不习惯，sysout、foreach等快捷方式找不到了，main方法也无法自动补全了，其实这个在IntelliJ中有一个异常强大的模块Live Template来实现。
例如，在class中尝试psvm+tab，则会发现main方法产生了；输入iter+tab，则生成了foreach语句。 live template还有一个surround的用法，选中某个变量，键入ctl+alt+j两次，则会出现自动补全的菜单。

3. 建类的时候输入包名
service.NewClass


4. Mybatis:
Could not autowire. No beans of 'ApplicationMapper' type found. less... (Ctrl+F1) Checks autowiring problems in a bean class.
要装Mybatis插件或者将这个功能禁用掉
http://stackoverflow.com/questions/25379348/idea-inspects-batis-mapper-bean-wrong


## Idea的缺陷：
1 多线程Debug起来太渣，经常会断点不起作用

2 provided的不在classPath，例如，想在Idea下运行Hadoop，因为子项目比较多，N多Provided其他子项目的依赖

3 Test和Main不分
