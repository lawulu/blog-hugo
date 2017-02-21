+++
title = "一种统一的API签名校验实现"
draft = false
date = "2015-08-10"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["java", "spring","安全"] 

+++


### 签名方法:

#### Header:
```

@RequestHeader(value = "x-header-timestamp") Long timestamp,
@RequestHeader(value = "x-header-appid") String appid,
@RequestHeader(value = "x-header-sign") String sign
```

#### 源串 orignalString:
```

HttpMethod=URI&Param1=Value1&Param2=Value2RequestBodySeckeyTimestamp
```

注1:Method大写
注2:为方便期间Param不排序,如果出问题再说
注3:ResustBody可能为空
注4:SecKey跟AppId是一一对应.
#### 签名 sign:
```
sign=Md5(orignalString).toUpperCase
```

暂时只支持MD5

### 实现:
原则上,应该和业务逻辑代码分离,不影响业务逻辑代码的开发
#### 1 根据注解Mapping需要检查签名的方法
例如:Spring @ControllerAdvice(annotations = RestController.class)
如需要,可自定义自己的注解
#### 2 获取参数
```

@ModelAttribute
	public void checkSign(HttpServletRequest request, HttpServletResponse response, @RequestBody(required=false) String requestbody,
			@RequestHeader(value = "x-header-timestamp") Long timestamp,
			@RequestHeader(value = "x-header-appid") String appid,
			@RequestHeader(value = "x-header-sign") String sign) 
```

#### 3 计算签名
可能需要在GLobalExceptionHandler中增加
```

    @ExceptionHandler(ServletRequestBindingException.class) 
    	@ResponseStatus(HttpStatus.BAD_REQUEST)	
    	@ResponseBody 
    	ErrorMessage handleServletRequestBindingException(ServletRequestBindingException ex) {	
    		
    		LOGGER.debug("handleServletRequestBindingException", ex);
    		return new ErrorMessage(ErrorCode.INVALID_HEADER.getCode(), ex.getMessage());//TODO
    	}
```


### 增加Filter 

注意：因为在ControllerAdvice消费了Request的InputStream，所以需要在前面的Filter中Copy一份Request出来
代码如下：
```
/**
 * Wrapper used to be able to consume multiple times the InputStream provided by
 * HttpServletRequest
 *
 * @author lawulu
 * @see https://github.com/ClouDesire/spring-utils/
 * 
 */

public class CustomHttpServletRequestWrapper extends HttpServletRequestWrapper {
    private final byte[] body;

    public CustomHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        if (request.getContentType() != null
                && request.getContentType().contains(MediaType.APPLICATION_JSON.toString())) {
            InputStream inputStream = request.getInputStream();
            if (inputStream != null) {
                body = IOUtils.toByteArray(inputStream);
                return;
            }
        }
        body = null;
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        if (body == null)
            return super.getInputStream();

        final ByteArrayInputStream stream = new ByteArrayInputStream(body);
        ServletInputStream inputStream = new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return stream.read();
            }
        };
        return inputStream;
    }

    @Override
    public BufferedReader getReader() throws IOException {
        if (body == null)
            return super.getReader();

        return new BufferedReader(new InputStreamReader(new ByteArrayInputStream(body), StandardCharsets.UTF_8));
    }

}
```
