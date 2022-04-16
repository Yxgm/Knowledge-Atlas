#### 响应式

Vue.js是采用数据劫持结合发布订阅者模式的方式，通过Object.defineProperty（）来劫持各子属性的setter，gettter，当数据访问时，也就是触发了getter方法，在调度中心中，我们把该数据的副作用收集起来（也就是订阅者在调度中心订阅），之后一旦数据改变时，触发setter方法，对应的dep对象调用notify函数，把所有的副作用函数都在执行一遍，从新渲染视图，就相当于调度中心发布消息给订阅者，触发响应的监听来渲染视图。

####  首先需要了解的几个对象

1. `Observer`: 这里的主要工作是递归地监听对象上的所有属性，在属性值改变的时候，触发相应的`Watcher`。我称它为`watcher Manager`

2. `Watcher`: 观察者，当监听的数据值修改时，执行响应的回调函数，在`Vue`里面的更新模板内容。

3. `Dep`: 链接`Observer`和`Watcher`的桥梁，每一个`Observer`对应一个`Dep`，它内部维护一个数组，保存与该`Observer`相关的`Watcher`。`Dep.target`关健属性

`dep`和`watcher`有双向绑定的关系:

> `render watcher`中有所有响应式数据的`dep`对象；
>
> 而数据对应的`dep`对象中，`sub`数组中有`render watcher`，还可能有`computed watcher`

#### 两个核心概念

1. 依赖收集：读取数据时（`get`操作），将依赖自身的watcher函数（**感觉说成副作用函数会更好理解**）收集到自己dep中，同时 `watcher `也会保存所有 `watcher` 依赖数据

   🤫：在我看来，依赖收集，是依赖自己把自己收集起来，new Watcher的时候 触发函数执行或者读值，然后把被数据当作副作用函数收集到自己dep中

2. 派发更新：设置数据时（`set`操作），将自身的副作用函数（dep上的subs数组）全部重新执行一遍，更新vdom数据，patch算出最小修改范围、修改真实dom

> 所以：我们要确定一个变量是怎么收集依赖和派发更新



## `render Watcher`：

函数意义：`render watcher` 即是`vnode._update(vm._render())`，组件重新构成vdom，patch，挂载

初始化：

> `initData`调用了observe函数，并且新建observe对象，挂载data对象的`__ob__`属性上，`Observer` 是 ``data ` 的观察者，实例上的`data`属性，添加一个`__ob__`属性，代表该`data`已经被`observe`监听。这个是为data服务的

副作用函数收集阶段：（依赖收集）

一顿初始化完毕后执行  `new Watcher(vm, vm.$options.render, () => {}, true)`

> `render`函数执行，读取数据，在 new Watch阶段就触发render函数，在render中，又将watcher收集。
>
> `all`的数据getter操作收集依赖



## `user Watcher `：

函数意义：相当于自定义的watcher，监听目标改变时，触发回调（user watcher）

副作用函数收集阶段：（依赖收集）：

> 1. 先走`$watch`，后到 `new watch`目的是配置watcher，初始化watcher， user属性设置为true；
> 2.  `user watcher`的`option`选项只有`user`为`true`，`new Watch`之后，会走`this.get()`，将  **c位给user `Watcher`**，接着执行 this.getter.call(vm, vm) ，对 vm 的属性进行层级访问, 触发 data 中目标属性的 get 方法, 触发属性对应的 [dep.depend](https://github.com/vuejs/vue/blob/2.6/src/core/observer/dep.js#L31) 方法, 进行依赖收集
> 3. 在我们写完了watch监听，相当于vue主动的get了一下监听的属性
> 4. ⚠️：**这里`user watcher`与众不同的是这里是读值，而不是函数调用**
>
> ```js
> const watcher = new Watcher(vm, expOrFn, cb, options) 
>  this.value = this.lazy
>       ? undefined
>       : this.get() //user watcher 走后者
>   }
> 
>   get () {
>     pushTarget(this)//几种watcher轮着占c位
>     let value
>     const vm = this.vm
>     value = this.getter.call(vm, vm)
>     //调用对应cb，user watcher是遍历读值，另外两种执行函数
>     popTarget()
>     return value
>  }
> ```
>
> 4. 　`user Watcher`就会被当前属性收集到自己的`dep`中，

派发更新：监听属性发生改变时，user watcher执行，

其他的配置项：

1. immediate：immediate判断，为true的话，invokeWithErrorHandling执行，也就是cb直接执行一遍

```js
if (options.immediate) {
  pushTarget()
  invokeWithErrorHandling(cb, vm, [watcher.value], vm, info)
  popTarget()
}
```

2. deep :true

   watch 中设置深层监听, 会执行 [traverse](https://github.com/vuejs/vue/blob/edf7df0c837557dd3ea8d7b42ad8d4b21858ade0/src/core/observer/traverse.js#L14) 对 value 进行深度访问，触发所有属性的 get 方法，实现依赖收集,，效果和 `parsePath函数 `一致（parsePath会返回下面的循环读值函数）

```js
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
```



## `computed Watcher`：

函数意义：计算属性是基于数据的响应式依赖进行缓存的，只在相关响应式依赖发生改变时它们才会重新求值，也就是说只要计算属性依赖的数据还没有发生改变，多次访问计算属性会立即返回之前的计算结果，而不必再次执行函数

设计初衷就是为了惰性求值。比如说有一个1000的数组。。。。我们第一次已经遍历过一次了，再次使用，依赖数据没有发生变化，就不必再遍历一遍了。

初始化：

> 1. new Watcher的时候计算属性的fn放入getter，初始化watcher的时候，并没有主动出发属性的get操作，即没有收集依赖。但标记上了lazy标识位，dirty标识位
> 2. 之后定义vm上读取computed的get操作，初始化完毕

```js
const watchers = vm._computedWatchers = Object.create(null)
watchers[key] = new Watcher( 
        vm,
        getter || noop,       //计算属性的fn放入getter
        noop,
        computedWatcherOptions//{lazy:true}
      )
 this.value = this.lazy? undefined: this.get()  //computed watcher 走前者，也就是说没有主动触发get，lazy：标记为懒watcher

