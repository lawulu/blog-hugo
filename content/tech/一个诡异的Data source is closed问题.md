+++
title = "一个诡异的Data source is closed问题"
draft = false
date = "2015-10-19"
Categories = ["Tech"] 
Description = "" 
Tags = ["java", "Dbcp"] 

+++

### 问题由起
因为某些问题（这个问题稍后再提），将线上的DBCP版本从1.4升级到2.1，大概扫了一下官方文档，没有迁移指南，只是几个属性名称变了，感觉问题不大，直接升级。升级之后，UT一下，通过。但是上线之后一直报下面错误：
>Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is java.sql.SQLException: Data source is closed
	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:23)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:107)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:98)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:386)
	... 43 more
### 问题背景
- 项目因为需要动态切换数据源，使用的是++AbstractRoutingDataSource++
- 需要动态切换的数据源比较多，项目初始化时候通过++BeanDefinitionBuilder++来根据数据库配置动态创建数据源，而报错的连接正是动态生成的的数据源。
### 问题定位
#### 为什么UT测试成功
原来是UT的时候，因为配置文件，测试其实走的是++AbstractRoutingDataSource++的默认的数据源，并没有走动态生成的数据源
#### 怀疑是参数问题或者其他原因
只设置JDBC Url，依旧报错；换成dbcp，不报错。
#### AbstractRoutingDataSource的问题？
使用XML配置，没有问题

**至此，基本确认是++DBCP2++和++BeanDefinitionBuilder++**的问题

### 查看源码和Debug
#### BasicDataSource DBCP1和2有什么区别？
DBCP2实现了更多的接口：
```
public class BasicDataSource implements DataSource, BasicDataSourceMXBean, MBeanRegistration, AutoCloseable 

```
#### 查看DBCP2创建链接和销毁的地方
是没有Create Connection还是什么时候被Close了？
被Close了

```

		this.beanDefinitionMap.put(beanName, beanDefinition);

		if (oldBeanDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
		
		
```
```
protected void resetBeanDefinition(String beanName) {
		// Remove the merged bean definition for the given bean, if already created.
		clearMergedBeanDefinition(beanName);

		// Remove corresponding bean from singleton cache, if any. Shouldn't usually
		// be necessary, rather just meant for overriding a context's default beans
		// (e.g. the default StaticMessageSource in a StaticApplicationContext).
		destroySingleton(beanName);

		// Reset all bean definitions that have the given bean as parent (recursively).
		for (String bdName : this.beanDefinitionNames) {
			if (!beanName.equals(bdName)) {
				BeanDefinition bd = this.beanDefinitionMap.get(bdName);
				if (beanName.equals(bd.getParentName())) {
					resetBeanDefinition(bdName);
				}
			}
		}
	}

```
原来Spring创建Bean的时候会检查是否有oldBeanDefinition,如果有则销毁之。而DBCP2实现了AutoCloseable接口，在这里直接被Close了。
#### 为什么有oldBeanDefinition？
为了支持另一种类型的数据源，引入了一个Bug，会在Spring BeanFactory里面引入重复的定义，因为在AbstractRoutingDataSource中是正确的，所以一直没有发现这个问题。第二次会销毁上次的Bean。
其实在DBCP1时候，也会销毁，但是因为在动态定义的时候没有定义destroy-method="close"，其实会引起内存的泄露？


结论：很多看似运行完好的软件其实都危机四伏..