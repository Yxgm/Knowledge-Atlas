### ref:

因为proxy不能对基本类型做代理，所有ref就是包装了一层RefImpl对象，RefImpl的value值还是proxy

本质就是reactive

```js
ref(0)=reactive({value:0})
```

对于vue3的响应式来说，除了proxy代理以外的区别，就是存储副作用函数数据结构的不同，使用了weakMap，一个reactive对应一个map，map中一个属性的key，又对应着一个集合结构的副作用函数。

2版本中，使用的watcher上的静态属性，保持了watcher的唯一性，3只使用了一个activeEffect全局唯一常量替代而已

