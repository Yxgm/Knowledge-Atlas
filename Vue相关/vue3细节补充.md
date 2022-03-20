### Vue2和Vue3的补充细节区别

1. teleport标签，可以让标签内的元素，挂在在teleport to属性的制定根结点

```vue
<teleport to="#other">
	<input/>
</teleport> input元素挂载到了id="other"元素下
```

2. Vue2中一个组件template中必须有一个根节点，Vue3可以多个

3. 函数式组件(大多都是无状态的简单组件，只关注`props`传值来展示)

   Vue2：`functional: true (.vue文件)`

​	 Vue3：类似于jsx写法`（.js）`

4. v-if和v-for的优先级：

   vue2中v-for优先级更高，所以浪费内存（vdom先创建，后销毁）

   vue3中调整了优先级if更高了

5. `vue3`中的响应组合`api`

   > 1. `reactive`：参数是对象，因为是要对对象做代理，而`ref`多用一般数据类型
   >
   > 2. `shallowReactive` 只有第一层是响应式
   >
   > 3. 从`reactive`对象中解构取值时，取出的不是响应式，需要`toRefs`包裹变成响应式
   >
   > 4. effect：effect包裹的函数中响应式数据，数据变化的时候，执行函数（用于只关注更新后的值）
   >
   >    **`effect`和react中的useEffect钩子有点类似**，在组件初始化执行，vue对effect的参数（watcher函数进行收集）
   >
   > 5. watch：第一个参数写成函数的时候，只有函数的返回值变了，才会执行回调函数
   >
   >    ```js
   >    watch(() => count.value> 5, (value) => alert(value) )
   >    ```
   >
   >    count为6的时候才会执行，之后不再执行，因为返回值的状态`false => true `
   >
   > 6. watchEffect和effect类似，watchEffect支持配置
   >
   > 7. provide传递的值应该会只读的，只能通过父组件传递的方法，改变数据（和redux的connect差不多从父组件传递state和dispatch）
   >
   > 8. 逻辑复用vue2中使用mixin但是无法明确数据源，vue3直接使用hook即可复用

