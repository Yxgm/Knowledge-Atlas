## XSS攻击

#### 反射型：

用户在可以输入的位置，输入非法标签代码，跨站点注入了`script`

- XSS 脚本来自当前 HTTP 请求
- 当服务器在 HTTP 请求中接收数据并将该数据拼接在 HTML 中返回

#### 存储型：

后端数据库注入`script`，存储非法代码

- XSS 脚本来自服务器数据库中
- 攻击者将恶意代码提交到目标网站的数据库中，普通用户访问网站时服务器将恶意代码返回，浏览器默认执行

#### dom型：

也为反射型xss攻击，只是在前端渲染，不会和后端交互

点击url，对url植入代码，用户点击后执行代码

### XSS攻击预防

#### 预防 DOM 型 XSS 攻击

> DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。
>
> 如果用 Vue/React 技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML` 功能，就在前端 render 阶段避免 `innerHTML`、`outerHTML` 的 XSS 隐患。
>
> DOM 中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等，`<a>` 标签的 `href` 属性，JavaScript 的 `eval()`、`setTimeout()`、`setInterval()` 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

#### 预防存储型和反射型 XSS 攻击

> 存储型和反射型 XSS 都是在服务端取出恶意代码后，插入到响应 HTML 里的，攻击者刻意编写的“数据”被内嵌到“代码”中，被浏览器所执行。
>
> 预防这两种漏洞，有两种常见做法：
>
> - 改成纯前端渲染，把代码和数据分隔开。
> - 对 HTML 做充分转义。




### React的jsx预防xss攻击：

####   如何解决：

1. react代码中，会对渲染前的代码进行转义，就算是注入了非法代码，浏览器请求回来也不法正常解析，解决了xss攻击。react本身有这个能力
2. jsx经过babel转化为reactElement，每一个元素都会被react.createElement函数转化，会表示上一个唯一的symbol标识，只识别react元素。而xss注入的元素是无法被react识别的。就算是xss模仿了一个一摸一样的形似元素进行植入，但是被json转化后，symbol将会丢失。

#### 反模式的问题

开发时最好避免使用 `dangerouslySetInnerHTML`，如果不得不使用的话，前端或服务端必须对输入进行相关验证，例如对特殊输入进行过滤、转义等处理。前端这边处理的话，推荐使用[白名单过滤](https://jsxss.com/zh/index.html)，通过白名单控制允许的 HTML 标签及各标签的属性。

## Vue模版语法预防xss攻击：

1. 无论是使用模板还是渲染函数，内容都会自动转义。



<iframe src="https://v3.cn.vuejs.org/guide/security.html#%E9%A6%96%E8%A6%81%E8%A7%84%E5%88%99-%E6%B0%B8%E8%BF%9C%E4%B8%8D%E8%A6%81%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%8F%97%E4%BF%A1%E4%BB%BB%E7%9A%84%E6%A8%A1%E6%9D%BF" scrolling="yes" border="0" frameborder="no" framespacing="0" allowfullscreen="true" height="1000" width="1500"></iframe>