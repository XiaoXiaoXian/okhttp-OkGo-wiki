## 注意事项
使用缓存前，必须让涉及到缓存`javaBean`对象实现`Serializable`接口，否者会报`NotSerializableException`。因为缓存的原理是将对象序列化后直接写入数据库中，如果不实现`Serializable`接口，会导致对象无法序列化，进而无法写入到数据库中，也就达不到缓存的效果。

## 代码示例
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/9ze5r.jpg)
涉及到缓存相关的api一共是如下五个：

### 1. cacheKey
每次框架是根据这个`cacheKey`去到数据库中寻找缓存的，所以一般来说，每个不同功能的请求都要设置不一样的`cacheKey`，如果相同，会导致数据库中的缓存数据发生覆盖或错乱。如果不指定`cacheKey`，默认是用`url`带参数的全路径名为`cacheKey`。

### 2. cacheMode
目前默认提供了五种`CacheMode`缓存模式，不同的模式会有不同的Callback回调顺序，详细见下方的回调顺序
-  `NO_CACHE`：不使用缓存，该模式下`cacheKey`、`cacheTime` 参数均无效
- `DEFAULT`：按照HTTP协议的默认缓存规则，例如有304响应头时缓存。
- `REQUEST_FAILED_READ_CACHE`：先请求网络，如果请求网络失败，则读取缓存，如果读取缓存失败，本次请求失败。
- `IF_NONE_CACHE_REQUEST`：如果缓存不存在才请求网络，否则使用缓存。
- `FIRST_CACHE_THEN_REQUEST`：先使用缓存，不管是否存在，仍然请求网络。

### 3. cacheTime
当前缓存的有效时间是多长，单位毫秒，上面示例是3600秒，也就是一个小时，这个根据自己需要设置，如果不设置，默认是`CacheEntity.CACHE_NEVER_EXPIRE=-1`，也就是永不过期。该参数对`DEFAULT`模式是无效的，因为该模式是完全遵循标准的http协议的，缓存时间是依靠服务端响应头来控制，所以客户端的`cacheTime`参数无效。

### 4. cachePolicy
这个是自定义的缓存策略，内置的五大缓存模式其实就是这个缓存策略`CachePolicy`接口的五种不同实现，如果这五种模式不能满足你，你完全可以自行实现这个接口，写出属于你自己的缓存规则。

### 5. onCacheSuccess
当缓存读取成功后，回调的是这个方法，如果你只复写了`onSuccess`方法，是无法获取缓存的，这里要注意。

## 回调顺序
不同的缓存模式具有不同的回调顺序，虽然顺序写的很复杂,但是理解后,是很简单,并且合情合理的
#### 1.) 无缓存模式 CacheMode.NO_CACHE
网络请求成功 onStart -> convertResponse -> onSuccess -> onFinish  
网络请求失败 onStart -> onError -> onFinish  

#### 2.) 默认缓存模式,遵循304头 CacheMode.DEFAULT
网络请求成功,服务端返回非304 onStart -> convertResponse -> onSuccess -> onFinish  
网络请求成功服务端返回304 onStart -> onCacheSuccess -> onFinish  
网络请求失败 onStart -> onError -> onFinish  

#### 3.) 请求网络失败后读取缓存 CacheMode.REQUEST_FAILED_READ_CACHE
网络请求成功,不读取缓存 onStart -> convertResponse -> onSuccess -> onFinish  
网络请求失败,读取缓存成功 onStart -> onCacheSuccess -> onFinish  
网络请求失败,读取缓存失败 onStart -> onError -> onFinish  

#### 4.) 如果缓存不存在才请求网络，否则使用缓存 CacheMode.IF_NONE_CACHE_REQUEST
已经有缓存,不请求网络 onStart -> onCacheSuccess -> onFinish  
没有缓存请求网络成功 onStart -> convertResponse -> onSuccess -> onFinish  
没有缓存请求网络失败 onStart -> onError -> onFinish  

#### 5.) 先使用缓存，不管是否存在，仍然请求网络 CacheMode.FIRST_CACHE_THEN_REQUEST
无缓存时,网络请求成功 onStart -> convertResponse -> onSuccess -> onFinish  
无缓存时,网络请求失败 onStart -> onError -> onFinish  
有缓存时,网络请求成功 onStart -> onCacheSuccess -> convertResponse -> onSuccess -> onFinish  
有缓存时,网络请求失败 onStart -> onCacheSuccess -> onError -> onFinish  

## 缓存的手动操作
**！！！警告：在没有完全搞懂缓存的原理前，不要尝试自己操作缓存**  
**！！！警告：在没有完全搞懂缓存的原理前，不要尝试自己操作缓存**  
**！！！警告：在没有完全搞懂缓存的原理前，不要尝试自己操作缓存**  

