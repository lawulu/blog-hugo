+++
title = "Freemarker下为静态资源增加Version"
draft = false
date = "2014-06-04"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["freemarker","idea","js"] 
toc = true

+++

## 源起
因为客户端浏览器缓存问题，每次发布时候之后，经常遇到新修改的JS/CSS不生效，需要手动清理老版本的缓存。其实业界已经有成熟的解决方案，无非加上Version或者禁止客户端缓存，未免遗忘，在这里记录一下。
## 具体方案
### 自定义一个FreemarkerVariables
```
public class FreemarkerVariables extends HashMap{
    public FreemarkerVariables(){
        super();
        this.put("resourceVersion",""+System.currentTimeMillis());
    }
}
```
### Freemarker配置
使用了FreemarkerVariables
```
<bean id="freemarkerVariables" class="xxx.FreemarkerVariables"/>
	<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
		<property name="templateLoaderPath" value="/WEB-INF/views/"/>
		<property name="freemarkerSettings">
			<props>
				<prop key="defaultEncoding">UTF-8</prop>
			</props>
		</property>
		<property name="freemarkerVariables" ref="freemarkerVariables"/>
	</bean>
```

### 批量替换
做之前搜索了下，N多人都是写一个小工具。感谢现代IDE，直接替换就可以了。把所有的 `js"></script>` 替换成`js?v=${resourceVersion}"></script>`即可，注意不要使用正则。

## 验证
开发者的Chrome一般都是`Disable Cache`的，这时候Header里面会有`no-cache`，否则，应该可以看到`Status Code:304 Not Modified (from memory cache)`之类的东西。
