## vue进阶用法
### 特征一：模板化
#### 插槽
##### 具名插槽：以name标识插槽的身份，从而在组件内部做到可区分
##### 作用域插槽：外部做结构描述勾勒，内部做传参

#### 模板数据的二次加工
1. watch、computed => 相应流过于复杂（computed赋值）
2. 方案一：函数 - 独立、管道 / 无法响应式
   方案二：`v-html `
   方案三：过滤器 - 追问：无上下文 `{{ time | format }}`

#### jsx 更自由的基于js书写
vue的编译路径：template->render->vm.render->vm.render() diff => 可以使用性能优化方案

如果在vue中直接写render写法，就不会走vue的template优化，导致性能下降.

### 特征二：组件化
#### 传统模板化
* 1. 抽象复用
* 2. 精简 & 内部的功能聚合

组件的模版化是更重，更聚合的，对可视化的部分做复用

逻辑复用的话就使用mixin或者extends

#### 1. 混入mixin - 逻辑混入
应用： 抽离公共逻辑（逻辑相同，模板不同，可用mixin）

缺点： 数据来源不明确 

合并策略
a. 递归合并（同一对象下的属性，也会合并）
b. data合并冲突时，以组件优先
c. 生命周期回调函数不会覆盖，优先级为**先mixin 后组件**

```js
  mixins: [demoMixin],
  extends: demoExtends,
```

#### 2. 继承拓展extends - 逻辑拓展
应用： 拓展独立逻辑

与mixin的区别，传值mixin为数组

合并策略
a. 同mixin，也是递归合并
b.**合并优先级 组件 > mixin > extends**
c. 回调优先级 extends > mixin

#### 3. 整体拓展类extend.    `Vue.extend(）`

```js
// 构造一个基础的构造器，然后让vue.extend，作为一个基础的逻辑
let _baseOptions = {
  data: {return {course: 'zhaowa',}
  created：()=> console.log('extend base')
}
const BaseComponent = Vue.extend(_baseOptions)//返回一个基础构造器
// 基于_BaseComponent，再拓展逻辑
new BaseComponent({ TODO...})
```

从预定义的配置中拓展出来一个独立的配置项，进行合并。

**`mergeOption`**：就是将我们写的一些config，不断的merge，并且依据一定的优先级

应用方法：动态拓展，针对于不同的场景需求，去构建不同的拓展器，参数写成函数

#### Vue.use - 插件
* 1. 注册外部插件，作为整体实例的补充
* 2. 会除重，不会重复注册，底层是 `map数据结构`
* 3. 手写插件
      a. 外部使用Vue.use(myPlugin, options)
      b. 默认调用的是内部的install方法
* 4. 微内核：自己本身的东西很少，都是通过插件的方式（有点中间件的意思）来注入，webpack就是如此

vue的组件化：丰富多样，

react：就是传参数

#### 组件的高级引用
* 1. 递归组件 - es6 vue-tree
* 2. 动态组件 - <component :is='name'/>
* 3. 异步组件 - router（用到的时候在加载，懒加载），一般在大型项目中，考虑首屏渲染的加载时间，资源大小问题，需要用到的技术方案，详细api查阅文档

使用$set的时候就是数据没有被收集依赖 
