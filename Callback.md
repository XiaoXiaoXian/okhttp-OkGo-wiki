## Callback介绍
`AbsCallback`接口默认只需要复写`onSuccess`方法，并不代表所有的回调都只走这一个，实际开发中，错误回调并没有成功回调使用频繁，所以我把`AbsCallback`的失败回调`onError`并没有声明为抽象的，如果有需要，请自行复写。

`Callback`一共有以下几个回调,除`onSuccess`必须实现以外,其余均可以按需实现,每个方法参数详细说明,请看下面第6点:
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/vr5l3.jpg)
 
由于`Callback`接口实现了`Converter`接口，所以这个接口里面的方法也必须实现
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/9auaw.jpg)

## 内置Callback
库中默认提供了4个Callback，也就是对`convertResponse()`这个方法的默认实现，具体如下：

### 1. AbsCallback
对`Callback`的默认包装，除`onSuccess()`和`convertResponse()`方法为抽象方法，其余均为空实现，源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/m8b2e.jpg)

### 2. StringCallBack
对`convertResponse()`按文本解析，解析的编码依据服务端响应头中的`Content-Type`中的编码格式，自动进行编码转换，确保不出现乱码，实现源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/n950v.jpg)

### 3. BitmapCallback
如果请求的数据是图片，则可以使用该回调，回调的图片进行了压缩处理，确保不发生OOM，部分源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/1xbbx.jpg)

### 4. FileCallBack
如果要做文件下载，可以使用该回调，内部封装了关于文件下载和进度回调的方法，部分源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/3xub1.jpg)

## 高级自定义Callback
由此可以看出来，该网络框架的核心，即为`Callback`的使用程度，例如我们可以尝试自定义一个JsonCallback，将网络返回的JSON数据自动解析成对象，[详细的自定义方法请猛戳这里]()。

### 1. DialogCallback
我们经常需要在网络请求前显示一个loading，请求结束后取消loading，这是个重复的工作，我们完全可以自定义一个`Callback`，让这个`Callback`帮我们完成这个工作，那么我们就需要用到`Callback`中的两个回调方法，`onStart()`和`onFinish()`，详细源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/xw6xy.jpg)

### 2. EncriptCallback
有时候我们需要在请求前，将我们的参数都加密或者签名，然而加密也是个重复的工作，我们没必要每次都写，也可以放在自定义的`Callback`中，源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/pp14n.jpg)

相信你要是看到这里，对`Callback`的功能一定有个全新的认识，他可以帮助你减少很多重复工作。

关于自定义`JsonCallback`，需要单独的一篇文档来说明，[需要的话请猛戳这里]()。