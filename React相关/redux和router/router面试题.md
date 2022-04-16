## react想实现路由守卫的功能，

`react-router`是没有和`vue`类似的路由守卫钩子的，所以想要实现路由守卫，需要手动实现

#### 1. 进入路由之前的钩子

用一个函数包裹起来，`auth`函数中，拿到用户的信息凭证，如果没有的话，就直接跳转到`login`组件（使用`Navigate`组件router6新出的api）

```jsx
//进入路由的函数钩子
export function Auth({ children }: any) {
  const auth = useAuth();
  if (!auth.userInfo) {return <Navigate to="/login" replace></Navigate>;}
  return children;
}
//路由组件
export default function Account() {
  return (<Auth><div>account...</div></Auth>);
}
```

对于如何拿到用户信息，我们一般用`context`传递，顶层注入，持久化存储

```js
const userContext = createContext(null);
//这个AuthProvider组件，只是一个将userInfo数据和一些方法传递至全部组件一个作用而已
export default function AuthProvider({ children }) {
  const [userInfo, setUserInfo] = useState({ name: '', cookie: '' });

  useEffect(() => {
    setTimeout(() => setUserInfo({ name: 'sxy', cookie: '00000 ' }), 500);//模拟网络请求
  }, []);

  const loginOut = () => {
    localStorage.setItem('userInfo', '');
    setUserInfo(null);
  };

  const value = { userInfo, loginOut };
  //对于一些用户登陆的操作，如退出，获取数据等等我们都应该把这类逻辑放在同一处
  return <userContext.Provider value= { value }>{children}</userContext.Provider>;
}

//同时可以封装一个hook,便于拿到userInfo
export function useAuth() => useContext(userContext)
```

> 业务组件中涉及到路由的，可以使用`useNavigate`来方便的操控路由，以前的版本可能使用的是`withRouter`高阶组件包裹或者`useHistory`钩子，直接`push`路由，来达到相同的效果。

#### 2. 离开路由之前的钩子

navigate上有block方法，我们把要做的逻辑放入block参数中，就会触发浏览器的原生`onbeforeunload`事件监听(离开页面触发回调函数)，一边后面的流程阻止掉，同时执行我们的回调函数，当回调函数的队列清空了，再把beforeunload事件移除掉。详细代码请见 <a href="./usePrompt.md">usePrompt 钩子</a>

> 大体逻辑如下：input或者form等触发了改变，就会有一个showDialog的boolean值，会置为true，之后就回调用我们自己实现的路由离开钩子usePrompt，true传入钩子中
>
> 钩子中useEffect判showDialog，为false直接返回不处理，为true，将要展示dialog，存储跳转路径。在dialog中确定跳转（监听是否要跳转用一个boolean值）的时候，触发路经跳转。
>
> 并不是出发了inputChange，dialog就会展示，而是在跳走的时候（原生层面上来讲就是页面卸载的时候，触发回调函数），才触发dialog的展示，
>
> 所以说，在跳走的时候，展示dialog，

####  3. lazy 

一般来说，用户需要登录状态，才可以看到页面的情况下，这样做懒加载是比较合适的。对于懒加载需要一个平衡

#### 4. 源码部分

1. `BrowserHistory`:

借助history库，将状态机保存在一个state中，并且history监听自己的变化，触发setState，用的是useLayoutEffect，是为了在dom挂载之前，完成监听。 

2. 返回的`Router`标签：

创建了两个全局的context（`navigationContext`和`location对象`），为了让所有的路由组件都可以拿到这两个对象，就可以使用`useLocation等hook`，

3. `Routes`标签：

内部也是使用了`useRoutes`，我们写的标签组匹配，最终还是会转化成useRoutes+匹配参数的形式。大致逻辑，遍历内部所有子节点（递归遍历），生成一份映射表放入Routes数组中。

> 关键的是useRoutes函数，他是处理`path`和组件展示的逻辑函数，拿到当前路径，遍历路由表，匹配对应组件，拿到对应组件，渲染节点

#### 5. History

⚠️：当调用`pushState`或者`replaceState`，一边操作浏览器的`history`对象，同时也要操作我们缓存的`history`对象，这是会触发`listerer`监听，从而改变我们自己改变的状态机（`state`）之后在重新渲染视图。

#### 6.react为什么分包，而vue没有

react官方提倡的一个方式，把核心实现逻辑和平台差异的逻辑解耦，方便去做跨平台。

react-router是平台无关的，主要核心是处理内存中组件匹配和渲染的过程，挂载context过程。

react-router-dom处理浏览器相关的，history对象等

react-native 移动端







