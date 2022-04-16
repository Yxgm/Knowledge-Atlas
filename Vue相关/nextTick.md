### 自我理解的nextTick

<img src="/Users/yxgm/Documents/知识图鉴/assets/04.jpg" style="width: 700px"/>

如果Vue使用`setTimeout`等**宏任务**函数，那么势必要等待UI渲染完成后的下一个**宏任务**执行，而如果Vue使用**微任务**函数，无需等待UI渲染完成才进行`nextTick`的回调函数操作，可以想象在**JS引擎线程**和**GUI渲染线程**之间来回切换，以及等待**GUI渲染线程**的过程中，浏览器势必要消耗性能

> **函数解释说明：**

1. timerFunc函数：判断promise的then微任务，其次使用MutationObserve，再次setImmediate函数，最后setTimeout（降级处理）的优先级。无可

2. flushCallbacks函数：是刷新（全部重新执行一遍）callbacks数组（nextTick的全局数组）的函数，并且清空callbacks数组。

3. flushBatcherQueue（flushSchedulerQueue）函数：排序watcher队列，父组件的watcher先于子组件执行，避免子组件的二次刷新的性能问题，刷新（全部重新执行一遍）watcher数组。
4. 这些函数都有闭包的应用，queue：watcher函数回调数组。
5. callbacks：刷新watcher函数，自己写的nextTick回调函数。
6. pending：boolean值，控制当前只有一个flushCallbacks函数。

> **大致流程如下：**

1. 当响应式数据更新后，会调用 dep.notify 方法，通知 dep 中收集的 watcher 去执行 update 方法，update又调用queueWatcher方法，将watcher push到一个queue队列。
2. queueWatcher中又调用了nextTick函数，flushSchedulerQueue（刷新watcher的函数）作为参数传入。
3. nextTick函数的调用，将参数push进入callbacks数组，也执行了timerFunc函数。
4. 调用timerFunc函数，优先级选择flushCallbacks函数采用何种异步方式，之后flushCallbacks进入浏览器的异步队列（多为微任务），等待script（整体代码）执行完毕，进入执行栈执行。只是将刷新callbacks的函数推入异步任务，即完成了vue的异步更新策略

> 这也就是nextTick函数命名的原因，在下一次的事件循环回调，实际上是本次事件循环的最后执行的

### 5.Vue中nextTick函数的实现

- 将传递的回调函数用 try catch     包裹然后放入 callbacks 数组
- **执行 timerFunc函数，在浏览器的异步任务队列放入一个刷新 callbacks 数组的函数**



### Node.js的libve

1. timers
2. I/O callbacks
3. idle, prepare
4. poll
5. check（setImmedaite）
6. close callbacks

`process.nextTick()` 的函数会在事件循环的当前迭代中（当前操作结束之后）被执行。 这意味着它会始终在 `setTimeout` 和 `setImmediate` 之前执行。

延迟 0 毫秒的 `setTimeout()` 回调与 `setImmediate()` 非常相似。 执行顺序取决于各种因素，但是它们都会在事件循环的下一个迭代中运行。