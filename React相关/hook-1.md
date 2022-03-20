### useState

每一次渲染都有它自己的 props 和 state，每一次渲染都有它自己的事件处理函数。

```js
const [count setState]= useState(0)//不想采用合并策略，使用回调函数更新方式，也称函数式更新
const click=()=>{setState(count => count + 1 );//如此方法解决setState合并问题}
```

惰性初始化`State`，`state`永远是第一次初始化的值，后续在函数组件`re-render`的时候，函数组件内所有代码都会重新执行一遍。此时`initialState`的初始值是一个相对开销较大的`IO`操作。每次函数组件`re-render`时，第一行代码都会被执行一次，引起不必要的性能损耗。

```js
   const initialState = Number(window.localStorage.getItem('count'))
   const [count, setCount] = React.useState(initialState)
   //那么我们完全可以使用如下写法
   const [count, setCount ]= React.useState( () => Number (window.localStorage.getItem 		 ('count')))
```

死循环

```js
useState声明出的数据，永远是初始化的闭包。
函数组件更新，使用const query={index:0,...}定义的数据就会重新定义，导致发送网络请求的hook重新执行，
const query ={index:0,...};
//
useEffect(()=>{
  do something //such as 网络请求
  setState(..) //页面挂载，执行useEffect内函数，网络请求后set数据，触发页面重新刷新，导致query重新定义一遍,而useEffect钩子又依赖query，导致重新执行
}，[query])
```

> 解决方法：需要使用useState、useMemo、useCallback包裹

### useReducer（reducer，initState）

导致组件重新渲染，和redux的使用方法一样

如果业务不是特别大，可以使用uesContext+useReducer替代redux

> const [ state , ] = useReducer()

### useMemo，userCallback

性能优化，解决父组件重新刷新，子组件重新渲染问题。（v8引擎生成一个匿名函数成本很低），**第二个参数为依赖数组，显式声明依赖数据**

```js
useCallback(fn, deps) 相当于 useMemo(() => fn, deps)
useMemo相当于react版本的computed计算属性
```

### useEffect

模拟组件第一次挂载，更新组件，组件销毁生命周期，第二个参数（依赖刷新数组）和vue中的依赖采集是一个意思

```js
useEffect(()=>{
	//定时器，发送网络请求等
  return （//卸载定时器等组件必要操作）
}，[依赖数据])
```

### useLayoutEffect

执行时机在paint（绘制）之前，函数签名和useEffect是一样的，

```js
useLayoutEffect和useEffect都是在commit阶段执行，但是不同时机，useEffect在commit阶段结尾异步调用，useLayout/componentDidMount同步调用
```

### useRef（和类组件的createRef一样）

不会随着函数式组件的刷新，而重新赋值，拿到的总是第一次初始化的值，也称持久化的一个数据，所以可以改变数据而不触发re-render的骚操作

```js
我们可以利用持久化数据这个特性,模拟函数组件的this，在初始化阶段，我们存入一些数据，在之后的useEffect等hook中，用到useRef存入的数据 
```

### useContext（不使用props，从而传递数据，透传）

在外部定义createContext，并且在外层组件通过value注入，内存的函数式组件即可使用useContext，拿到注入的值。

```js
import React, { useRef, useState, useContext, createContext } from 'react';
const Context = createContext(); //外部模块声明

export default function Parent() {
  const [count, setCount] = useState(0);
  const store = { count, setCount };
  return (
    <Context.Provider value={store}>
      <Child />
    </Context.Provider>
  );
}

const Child = () => {
  //   const { count, setCount } = useContext(Context); //拥有了hook写法
  return (
    <div>
      {/* <div>{count}</div> 
      <button onClick={(e) => setCount(count + 1)}>改变context中的数据</button> //如此写法如同connect函数*/}
      <Context.Consumer>
        {(value) => {
          const { count, setCount } = value;
          return (
            <div>
              {count}
              <button onClick={(e) => setCount(value.count + 1)}children={' 改变context中的数据'}/>
            </div>
          );
        }}
      </Context.Consumer>
    </div>
  );
};
```

使不使用`hook`的写法，都是在子组件中拿到外层组件注入的value值（其中可以有改变数据的方法，**Context函数注入和redux非常相似 **，只不过`context`是从`context`中拿到数据和`change`，`redux`中是从`props`中拿取）。

### useImperativeHandle(了解)

在父组件中拿到子组件的ref对象，对子组件的操作进行限制，在父组件中调用子组件中规定的ref劫持方法，来实现对子组件的控制，子组件组件对ref进行限制，直将focus暴露给父组件，相当于做个ref缩小劫持

```js
const sub = forwardRef((props, ref) => {
  const input = useRef();
  //这里有意切断ref与input标签的联系，只让组件拿到input标签的引用
  useImperativeHandle(ref, () => {
    input.current.focus();
  });
  return <input {...props} ref={input} />;
});
```

### useLocation，useHistory，useParams等等

### 自定义钩子

```js
function useUpdated(callback) {
  let isMounted = useRef(false);
  useEffect(() => {//只有组件状态更新的时候才会发生变化
    isMounted.current ? callback() : (isMounted.current = true);
  });
}

export default function aaa(props) {
  useUpdated(() => console.log("使用自定义钩子，只有在组件更新的时候改变"));
  const [count, setCount] = useState(0);
  return <div onClick={(e) => setCount(count + 1)}>{count} </div>;
}
```

### forwardRef

原来我们在高阶组件中写的一些ref转发，使用这个hook就不再需要了,`ref`转发的目的是为了在父组件中可以拿到子组件中的元素。如此操作我们可以拿到input的所有方法，可以所以修改，违反开闭原则，新的hook来处理`useImperati veHandle`

```js
const Sub = forwardRef((props, ref) => {return <input ref={ref} />});

export default class Parent extends React.Component {
  input = createRef();
  componentDidMount() {   this.input.current.focus(); }
  render() {return <Sub ref={this.input} />;}
}
```



> 面试题：hook不可用于条件判断，如何自己实现hook。
>
> 答：单向链表存储hook，决定了hook的注册顺序
>
> ```js
> 手写hook
> ```

