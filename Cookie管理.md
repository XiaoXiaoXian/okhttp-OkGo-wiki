## 科普概念
具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制是需要借助于cookie机制来达到保存标识的目的，所谓session保持会话，对于客户端来说，就是cookie的自动管理。  

cookie的内容主要包括：名字，值，过期时间，路径和域。路径与域一起构成cookie的作用范围。若不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就消失。这种生命期为浏览器会话期的cookie被称为会话cookie。会话cookie一般不存储在硬盘上而是保存在内存里，若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。存储在硬盘上的cookie可以在不同的浏览器进程间共享，比如两个IE窗口。而对于保存在内存里的cookie，不同的浏览器有不同的处理方式。  

session机制。session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为sessionId，也就是请求头是否有cookie），如果已包含则说明以前已经为此客户端创建过session，服务器就按照sessionId把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含sessionId（也就是不携带cookie的请求头），则为此客户端创建一个session并且生成一个与此session相关联的sessionId，sessionId的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个sessionId将被在本次响应中通过set-cookie响应头返回给客户端保存。客户端检查到这个响应头后，根据需要就会保存这个sessionId，下次在请求交互过程中便可以自动的按照规则把这个标识发送给服务器。这样就完成了session的保持。  

## okgo配置
对于okgo来说，okgo完全遵循了http协议，所以，如果你的服务端的`session`是按照`set-cookie`头返回给客户端，并且希望在下次请求的时候自动带上这个cookie值，那么你只需要在okgo初始化的时候添加这么一行代码：
![](http://7xss53.com1.z0.glb.clouddn.com/markdown/etebq.jpg)

以上三种方式是okgo默认内置的三种cookie管理策略，任选其一就可以了，其中
- **SPCookieStore**：使用SharedPreferences保持cookie，如果cookie不过期，则一直有效
- **DBCookieStore**：使用数据库保持cookie，如果cookie不过期，则一直有效
- **MemoryCookieStore**：使用内存保持cookie，app退出后，cookie消失

到此，以后所有的请求不需要你有任何的额外代码，就只要上面这一行，就完成了所有请求的cookie与session全自动管理。就是这么的强大！

## 自定义cookie管理策略
如果你的业务中有些特殊的cookie逻辑，那么可以自定义cookie管理策略，有两种方法：
- 按照okhttp原生提供的方法，实现`CookieJar`接口，传入`OkHttpClient.Builder`中即可
- 按照okgo内置的方式，实现`CookieStore`接口，同样传入`OkHttpClient.Builder`中即可
- 详细代码可以参考默认的实现

## 注意事项
cookie是有作用域的，他绑定的是url对应的host，比如你的请求两个接口，一个是 www.domain1.com 一个是 www.domain2.com 那么这个时候，domain1所具有的cookie是不会自动在domain2中携带的，如果一定要让他们两个都可以用，那么需要手动设置，方法请看下面的cookie交互。


## cookie交互
如果你需要与webview交互，okgo需要向webview传递cookie，或者webview需要向okgo传递cookie，或者你需要把okgo管理的cookie做一些处理，那么对cookie的操作方式如下，使用方法依然极其简单：

查看okgo管理的cookie中，某个url所对应的全部cookie
```java
CookieStore cookieStore = OkGo.getInstance().getCookieJar().getCookieStore();
HttpUrl httpUrl = HttpUrl.parse("http://server.jeasonlzy.com/OkHttpUtils/method/");
List<Cookie> cookies = cookieStore.getCookie(httpUrl);
showToast(httpUrl.host() + "对应的cookie如下：" + cookies.toString());
```

查看okgo管理的所有cookie 
```java
CookieStore cookieStore = OkGo.getInstance().getCookieJar().getCookieStore();
List<Cookie> allCookie = cookieStore.getAllCookie();
showToast("所有cookie如下：" + allCookie.toString());
```

手动向okgo管理的cookie中，添加一些自己的cookie，那么以后满足条件时，okgo就会带上这些cookie
```java
HttpUrl httpUrl = HttpUrl.parse("http://server.jeasonlzy.com/OkHttpUtils/method/");
Cookie.Builder builder = new Cookie.Builder();
Cookie cookie = builder.name("myCookieKey1").value("myCookieValue1").domain(httpUrl.host()).build();
CookieStore cookieStore = OkGo.getInstance().getCookieJar().getCookieStore();
cookieStore.saveCookie(httpUrl, cookie);
```

手动把okgo管理的cookie移除
```java
HttpUrl httpUrl = HttpUrl.parse("http://server.jeasonlzy.com/OkHttpUtils/method/");
CookieStore cookieStore = OkGo.getInstance().getCookieJar().getCookieStore();
cookieStore.removeCookie(httpUrl);
```