## JsonCallback自定义
这篇文档主要讲解JsonCallback的自定义方法与原理，该文档以谷歌的Gson框架来讲，如果你用的是fastJson，其实原理也大同小异。

## 数据格式
讲解之前，我们先看看一些服务端返回的格式类型，一般服务端返回的数据格式都是这样的，我相信这4种数据格式包含了99%以上的业务。

- 数据类型A-最外层数据类型是`JsonObject`，data 数据也是`JsonObject`
```java
{
	"code":0,
	"msg":"请求成功",
	"data":{
		"id":123456,
		"name":"张三",
		"age":18
	}	
}
```

- 数据类型B-最外层数据类型是`JsonObject`，data 数据是`JsonArray`
```java
{
	"code":0,
	"msg":"请求成功",
	"data":[
		{
			"id":123456,
			"name":"张三",
			"age":18
		},
		{
			"id":123456,
			"name":"张三",
			"age":18
		},
		{
			"id":123456,
			"name":"张三",
			"age":18
		}
	]
}
```

- 数据类型C-没有固定的 msg、code 字段格式包装，服务器任意返回对象
```java
"data":{
	"id":123456,
	"name":"张三",
	"age":18
}	
```

- 数据类型D-最外层数据类型是`JsonArray`，内部数据是`JsonObject`
```java
[
	{
		"id":123456,
		"name":"张三",
		"age":18
	},
	{
		"id":123456,
		"name":"张三",
		"age":18
	},
	{
		"id":123456,
		"name":"张三",
		"age":18
	}
]
```

那么我们可以严格的按照服务端的格式来定义，如下，将code和msg都定义在javabean中。
- 数据类型A定义方式
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/e2itp.jpg)
- 数据类型B定义方式
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/yv73f.jpg)
- 数据类型C和D定义方式一样，这里就按照服务端的数据一一对应好字段就行了，没什么其他的妙招
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/v6pj6.jpg)

## Gson解析
如果你使用过Gson，那么你一定不会陌生如何解析以上几种数据类型，解析代码如下：

- 数据类型A，B，C，都是最正常的解析方式
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/3m2tp.jpg)
- 数据类型D，因为他是集合类型，我们不能传个List.class让框架去解析数据，框架怎么知道我们集合的泛型是什么呢，毕竟大家都知道java的泛型是擦除的，那么怎么办呢，Gson还提供了另一个方法，我们可以传一个Type，Type是个接口，我们的Class对象就是实现了Type接口的，解析代码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/caprh.jpg)
- 我们都知道网路的数据是以流的形式进行传递的，我们再解析数据的时候，把流先解析成JSON字符串，然后在通过Gson解析成对象这个过程是没必要的，如果大量的请求发生，是不是就有大量的数据要被转成JSON字符串，数据量小还行，一旦较大，连请求都发生OOM了，这并不是我们希望看到的，那么有没有办法能直接把流解析成对象呢，不要转成中间的字符串对象，当然是可以的，Gson框架也为我们提供了这个方法，解析代码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/8csma.jpg)

你一定会疑惑为什么集合类数据要传Type，这个Type为什么要这么得到，得到的这个Type又是什么东西，Gson又是如何根据Type来得到正确的数据的，这个就很复杂了，要从Java的泛型擦除机制说起，我就不展开说了，详细的我给几篇参考链接，讲的不错的。

