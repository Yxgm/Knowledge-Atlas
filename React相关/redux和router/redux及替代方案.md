# redux

### 配套使用的钩子useDispatch、useSelector（函数组件）可替代connect（类组件）

### react-redux是将redux和react结合起来的工具，而connect函数就是这个工具的具体实现

1. 函数本身接受两个映射函数，返回一个高阶组件（就是一个高阶函数，没啥了不起的），让store的依赖只依赖于connect函数，
2. connect本身就是一个函数柯西化而已，在返回的高阶组件中，又传入基础组件
3. **函数意义：分离组件，分离为视图组件和容器组件。视图组件：只是数据的展示、 外层容器：处理state和dispatch等逻辑**

```jsx
//connect.js
import StoreContext from "./context";

export default function connect(mapStateToProps, mapDispatchToProps) {
  return function (WrappedComponent) {
    class exhanceComponent extends PureComponent {
      constructor(props, context) {
        super(props);
        this.state = {
          storeState: mapStateToProps(context.getState()),
        };
      }
      componentDidMount() {
        this.unsubscribe = this.context.subscribe(() => {
          this.setState({storeState: mapStateToProps(this.context.getState())});
        });
      }
      componentWillUnmount() {this.unsubscribe()}
      render() {
        return (
          <div>
            <div>容器组件的props:{this.props.name} </div>
            <WrappedComponent
              {...this.props}
              {...mapStateToProps(this.context.getState())}
              {...mapDispatchToProps(this.context.dispatch)}
            />
          </div>
        );
      }
    }

    exhanceComponent.contextType = StoreContext;//接收到顶层注入的store，放入类组件中
    return exhanceComponent;
  };
}
```

```js
//about.jsx
function About(props) {
  return (
    <div>
      <h1>About</h1>
      <h2>当前计数:{props.counter}</h2>
      <button onClick={(e) => props.decrement(10)}>+1</button>
      <button onClick={(e) => props.reset()}>归0</button>
    </div>
  );
}
//容器上的逻辑
const mapStateToProps = (state) => {
  return { counter: state.counter };
};
const mapDispatchToProps = (dispatch) => {
  return {
    decrement: (data) => dispatch(add(data)),
    reset: () => dispatch(reset()),
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(About);
```



结合上面的代码，就是在connect中集中了store，然后通过两个映射函数的直接执行，然后这两个函数的执行结果返回的state和dispatch直接返回给视图组件，让其展示或者处理逻辑。

```jsx
exhanceComponent.contextType = StoreContext//在高阶函数中的类组件使用this.context代替store。
```

为了让这个connect函数不对store有依赖性，在connect中引入connext，在index.js中，引入context和store,传入stote就可以在connect函数中使用了。

```js
  <StoreContext.Provider value={store}> <App/> </StoreContext.Provider>
```

### connect函数内部的性能优化

在mapStateToProps函数返回的对象中，只依赖于当前数据，也就是说不依赖的数据发生改变，该组件是不会重新render的。

为什么？在state发生改变之后，会拿到新旧两次state，进行对比（浅比较，如不理解浅比较，请去查看oneNote）因为两次的state是一样的，所以组件不重新render

### connect函数（也是是用了context）这和useSelector有所区别

useSelector默认是不做浅层比较的，内部是直接使用了一个===，组件会重新render，我们会在加上第二个参数，useSelector（fn，shallowEqual）

# 使用useReducer和useContext替代redux

```js
//Provider.js
import { createContext, useReducer } from 'react';
export const Context = createContext();
//initStore和reducer应该抽离为单独模块
const initStore = {
  count: 0,
  number: []
};

function reducer(state, action) {
  switch (action.type) {
    case 'add':
      return { ...state, count: action.data };
    default:
      return state;
  }
}
向外暴露一个Provider api，Provider函数可将数据和改变数据的方法顶层注入，useReducer就是将数据和方法结合起来的一个钩子
export const Provider = (childrenComponent) => {
  const [state, dispatch] = useReducer(reducer, initStore); //集成reducer和store
  return (
    <Context.Provider value={{ state, dispatch }}>
      {/* 将数据state和改变数据的dispatch顶层注入进去 */}
      {childrenComponent}
    </Context.Provider>
  );
};

//App.js
function App() {
  return Provider(<App />);//顶层注入
}
//Component.js
import { Context } from './context';
export default function Home() {
  const { state, dispatch } = React.useContext(Context);//直接从Context中结构即可拿到
  return (
    <p onClick={(e) => dispatch({ type: 'add', data: state.count + 1 })}>
      {state.count}
    </p>
  );
}
```

# 总结

> 思想都是一样的，都是从外层组件注入数据和改变方法，内层组件拿到数据和方法，从而进行视图的展示和逻辑处理。
>
> 对于替代redux的两种钩子方案，useReducer是将数据和方法暴露出来，而useContext是将数据和方法注入到所需组件。
