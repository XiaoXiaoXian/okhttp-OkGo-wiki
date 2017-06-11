## OkGo主要功能
* 基本的get、post、put、delete、head、options、trace、patch八种请求
* 支持upString，upJson，upBytes，upFile等up类方法上传特定数据
* 支持一个key上传一个文件，也可以一个key上传多个文件，也可以多文件和多参数一起上传
* 大文件下载和下载进度回调
* 大文件上传和上传进度回调
* 支持cookie的自动管理，并可自定义cookie管理策略
* 支持缓存模式，不仅支持http缓存协议，也支持自定义缓存策略
* 支持重定向
* 支持自定义超时自动重连次数
* 支持链式调用
* 支持https访问，支持双向认证
* 支持根据tag取消请求，也可全部取消
* 支持自定义Callback，自动解析网络数据

## 请求

### 1.基本请求方式介绍
```java
OkGo.get(Urls.URL_METHOD)     // 请求方式和请求url
	.tag(this)                       // 请求的 tag, 主要用于取消对应的请求
	.cacheKey("cacheKey")            // 设置当前请求的缓存key,建议每个不同功能的请求设置一个
	.cacheMode(CacheMode.DEFAULT)    // 缓存模式，详细请看缓存介绍
	.execute(new StringCallback() {
		@Override
		public void onSuccess(String s, Call call, Response response) {
		    // s 即为所需要的结果
		}
	});
```
### 2.请求 Bitmap 对象
```java
OkGo.get(Urls.URL_IMAGE)//
	.tag(this)//
	.execute(new BitmapCallback() {
	    @Override
	    public void onSuccess(Bitmap bitmap, Call call, Response response) {
		    // bitmap 即为返回的图片数据
	    }
	});
```
### 3.请求 文件下载
`FileCallback`具有三个重载的构造方法,分别是
> `FileCallback()`:空参构造<br>
> `FileCallback(String destFileName)`:可以额外指定文件下载完成后的文件名<br>
> `FileCallback(String destFileDir, String destFileName)`:可以额外指定文件的下载目录和下载完成后的文件名

文件目录如果不指定,默认下载的目录为 `sdcard/download/`,文件名如果不指定,则按照以下规则命名:

> 1.首先检查用户是否传入了文件名,如果传入,将以用户传入的文件名命名<br>
> 2.如果没有传入,那么将会检查服务端返回的响应头是否含有`Content-Disposition=attachment;filename=FileName.txt`该种形式的响应头,如果有,则按照该响应头中指定的文件名命名文件,如`FileName.txt`<br>
> 3.如果上述响应头不存在,则检查下载的文件url,例如:`http://image.baidu.com/abc.jpg`,那么将会自动以`abc.jpg`命名文件<br>
> 4.如果url也把文件名解析不出来,那么最终将以`nofilename`命名文件

```java
OkGo.get(Urls.URL_DOWNLOAD)//
	.tag(this)//
	.execute(new FileCallback() {  //文件下载时，可以指定下载的文件目录和文件名
	    @Override
	    public void onSuccess(File file, Call call, Response response) {
		    // file 即为文件数据，文件保存在指定目录
	    }
	    
	    @Override
        public void downloadProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
            //这里回调下载进度(该回调在主线程,可以直接更新ui)
        }
	});
```
### 4.普通Post，直接上传String类型的文本
一般此种用法用于与服务器约定的数据格式，当使用该方法时，params中的参数设置是无效的，所有参数均需要通过需要上传的文本中指定，此外，额外指定的header参数仍然保持有效。</br>

默认会携带以下请求头
> Content-Type: text/plain;charset=utf-8

如果你对请求头有自己的要求，可以使用这个重载的形式，传入自定义的`content-type`
> upString("这是要上传的长文本数据！", MediaType.parse("application/xml")) // 比如上传xml数据，这里就可以自己指定请求头

```java
OkGo.post(Urls.URL_TEXT_UPLOAD)//
	.tag(this)//
//	.params("param1", "paramValue1")//  这里不要使用params，upString 与 params 是互斥的，只有 upString 的数据会被上传
	.upString("这是要上传的长文本数据！")//
//	.upString("这是要上传的长文本数据！", MediaType.parse("application/xml")) // 比如上传xml数据，这里就可以自己指定请求头
	.execute(new StringCallback() {
	    @Override
	    public void onSuccess(String s, Call call, Response response) {
			//上传成功
	    }
	    
	    @Override
        public void upProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
            //这里回调上传进度(该回调在主线程,可以直接更新ui)
        }
	});
```

