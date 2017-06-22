### 1. NoClassDefFoundError异常，或者在5.0以上系统程序正常，5.0以下系统crash等问题
如果你的报错日志如下：

![](https://ws1.sinaimg.cn/large/006tNc79ly1fgslhf13tsj30g70a4q4m.jpg)

那么这个问题是你使用的，mutidex打包导致的，详细解决方案查看这个链接：<https://stackoverflow.com/questions/35578135/noclassdeffounderror-for-okhttpclient>

如果你不想看，直接看这个截图：
![](https://ws2.sinaimg.cn/large/006tNc79ly1fgsllp2pv5j30z00t8wkd.jpg)

### 2. 设置Content-Type为什么无效
首先，记住任何时候，任何地方，以下这两行设置代码永远不起作用。
```
headers("Content-Type", "你自己设置的值")
headers("Content-Length", 一个数值)
```
`Content-Type`将遵循以下规则，`Content-Length`永远自动根据你上传的内容的真实大小自动添加，不可修改

### 警告：
所有`up开头`的方法不能与`params()`方法混用，如果混用，将按`up方法`的行为来，所有`params()`设置的参数将丢失。  

以下表格来自[OkGO文档](https://github.com/jeasonlzy/okhttp-OkGo/wiki/OkGo#6%E4%B8%8A%E4%BC%A0string%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%96%87%E6%9C%AC)的总结：

|调用方法|对应的`Content-Type`值|
|--|---|
|upStrig()|text/plain;charset=utf-8|
|upString(string,mediaType)|值就是你传进来的这个值，只是你只能上传文本数据|
|upJson()|application/json;charset=utf-8|
|upBytes()|application/octet-stream|
|upFile()|1. 会尝试自动根据你的文件后缀名去自动找到最适合他的`Content-Type`</br>2. 如果找不到，那么默认使用`application/octet-stream`|
|upRequestBody()|这里就是直接上传okhttp原生的RequestBody，你怎么构建的，他就传什么|
|params(string,string)|如果你的所有`params()`方法都是传的字符串，并没有文件，那么默认使用以下值：</br>` application/x-www-form-urlencoded`|
|params(string,file)|不论你有多少个`params()`，只要有一个传的是`File`，那么本次请求就将使用以下值：</br>`multipart/form-data; boundary=f6b76bad-0345-4337-b7d8-b362cb1f9949`|
|isMultipart(true)|当你所有的`params()`都是字符串，同时你还想你的`Content-Type`是`multipart/form-data`，那么使用该方法|

### 3. okgo支持断点下载，或者断点上传吗？
okgo不支持断点下载  
okserver支持断点下载  
断点上传都不支持，至于为什么，看这里吧：[有没有可能加入 断点/分片 上传的功能](https://github.com/jeasonlzy/okhttp-OkGo/issues/205)

### 4. 下载进度为什么是负数？
下载文件的时候，如果仅仅是读写文件流，客户端是无法知道文件的总大小的，但是服务端可以额外通过`Content-Length`头来告诉客户端，如果返回了，那么就是文件的大小，如果不返回，默认就是-1，所以如果你发现进度为-1，请联系服务端返回`Content-Length`响应头。