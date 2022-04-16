### redux中间件异步操作

1. 为了使用一些异步的代码，使用Middleaware中间件,比如说cookie解析，日志记录，文件压缩，调用接口等操作。
2. 中间件的概念：在dispatch派发action到reducer这个过程中，拓展一些自己的代码

redux-thunk：网络请求中间件（只是将传入的action函数帮我们执行了，并且又dispatch了一次同步action）

#### 自己实现中间件

1. hack技术：monkeyingpatch（修改原有的api）

```js
const next=store.dispatch
function dispatchAndLodding(action){
  //在派发之前做一些逻辑处理
  next（action）
  //派发之后做一些逻辑处理
}
store.dispatch=dispatchAndLodding
```

2. 将上面的函数封装： 

```js
function patchLogging(store){
  const next=store.dispatch
  function dispatchAndLogging(action){
		console.log("在派发之前做一些逻辑处理,比如说日志记录等")
  	next（action）
    console.log("在派发之后做一些逻辑处理,比如说日志记录等")
  }
 // store.dispatch=disopatchAndLogging
  return disopatchAndLogging
}
```

3. 封装patchThunk的功能：

```js
function patchThunk(store){
	const next=store.dispatch
  function dispatchAndThunk(action){
    if(typeof action === 'function'){
			action(next,store.getStore)
    }else{
    	next(action)  
    }
  } 
 //store.dispatch=dispatchThunk
  return dispatchThunk
}
```

如果我们想使用这个中间件的话，在index文件夹中直接调用即可,但是不会直接调用，而是使用applyMiddleware 

4. applyMiddleware封装：

```js
function appleMiddleware (...middlewares){
  const newMiddlewares = [...middlewares]
	newMiddlewares.forEach((middleware)=>{
    //middleware(store) 下面的写法是patchLogging,patchThunk纯函数化写法
		store.dispatch = middleware(store)
})
}
//外层调用方式：
appleMiddleware(patchLogging,patchThunk)
```



### combineReducer函数

combineReducer函数内部源码中判断派发的action是否改变了state，如果没有改变的话，代表不需要刷新界面，返回的state还是旧的state。

1. ui相关的组件内部可以维护的状态，在组件内部自己来维护
2. 只要是需要共享的状态，组件共享的数据，都交给redux来管理和维护
3. 从服务器请求回来的数据（包括请求的操作），交给redux来维护
4. 在路由发生改变的时候，数据需要保留的情况下（数据持久化），应放入redux之中



## reduer是纯函数

其一是保证新旧 state 不是同一个对象引用, 所以不能直接修改旧 state 中的数据, 然后返回旧 state, 这会导致 combineReducers 中的判断失效( hasChanged = hasChanged || nextStateForKey !== previousStateForKey ), 最后redux会直接返回旧 state, 从而导致操作无效;

其二为了保证返回的新 state 是确定的, 而不是会因为某些副作用返回不确定值的 state.

然后说一下为什么reducer最好是纯函数，首先你得看看文档怎么说reducer的作用的，‘接收旧的 state 和 action，返回新的 state’，您可得瞧好咯，他就是起一个对数据做简单处理后返回state的作用，为什么只起这个作用，这时用设计这个词回答这个问题才恰当，因为redux把reducer设计成只负责这个作用。很白痴的问答对吧，所以题目的答案也就简单了，reducer的职责不允许有副作用，副作用简单来说就是不确定性，如果reducer有副作用，那么返回的state就不确定，举个例子，你的reducer就做了一个value = value + 1这个逻辑，然后返回state为{value}，ok，这个过程太jr纯了，然后你可能觉得要加个请求来取得value后再加1，那么你的逻辑就是value = getValue() + 1, getValue是个请求函数，返回一个值，这种情况，退一万步讲，如果你的网络请求这次出错，那么getValue就返回的不是一个数值，value就不确定了，所以return的state你也不确定了，前端UI拿到的数据也不确定了，所以就是这个环节引入了副作用，他娘的redux设计好的规范就被你破坏了，redux就没卵用了。

最后我回答下如何解决这个副作用，实际上也很白痴的问题，这里的请求可以放在reducer之前，你先请求，该做出错处理的就做出错处理，等拿到实际数据后在发送action来调用reducer。这样通过前移副作用的方式，使reducer变得纯洁。

纯函数没有副作用，方便做tree shaking

当网络请求回来的数据，发生改变的时候，useSelector依赖的书redux中的数据的页面，就会自动刷新页面，就可以重新拿到redux中数据，就可以正常显示了，