### 5.普通Post，直接上传Json类型的文本
该方法与postString没有本质区别，只是数据格式是json,一般来说，需要自己创建一个实体bean或者一个map，把需要的参数设置进去，然后通过三方的Gson或者fastjson转换成json字符串，最后直接使用该方法提交到服务器。</br>

默认会携带以下请求头，请不要手动修改，okgo也不支持自己修改
> Content-Type: application/json;charset=utf-8

```java
HashMap<String, String> params = new HashMap<>();
params.put("key1", "value1");
params.put("key2", "这里是需要提交的json格式数据");
params.put("key3", "也可以使用三方工具将对象转成json字符串");
params.put("key4", "其实你怎么高兴怎么写都行");
JSONObject jsonObject = new JSONObject(params);
        
OkGo.post(Urls.URL_TEXT_UPLOAD)//
	.tag(this)//
//	.params("param1", "paramValue1")//  这里不要使用params，upJson 与 params 是互斥的，只有 upJson 的数据会被上传
	.upJson(jsonObject.toString())//
	.execute(new StringCallback() {
	    @Override
	    public void onSuccess(String s, Call call, Response response) {
			//上传成功
	    }
	    
	    	    
	    @Override
        public void upProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
            //这里回调上传进度(该回调在主线程,可以直接更新ui)
        }
	});
```


### 6.上传图片或者文件
上传文件支持文件与参数一起同时上传，也支持一个key上传多个文件，以下方式可以任选</br>

特别要注意的是</br>

#### 1).很多人会说需要在上传文件到时候，要把`Content-Type`修改掉，变成`multipart/form-data`，就像下面这样的。其实在使用OkGo的时候，只要你添加了文件，这里的的`Content-Type`不需要你手动设置，OkGo自动添加该请求头，同时，OkGo也不允许你修改该请求头。
> Content-Type: multipart/form-data; boundary=f6b76bad-0345-4337-b7d8-b362cb1f9949

#### 2).如果没有文件，那么OkGo将自动使用以下请求头，同样OkGo也不允许你修改该请求头。
> Content-Type: application/x-www-form-urlencoded 

#### 3).如果你的服务器希望你在没有文件的时候依然使用`multipart/form-data`请求，那么可以使用`.isMultipart(true)`这个方法强制修改，一般来说是不需要强制的。

```java
OkGo.post(URL_FORM_UPLOAD)//
	.tag(this)//
	.isMultipart(true)       // 强制使用 multipart/form-data 表单上传（只是演示，不需要的话不要设置。默认就是false）
	.params("param1", "paramValue1") 		// 这里可以上传参数
	.params("file1", new File("filepath1"))   // 可以添加文件上传
	.params("file2", new File("filepath2")) 	// 支持多文件同时添加上传
	.addFileParams("key", List<File> files)	// 这里支持一个key传多个文件
	.execute(new StringCallback() {
	    @Override
	    public void onSuccess(String s, Call call, Response response) {
			//上传成功
	    }
	    
	    	    
	    @Override
        public void upProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
            //这里回调上传进度(该回调在主线程,可以直接更新ui)
        }
	});
```

### 7.https请求，需要在初始化的时候配置以下代码
```java
OkGo.getInstance()
    ...
    //可以设置https的证书,以下几种方案根据需要自己设置
       .setCertificates()                                  //方法一：信任所有证书,不安全有风险
    // .setCertificates(new SafeTrustManager())            //方法二：自定义信任规则，校验服务端证书
    // .setCertificates(getAssets().open("srca.cer"))      //方法三：使用预埋证书，校验服务端证书（自签名证书）
    //方法四：使用bks证书和密码管理客户端证书（双向认证），使用预埋证书，校验服务端证书（自签名证书）
    // .setCertificates(getAssets().open("xxx.bks"), "123456", getAssets().open("yyy.cer"))//
    
    //配置https的域名匹配规则，详细看demo的初始化介绍，不需要就不要加入，使用不当会导致https握手失败
    // .setHostnameVerifier(new SafeHostnameVerifier())
    ...
```
### 8.请求功能的所有配置讲解

