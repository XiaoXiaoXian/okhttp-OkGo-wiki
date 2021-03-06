## OkUpload主要功能
- 结合OkGo的request进行网络请求，支持与OkGo保持相同的配置方法和传参方式
- 上传只能使用有请求体的方法
- 该上传管理为简单管理，不支持断点续传或分片上传，只是简单的将所有上传任务使用线程池进行了统一管理
- 上传任务存入数据库，按tag区分，意义与OkUpload的tag一致
- 默认同时上传数量为1个,该数列可以在代码中配置修改

### 注意！！！
> OkGo与OkUpload的区别就是，OkGo只是简单的做一个上传功能。OkUpload是具有上传管理功能，可以让很多任务有序上传，并且能记录上传状态

OkUpload只是对OkGo功能的一个扩展升级，如果你不需要使用上传管理的功能，那么就不需要导入该包。

很多人经常问，上传可以不可以支持断点/分片上传，首先这个技术是可以实现的，但是问题是，这个很大程度依赖于服务端怎么写，而不同的服务端实现逻辑和思路都不一样，okgo就一个网络框架，不能约束服务端代码，所以这个功能技术上能做到，但是框架不会去做，如果确实需要，请自行与服务端商量方案，自行实现。

[这里有个issue，解释了为什么不做断点上传，点击查看](https://github.com/jeasonlzy/okhttp-OkGo/issues/205)。

废话不多说，看看怎么用。

## 全局配置
直接上代码，如图：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/r7amu.jpg)
1. 可以设置同时上传的任务数量，如果不设置，默认就是1个，改方法只在第一次调用时生效，以后无效
3. 可以设置所有任务的监听，比如全部任务开始的时候做点事情，全部完成后做点事情。
4. 记得监听可以取消，否则可能会发生内存泄露。

此外，OkUpload不仅有全局相关的设置，还有对全部任务的同时操作能力。
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/9p1zf.jpg)
例如如下方法：
1. **startAll()**：开始所有任务。
2. **pauseAll()**：将全部上传中的任务暂停，注意，上传的暂停并不是正真的暂停，虽然确实不会上传了，但是下次继续上传的话，进度是重头开始的，也就是上传其实是不支持断点的。
3. **removeAll()**：移除所有任务，无论这个任务是在上传中、暂停、完成还是其他任何状态，都可以直接移除这个任务。
4. **removeTask()**：根据tag移除任务
5. **getTaskMap()**：获取当前所有上传任务的map
6. **getTask()**：根据tag获取任务
7. **hasTask()**：标识为tag的任务是否存在

## 基本上传方法
我们首先看看一个最基本的上传任务是怎么写的，先给代码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/c1i1o.jpg)
以上的几行代码就完成了文件的上传，可以开始，暂停，移除，重新上传等，是不是很简单，关于以上代码，详细说明如下：

1. 构建一个上传请求Request，这个构建方法和OkGo是一样的，params参数和headers参数是只是演示使用，一切OkGo的使用方法，这里都是一样的。**需要注意的是最后一定需要传递一个Converter对象，表示上传成功后，如何解释上传的结果，不传的话，会报错**。
2. 构建上传任务，使用OkUpload中的request方法，传入一个tag和我们上一步创建的request对象，创建出上传任务，其他的方法我们下文在细讲。
3. 启动任务，我们已经得到了UploadTask任务对象，那么简单调用start启动他就好了，同时他还支持这么几个方法：
   - **start()**：开始一个新任务
   - **pause()**：将一个上传中的任务暂停
   - **remove()**：移除一个任务，无论这个任务是在上传中、暂停、完成还是其他任何状态，都可以直接移除这个任务
   - **restart()**：重新上传一个任务。也就是从头开始重新上传该文件。
   
上传任务需要一个回调，我们上面随意注册一个打日志的回调，代码如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/tlk27.jpg)
控制台的输出如下：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/ravyu.jpg)

这个log要做以下几点说明：
1. onStart()方法是在上传请求之前执行的，所以可以做一些请求之前相关的事，比如修改请求参数，加密，显示对话框等等。
2. 上传文件的时候，请求体会打印一句话：`maybe [binary boby], emitted!`，这句话表示当前的数据是二进制文件，控制台没法打印也没必要打印出来，所以不用打印了。很多人经常拿着这个log告诉我出问题了，搞的我真是欲哭无泪，所以不要认为是bug，这是正常的。
4. 上传完后，最后会调用onFinish()，不过我设计成在调用onFinish()之前，还会额外调用一次onProgress()方法，这样的好处可以在onProgress方法中捕获到所有的状态变化，方便管理。

## 上传API介绍
以上的介绍只是简单的说了一下使用方法，那么OkUpload究竟提供的api一共有哪些呢，我们来看看，保证满足你的任何需求。

![](http://7xss53.com1.z0.glb.clouddn.com/markdown/djkqy.jpg)

Request的构建详细参考OkGo的用法，这里重点介绍DownloadTask的构建，这里面的方法一个个介绍：
1. **request()**：静态方法创建UploadTask对象，接受两个参数，第一个参数是tag，表示当前任务的唯一标识，就像介绍中说的，**所有上传任务按照tag区分**，不同的任务必须使用不一样的tag，否者进度会发生错乱，如果相同的上传url地址，如果使用不一样的tag，也会认为是两个上传任务，不同的上传url地址，如果使用相同的tag，也会认为是同一个任务，导致进度错乱。切记，切记！！
2. **priority()**：表示当前任务的上传优先级，他是一个int类型的值，只要在int的大小范围内，数值越大，优先级越高，也就会优先上传。当然也可以不设置，默认优先级为0，当所有任务优先级都一样的时候，就会按添加顺序上传。
3. **extra()**：这个方法相当于数据库的扩展字段，我们知道我们是需要保存上传记录信息的，而我们这个框架是保存在数据库中，数据库的字段都是写死的，如果用户想在我们的上传数据库中保存自己的数据就做不到了，所以我们这里提供了三个扩展字段，允许用户保存自己想要的数据，如果不需要的话，也不用调用该方法。
4. **register()**：这是个注册监听的方法，我们既然要上传文件，那么我们肯定要知道上传的进度和状态是吧，就在这里注册我们需要的监听，监听可以注册多个，同时生效，当状态发生改变的时候，每个监听都会收到通知。当然如果你只是想上传文件，不关心他的回调，那么你不用注册任何回调。

细心你一定会发现我有个方法没有讲，那就是`save()`方法，这个方法很重要，重要到我要单独拿出来说他。我们先看看这个方法的源码：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/rwxzg.jpg)
简直简单到不行，就一行代码，就是把当前的progress进度对象，保存到数据库。

想想，如果数据库中没有你这条记录，然后你还开始上传，我如何去更新这个记录，一旦你这么做，OkUpload就会抛出一个异常如下，告诉你必须在start()前，先调用save()方法。所以我们知道了，save()用在什么地方呢？
1. 如果你是第一次上传这个标识为这个tag的任务，那么你一定要调用save()方法，先将该数据写入数据库。
2. 如果你对标识为tag的任务进行了一些参数的修改，比如修改了extra数据等，也必须调用save()方法，更新数据库。

## 结束
其实OkUpload还有很多用法，但是和OkDownload几乎一模一样，你只需要看着OkDownload的文档，把Down换成Up就是OkUpload的文档，所以，如果还有其他疑问，可以看看OkDownload的文档。