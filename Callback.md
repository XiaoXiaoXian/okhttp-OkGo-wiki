## Callback介绍
`AbsCallback`接口默认只需要复写`onSuccess`方法，并不代表所有的回调都只走这一个，实际开发中，错误回调并没有成功回调使用频繁，所以我把`AbsCallback`的失败回调`onError`并没有声明为抽象的，如果有需要，请自行复写。

`Callback`一共有以下几个回调,除`onSuccess`必须实现以外,其余均可以按需实现,每个方法参数详细说明,请看下面第6点:
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/vr5l3.jpg)
 
由于`Callback`接口实现了`Converter`接口，所以这个接口里面的方法也必须实现
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/9auaw.jpg)

## 内置Callback
库中默认提供了4个Callback，也就是对`convertResponse()`这个方法的默认实现，再介绍`Callback`之前，我先详细说一下convertResponse这个方法的作用，以及他在源码中的调用时机，这个理解了，对自定义callback有很大帮助，先给张图，这是部分源码：
![](https://ws2.sinaimg.cn/large/006tNc79ly1fh468bonhlj313u0mytcp.jpg)
图中画红线的地方就是调用`convertResponse`的地方，之所以能这么写，是因为我们的`Callback`接口继承了`Converter`接口，所以我们能获取到这个方法并调用。而最外层整个这个方法叫`onResponse`方法，其实就`是okhttp`默认的成功回调 的方法，他是在子线程的，我们直接在子线程中调用了`convertResponse`方法，所以这个方法也是在子线程的。

我们再把这个方法重头梳理一下逻辑，当我们请求成功后，当然`okhttp`只会给我们判断当前网络是不是通的，除非没有网络，或者请求超时等，才会回调`onFaile`方法，否则无论什么响应码，统统都是回调`onResponse`方法，所以我们看到源码中，第一行就先判断响应码是不是`404`或者大于等于`500`，如果是就直接回调触发`onError`方法，并且`return`掉。

这里要重点解释下为什么404和大于等于500这两个要会直接回调`onError`并且`return`，因为大家都知道
- **404**是网页找不到了，要么就是服务端资源改了，要么就是url地址写错了，这时候服务端响应的数据是一堆404的html格式的数据，对我们客户端是没有意义的，所以直接调用`onError`，并且没必要执行后续代码了。
- **大于等于500呢**，是服务端发生了内部错误，意思就是你一个请求发过去，也许是你请求参数写的有问题，也许是服务端代码逻辑有问题，总之结果就是服务端挂了，抛异常了，并且给你返回来一堆服务端的异常堆栈信息，那么你拿到这些个信息有什么用呢，所以也直接调用`onError`了。
- 那么问题来了，有些响应码比如400，401，有些人也希望回调`onError`怎么办呢？别急，接着往下看代码。

接下来的`onAnalysisResponse`这个方法我们暂且不用管它，主要是处理了304相关的缓存协议头，这里我们就当他永远不会`return`就行了。

**再往下就是最重要的`try-catch`代码块了，这段代码特别重要，重要到你理解了这段代码，那么在你做网络请求的过程中，可以少很多疑问。**
- 首先第一行就是我用红线标出来的调用的`convertResponse`方法，这里就能看出来这个方法的作用，他是把网络请求返回的原始`response`对象里面的数据取出来，转换成我们泛型约定好的数据对象。
- 至于怎么转换，能不能转成功，其实`okgo`并不关心，对于`okgo`来说，无论你怎么解析，你只要返回了一个和泛型数据类型一致的数据，那么就是成功了，如果你抛异常了，那么就会被`try-catch`捕获到，进入`catch`代码块回调`onError`。
- 所以这段代码很神奇发现没，神奇到最后的结果是回调`onSuccess`还是`onError`，不是`okgo`框架写死的，而是由用户在`convertResponse`方法中的逻辑决定的，如果抛异常，就会回调`onError`，不抛异常，有返回值了，那就是`onSuccess`。

由此而来就会出现两个典型的问题。

#### 问题一：
大家喜欢用`JsonCallbac`k，这个`JsonCallback`是大家自定义的，由于自己对`json`解析的不熟悉，导致控制台打印出这个异常：

![](http://7xss53.com1.z0.glb.clouddn.com/markdown/hdf7k.jpg)

有人看到这个就很疑惑了，说我网络请求都成功了，log的数据都打出来了，为什么回调了`onError`方法，实际上你要是搞懂了上面的逻辑，这个问题就很好解释了。所有绿色的日志是`okgo`自己的拦截器自动打印的，表示当前的网络请求状态，前两个`-->`表示请求的相关数据，后两个`<--`表示服务端打回来的响应数据，我们能清楚的看到这一次交互的相关信息，并别能清楚的看到服务端打回来的正确响应的数据。

之所以回调了`onError`，继续看下面异常，核心就一句话：

```
Expected BEGIN_ARRAY but was BEGIN_OBJECT
```

翻译成中文就是，我期望要转换的数据格式应该是个`JSONArray`的格式，但是我实际要转换的数据格式却是`JSONObject`的格式，`Gson`框架不知道怎么转换了，所以抛出了这个异常。我们再这个异常信息中能看到这么一行信息:

```
at com.lzy.demo.callback.JsonCallback.convertResponse(JsonCallback.java:89)
```

发现错误抛到了`Callback`的`convertResponse`的方法中，而这个方法被我们源码给`try`住了，然后在`catch`代码中，回调了`onError`了。意思就是，虽然你网络请求成功了，但是你解析网路的数据发生异常了，你任然拿不到正确结果，所以失败，是不是清晰多了，遇到这种问题，就好好检查自己在`convertResponse`中的代码吧。

#### 问题二：
为什么服务端给我返回了400的响应码，我还回调了`onSuccess`？

这个问题其实你要是仔细看前面的代码，就很简单了，可能我都不需要过多解析了。我补充一点，如果你希望在400的时候回调`onError`，那么你可以自己在`convertResponse`中写一些逻辑，自己手动抛出异常，这样就回调了`onError`。

#### 问题三：
服务端返回的错误信息我怎么拿到？
比如当请求有问题的时候，有的服务端会返回响应码400，也就是上面的问题，顺便返回一段数据，大概这样：

```
{
	"code":300,
	"msg":"用户名或密码错误",	
	"data":null
}
```

有人说，我如何在`onError`中能获取到这段`msg`信息。答案就是服务端返回这段信息的时候，是不是必然会导致客户端会调用`convertResponse`方法，那么你需要做的就是在`convertResponse`中解析这段数据，获取`msg`数据，然后抛一个异常，比如这样:

```
throw new IllegalStateException(msg);
```

就把有效的错误信息用异常抛出去了，既然抛异常了，那么肯定会被源码的`catch`捕获，回调`onError`是吧，那么我们再`onError`中写如下代码：
![](https://ws2.sinaimg.cn/large/006tNc79gy1fh48ia82eoj30z60ba3zq.jpg)
你会发现，是不是数据就被你通过异常的方法传递过来了。

到这里，关于`convertResponse`的相关作用和用法就说完了，接下来我们看看`Callback`中到底是怎么用这个方法的，首先是`okgo`框架内置的几个`Callback`，相信看完上面的原理讲解后再看这些内置`Callback`，会无比简单了。。

### 1. AbsCallback
对`Callback`的默认包装，除`onSuccess()`和`convertResponse()`方法为抽象方法，其余均为空实现，源码如下：
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi75vp68fj314w0q6why.jpg)

### 2. StringCallBack
对`convertResponse()`按文本解析，解析的编码依据服务端响应头中的`Content-Type`中的编码格式，自动进行编码转换，确保不出现乱码，实现源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/n950v.jpg)

### 3. BitmapCallback
如果请求的数据是图片，则可以使用该回调，回调的图片进行了压缩处理，确保不发生OOM，部分源码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/1xbbx.jpg)

