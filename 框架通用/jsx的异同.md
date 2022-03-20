vue中template和render之间的关系：vue默认render的优先级更高，也就是会将template转换成render函数

vue和react，diff的时候vue是两侧向中间递归，react从左到右，手动挡（vue），自动挡（react）更定制化，没有双向绑定，

## react中的JSX：

### 预防xss攻击

##### 反射型：

用户输入标签，跨站点注入了script，

##### 存储型：

注入script，后端存储非法代码

> ####   如何解决：
>
> 1. react代码中，会对渲染前的代码进行转移，就算是注入了非法代码，浏览器请求回来也不法正常解析，解决了xss攻击。react本身有这个能力
>
> 2. jsx经过babel转化为reactElement，每一个元素都会被react.createElement函数转化，会表示上一个唯一的symbol标识，只识别react元素。而xss注入的元素是无法被react识别的。就算是xss模仿了一个一摸一样的形似元素进行植入，但是被json转化后，symbol将会丢失。

### dom攻击

后端处理