以下代码包含了以下内容：

 * 一次普通请求所有能配置的参数，真实使用时不需要配置这么多，按自己的需要选择性的使用即可
 * `params`添加参数的时候,最后一个`isReplace`为可选参数,默认为`true`,即代表相同`key`的时候,后添加的会覆盖先前添加的
 * 多文件和多参数的表单上传，同时支持进度监听
 * 为单个请求设置超时，比如涉及到文件的需要设置读写等待时间多一点。
 
```java
OkGo.post(Urls.URL_METHOD)    // 请求方式和请求url, get请求不需要拼接参数，支持get，post，put，delete，head，options请求
	.tag(this)               // 请求的 tag, 主要用于取消对应的请求
	.isMultipart(true)       // 强制使用 multipart/form-data 表单上传（只是演示，不需要的话不要设置。默认就是false）
	.connTimeOut(10000)      // 设置当前请求的连接超时时间
	.readTimeOut(10000)      // 设置当前请求的读取超时时间
	.writeTimeOut(10000)     // 设置当前请求的写入超时时间
	.cacheKey("cacheKey")    // 设置当前请求的缓存key,建议每个不同功能的请求设置一个
	.cacheTime(5000)         // 缓存的过期时间,单位毫秒
	.cacheMode(CacheMode.FIRST_CACHE_THEN_REQUEST) // 缓存模式，详细请看第四部分，缓存介绍
	.addInterceptor(interceptor)            		// 添加自定义拦截器
	.headers("header1", "headerValue1")     		// 添加请求头参数
	.headers("header2", "headerValue2")     		// 支持多请求头参数同时添加
	.params("param1", "paramValue1")        		// 添加请求参数
	.params("param2", "paramValue2")        		// 支持多请求参数同时添加
	.params("file1", new File("filepath1")) 		// 可以添加文件上传
	.params("file2", new File("filepath2")) 		// 支持多文件同时添加上传
	.addUrlParams("key", List<String> values) 	// 这里支持一个key传多个参数
	.addFileParams("key", List<File> files)		// 这里支持一个key传多个文件
	.addFileWrapperParams("key", List<HttpParams.FileWrapper> fileWrappers)//这里支持一个key传多个文件
	//这里给出的泛型为 ServerModel，同时传递一个泛型的 class对象，即可自动将数据结果转成对象返回
	.execute(new DialogCallback<ServerModel>(this) {
		@Override
		public void onBefore(BaseRequest request) {
		    // UI线程 请求网络之前调用
		    // 可以显示对话框，添加/修改/移除 请求参数
		}
	
		@Override
		public ServerModel convertSuccess(Response response) throws Exception{
		    // 子线程，可以做耗时操作
		    // 根据传递进来的 response 对象，把数据解析成需要的 ServerModel 类型并返回
		    // 可以根据自己的需要，抛出异常，在onError中处理
		    return null;
		}
		
		@Override
		public void parseError(Call call, IOException e) {
			// 子线程，可以做耗时操作
			// 用于网络错误时在子线程中执行数据耗时操作,子类可以根据自己的需要重写此方法
		}
	
		@Override
		public void onSuccess(ServerModel serverModel, Call call, Response response) {
		    // UI 线程，请求成功后回调
		    // ServerModel 返回泛型约定的实体类型参数
		    // call        本次网络的请求信息，如果需要查看请求头或请求参数可以从此对象获取
		    // response    本次网络访问的结果对象，包含了响应头，响应码等		
		}
		
		@Override
		public void onCacheSuccess(ServerModel serverModel, Call call) {
		    // UI 线程，缓存读取成功后回调
		    // serverModel 返回泛型约定的实体类型参数
		    // call        本次网络的请求信息
		}
	
		@Override
		public void onError(Call call, Response response, Exception e) {
		    // UI 线程，请求失败后回调
		    // call        本次网络的请求对象，可以根据该对象拿到 request
		    // response    本次网络访问的结果对象，包含了响应头，响应码等
		    // e           本次网络访问的异常信息，如果服务器内部发生了错误，响应码为 404,或大于等于500
		}
	
		@Override
		public void onCacheError(Call call, Exception e) {
			// UI 线程，读取缓存失败后回调
			// call        本次网络的请求对象，可以根据该对象拿到 request
			// e           本次网络访问的异常信息，如果服务器内部发生了错误，响应码为 404,或大于等于500
		}
	
		@Override
		public void onAfter(ServerModel serverModel, Exception e) {
		    // UI 线程，请求结束后回调，无论网络请求成功还是失败，都会调用，可以用于关闭显示对话框
		    // ServerModel 返回泛型约定的实体类型参数，如果网络请求失败，该对象为　null
		    // e           本次网络访问的异常信息，如果服务器内部发生了错误，响应码为 404,或大于等于500
		}
	
		@Override
		public void upProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
		    // UI 线程，文件上传过程中回调，只有请求方式包含请求体才回调（GET,HEAD不会回调）
		    // currentSize  当前上传的大小（单位字节）
		    // totalSize 　 需要上传的总大小（单位字节）
		    // progress     当前上传的进度，范围　0.0f ~ 1.0f
		    // networkSpeed 当前上传的网速（单位秒）
		}
	
		@Override
		public void downloadProgress(long currentSize, long totalSize, float progress, long networkSpeed) {
		    // UI 线程，文件下载过程中回调
		    //参数含义同　上传相同
		}
    });
```
### 9.取消请求
每个请求前都设置了一个参数`tag`，取消则通过` OkGo.cancel(tag)`执行。
例如：在Activity中，当Activity销毁取消请求，可以在onDestory里面统一取消。
```java
@Override
protected void onDestroy() {
    super.onDestroy();

    //根据 Tag 取消请求
    OkGo.getInstance().cancelTag(this);

    //取消所有请求
    OkGo.getInstance().cancelAll();
}
```
### 10.同步的请求
execute方法不传入callback即为同步的请求，返回`Response`对象，需要自己解析
```java
Response response = OkGo.get("http://www.baidu.com")//
                                .tag(this)//
                                .headers("aaa", "111")//
                                .params("bbb", "222")
                                .execute();
```
### 11.参数的顺序
添加header和param的方法各有三个地方,在提交的时候,他们是有顺序的,如果对提交顺序有需要的话,请注意这里

 * 第一个地方,全局初始化时,使用`OkGo.getInstance().addCommonHeaders()`,`OkGo.getInstance().addCommonParams()` 添加

