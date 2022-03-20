# 开始学习html5

### \<head>标签里有什么？

#### 1. 元数据\<meta>

```html
<meta charset="utf-8">
```

这个元素简单的指定了文档的字符编码 —— 在这个文档中被允许使用的字符集。 `utf-8` 是一个通用的字符集，它包含了任何人类语言中的大部分的字符。 意味着该 web 页面可以显示任意的语言；

```html
<meta name="author" content="Chris Mills">
```

- `name` 指定了meta 元素的类型； 说明该元素包含了什么类型的信息。
- `content` 指定了实际的元数据内容。

#### 2.\<link>

```html
<link rel="stylesheet" href="my-css-file.css">//引入样式表
```

```html
<link rel="apple-touch-icon-precomposed" sizes="144x144" href="https://developer.mozilla.org/static/img/favicon144.png"> 为ipad单独引入icon图标
```

#### 3.\<html>

```html
<html lang="zh-CN"> 设置主语言
```

### 4.scrollHeight

scrollHeight：盒子中，内容的高度，包括超出盒子高的的滑动内容，不包括边距，scrollTop：即滑动内容的上边沿高度，如果元素不能滑动，那么top为0

clientHeight：

