+++
title = "利用X-Accel-Redirect做静态资源的身份验证"
draft = false
date = "2015-10-20"
Categories = ["爆栈&基线器"] 
Description = "" 
Tags = ["nginx","安全"] 
toc = true

+++


## 背景
### 需求
- 用户能查到自己历史体检报告
- 用户能尽快查到刚出的体检报告
- 数据安全性
### 初步方案
Api server根据索引表生成token, 返回给客户端,客户端去数据中心提供文件下载的服务

- 生成Token方案:类似于现在登陆token或者签名机制. Api Server根据时间和deviceId,barcode生成一个,客户端请求时候带上DeviceId

- 认证方案: Nginx/Apache X-Accel-Redirect, Java/PHP校验token,然后转发给Nginx的标志为Internal Location.

- Nginx/Apache需要限制同一deviceId和ip的访问次数
## 实现
### Java实现
为了减少依赖，使用Servlet来做
```
Map<String, String> paramMap = new HashMap<String, String>();

        paramMap.put(Constants.P_DEVICE_ID, deviceId);
        paramMap.put(Constants.P_BARCODE, barcode);
        paramMap.put(Constants.P_VERSION, version);
        paramMap.put(Constants.P_RANDOM_STRING, randomString);
        paramMap.put(Constants.P_CHANNEL,channel);

        if(!validateParam(paramMap,timestamp,sign)){
            hanldExceptionResponse(response,"invalid sign");
            return;
        }


        response.setHeader("Content-Type", "application/octet-stream");
        String date =barcode.substring(7,7+6);
         //opt/nfsdata/reports/2/150915/report-h5_000800715091551133.zip
        String path = Constants.U_PRIFIX+"/"+version+"/"+date+"/"+Constants.U_PATH_PRFIX+barcode+".zip";
        logger.debug("The path is  {}", path);

        response.setHeader("X-Accel-Redirect",path);


private void hanldExceptionResponse(HttpServletResponse response,String msg) throws IOException {
        response.sendError(403,msg);


    }

```


### nginx修改
```

location ~ /file/(.*)/(.*)$ {
       alias "/opt/reports/$1/$2";
add_header Content-Disposition "attachment; filename=$2";
#add_header Content-Disposition "attachment; filename=fileceshi111";
        internal;

    #   error_page 404 =200 @backend ;
    }

    location @backend{
        proxy_pass http://localhost:8080;

    }

    location /reports/ {
     #   root   /usr/share/nginx/html;
      #  index  index.html index.htm;
         proxy_pass http://reportAuth;
    }


 upstream  reportAuth {
        ip_hash;
        server 127.0.0.1:8080;
    }
```