```java
HttpHeaders headers = new HttpHeaders();
headers.put("HKAAA", "HVAAA");
headers.put("HKBBB", "HVBBB");
HttpParams params = new HttpParams();
params.put("PKAAA", "PVAAA"); 
params.put("PKBBB", "PVBBB");

OkGo.getInstance()
    .addCommonHeaders(headers) //设置全局公共头
    .addCommonParams(params);  //设置全局公共参数
```

 * 第二个地方,`callback`的`onBefore`方法中添加
 
```java
public abstract class CommonCallback<T> extends AbsCallback<T> {
    @Override
    public void onBefore(BaseRequest request) {
        super.onBefore(request);
        
        request.headers("HKCCC", "HVCCC")//
                .headers("HKDDD", "HVDDD")//
                .params("PKCCC", "PVCCC")//
                .params("PKDDD", "PVDDD")//
    }
}
```

 * 第三个地方,执行网络请求的时候添加

```java
OkGo.get(Urls.URL_METHOD)//
        .tag(this)//
        .headers("HKEEE", "HVEEE")//
        .headers("HKFFF", "HVFFF")//
        .params("PKEEE", "PVEEE")//
        .params("PKFFF", "PVFFF")//
        .execute(new MethodCallBack<>(this, ServerModel.class));
```
 
 那么,最终执行请求的参数的添加顺序为
 
> * Header顺序: HKAAA -> HKBBB -> HKEEE -> HKFFF -> HKCCC -> HKDDD
> * Params顺序: PKAAA -> PKBBB -> PKEEE -> PKFFF -> PKCCC -> PKDDD
 
### 总结一句话就是,全局添加的在最开始,callback添加的在最后,请求添加的在中间
 

