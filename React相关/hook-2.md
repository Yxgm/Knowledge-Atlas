## useState

useState中的setState和类组件组件中的setState使用效果不同。

```js
function App(props) {
  const [count, setCount] = useState({ value: 0, name: 'sxy' });
  return (<div>
					{count.value}:{count.name}
					<button onClick={(e) => setCount({ value: count.value + 1 })} />
					//如此写state的值只会有value，name将会消失
					</div>);}
  
class extends React.Component {
  state = { name: 'sxy', value: 0 };
  handleValue = () => {this.setState({ value: 10 });
    //如此写的setState，会将原来state合并，还是会留有name属性
  };
  render() {
    return (<>
        {this.state.name}:{this.state.value}
        <button onClick={(e) => this.handleValue()} />
       			</>  )}}
```

## useEffect （闪动效果）

**它回调函数是【异步宏任务**】，**在下一轮事件循环才会执行，在dom渲染完成，后再次回调**

> 1. `好处`：这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因为绝大多数操作不应阻塞浏览器对屏幕的渲染更新。
> 2. `坏处`：产生二次渲染问题，第一次渲染的是旧的状态，接着下一个事件循环中，执行改变状态的函数，组件又携带新的状态渲染，在视觉上，就是二次渲染。

## useLayoutEffect（卡顿，白屏更多）

而 useLayoutEffect 与 componentDidMount、componentDidUpdate 生命周期钩子是【**异步微任务**】，**在渲染线程被调用之前就执行**。这意味着回调内部执行完才会更新渲染页面，没有二次渲染问题。

> 1. `好处`：没有二次渲染问题，页面视觉行为一致。
> 2. `坏处`：在回调内部有一些运行耗时很长的代码或者循环时，页面因为需要等 JS 执行完之后才会交给渲染线程绘制页面，等待时期就是白屏效果，即阻塞了渲染。