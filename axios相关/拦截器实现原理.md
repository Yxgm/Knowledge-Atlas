#### 1. Axios构造函数

```js
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  //这里声明了请求拦截，和相应拦截
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}
```

##### axios的interceptors属性是一个对象，而request和response都是是InterceptorManager对象，没有区别，只是先后执行的问题

#### 2. InterceptorManager构造函数

```js
function InterceptorManager() {
  this.handlers = [];
}
并且原型上拥有use，eject，foreach方法
1. use方法：只是把两个拦截器的回调函数，放入到拦截器实例中的handlers数组中并且返回一个ID标识
2. eject方法：将拦截器中的回调函数置空，取消拦截
3. foreach方法：重写了原来有的foreach，但是内部差不多，有一步判空的条件语句，并且默认遍历是自己实例上的handlers数组。

InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};

InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};

InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function (handler) {
    if (handler!== null) {
      fn(handler);
    }
  });
};
```

#### 3. Axios函数中的拦截器片段代码

```js
 var chain = [dispatchRequest, undefined];
 var promise = Promise.resolve(config);

 this.interceptors.request.forEach((interceptor) =>
  chain.unshift(interceptor.fulfilled, interceptor.rejected)
 );//请求拦截器

 this.interceptors.response.forEach((interceptor) =>
  chain.push(interceptor.fulfilled, interceptor.rejected)
 );//响应拦截器

 //自我理解：不暴露interceptor实例上的handler属性，所以interceptor内部自己封装了foreach方法，将handler数组封装在了内部
 while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
 }
 return promise;
```

chain：链数组，dispatchRequest是真正发出请求的函数。

请求拦截器执行顺序，后添加的先执行的问题

>  是因为每一次添加都是unshift进去的，都是放在了chain数组的前面，所以才会出现这个情况

响应拦截器，正常顺序执行问题

> 因为chain是将响应拦截器push到数组的，所以正正常顺序

#### 4. 纠结的问题

请求拦截器失败回调函数，什么时候才执行？ （现在并没有研究明白），但是依旧源码，自己总结如下：

就算是参数错误，url拼写不正确的条件下，也不会执行请求拦截器失败的回调函数,源码中是将config包裹了Promise（promise.resolve(config)），然后promise.then(chain.shift(), chain.shift())链式调用，请求拦截成功的函数无论请求情况如何，都会执行一遍（明确的来说，这个函数，是在请求发出之前做的一些事情，无论请求成功还是失败）。

之后正常的执行dispatchRequest函数（发出网络请求），如果请求失败了

```js
 PROMISE = promise.then(dispatchRequest,undefined)
```

这时dispatchRequest函数返回了Promise.reject（reason），所以说PROMISE就是dispatchRequest返回的promise

```js
PROMISE.then("response.fulfilled","response.rejected")
```

所以响应拦截器的rejected函数执行