### 4. FileCallBack
如果要做文件下载，可以使用该回调，内部封装了关于文件下载和进度回调的方法，部分源码如下：
![](https://ws1.sinaimg.cn/large/006tNc79ly1fho5qf4y7pj316u1p6guo.jpg)

## 高级自定义Callback
由此可以看出来，该网络框架的核心，即为`Callback`的使用的熟练程度，例如我们可以尝试自定义一个JsonCallback，将网络返回的JSON数据自动解析成对象，[详细的JsonCallback自定义方法请猛戳这里](https://github.com/jeasonlzy/okhttp-OkGo/wiki/JsonCallback)。

### 1. DialogCallback
我们经常需要在网络请求前显示一个loading，请求结束后取消loading，这是个重复的工作，我们完全可以自定义一个`Callback`，让这个`Callback`帮我们完成这个工作，那么我们就需要用到`Callback`中的两个回调方法，`onStart()`和`onFinish()`，详细源码如下：
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgi76xoo9vj315k0ton1o.jpg)

### 2. EncriptCallback
有时候我们需要在请求前，将我们的参数都加密或者签名，然而加密也是个重复的工作，我们没必要每次都写，也可以放在自定义的`Callback`中，源码如下：
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi77etrfbj315k1jpk17.jpg)

相信你要是看到这里，对`Callback`的功能一定有个全新的认识，他可以帮助你减少很多重复工作。

关于自定义`JsonCallback`，需要单独的一篇文档来说明，[需要的话请猛戳这里](https://github.com/jeasonlzy/okhttp-OkGo/wiki/JsonCallback)。

