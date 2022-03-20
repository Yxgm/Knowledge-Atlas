### mvc模式，单向数据流

`control`注册`modal`层注入的方法(改变`modal`的方法)，`view`向`modal`注册（观察者模式）当`model`更新就会通知`view`更新时图。

`view`视图进行变动，不会直接操作`modal`，`control`是做一个中转，而是通过`control`去接受一些`view`的变动，然后让`control`去触发`modal`的改变。modal的改变从而触发`view`的更新

> react：唯一公式UI=Render（data），专注于视图的渲染，实际并不是mvc模式，只是一个library库，只是对于视图的变动，更新视图，只算是`view`的一部分

### mvvm模式，双向数据流

更多的是处理view和model之间的关系数据是双向的，mvvm模式（viewmode将这两者解耦）

>  vue：基于mvvm模式之上，支持了一个ref直接操作dom的方式而形成了的框架。严格意义上来说，不是mvvm模式，（vue中有一个ref，直接操作了DOM，跳过了ViewModal），没有完全符合mvvm标准。
>
> **自己引出问题**：vue是什么，他只是我们操作dom的一个中间层，他将数据响应式，不必让我们自己去处理数据变化后的事情了（也就是页面刷新），vue帮我们做了。

