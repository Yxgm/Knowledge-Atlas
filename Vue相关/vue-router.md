# Vue-router

>  一句话总结：**封装了一个`全局混入`，定义了两个挂载在`原型`上的`变量`，注册了两个`组件`。**

`hash`模式和`history`模式是一样的API设计，把`hash`的`#`截取掉，和`history`做相同的逻辑处理，不再使用`hashchange`监听事件。两个模式都是使用的浏览器H5的`history`新特性来实现的路由。将路由变成响应式的。

`router.push`的时候，会先调用`replaceState()`，并且在把状态存入，随后在调用`pushState()`也就是向浏览器的history栈中，推入一个状态，只是增加了一个历史记录，并且不会刷新页面，`locaiton`的`url`改变，同时也改变router中维护的唯一变量`currentpath`，`currentPath`为响应式（详细请看下文）。

在初始化路由的时候，也初始化了`history`栈结构的监听器，前进或者后退的时候，都会触发视图的重新渲染（也是路由响应式的那套），**这个监听器是为history.go()等API服务的，和我们点击跳转的路由并没有关系。**

> 至于为什么两种模式为什么都使用`history`这一套逻辑，请看下文

请看这条记录`commit` [feat: enhance hashHistory to support scrollBehavior](https://link.segmentfault.com/?enc=Lwk1%2BXCDlRJ%2FWKuMDMnj1Q%3D%3D.lu%2B9Rcn%2FikaqpZEMC64VTEiaZFWV%2Fu1fJ9xR2yGgACJjIZGJyDw%2F8ZEnH8UtLK4XZAfvxOlZ%2FHFPZxeoed2cQXRWZZYYv186dooKw%2FIgrouTkQouZtdTolYDV09cmDKw)是为了支持[scrollBehavior](https://link.segmentfault.com/?enc=cY30LvSYxV6EVilBJU66Ww%3D%3D.U69Nyo0Zftdu%2FluomExNelAubRz7uEhbP3AmYq%2B0GyGmjXiYD7D%2F5623pxhMTUsSWP166F4%2B5mFZO%2BM1ZVjnNZaNT118soaswNlYZmfsrt3%2BJc1%2B4hoDdivpkVSXhIqTgkhiONDRrftS0Rr8NCyR%2FQ%3D%3D)

具体原因是`window.history.pushState`可以传参`key`过去，这样每个`url`历史都有一个`key`，用`key`保存了每个路由的位置信息

1. 初始化路由的时候，没有状态，所以说，我维护一个状态机，将当前的location的url包装成一个对象，

   ```js
    {
             back: null,
             current: currentLocation.value,//当前路径
             forward: null,
             position: history.length - 1,// 0位置
             replaced: true,//是否是replace操作
             scroll: null,
    },
   ```

   当我们操作浏览器的前进或者后退的时候，通过positoin属性，来判断当前的操作，在调用原生的replaceState，将此状态，url replace到history的第一条记录上。

# 大彻大悟：



`vue-router`源码中，维护着一整套的状态机制，存在了`history`的栈结构中，并没有在`pushState`的时候执行挂载函数等等（自己先入为主的思想跑偏了），**`vue-router`在将状态推入栈中，仅仅是做了一个历史记录，并没有去渲染组件（原来自己理解的渲染函数），实际上是改变了`url`的同时，也改变`currentRoute`，监听`currentRoute ` ，并且做出渲染。**

```javascript
 vue.watch(router.currentRoute, () => { //就是监听currentRoute的变化，一旦变化，就更新视图
     // refresh active state
 		 refreshRoutesView();
		 api.notifyComponentUpdate();
		 api.sendInspectorTree(routerInspectorId);
		 api.sendInspectorState(routerInspectorId);
});
以前的router：
		route 在 vue-router 初始化时变成了一个响应式对象，所以会触发 route 的 getter，收集当前的渲染watch -er。当路由跳转后，会触发其 setter，重新运行 render 函数更新视图，Vue-rouer自己管理着一套状态机，拥有着currentRoute这个属性，也就是当每次我们通过vue封装的push操作时，当前状态会进入history的栈结构中，记录着历史记录，同时，会更新router身上的currentRoute属性，这就触发了setting，就会根据路径去寻找到视图的渲染函数，之后在执行路由守卫等等操作

现在的router：
		直接使用watch监听
```



### 自我理解：

先更改路由的路径，然后在记录状态，push到栈中，在更改路由的currentPath值，触发setting，promise链式调用，然后再渲染视图。

## 路由守卫：

**发布订阅模式，promise的链式调用来实现的，和axios的拦截器类似**，路由守卫可以写成异步的。

首屏加载要在bundle size和网络请求的数量中制衡，让这两者均衡，性能达到最优

`addRoute()`动态添加路由

## 完整的导航解析流程：

- 导航被触发。
- 在失活的组件里调用离开守卫。
- 调用全局的 beforeEach 守卫。
- 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
- 在路由配置里调用 beforeEnter。
- 解析异步路由组件。
- 在被激活的组件里调用 beforeRouteEnter。
- 调用全局的 beforeResolve 守卫 (2.5+)。
- 导航被确认。
- 调用全局的 afterEach 钩子。
- 触发 DOM 更新。
- 用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数。

详解可鉴花帽子的博客