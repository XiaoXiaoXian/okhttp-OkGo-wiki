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

### 1.请求所有配置

注意以下几点：

- 这里演示的是一次普通请求所有能配置的参数，真实使用时不需要配置这么多，按自己的需要选择性的使用即可
- 第一行的泛型一定要特别注意，这个表示你请求网络的数据类型是什么，必须指定，否则无法解析网络数据。
- **.post(url)**：这个表示当前请求是post请求，当然一共支持GET，HEAD，OPTIONS，POST，PUT，DELETE, PATCH, TRACE这8种请求方式，你只需要改改这个方法名就行了，很方便。
- **`.params()`**：添加参数的时候,最后一个`isReplace`为可选参数,默认为`true`，即代表相同`key`的时候，后添加的会覆盖先前添加的。
- **.tag(this)**：请求的tag，用于标识当前的请求，方便后续取消对应的请求，如果你不需要取消请求，也可以不用设置。
- **.isMultipart()**：该方法表示是否强制使用`multipart/form-data`表单上传，因为该框架在有文件的时候，无论你是否设置这个参数，默认都是`multipart/form-data`格式上传，但是如果参数中不包含文件，默认使用`application/x-www-form-urlencoded`格式上传，如果你的服务器要求无论是否有文件，都要使用表单上传，那么可以用这个参数设置为true。
- **.isSpliceUrl()**：该方法表示是否强制将params的参数拼接到url后面，默认false不拼接。一般来说，post、put等有请求体的方法应该把参数都放在请求体中，不应该放在url上，但是有的服务端可能不太规范，url和请求体都需要传递参数，那么这时候就使用该参数，他会将你所有使用`.params()`方法传递的参数，自动拼接在url后面。
- **.retryCount()**：该方法是配置超时重连次数，也可以在全局初始化的时候设置，默认使用全局的配置，即为3次，你也可以在这里为你的这个请求特殊配置一个，并不会影响全局其他请求的超时重连次数。
- **.cacheKey() .cacheTime() .cacheMode()**：这三个是缓存相关的配置，[详细请看缓存介绍](https://github.com/jeasonlzy/okhttp-OkGo/wiki/Cache)
- **.headers()**：该方法是传递服务端需要的请求头，如果你不知道什么是请求头，[看wiki首页关于网络抓包中的http协议链接](https://github.com/jeasonlzy/okhttp-OkGo/wiki#%E7%BD%91%E7%BB%9C%E6%8A%93%E5%8C%85)。
- **.params()**：该方法传递键值对参数，格式也是http协议中的格式，详细参考上面的http协议连接。
- **.addUrlParams() .addFileParams() .addFileWrapperParams()**：这里是支持一个key传递多个文本参数，也支持一个key传递多个文件参数，详细也看上面的http协议连接。
 
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi7o0n1itj315s1medpg.jpg)

### 2.基本请求
那么我们不可能每次请求都写上面那么一大堆，这里可以看到，一次简单的请求，这么写就够了。
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/zj44z.jpg)

### 3.请求Bitmap
如果你知道你的请求数据是图片，可以使用这样的方法
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/rr1of.jpg)

### 4.基本文件下载
如果你要下载文件，可以这么写。

`FileCallback`具有三个重载的构造方法,分别是
> **FileCallback()**：空参构造<br>
> **FileCallback(String destFileName)**：可以额外指定文件下载完成后的文件名<br>
> **FileCallback(String destFileDir, String destFileName)**：可以额外指定文件的下载目录和下载完成后的文件名

文件目录如果不指定,默认下载的目录为 `sdcard/download/`,文件名如果不指定,则按照以下规则命名:

> 1.首先检查用户是否传入了文件名,如果传入,将以用户传入的文件名命名<br>
> 2.如果没有传入,那么将会检查服务端返回的响应头是否含有`Content-Disposition=attachment;filename=FileName.txt`该种形式的响应头,如果有,则按照该响应头中指定的文件名命名文件,如`FileName.txt`<br>
> 3.如果上述响应头不存在,则检查下载的文件url,例如:`http://image.baidu.com/abc.jpg`,那么将会自动以`abc.jpg`命名文件<br>
> 4.如果url也把文件名解析不出来,那么最终将以"unknownfile_" + System.currentTimeMillis()命名文件