Object.defineProperty(vm, key, {  //在vm上挂载computed的key，并且设置computed的get和set
    enumerable: true,
    configurable: true,
    get:  function computedGetter () {            //computed的get方法
   				  if (watcher.dirty) watcher.evaluate() //dirty属性标记意义
      			if (Dep.target) watcher.depend()
  			    return watcher.value                 //return fn()结果
  }，
    set: function() {} // 暂不考虑set情况
  })                
```

副作用函数收集阶段（依赖收集）：

> 1. 在首次挂载的时候，初始化了render watcher， 之后调用了this.get()方法，先render watcher 进入c位，在执行vm.update(vm.render)，再vm.render执行
>
> 2. render过程中，读取到了计算属性，触发get，即下面函数，会拿到我们之前初始化的computed watcher，这步很关键，后续要把它推入c位
>
>    ```js
>     const watcher = this._computedWatchers[key]
>    //取到我们之前初始化的computed watcher，这个watcher上有computed对应的fn
>    ```
>
> 3. 首先会判断dirty标识位，第一次进入的时候为true，直接执行`watcher.evaluate()`,就又去执行watcher的this.get()，**computed watcher进入c位，此时的全局watcher中已经有两个watcher（computed，render）**
>
>    ```js
>    watcher.evaluate()等价于下面
>      //1. this.value = this.get()
>    
>      //  get () {
>      //  pushTarget(this) //把当前的watcher置于c位
>      //  const vm = this.vm
>      //  let value = this.getter.call(vm, vm)//调用updateComponent或者conputed函数
>      //  popTarget()
>      //  return value
>    	//  }
>    
>      //2. this.dirty = false
>      //这一步走完，就代表所有的computed函数中的数据，都已经将computed watcher收集到自己的dep中了。
>    ```
>
> 4. 执行计算函数，同时内部触发数据的get操作，内部的数据收集computed watcher，这一步走完，就代表所有的computed函数中的数据，都已经将computed watcher收集到内部属性的dep中。
>
> 5. 接着执行 watcher.depend() ，判断当前targetStack中是否还有watcher（因为render watcher还在队列之中），将渲染watcher 添加到属性的dep中，computed watcher已经出栈，并且把target的指向改为前一个watcher，现在render watcher 在c位
>
>    ```js
>    watcher.depend()
>    //   depend () {
>    //		let i = this.deps.length
>    //    while (i--) {
>    //    this.deps[i].depend()
>    //  } 所有computed watcher的依赖数据的subs都添加上render watcher
>    ```
>
> 6. 此步目的就是计算函数内部数据**不仅要把computed watcher 收集起来，还需要把render watcher收集起来**

dirty:意义：再次读取到computed的时候，进入判断，因为dirty在第一次读的时候，设置为false，所以直接返回value

```javascript
get:  function computedGetter () {            //computed的get方法
			  if (watcher.dirty) watcher.evaluate() //dirty属性标记意义
  			if (Dep.target) watcher.depend()
		    return watcher.value                 //return 计算fn()结果
```

计算属性内部数据如果发生更新：会把`this.dirty` 设置成 `true`，如果数据变化的时候再执行`watcher.evaluate()`进行`info`更新，没有变化的的话`this.dirty` 就是`false`，不会执行`info`方法。这就是computed缓存机制。



```js
//全局的watcher数组
const targetStack = []
export function pushTarget (target: ?Watcher) {
  targetStack.push(target)
  Dep.target = target
}
export function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```

[三种watcher]: https://segmentfault.com/a/1190000023196603



