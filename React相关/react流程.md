## 三个前提：

##### 1. 只做同级比较，跨层级的dom不会复用

##### 2. 不同类型节点生成的dom树不同，此时会直接销毁老节点以及子孙节点，并新创建节点

##### 3. 通过key值，对元素diff的过程，提供复用（真实dom节点复用，不再创建或者别的操作真实dom了）的线索

### 单节点DIFF

### 多节点DIFF



虚拟dom：

1. 某种程度上磨平性能下限
2. 作为一个中间层，跨端
3. 在大量操作元素的时候，提升性能

## react的三个核心包

在没有进入`react`这三个包之前，所有的标签都已经转化成了`React.createElement`的形式，在react的处理之前，就已经成为了`vdom`元素（正常的业界`vdom`），

### 1.  React

> 提供处理虚拟dom的一些函数，和用户侧的一些hooks

#### 经过React包的处理，所有的虚拟dom，都变成了react虚拟dom，标记上了type类型，onwer原生节点等等

### 2. React Reconciler！

`BeginWork（diff)、CompleteWork`，构建出fiber树`

`beginWork`阶段：根据`current fiber`和新`render`出来的`recat-Vdom`生成新的fiber节点，深度探索fiber树。但是此时的新构建出来的fiber树上的原生dom，并没有发生刷新。最后在`effectTag`上标记上了变动的标签

`completeWork`阶段：自下而上的收集这些带有标签的fiber节点，依次链接起来，并且挂在rootFiber上。(说的是更新阶段)，

`commitWork`阶段： 又分三个阶段

before mutation

mutation

layout

### 3. ReactDom

接收到effectList链，执行真正的dom操作，更改current Fiber上的真实dom节点。（commit操作）

两颗fiber树，共用一颗真实的dom树

当任务打断了之后，即交出执行权，让浏览器去渲染，之后在postMessage中又通知这个任务再次执行

已进入循环的时候，就会判断当前一帧的剩余时间还有没有空余，空余情况才

diff：（发生在beginWork阶段，生成链表的时候）

vdom和current比较生成workInProgress Fiber，同时生成了effect链表，



### 模拟的requestIdleCallback函数

用MessageChange，将一帧开始的时间传出（requestAnimationFraml的回调函数中有一帧开始的时间），随后在下一次事件循环中的宏任务首部（messagechange的回调在宏任务一开始就会执行），

>  补充：onmessage的回调函数的调用时机是在一帧的paint完成之后，所以说在onmessage中又调用那个循环的方法 

 为什么不用生成器？

生成器有状态，重新执行的话，又会从头执行

### 对于新旧多节点diff

首先从头遍历新children，该节点新children有，而旧children没有的，直接插入旧children首部，都一次操作后都会记录新children的最后的修改位置（lastIndex）

因为只有insertBefore API，如果新c遍历可以复用的节点的话（key，type相同等等，直接复用dom节点），会移动改节点，就移动到lastIndex位置。

当新的children节点都已经全部插入到了旧children之后，会标识为最后一个节点，之后的节点不再需要，直接删掉

用户输入的时候，渲染相应的列表，18版本之前，使用防抖或者截流等，来控制视图渲染

18版本之后，startTrantale ：cd方法不在主线程上，相当于在一个分支上

节点移动最为复杂，先固定节点，（最小上升子序列），



#### Fiber Reconciliation流程分析

1. 如果当前节点不需要更新，则把子节点clone过来，跳到步骤5
2. 跟新当前节点的props,state,context等，调用getDerivedStateFromProps静态方法
3. 调用shouldComponentUpdate()，false的话，跳到步骤5
4. 调用组件的render方法获得新的children, 进行reconcileChildren，为子节点创建fiber（创建过程会尽量复用现有fiber，子节点增删移动发生在这里，不是真实的dom节点，并标记effectTag，收集effect）
5. 如果没有workInProgress.child，则工作单元结束，调用completeWork，如果是dom节点就diffProperties,标记tag，把effect list归并到return fiber，并把当前节点的sibling作为下一个工作单元；否则把child作为下一个工作单元
6. 如果没有剩余可用时间了，等到下一次主线程空闲时才开始下一个工作单元；否则，立即开始做
7. 当返回到root节点的时候，工作循环结束，进入commit阶段
8. commit阶段，标记isWorking = true;此阶段不可打断
9. 有三个大循环遍历effect list。主要是调用生命周期，将更新flush到屏幕上



每当更新发生时，react会从fiber的根节点开始，一个一个地循环遍历所有的fiber。

**react 通过循环，一个一个地对fiber执行performUnitOfWork操作，以此实现了任务分片。**

时间分片

时间分片的逻辑藏在循环里。

```javascript
//react-reconciler -> ReactFiberWorkLoop.js
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

performUnitOfWork可能返回null或者下一个需要被执行的fiber，返回结果存在workInProgress中。workInProgress在react-reconciler模块中是全局变量。

当shouldYield返回true的时候，循环语句中断，一个时间分片就结束了，浏览器将重获控制权。

以下任意条件成立时，shouldYield会返回true

- 时间片到期（默认5ms）
- 更紧急任务插队

**react 通过中断任务循环，实现了时间分片。**

任务恢复

循环中断时，下一个未被完成的任务已经被保存到react-reconciler模块的**全局变量workInProgress**中。下一次循环开始时就从workInProgress开始。

以上都发生在浏览器重获控制权之前。

而监听这个事件的，正是循环的发起者**performWorkUntilDeadline**。

**循环中断之后react执行 \*port.postMessage\* 发起了一个message事件，并且通过事件监听又恢复了循环**。在循环中断到事件响应的间隙，浏览器重获了控制权，执行必要的渲染工作（如果有的话）。

以上特性，目前只在开启concurrent模式时有效。默认模式下只有任务分片，但是是同步执行的，所以不存在时间分片。



## 微信公粽号React开发思路

1. 使用 `ESLint` 来静态分析你的代码，开启 `rule-of-hooks` 和 `exhaustive-deps`这两个规则来捕获 `React` 错误。
2. 开启 JS `严格模式`吧，都 2202 年了。
3. `直面依赖`，解决在`useMemo`，`useCallback` 和 `useEffect` 上 `exhaustive-deps` 规则提示的 warning 或 error 问题。可以将最新的值挂在 ref 上来保证这些 hook 在回调中拿到的都是最新的值，同时避免不必要的重新渲染。
4. 使用 map 批量渲染组件时，`都加上 key`。
5. `只在最顶层使用 hook`，不要在循环、条件或嵌套语句中使用 hook。
6. 理解`不能对已经卸载的组件执行状态更新`的控制台警告。
7. 给不同层级的组件都添加`错误边界（Error Boundary）`来防止白屏，还可以用它来向错误监控平台（比如 `Sentry`）上报错误，并设置报警。
8. 不要忽略了控制台中打印的错误和警告。
9. 记得要 `tree-shaking`!
10. 使用 `Prettier` 来保证代码的格式化一致性！
11. 使用 `Typescript` 和 `NextJS`这样的框架来提升开发体验。
12. 强烈推荐 `Code Climate`（或其他类似的）开源库。这类工具会自动检测代码异味（Code Smell，代码中的任何可能导致深层次问题的症状），它可以促使我去处理项目里留下的技术债。
