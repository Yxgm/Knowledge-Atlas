### TypeScript 理解

###### 1. interface 定义之后，如果有范型继承接口，或者别的接口继承，一定要实现，否则会报错，在子容器中一定要实现继承的属性

###### 2. unknown 和 any 的主要区别是 unknown 类型会更加严格：在对 unknown 类型的值执行大多数操作之前，我们必须进行某种形式的检查。而在对 any 类型的值执行操作之前，我们不必进行任何检查。

### type和interface

#### 不同之处

> 1. type（类型别名）:侧重于描述数据，可以使用运算符（& ｜）来实现联合类型或者交叉类型
> 2. interface：重于描述数据结构，使用 extends 来继承 type 或者 interface 来拓展自己的类型，但是如何他想使用或的这个关系，可以使用可选类型来用
> 3. 联合类型（一次只能使用一个类型，或的关系 |），或者交叉类型（每次是合并后的类型且的关系 &）
> 4. 可以使用 typeOf 去得到自己想要的一个类型，基于一个已有的函数或者对象来获得这个类型**
>    type 可以给基本类型别名

#### 相同之处

> 1. 都可以描述一个对象或者函数 </br>
>2. 都允许拓展（extends）

###### 4. 类型断言：缩小类型，明示 ts 编译器，这个变量的类型我是明确知道的，绕过 ts 的类型检查

###### 5. 类型约束的时候，实参传入函数的时候，必须要保证传递明确声明的类型，（extends 等），也可以传入一些没有定义的类型



### 条件类型（U ? X : Y）

条件类型的语法规则和三元表达式一致，经常用于一些类型不确定的情况。

```typescript
T extends U ? X : Y //意思就是，如果 T 是 U 的子集，就是类型 X，否则为类型 Y。
```

### Partial\<A> 将 A 类型中的属性全部转化为可选类型

```typescript
type Partial<T> = {
    [P in keyof T]?: T[P]
}
```

`Partial` 用于将一个接口的所有属性设置为可选状态，首先通过 `keyof T`，取出类型变量 `T` 的所有属性，然后通过 `in` 进行遍历，最后在属性后加上一个 `?`。

### Required\<A> 将 A 类型中的属性全部转化为必选类型

```typescript
type Record<K extends keyof any, T> = {
    [P in K]: T
}
```



### Pick主要用于提取接口的某几个属性。

```typescript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P]
}
interface Todo {title: string completed: boolean description: string}
type TodoPreview = Pick<Todo, "title" | "completed">
const todo: TodoPreview = {
  title: 'Clean room',
  completed: false
}
```

`Pick` 主要用于提取接口的某几个属性。做过 Todo 工具的同学都知道，Todo工具只有编辑的时候才会填写描述信息，预览的时候只有标题和完成状态，所以我们可以通过 `Pick` 工具，提取 Todo 接口的两个属性，生成一个新的类型 TodoPreview。

### Omit

```typescript
type Omit<T, K extends keyof any> = Pick<
  T, Exclude<keyof T, K>
>
interface Todo {
  title: string
  completed: boolean
  description: string
}

type TodoPreview = Omit<Todo, "description">

const todo: TodoPreview = {
  title: 'Clean room',
  completed: false
}
```

`Omit` 的作用刚好和 Pick 相反，先通过 `Exclude<keyof T, K>` 先取出类型 T 中存在，但是 K 不存在的属性，然后再由这些属性构造一个新的类型。还是通过前面的 Todo 案例来说，TodoPreview 类型只需要排除接口的 description 属性即可，写法上比之前 Pick 精简了一些。





### `Extract`      type Extract<T, U> = T extends U ? T : never;

如果 T 中的类型在 U 存在，则返回，否则抛弃。假设我们两个类，有三个公共的属性，可以通过 Extract 提取这三个公共属性。

```tsx
type CommonKeys = Extract<keyof Worker, keyof Student>   // 'name' | 'age' | 'email'
```

### `Exclude`     type Exclude<T, U> = T extends U ? never : T

```typescript
type ExcludeKeys = Exclude<keyof Worker, keyof Student> // 'salary'
```

与`Extract` 刚好相反，如果 T 中的类型在 U 不存在，则返回，否则抛弃。现在我们拿之前的两个类举例，看看 `Exclude` 的返回结果。

```ts
例子🌰：
interface Worker {name: stringage age: number email:string salary: number}
interface Student {name: string age: number email: string grade: number}
```

