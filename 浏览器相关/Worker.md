### Web Worker官方解释

```
Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面

一个worker是使用一个构造函数创建的一个对象(e.g. Worker()) 运行一个命名的JavaScript文件 

这个文件包含将在工作线程中运行的代码; workers 运行在另一个全局上下文中,不同于当前的window

因此，使用 window快捷方式获取当前全局的范围 (而不是self) 在一个 Worker 内将返回错误
```

这样理解下：

- 创建Worker时，JS引擎向浏览器申请开一个子线程（子线程是浏览器开的，完全受主线程控制，而且不能操作DOM）
- JS引擎线程与worker线程间通过特定的方式通信`MessageChannel`（postMessage API，需要通过序列化对象来与线程交互特定的数据）

所以，如果有非常耗时的工作，请单独开一个Worker线程，这样里面不管如何翻天覆地都不会影响JS引擎主线程， 只待计算出结果后，将结果通信给主线程即可，perfect!

而且注意下，**JS引擎是单线程的**，这一点的本质仍然未改变，Worker可以理解是浏览器给JS引擎开的外挂，专门用来解决那些大量计算问题。

### Web Worker与Shared Worker

- WebWorker只属于某个页面，不会和其他页面的Render进程（浏览器内核进程）共享
  - 所以Chrome在Render进程中（每一个Tab页就是一个render进程）创建一个新的线程来运行Worker中的JavaScript程序。
- SharedWorker是浏览器所有页面共享的，不能采用与Worker同样的方式实现，因为它不隶属于某个Render进程，可以为多个Render进程共享使用
  - 所以Chrome浏览器为SharedWorker单独创建一个进程来运行JavaScript程序，在浏览器中每个相同的JavaScript只存在一个SharedWorker进程，不管它被创建多少次。

看到这里，应该就很容易明白了，本质上就是进程和线程的区别。SharedWorker由独立的进程管理，WebWorker只是属于render进程下的一个线程



### Service Worker

一种专门用于浏览器与网络或者缓存的中间代理。

Service workers 本质上充当 Web 应用程序、浏览器与网络（可用时）之间的代理服务器。这个 API 旨在创建有效的离线体验，它会拦截网络请求并根据网络是否可用来采取适当的动作、更新来自服务器的的资源。它还提供入口以推送通知和访问后台同步 API。

服务器和浏览器的中间人，充当代理服务器这一能力（通过拦截所有请求实现），实现HTTP缓存无法实现的功能:**离线应用**。因为在HTTP缓存策略下，如果一个资源过了服务器规定的到期时间，则必须要发起请求，一旦网络连接有问题，整个网站就会出现功能问题。而在service worker控制下的缓存，能够在代码中发现网络连接问题并直接返回缓存的资源。这种方式返回的响应对于浏览器来说是透明的，它会认为该响应就是服务器发送回来的资源。

 PWA 的离线缓存特性主要是依赖 Web 提供的 Service Worker 机制和 Cache API 来配合实现的

还可以解决Web应用推送、后台长计算等问题，本质上是一个全新的JavaScript线程，运行在与主Javascript线程不同的上下文。service worker线程被设计成完成异步，一些原本在主线程中的同步API，如`XMLHTTPRequest`和`localStorage`是不能在`service worker`中使用的，

主线程是负责dom的线程，servier work线程是无法访问dom的，ui线程只能有一个，而保证ui线程不卡顿的原则是尽量不在ui线程中做大量计算和同步IO处理。

> ```js
> sw线程能够用来和服务器沟通数据（service worker的上下文内置了fetch和Push API）
> 能够用来进行大量复杂的运算而不影响UI响应。
> 它能拦截所有的请求（通过监听fetch事件，任何对网络资源的请求都会触发该事件），并内置了一个完全异步的存储系统（Caches属性，完全异步并能存储全部种类的网络资源），这是它能精细化控制缓存的关键。
> ```

service work的缓存关键在于监听监听fetch事件和管理Cache资源

##### Service Worker的特点

Service Worker本质上就是一个Web Worker，因此它具有Web Worker的特点：

1.无法操作DOM；
2.脱离主线程；3.独立上下文；