- [泛型相关周边信息获取](http://blog.csdn.net/harvic880925/article/details/50085595)，这个人的五篇文章，关于泛型的讲解无比的详细，强烈推荐。
- [Java能够运行时获取到泛型的真实类型的原则](http://rednaxelafx.iteye.com/blog/586212)
- [Java为什么要添加运行时获取泛型的方法](https://www.zhihu.com/question/36645143)
- [Java 泛型进阶](http://www.jianshu.com/p/4caf2567f91d)
- [Java 泛型详解](https://juejin.im/post/584d36f161ff4b006cccdb82)

## Callback中解析

上面讲了一堆废话，如果你熟悉Gson解析的话，上面那一大堆都不用看，因为实在是太基础了，我只是把我们常见的数据格式和对应的解析方法给了个总结而已，接下来进入自定义环节。

通过上面的基础结束，我们了解到，解析数据需要一个class或者type，我们用构造传递进去就可以了，如下：
![](https://ws1.sinaimg.cn/large/006tNbRwly1fgi79fq6y0j31480p00wf.jpg)

自定义的JsonCallback写完了，我们需要使用它，使用的代码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/2gx2m.jpg)

重要的事情说三遍：
### 其实到这里`JsonCallback`就全部自定义结束了
### 其实到这里`JsonCallback`就全部自定义结束了
### 其实到这里`JsonCallback`就全部自定义结束了

就这么简单的一个`JsonCallback`就能无论你什么网络数据都能解析成功并返回，而且以上的代码是通用的，可以直接复制，到哪都能用，是不是很简单，Retrofit默认的GsonConvert的核心也就是上面这几行代码。

如果上面这点代码还是有问题，强烈建议不要自定义`Callback`了，直接用框架内置的`StringCallback`，写法极其简单，就这样，再次强调第一行的泛型一定要写`String`，才能用`StringCallback`，否则无法编译通过：
![](https://ws2.sinaimg.cn/large/006tKfTcly1fgsxtabtmcj310i0h00v7.jpg)

## JsonCallback优化

#### 高能预警！！！警告！！！
- 如果你还没工作，或者刚工作不久，对泛型解析不熟悉，那么接下来的代码不要继续看
- 以下的代码都是和业务逻辑挂钩的，不同的项目写法均不一样，如果你只是要个通用的解析，以下代码也不必要看
- 如果你只想简单的用一下，以下的代码也不要看
- 如果你想挑战自己，继续深入，我们就一起往下走
- 如果你发现看完了，一头雾水，并不知道在干嘛，那么就用上面最简单的`JsonCallback`，效果依然很好

来继续，我们要优化什么呢？我明明已经在JsonCallback中写了泛型Login，我不想还要构造方法中传递进去，我感觉很多余，没有必要，行，那我们先上代码，改完后的样子如下：
![](https://ws3.sinaimg.cn/large/006tNbRwly1fgi7a0rme4j313q0h20vg.jpg)

调用代码如下：
![](https://ws2.sinaimg.cn/large/006tNbRwly1fgi7a97zp6j31280iumzj.jpg)

看起来简单干净多了，没有额外的构造方法，不需要传递type或者class，就改了两行很神奇的代码，这两行代码的作用就是**解析出当前类的父类上的泛型的真实泛型**，注意这个黑体加粗的字，如果有不明白的，参考前面给出讲解泛型的几篇文章的链接，特别是第一篇，好好看看，就明白了。

既然这么简单有没有什么弊端呢，当然有的。有些人喜欢把通用的请求操作放在基类写好，就像这样：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/w5i9a.jpg)

还有的希望继承JsonCallback再额外做些操作，比如这样：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/m0f1p.jpg)

当你在使用BaseModel或者SomeCallback的时候，你会发现，不封装前代码好好的，就这么封装一下，什么都不干，代码居然就挂了，报的异常就是这样：
![](https://ws3.sinaimg.cn/large/006tKfTcly1fgsyrqosr1j314u070q6d.jpg)

这是为什么呢，根本原因就是使用反射获取类上泛型的真实类型是有局限性的，只能获取一层继承结构的数据，层次再多，获取到的数据类型他就是T，不是什么真实类型，Gson无法解析，所以就挂了，为什么就获取不到了呢，详细还是参考前面给出讲解泛型的几篇文章的链接，里面都有很清晰的讲到。

所以，最稳妥的办法，还是不要这么优化了，或者兼容一下写法，如下：
![](https://ws3.sinaimg.cn/large/006tKfTcly1fgsys015nij31460ykgqj.jpg)

是不是很失望，当然简单有简单的好，复杂有复杂的好，万物都是相生相克，物极必反就是这个道理。

## JsonCallback继续优化
居然还有优化，我只能呵呵！！

场景：
大多数情况，服务器返回的数据格式都是有个统一的规范的，就像我写的数据类型A和B那样，那么我们可以使用泛型，分离基础包装与实际数据，这样子需要定义两个javabean，一个全项目通用的`LzyResponse`，一个单纯的业务模块需要的数据。

这种方法只适合服务器返回有固定数据格式的情况，如果服务器的数据格式不统一，那就放弃治疗吧，不建议使用该种方式，具体的定义方法如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/7grqq.jpg)

在okgo提供的demo中是按照这种方式实现的，对于这种方式，我们在创建`JsonCallback`的时候，需要这么将`LzyResponse<ServerModel>`或`LzyResponse<List<ServerModel>>`整体作为一个泛型传递，相当于传递了两层，泛型中又包含了一个泛型，使用方法如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/z57j2.jpg)

那么`JsonCallback`中就需要做改动了，详细的原理就不说了，直接上代码，详细看注释：
![](https://ws1.sinaimg.cn/large/006tKfTcly1fgsys9yzykj315s1lcn7i.jpg)

## 总结
分析优化前与优化后的写法，很明显优化前要方便、简单、通用，优化后更麻烦，还难以理解。那我在demo中为什么不使用优化前的方式呢？

先做如下分析，如果服务端返回了这么个数据，这种数据应该是经常返回的。

```java
{
	"code":300,
	"msg":"用户名或密码错误",	
}
```

优化前方式，很明显，这样的的数据应该要回调`onError`的，但是我们在`convertResponse`中只是解析了数据，并不知道数据中的`code`码`104`是表示错误数据的，我们仍然会解析数据并返回，这就导致了会回调`onSuccess`，之后的逻辑我们需要在`onSuccess`中判断`code`码，给予用户错误提示或者做成功跳转的逻辑。这样做的结果是，把无论正确的还是错误的数据，都交给了`onSuccess`处理，在使用上不太友好，逻辑分层混乱。

优化后的方式，这种方式不仅可以正确的解析数据，而且能在解析的过程中判断错误码，并根据不同的错误码抛出不同的异常（这里抛出异常并不会导致okgo挂掉，而是以异常的形式通知okgo回调`onError`，并且将该异常以参数的形式传递到`onError`，这样做后，在`onSuccess`中处理的一定是成功的数据，在`onError`中处理的一定是失败的数据，达到了友好的逻辑分层。

正因为优化后可以很方便的处理业务逻辑，而我也推荐使用优化后的写法，但是这种写法是依赖于业务中定义的数据结构的，不同的项目定义的数据结构都不一样，无法封装到库中，所以用demo的形式提供示例代码，根据需要自己修改。同时我也推荐服务端对返回的数据做一次统一包装，就像上面的示例一样，用code和msg包一层。