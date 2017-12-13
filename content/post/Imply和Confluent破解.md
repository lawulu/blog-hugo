+++
title = "Imply和Confluent破解"
draft = false
date = "2017-11-02"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["imply","druid","kafka","confluent"] 
toc = true

+++

其实这两个软件还是比较容易破解的，首先开发者并没有把精力放到反破解上面，相关功能做的很简单;第二没有服务器交互的反破解还是很难的，尤其是对于Java这种容易反编译的软件来说。话说过来，即时是有服务器交互，例如Idea，也有人搞出各种License Server出来。

## Imply Pivot
搜索
```
grep -ri 'Pivot evaluation license expired' .
```
发现有个文件`./node_modules/@implydata/im-auth/build/server/utils/license-manager/license-manager.js`

里面有这个逻辑：

```
        var e = path.join(this.varDir, ".pivot-first-run");
        return Q(fs.readFile(e, {encoding: "utf-8"})).then(function (e) {
            var r = new Date(e.trim());
            if (isNaN(r)) throw new Error("invalid date");
            return {created: false, firstRun: r}
        }).catch(function (r) {
            var t = new Date;
            return Q(fs.writeFile(e, t.toISOString())).then(function () {
                return {created: true, firstRun: t}
            })
        })
```
找到文件`find . -name ".imply-first-run"`
把里面日期改一下就行了

## Confluent
坑一：一开始搞反了..一直没报错 还以为没生效
```
jar uf /service/app/confluent-4.0.0/share/java/confluent-control-center/control-center-4.0.0.jar io/confluent/controlcenter/license/LicenseModule.class
```
坑二：zsh ll不显示隐藏文件，习惯把ll = ls-al 而不是ls -ah 


```
 return License.baseClaims("demo", bfa.creationTime().toMillis() + TimeUnit.DAYS.toMillis(30L), true);
```
所以其实修改该文件的createTime就行了..

