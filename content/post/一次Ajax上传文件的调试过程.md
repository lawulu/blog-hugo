+++
title = "一次Ajax上传文件的调试过程"
draft = false
date = "2015-12-19"
Categories = ["Tech"] 
Description = "" 
Tags = ["JS", "Chrome"] 

+++
### Js端
使用的FormData
```
var formData = new FormData();
    var actionUrl = "/report/agentUpload";
    var form = new FormData();
    form.append("file", $("#reportFile" )[0]);

    var xhr = new XMLHttpRequest();
    xhr.open("post", actionUrl, true);
    xhr.onload = function () {
         alert("上传完成!");
    };

```
### Server端

```
public String upload(HttpServletRequest request, @RequestParam(value="file",required = true) MultipartFile file, ModelMap model)
```
但是就是取不到，报错：

```
Required MultipartFile parameter 'file' is not present
```


### Server端的问题？
#### 是否发到了Server？
看Network：

```
------WebKitFormBoundaryNLPhxKE21THaBaN1
Content-Disposition: form-data; name="file"

[object HTMLInputElement]

```


貌似是正确的，在Server端确认一下，将参数加上required=false
查看request:

```
multipartParameters = {HashMap@8882}  size = 1
multipartParameterContentTypes = {HashMap@8883}  size = 1
multipartFiles = {LinkedMultiValueMap@8884}  size = 0
```
即其实Request里面有东西，但是multipartFiles没有东西……
其实这时候应该已经看出问题了，因为那个值是一个String，但是没注意，自己盲目以为只要带multipartParameters就是文件

debug到CommonsMultipartResolver的resolveMultipart方法，里面有这段判断
```
protected MultipartParsingResult parseFileItems(List<FileItem> fileItems, String encoding) {
		MultiValueMap<String, MultipartFile> multipartFiles = new LinkedMultiValueMap<String, MultipartFile>();
		Map<String, String[]> multipartParameters = new HashMap<String, String[]>();
		Map<String, String> multipartParameterContentTypes = new HashMap<String, String>();

		// Extract multipart files and multipart parameters.
		for (FileItem fileItem : fileItems) {
			if (fileItem.isFormField()) {
```
如果是false…就往MultipartFile里面填值。
难道是Content-type的问题？
### 去Js端
先找到如何打印Formdata的：
http://stackoverflow.com/questions/17066875/how-to-inspect-formdata

```
for (var pair of formData.entries()) {
    console.log(pair[0]+ ', ' + pair[1]); 
}
```
打印的东西是：

```
file, [object HTMLInputElement]

```
决定采用原始的document.getElementById("reportFile").files[]

```
lastModifiedDate: Tue Dec 20 2015 10:39:39 GMT+0800 (CST)name: "工作簿1.xlsx"size: 23168type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"webkitRelativePath: ""__proto__: File

```
最终发现问题：
应该用

```
 $("#reportFile" )[0].files[0]

```

### 结论
- ++**盲目的去搜索其实非常浪费时间**++
- 要善用Chrome Console





