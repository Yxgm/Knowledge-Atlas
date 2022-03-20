### 一般组件和路由组件之间的区别：

1. 写法不同：一般组件<Demo/> 路由组件：<Route path="/demo" component={Demo}>

2. 存放位置不同：一般组件：component 路由组件：pages

3. 路由组件，在props中会接受三个参数：

   | 属性                                                         |      |      |
   | ------------------------------------------------------------ | ---- | ---- |
   | history    : ƒ go(n)   ƒ goBack()  ƒ goForward() ƒ push(path, state) ƒ replace(path, state) |      |      |
   | loaction :pathname: "/about"	search: 		state: undefined |      |      |
   | match    :params: {}.                 path: "/about"	url: "/about" |      |      |

只有路由组件<NavLink>,<Link>拥有this.props.history这个属性，可以进行一些基于historyapi的一些push等等操作

1. 使用withRouter高阶组件，将普通组件传入，就可以拥有这三个属性，当使用通过withRouter包裹的组件，需要被<BrowerRouter>标签包裹
2. 因为在<BrowerRouter>中有history等等属性，实际上在源码中的做法是给BrowerRouter这个组件原型上加上了render函数，当组件实例化的时候，就会执行render函数，加上了history属性，location和match属性也是如此。

### 路由传值

1. 如果是简单的数据，可以使用动态路由，定义路由的时候，不把path写死，后面加上：id等 

   eg： <Route path="/detail/:id" component={}/> 在路由组件中，在match中拿到参数

2. 使用location的search传递，是直接拼接在路由的path路径中

   eg：<Route path="/detail？name="sxy" component={}/> 

   如果使用？拼接的url传递到服务器的话这串字符串是query参数， 如果是前端路由的话，是在location中的search参数

3. 复杂数据，使用<NavLink to={..}/>  to后面加对象，会传递到路由组件的this.props.loaction属性中



> renderRoutes函数，只有使用了renderRoutes函数渲染出来的路由映射关系，组件的props才有route这个属性.
>
> renderRoutes函数源码实现中，也是生成的<Switch/> <Route/>标签

<img src="../assets/01.jpg" style="width:600px"/>