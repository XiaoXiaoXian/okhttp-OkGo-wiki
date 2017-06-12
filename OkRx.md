## 注意事项
OkRx是针对RxJava，将OkGO扩展的项目，使用的RxJava的版本如下：
> compile 'io.reactivex:rxjava:1.3.0'

OkRx2是针对RxJava2，将OkGO扩展的项目，使用的RxJava2的版本如下：
> compile 'io.reactivex.rxjava2:rxjava:2.1.0'

OkRx与OkRx2具有相同的使用方法，相同的Api调用方式，所以该文档以OkRx2为例，给出相关用法，OkRx的文档请参考该文档。

## OkRx主要功能
- 可以很完美结合RxJava做网络请求
- 在使用上比Retrofit更简单方便，门槛更低，灵活性更高
- 网络请求和RxJava调用可以做成一条链试，写法优雅
- 使用Converter接口，支持任意类型的数据自动解析
- OkRx是扩展的OkGo，所以OkGo包含的所有功能和写法，OkRx全部支持

## 请求写法
### 1. 获取Observable对象
我们还是像正常使用OkGo的方式一样，传入我们需要请求的url，和我们需要的参数，写法如下：
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi9pfkie9j310u05q3zk.jpg)

这里有两个特殊的方法：
- **.converter()**：该方法是网络请求的转换器，功能是将网络请求的结果转换成我们需要的数据类型。
- **.adapt()**：该方法是方法返回值的适配器，功能是将网络请求的`Call<T>`对象，适配成我们需要的`Observable<T>`对象。

### 2. 调用Rx转换代码
现在我们已经获取了`Observable`对象了，熟悉`RxJava`的你难道还不会使用了吗，直接上代码：
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi9vkddb2j311o0uc42m.jpg)

### 3. 代码整合
上面的调用很简单，我们可以把这两步放在一起写，如下：
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi9y6qjxcj312k0yugqg.jpg)

### 4. 结构优化
有人可能觉得链试代码太长，没关系，我们完全可以像`Retrofit`一样,自己写一个`ServerApi`类，这里面管理了所有的接口请求和参数，只是OkGo并不是采用的注解和反射实现的，而是通过传参来实现，相信对你你来讲，这样的方式更加直观。
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgia0n3ukmj31540wqgsl.jpg)

## Converter详解
我们知道Converter接口的作用就是将服务端的数据做解析转换的，仿佛这句话很熟悉，在哪看见过。如果你用过OkGo的基本请求，那么你一定知道我们会自定义Callback，实现里面的convertResponse方法，而Callback结构就是继承至Converter接口，所以他两本质就是一个东西。
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgia4fpy28j30ws01u3yn.jpg)

那么其实Converter的写法，和自定义Callback的写法原理是一样的，详细参考
- [OkGo内置Callback介绍](https://github.com/jeasonlzy/okhttp-OkGo/wiki/Callback)
- [JsonCallback自定义方法](https://github.com/jeasonlzy/okhttp-OkGo/wiki/JsonCallback)。

我们也内置了
- **StringConverter**：按文本解析，解析的编码依据服务端响应头中的`Content-Type`中的编码格式，自动进行编码转换，确保不出现乱码。
- **BitmapConverter**：如果请求的数据是图片，则可以使用该转换器，该方法对图片进行了压缩处理，确保不发生OOM。
- **FileConverter**：如果要做文件下载，可以使用该转换器，内部封装了关于文件下载和进度回调的方法。
- 如果这些不能满足你的需求，参考上面的自定义Converter链接。

## CallAdapter详解
我们写Rx请求的时候，最后调用了一个`adapt()`方法，这里面需要一个`CallAdapter`适配器接口，那么这个适配器是什么东西呢，主要功能就是将我们得到的`Call<T>`对象，适配成我们需要的`Observable<T>`对象。
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgiadrvl7rj30y205owez.jpg)

参考okrx2源码的目录结构如下：
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgiag3j5k6j31120m8gnm.jpg)
在adapter包下，命名规则分为这么几组：

开头的意思：
- **Observable**开头，表示返回的对象是Observable对象
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgiew0w30jj30ys05cwfj.jpg)
- **Flowabl**e开头，表示返回的对象是Flowable对象
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgiewbgwqfj30xc05k75d.jpg)
- **Maybe**开头，表示返回的对象是Maybe对象
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgiewlrmfaj30vu05cmy5.jpg)
- **Single**开头，表示返回的对象是Single对象
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgiewvqi64j30vc05675c.jpg)
- **Completable**开头，表示返回的对象是Completable对象
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgiex7zvudj30vo05ejse.jpg)

结尾的意思：
- **Response**结尾，表示返回对象的泛型是Response&lt;T>
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgier5ja5vj312y05c3zr.jpg)
- **Body**结尾，表示返回对象的泛型是T，没有外层的Response包装
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgierm8s4aj310e058my8.jpg)
- **Result**结尾，表示返回对象的泛型是Result&lt;T>
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgies0jqclj310405adh1.jpg)

## 取消请求
我们可以在基类创建这么两个方法，把每一个请求的`Disposable`对象都交给由统一的`CompositeDisposable`对象去管理。
![](https://ws3.sinaimg.cn/large/006tNbRwly1fgiez9loooj310q0eimzo.jpg)

在`Observer`对象的`onSubscibe`方法中，把`Disposable`添加到`CompositeDisposable`中
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgif2wzx1rj31100ok0w4.jpg)

然后我们在界面销毁或者需要取消的地方调用取消请求的方法。
![](https://ws4.sinaimg.cn/large/006tNbRwly1fgif1g8ey3j30xs068gm5.jpg)