### 1）CacheEntity
首先介绍缓存的实体类，先看代码中对该类的定义：
![](https://ws4.sinaimg.cn/large/006tKfTcly1fh11gevb5xj31220fqwhs.jpg)

OkGo就是使用的CacheEntity这样的数据格式保存的缓存，缓存保存在数据库中，一共有四个字段如下：
- **key**：表示缓存的key，在代码中通过`.cacheKey()`方法指定，这个key在数据库中对应的是主键，定义为unique，所以这也就是我为什么说不同的数据，缓存的key一定不能一样，因为你要是一样了，数据库查询删除替换缓存，必然就出现混乱了。
- **localExpire**：表示缓存的过期时间，在代码中通过`.cacheTime()`方法指定，单位毫秒，如果传-1，表示永不过期。过期的意思其实就是当框架取缓存的时候，框架会多判断一次系统的当前时间与过期时间，发现当前时间大于过期时间了，那么就表示过期，返回null了，并且把数据库中对应的缓存删除。
- **head**：缓存的响应头，表示当前服务端的响应头信息，如果你不知道什么是响应头，请看wiki首页http协议介绍，或者忽略这句话吧。
- **data**：缓存的实体数据，这个是真正服务端返回的数据，但是注意，我并不是直接缓存的服务端返回的原始信息，而是缓存的`convertResponse()`方法返回的值，什么意思呢？就是一个网络请求okgo不关心它真正的数据是什么，他只关心你在`convertResponse()`中给他返回的数据是什么，他就把这个数据缓存下来了，下次从数据库中取对应`cachekey`的缓存的时候，取到的也就是你上次返回的值。下面是源码中保存缓存的时机，正好说明了这点。
![](https://ws1.sinaimg.cn/large/006tKfTcly1fh11vbj7yzj30yc050aax.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcly1fh11x9g8ayj31380j80w2.jpg)

### 2）CacheManager
我们知道了缓存是保存在数据库中的，那么其实缓存的操作也就是数据库的操作，那么操作这个数据库中缓存表的对象是这个`CacheManager`，它是个单例模式，里面提供了很多方法，常用的方法如下：

1. 获取所有的缓存，但是一般每个缓存的数据类型都不一样，所以缓存的泛型使用 ？
![](https://ws4.sinaimg.cn/large/006tKfTcly1fh12eu3cd0j311s03wgm8.jpg)
2. 根据cacheKey获取对应的缓存，一共有以下几种写法
![](https://ws1.sinaimg.cn/large/006tKfTcly1fh12e7qvwzj31820ecn1g.jpg)
3. 根据cacheKey删除指定缓存，返回值是否删除成功
![](https://ws1.sinaimg.cn/large/006tKfTcly1fh12gblccmj30yi03qwes.jpg)
4. 清除所有缓存，返回值是否删除成功
![](https://ws2.sinaimg.cn/large/006tKfTcly1fh12gq0d6wj30vm034mxd.jpg)
5. 其他操作，自己查看CacheManager中的方法
![](https://ws4.sinaimg.cn/large/006tKfTcly1fh12itzs8ej311i0c2q50.jpg)
6. 有人问了，怎么获取缓存的大小，那么这里有答案了，缓存都保存在数据中，数据库里面的内容能获取大小么？至少我不会，如果你会请告诉我。有人说可以获取数据库文件的总大小就行了，而这个数据库中不止有缓存一张表，所以也不准确。所以这事就别纠结了，虽然我们把这个东西叫缓存，但其实是在数据库中，数据库属于app的应用数据，不属于缓存数据。你要是真的很在意这个大小，你就每隔一段时间，把缓存表清空一次就行了是吧。

## 项目中使用缓存
一般使用缓存的时候，指定`cacheKey`和`cacheMode`就行了，但是遇到列表缓存的时候，就有点不一样了，这里推荐一种解决方案，不一定是最好的，仅供参考。

如果我们有个列表有很多页的数据，下拉刷新的时候默认加载第一页的数据，上拉加载的时候，加载下一页的数据，如果这两个请求的`cacheKey`给一样肯定是不行的，数据必然就覆盖了，所以下拉刷新和上拉加载的`cacheKey`一定不一样，下拉好说，当时上拉的时候如果每一页都是一样的`cacheKey`，那么数据又会发生覆盖了，所以上拉的`cacheKey`最好还加上页数相关的字段，确保每页的`cacheKey`也不一样。那么问题来了，这么做有必要吗？

其实我在我们自己的项目中，对于上拉加载这种操作，我是不给缓存的，为什么呢？作为21时间的app，还有单机的app吗？几乎没有，那么缓存的目的和初衷是什么，是给用户一个良好的体验，当我们网路不好的时候，或者网络异常的时候，能展示一部分最近看过的数据，不至于白屏什么都没有，但是有必要把他看过的所有数据都缓存下来么。肯定没什么用，第一占内存空间，第二用户根本不会看，何必做这种费力不讨好的事呢。所以最后的结论就是我们只需要在下拉刷新的代码中缓存第一页的数据就行，上拉加载的数据不用理会。

详细的代码在 [Demo --> supercache包 --> NewsTabFragment类](https://github.com/jeasonlzy/okhttp-OkGo/blob/master/demo/src/main/java/com/lzy/demo/supercache/NewsTabFragment.java) 中，或者看如下截图，部分代码如下：
![](https://ws1.sinaimg.cn/large/006tKfTcly1fh13b5valjj318e0oan2a.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcly1fh13bz05cqj315i0t4afr.jpg)