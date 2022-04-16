#### 动画：

```css
/*animation (动画) :绑定选择器, 3s完成动画 linear(匀速) infinite(循环) */
  animation: loading 3s linear infinite
```

#### 滚动：

```css
只有内容高度大于容器本身的高度，才会触发滚动，并且容器还需要设置一些属性
height:600px;  ！!现实设置高度
overflow-y: scroll；或者
overflow-x: scroll
```

#### 单行设置省略号：文字内容长度超出容器宽度（文本溢出），需要显式设置宽度，超出部分隐藏，并且不换行

```css
text-overflow: ellipsis;
overflow: hidden;
white-space: nowrap;
width: 100%;
多行省略：
-webkit-line-clamp:2; (两行文字)
-webkit-box-orient:vertical;
```

#### 可以继承的css属性

```css
1、字体系列属
font：组合字体	font-family：规定元素的字体系列	font-weight：设置字体的粗细	font-size：设置字体的尺寸	font-style：定义字体的风格
2、文本系列属性
text-indent：文本缩进 											text-align：文本水平对齐					text-shadow：设置文本阴影			line-height：行高
word-spacing：增加或减少单词间的空白（即字间隔）text-transform：控制文本大小写
direction：规定文本的书写方向color：文本颜色
3、元素可见性：visibility
4、表格布局属性：caption-side、border-collap、seempty-cells
5、列表属性：list-style-type、list-style-image、list-style-position、list-style
6、设置嵌套引用的引号类型：quotes
7、光标属性：cursor
8、页面样式属性：page、page-break-inside、windows、orphans
```

#### flex:5 的意义是什么

小程序适配布局（用rpx和flex布局来做，动态决定样式的长度，之中有图片的话，具体情况使用mode属性，设置高宽）

```css
flex:1;		flex:5;   
对于一个内容，去除一些固定的长度（没有写flex的），剩余的长度，根据flex总和占比来区分，对于移动端适配是经常手段
```

#### Grid响应式布局

```css
grid-template-columns: repeat(var(--columnsNum), 1fr);
动态决定珊格布局的列数，用css变量从外界传入 1fr代表总体占比，也就是每个item平分容器宽度
```

```css
grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
响应式布局，规定item最小的width，如果容器不够放在一行item，会自动换行
```

#### 两栏布局：

##### grid布局

```css
.wrapper { display: grid; grid-template-columns: 70% 30%;}
```

##### flex布局

```css
.wrapper {display:flex;} 
.item1 {width: 400px}
.item2 {flex: 1} 
flex:1 代表伸缩自如，不设置flex-basic下，item的width为自适应，
⚠️如果在响应式下，想让某一个item的width不被压缩的话，flex-shrink设置为0（不缩）
```

#### 三栏布局：

##### grid布局

```css
.wrapper { display: grid; grid-template-columns: 100px 1fr 100px} 
根据不同的需求设置item占比，也可使用minmax（100px，1fr）规定minWidth等
```

#### 背景图片撑开：

 background-size: 100% 100%;