![](http://7xss53.com1.z0.glb.clouddn.com/markdown/vpo1d.jpg)

### 5.上传String类型的文本
一般此种用法用于与服务器约定的数据格式，当使用该方法时，params中的参数设置是无效的，所有参数均需要通过需要上传的文本中指定，此外，额外指定的header参数仍然保持有效。</br>

默认会携带以下请求头
> Content-Type: text/plain;charset=utf-8

如果你对请求头有自己的要求，可以使用这个重载的形式，传入自定义的`content-type`
> // 比如上传xml数据，这里就可以自己指定请求头</br>
> upString("这是要上传的长文本数据！", MediaType.parse("application/xml"))

![](http://7xss53.com1.z0.glb.clouddn.com/markdown/nfli8.jpg)

### 6.上传JSON类型的文本
该方法与upString没有本质区别，只是数据格式是json,一般来说，需要自己创建一个实体bean或者一个map，把需要的参数设置进去，然后通过三方的Gson或者fastjson转换成json对象，最后直接使用该方法提交到服务器。</br>

默认会携带以下请求头，请不要手动修改，okgo也不支持自己修改
> Content-Type: application/json;charset=utf-8

![](http://7xss53.com1.z0.glb.clouddn.com/markdown/gy716.jpg)

### 7.上传文件
上传文件支持文件与参数一起同时上传，也支持一个key上传多个文件，以下方式可以任选</br>

特别要注意的是</br>

#### 1). 很多人会说需要在上传文件到时候，要把`Content-Type`修改掉，变成`multipart/form-data`，就像下面这样的。其实在使用OkGo的时候，只要你添加了文件，这里的的`Content-Type`不需要你手动设置，OkGo自动添加该请求头，同时，OkGo也不允许你修改该请求头。
> Content-Type: multipart/form-data; boundary=f6b76bad-0345-4337-b7d8-b362cb1f9949

#### 2). 如果没有文件，那么OkGo将自动使用以下请求头，同样OkGo也不允许你修改该请求头。
> Content-Type: application/x-www-form-urlencoded 

#### 3). 如果你的服务器希望你在没有文件的时候依然使用`multipart/form-data`请求，那么可以使用`.isMultipart(true)`这个方法强制修改，一般来说是不需要强制的。
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/eh7a3.jpg)

### 8.取消请求
之前讲到，我们为每个请求前都设置了一个参数`tag`，取消就是通过这个`tag`来取消的。通常我们在`Activity`中做网络请求，当`Activity`销毁时要取消请求否则会发生内存泄露，那么就可以在`onDestory()`里面写如下代码：
![](https://ws3.sinaimg.cn/large/006tNbRwly1fgi8fiotk7j31180a60ts.jpg)

### 9.同步请求

同步请求有两种方式，第一种是返回原始的Response对象，自行解析网络数据，就像这样：
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi8keykcaj31420hcdi3.jpg)

或者可以返回解析解析完成的对象，如果你使用过`Retrofit`，那么你对这个`Call`对象一定不会陌生，这里面有个方法是`converter()`，需要传递一个`Converter`接口，作用其实就是解析网络返回的数据，你也可以自定义`Converter`代码如下：
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi8mht3naj313s0i4q5g.jpg)

### 10.参数的顺序
添加`headers`和`params`的方法各有三处，在提交的时候,他们是有顺序的,如果对提交顺序有需要的话,请注意这里

- 第一个地方，全局初始化时添加
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi8r4l8ulj310m0acjsz.jpg)

- 第二个地方，`Callback`的`onStart()`方法中添加
 ![](https://ws4.sinaimg.cn/large/006tNbRwly1fgi8sezt6dj30ys09g75i.jpg)

- 第三个地方，执行网络请求的时候添加
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi8tqkwigj310q0ckdhf.jpg)
 
 那么，最终执行请求的参数的添加顺序为
 
> Header顺序: HKAAA -> HKBBB -> HKEEE -> HKFFF -> HKCCC -> HKDDD</br>
> Params顺序: PKAAA -> PKBBB -> PKEEE -> PKFFF -> PKCCC -> PKDDD
 
### 总结一句话就是，全局添加的在最开始，callback添加的在最后，请求添加的在中间