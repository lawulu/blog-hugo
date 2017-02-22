+++
title = "Logback的一些奇技淫巧"
draft = false
date = "2015-08-23"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["logback"] 
toc = true

+++

Logback是一款非常优秀的日志框架.但是每个开发面对的需求也是多种多样的.
如果你有下面的需求,那你看了这篇文章就可以了：

- 每次调用链都生成一个唯一的traceId
- 修改配置文件即时生效,例如,为了定位一个问题,临时将log级别修改为debug
- 不同环境使用不同的配置
- Logback的默认Rolling策略是：有新的Log产生,如果需要rolling,则rename原来的文件.这样会存在的一个问题就是,如果没有新的log产生,就不会重命名原来的文件.如果遇上需要同步日志(例如Rsync),就会出现问题.

### 日志的TraceId

- 在Filter里面增加一个,ThreadLocal based 变量：


```

public class CustomFilter extends OncePerRequestFilter {
	
	private static final Random random = new Random(System.currentTimeMillis()); //TODO

	@Override
	protected void doFilterInternal(HttpServletRequest request,
			HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		
	
		request.setAttribute(ApiConstants.RQID, rnd);
		
		response.setHeader(ApiConstants.HEADER_RQID, rnd);
		MDC.put(ApiConstants.RQID, rnd);
		try{
			filterChain.doFilter(request, response);
		}finally{
			MDC.remove(ApiConstants.RQID);
		}

	}

}
 ```

 - 配置文件

```
 <encoder>
	        <pattern>%d{yyyy-MM-dd HH:mm:ss} %-4relative [%thread] %-5level %logger{35} [%X{x-jjk-rqid:-notFound}]- %msg%n</pattern>
	    </encoder>
```

### 自动更新配置
显然会影响效率

```
<configuration debug="true" scan="true" scanPeriod="1 minutes">

```
### 各个环境引入不同的Profile

```
	<property resource="config/config-${envTarget}.properties"/>

```
### 基于条件写不同的文件：SiftingAppender


```
 <appender name="stats" class="ch.qos.logback.classic.sift.SiftingAppender">
        <!-- in the absence of the class attribute, it is assumed that the
             desired discriminator type is
             ch.qos.logback.classic.sift.MDCBasedDiscriminator -->
        <discriminator>
            <key>date</key>
            <defaultValue>unknown</defaultValue>
        </discriminator>
        <sift>
            <appender name="FILE-${date}" class="ch.qos.logback.core.FileAppender">
                <file>/mnt/media-${date}.log</file>
                <!--<append>false</append>-->
                <encoder>
                    <pattern>%msg%n</pattern>
                    <charset>UTF-8</charset>
                </encoder>
            </appender>
        </sift>
    </appender>
```
代码中需要写MDC变量：


```
        MDC.put("date", dateString);

```





