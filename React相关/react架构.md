

# 什么是react ？学设计理想，数据结构。

1. 基于函数式思想，践行代数效应 view=fn（state），而函数的最大的问题：副作用问题，影响外部一些变量
2. react是基于mvc模式的一种设计模式，单向数据流
3. 解决代数效应：hook解决方案（解决了函数副作用的响应），剥离副作用
4. 原理：调度和vdom
5. learn once ，write anywhere
5. <15  stack reconciler： 不可暂停 、16fiber reconciler  、18 concurrent mode （分别优先级，低级的优先级任务会在那个函数中运行）

# react解决了什么问题 ？

### 1. CPU 瓶颈：

输入事件的处理（阻塞事件和非阻塞事件）-> js 解析运行阶段 —>样式布局-》 raF 请求下一帧1帧-〉珊格化-》paint

为什么原生比js快：原生编译性质的代码，没有js解析之前的阶段

如果js执行时间过长，阻塞重排和重绘，导致页面掉帧，react就将任务拆分，解决这个问题

### 2. IO瓶颈：

前端操作最多的io操作是网络请求

react提出一个suspence组件loading加载（大项目中使用），减少用户对网络延迟的感知（内部做了很多优化，复用），如果很快就请求回来了，就不展示了，版本18中提出的。

code spliiting（代码分割）：降低包体积，将一些不常用的组件放在网络，用的时候在请求，react组件懒加载

data fetching

一个html的标签过多，如果用js原生，浏览器会做大量的操作。

## 初衷：

为了构建一个快速相应的大型web项目，践行快速响应的理念

## 调和：

不等于diff，调和是让vnode和真实dom一致，而diff是找出不同，调和=diff+操作真实dom

## React必须重新实现遍历树的算法

**从依赖于内置堆栈的同步递归模型，变为具有链表和指针的异步模型**



## 两种工作方式

workLoopSync：同步、workLoopConcurrent：时间切片

```js
function workLoopConcurrent() {//时间切片执行，执行performUnitOfWork函数。
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

## 宏观

`render`阶段：只会做一些复杂的运算，并不会真正的操作页面，主要就是用来生成新的fiber树并diff出有变化的节点，生成effectList链表，直到commit阶段才会去通过js的原生api去修改dom。

> ##### workLoop循环♻️：
>
> `beginWork`:执行effect函数，对class组件完成state更新，完成fiber创建，diff，复用，并对有effect的节点flag标记，传入当前的fiber节点，创建子节点
>
> ```js
> function beginWork(
>   current: Fiber | null,//当前组件对应的上次更新的fiber节点
>   workInProgress: Fiber,//当前组件对应的fiber节点
>   renderLanes: Lanes,
> ): Fiber | null {
>   // ...省略函数体
> }
> ```
>
> 两种路径：首屏渲染，不存在current指向，会根据fiber的tag生成不同的fiber节点
>
> ​                     页面更新
>
> ##  reconcileChildren

> `completeWork`：收集effectList，对真实dom操作

`commit`阶段: 获取到render阶段中diff出来的发生了变化的节点的fiber，通过原生api更新页面

时间分片：在`render`阶段进行，也就是构建新的`workInProcess`的时候，**每一帧结束的时候，进行一次时间剩余检查，如果没有剩余时间的话，直接break进入下一帧，下一帧来到之后在做同样的判断**。



## 一些数据结构

fiberRoot： 容器节点（我觉得用这个形容更容易理解一些），只能有一个

> tag：legencyRoot/BlockingRoot/ConcurrentRoot
>
> containerInfo：挂在目标容器
>
> current：currentFiber对象， 第一个子节点（fiber对象），对应着root节点，整个应用的根结点
>
> finishWork：commit处理的任务

rootFiber：可以有多个

> - Scheduler（调度器）： 排序优先级，让优先级高的任务先进行协调
> - Reconciler（协调器）： 找出哪些节点发生了改变，并打上不同的Flags（旧版本react叫Tag）就是diff操作
> - Renderer（渲染器）： 将Reconciler中打好标签的节点渲染到视图上

#### `scheduler`（调度器）：

对于react15来说，没有调度部分，任务没有优先级，不可中断，只能同步执行。

**优先级调度方案：使用过期时间来表示，过期时间越短，表示优先级越高，需要立即执行。**

**没有过期的任务放在`timerQuere`中，过期的任务放在`taskQueue`中**

**都是小顶堆的结构，所以`peek`取出任务时候，都是过期时间最短的（即取出了最高优先级任务）。**



#### `reconciler`（协调器）：

发生在`render`阶段，`render`阶段会为节点执行`beginWork`和`completeWork`函数，或者计算state，对比节点差异，为有差异的节点赋予相应的`effectFlags`标识。

协调器是在render阶段工作的，概括：`Reconciler`会创建或者更新Fiber节点。在mount的时候会根据jsx生成Fiber对象，在update的时候会根据最新的state形成的jsx对象和current Fiber树对比构建workInProgress Fiber树，这个对比的过程就是diff算法。

打上的effectFlags标识（增删改操作），在commit阶段，把这些标签应用到真实的dom上。



#### `Renderer`（浏览器中的渲染器是react-dom）：

发生在commit阶段，commit阶段遍历effectLists执行对应的真实dom操作，或者部分的生命周期。



#### `Concurrent`

它是一类功能的合集（如fiber、schduler、lane、suspense），其目的是为了提高应用的响应速度，使应用cpu密集型的更新不在那么卡顿，**核心是实现了一套异步可中断、带优先级的更新**。

react17会在每一帧分配一个时间（时间片）给js执行，如果在这个时间内js还没执行完，那就要暂停它的执行，等下一帧继续执行，把执行权交回给浏览器去绘制。

###### react如果开启了异步渲染模式，会把所有复杂的操作放到每一帧的空余时间去做（idle）

> 复杂的操作比如“diff算法”，“初始化函数或class组件”，“执行组件生命周期”，“创建或更新fiber树”等。通过将这种复杂的同步算法，放到了不影响视觉流畅的空闲时间当中，这就是react16中的时间分片的假面！
>
> 在浏览器一帧中的空余时间，利用这段时间，进行任务调度，调整任务的执行顺序



###### 面试题：任务是如何中断，如何执行的？





commit阶段转化为同步阶段，因为要操作dom了

**fiber是单向链表的数据结构（更方便的进行更新、操作等），fiberNode是替代vnode的产物，更方便的任务切分，虚拟dom的深度遍历不可以停止，迭代是可以停止的**，深度优先遍历树，然后找到所有的子元素生成链表

lanes：判断当前节点是否需要更新（调度优先级作用）

对于更新来说，不是更新的页面上的dom，是更新的workInProcess树，完毕之后，将更新的树挂载，变成current Fiber

用户的交互优先级最高。





fiberRootNode 与 rootFiber，是因为在应用中我们可以多次调用 ReactDOM.render 渲染不同的组件树，他们会拥有不同的 rootFiber。但是整个应用的根节点只有一个，那就是 fiberRootNode。

fiberRootNode 的 current 会指向当前页面上已渲染内容对应 Fiber 树，即 current Fiber 树。