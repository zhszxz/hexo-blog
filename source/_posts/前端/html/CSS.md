---
title: CSS
tags:
  - CSS
categories:
  - 前端
  - html
date: 2026-09-03 21:20:00
---

# CSS

HTML 负责网页结构，CSS 负责网页样式。学习 CSS 的重点不是死记属性，而是理解“选中谁、改什么、改成什么样”。

这篇笔记覆盖 CSS 引入方式、选择器、文字样式、文本排版、继承与优先级、伪类伪元素、盒子模型、背景、阴影、过渡、文本省略、字体图标和精灵图等常用知识点。

<!--more-->

## 一、CSS是什么

CSS 全称是 Cascading Style Sheets，中文叫“层叠样式表”。

它主要解决这些问题：

- 文字显示成什么颜色、大小、字体。
- 盒子有多宽、多高、什么背景。
- 元素之间有多少间距。
- 鼠标移上去、输入框获得焦点时样式如何变化。
- 页面整体布局、卡片、按钮、导航栏、列表等如何呈现。

CSS 的基本语法如下：

```css
选择器 {
  属性名: 属性值;
  属性名: 属性值;
}
```

示例：

```css
p {
  color: red;
  font-size: 16px;
}
```

意思是：选中页面中所有 `p` 标签，把文字颜色设置为红色，字体大小设置为 `16px`。

## 二、CSS的三种引入方式

### 1. 内联样式表

直接写在 HTML 标签的 `style` 属性中，只影响当前这一个标签。

```html
<p style="color: pink;">我是粉色文字</p>
```

特点：

- 优先级高。
- 只适合临时测试或少量特殊样式。
- 不利于复用，真实项目中不建议大量使用。

### 2. 内部样式表

写在 HTML 文件的 `head` 标签中，用 `style` 包裹。

```html
<head>
  <style>
    div {
      color: blue;
    }

    strong {
      color: purple;
    }
  </style>
</head>
```

特点：

- 控制当前页面。
- 适合单个页面练习。
- 页面样式多了以后，HTML 会变得很长。

### 3. 外部样式表

把 CSS 单独写到 `.css` 文件中，再用 `link` 引入。

```html
<link rel="stylesheet" href="./css/my.css">
```

`my.css`：

```css
div {
  color: purple;
}
```

特点：

- 可以被多个页面复用。
- HTML 结构和 CSS 样式分离。
- 项目开发中最常用。

## 三、CSS注释

CSS 注释写法：

```css
/* 这是一段 CSS 注释 */
```

注释不会影响页面显示，常用于解释一段样式的作用。

## 四、基础选择器

选择器用来告诉浏览器“这条 CSS 规则要作用到谁身上”。

### 1. 标签选择器

按 HTML 标签名选择元素。

```css
div {
  color: pink;
}

li {
  color: blue;
}
```

特点：

- 会选中页面中所有同名标签。
- 适合统一设置某类标签的默认样式。

### 2. 类选择器

用 `.类名` 选择带有指定 `class` 的元素。

```css
.pink {
  color: pink;
}

.sub-nav {
  font-size: 20px;
}
```

HTML：

```html
<div class="pink">大春</div>
<div class="sub-nav">夏洛</div>
<div class="pink sub-nav">秋雅</div>
```

特点：

- 一个类可以给多个元素使用。
- 一个元素也可以拥有多个类名，类名之间用空格隔开。
- 项目中最常用的选择器之一。

### 3. ID选择器

用 `#id名` 选择指定元素。

```css
#pink {
  color: pink;
}
```

HTML：

```html
<div id="pink">夏洛</div>
```

特点：

- 一个页面中同一个 `id` 通常只给一个元素使用。
- 权重比类选择器高。
- 更适合唯一模块或 JavaScript 获取元素。

### 4. 通配符选择器

用 `*` 选择页面中所有元素。

```css
* {
  margin: 0;
  padding: 0;
}
```

常用于 CSS 初始化，去掉浏览器默认内外边距。

## 五、复合选择器

### 1. 后代选择器

用空格表示，选择某元素内部所有符合条件的后代。

```css
ul li a {
  color: red;
  font-size: 20px;
}

.footer a {
  color: pink;
}
```

`ul li a` 表示：选中 `ul` 里面的 `li` 里面的所有 `a`，不管嵌套多深。

### 2. 子代选择器

用 `>` 表示，只选择直接子元素。

```css
.box > span {
  color: green;
}
```

HTML：

```html
<div class="box">
  <span>我是儿子</span>
  <p>
    <span>我是孙子</span>
  </p>
</div>
```

上面的 CSS 只会选中 `.box` 的直接子元素 `span`，不会选中 `p` 里面的 `span`。

### 3. 邻接兄弟选择器

用 `+` 表示，选择紧挨在后面的一个兄弟元素。

```css
h1 + p {
  color: red;
}

.brother + li {
  color: red;
}
```

### 4. 通用兄弟选择器

用 `~` 表示，选择后面所有符合条件的兄弟元素。

```css
h2 ~ p {
  color: pink;
}

.brother ~ li {
  color: blue;
}
```

### 5. 分组选择器

用逗号把多个选择器合并，共用一组样式。

```css
.content img,
.content video {
  width: 100%;
}
```

适合减少重复代码。

## 六、属性选择器

属性选择器根据 HTML 属性来选择元素。

```css
/* 选择带有 class 属性的 a */
a[class] {
  color: red;
}

/* 选择 class 正好等于 font 的 a */
a[class="font"] {
  color: green;
}

/* 选择 class 以 font 开头的 a */
a[class^="font"] {
  color: blue;
}

/* 选择 class 以 14 结尾的 a */
a[class$="14"] {
  color: pink;
}

/* 选择 class 中包含 font 的 a */
a[class*="font"] {
  color: yellow;
}
```

常见写法：

| 选择器 | 含义 |
| --- | --- |
| `[attr]` | 选择有某个属性的元素 |
| `[attr="value"]` | 选择属性值等于指定值的元素 |
| `[attr^="value"]` | 选择属性值以指定内容开头的元素 |
| `[attr$="value"]` | 选择属性值以指定内容结尾的元素 |
| `[attr*="value"]` | 选择属性值包含指定内容的元素 |

案例：清除文本框和密码框的默认轮廓线。

```css
input[type="text"],
input[type="password"] {
  outline: none;
}
```

## 七、伪类选择器

伪类选择器用于选中元素的某种状态或某个位置。

### 1. 链接伪类

```css
a:link {
  color: #000;
  text-decoration: none;
}

a:visited {
  color: orange;
}

a:hover {
  color: red;
  text-decoration: underline;
}

a:active {
  color: blue;
}
```

| 伪类 | 状态 |
| --- | --- |
| `:link` | 未访问过的链接 |
| `:visited` | 已访问过的链接 |
| `:hover` | 鼠标悬停 |
| `:active` | 鼠标按下、链接被激活 |

建议顺序：`:link`、`:visited`、`:hover`、`:active`。

### 2. 用户行为伪类

```css
.box:hover {
  background-color: red;
  color: #fff;
}

.search:focus {
  background-color: red;
  width: 200px;
}
```

| 伪类 | 作用 |
| --- | --- |
| `:hover` | 鼠标经过时触发 |
| `:focus` | 输入框等元素获得焦点时触发 |

### 3. 结构伪类

```css
.ul1 li:first-child {
  color: red;
}

.ul1 li:last-child {
  color: blue;
}

.ul1 li:nth-child(3) {
  color: green;
}
```

`nth-child()` 常见取值：

| 写法 | 作用 |
| --- | --- |
| `nth-child(3)` | 选择第 3 个 |
| `nth-child(odd)` | 选择奇数项 |
| `nth-child(even)` | 选择偶数项 |
| `nth-child(3n)` | 选择 3 的倍数项 |
| `nth-child(n+4)` | 选择第 4 个及以后 |
| `nth-child(-n+3)` | 选择前 3 个 |

表格隔行变色常用结构伪类：

```css
.data tr:nth-child(odd) {
  background-color: #f9f9f9;
}

.data tr:hover {
  background-color: #f1f1f1;
}
```

### 4. 表单伪类

```css
button:disabled {
  background: #ccc;
  opacity: .4;
}

#agree:checked + label {
  color: #ff6900;
}
```

| 伪类 | 作用 |
| --- | --- |
| `:disabled` | 选择禁用状态的表单控件 |
| `:checked` | 选择被勾选的单选框或复选框 |

`#agree:checked + label` 的意思是：当 `id="agree"` 的复选框被选中时，选中它后面紧挨着的 `label`。

## 八、伪元素选择器

伪元素用于选中元素中的某一部分，或者额外创建一个“看得见但不在 HTML 中”的内容。

```css
p::first-line {
  color: #ff6900;
}

p::first-letter {
  color: red;
}

textarea::placeholder {
  color: red;
  font-size: 12px;
}

div::before {
  content: "我是";
  color: red;
}

div::after {
  content: "老师";
  color: red;
}
```

| 伪元素 | 作用 |
| --- | --- |
| `::first-line` | 选择第一行文字 |
| `::first-letter` | 选择第一个字母或文字 |
| `::placeholder` | 选择输入框占位提示文字 |
| `::before` | 在元素内容前插入内容 |
| `::after` | 在元素内容后插入内容 |

注意：`::before` 和 `::after` 必须写 `content` 属性，否则通常不显示。

## 九、CSS优先级和继承

### 1. 继承性

有些 CSS 属性会被子元素继承，例如：

- `color`
- `font-size`
- `font-family`
- `font-weight`
- `font-style`
- `line-height`
- `text-align`

示例：

```css
div {
  color: red;
  font-size: 25px;
  text-align: right;
}

p {
  color: pink;
}
```

如果 `p` 在 `div` 里面，`p` 会继承 `div` 的部分文字样式；但如果 `p` 自己设置了 `color: pink`，自己的样式会覆盖继承来的红色。

### 2. 优先级

当多个选择器同时选中同一个元素，并设置了同一个属性时，浏览器会比较权重。

常见权重从高到低：

| 写法 | 权重理解 |
| --- | --- |
| `style=""` | 行内样式，最高 |
| `#id` | ID 选择器 |
| `.class`、`:hover`、`[type="text"]` | 类、伪类、属性选择器 |
| `div`、`p`、`span` | 标签选择器 |
| `*`、继承 | 权重很低 |

示例：

```html
<div class="box" id="header" style="color: pink;">CSS权重优先级</div>
```

```css
#header {
  color: purple;
}

div {
  color: blue;
}

.box {
  color: green;
}
```

最终文字颜色是 `pink`，因为行内样式优先级最高。

### 3. `!important`

```css
.data tr:first-child {
  background-color: #8fbcf1 !important;
}
```

`!important` 可以强行提高某条声明的优先级，但不建议滥用。它会让后续维护变困难。

## 十、颜色、字体和文本

### 1. 颜色 color

```css
.pink {
  color: pink;
}

.color16 {
  color: #2CB5A5;
}

.rgb {
  color: rgb(255, 0, 0);
}

.rgba {
  color: rgba(255, 0, 0, 0.6);
}
```

常见取值：

| 写法 | 示例 | 说明 |
| --- | --- | --- |
| 颜色单词 | `red`、`pink`、`skyblue` | 简单直观 |
| 十六进制 | `#000`、`#fff`、`#2CB5A5` | 项目中常用 |
| RGB | `rgb(255, 0, 0)` | 红绿蓝三通道 |
| RGBA | `rgba(255, 0, 0, 0.6)` | 最后一个值表示透明度，范围 `0` 到 `1` |

### 2. 字体族 font-family

```css
.font {
  font-family: "微软雅黑";
}
```

项目中常见写法：

```css
body {
  font-family: Helvetica Neue, Helvetica, Arial, Microsoft Yahei, sans-serif;
}
```

浏览器会从左到右依次查找可用字体。

### 3. 字体大小 font-size

```css
.font12 {
  font-size: 12px;
}

.font16 {
  font-size: 16px;
}
```

常见单位：

| 单位 | 说明 |
| --- | --- |
| `px` | 像素，初学阶段最常用 |
| `em` | 相对当前元素字体大小 |
| `%` | 百分比 |

### 4. 字体倾斜 font-style

```css
.italic {
  font-style: italic;
}

.normal {
  font-style: normal;
}
```

常见取值：

- `normal`：正常。
- `italic`：倾斜。

### 5. 字体粗细 font-weight

```css
.bold1 {
  font-weight: bold;
}

.bold2 {
  font-weight: 700;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `normal` | 正常，约等于 `400` |
| `bold` | 加粗，约等于 `700` |
| `400` | 正常 |
| `700` | 加粗 |

### 6. 文本装饰 text-decoration

```css
.underline {
  text-decoration: underline;
}

.overline {
  text-decoration: overline;
}

.line-through {
  text-decoration: line-through;
}

a {
  text-decoration: none;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `none` | 无装饰，常用于取消 a 标签下划线 |
| `underline` | 下划线 |
| `overline` | 上划线 |
| `line-through` | 删除线 |

### 7. 文本对齐 text-align

```css
.box {
  text-align: center;
}

p {
  text-align: justify;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `left` | 左对齐 |
| `center` | 居中对齐 |
| `right` | 右对齐 |
| `justify` | 两端对齐 |

注意：`text-align: center` 可以让行内内容在父盒子中水平居中，例如 `p` 里面的 `span`。

### 8. 文本缩进 text-indent

```css
p {
  text-indent: 2em;
}
```

`2em` 表示缩进两个当前文字大小，中文段落首行缩进常用这个值。

### 9. 行高 line-height

```css
p {
  line-height: 26px;
}

.box {
  height: 50px;
  line-height: 50px;
}
```

常见用途：

- 控制多行文字的行距。
- 单行文字垂直居中：让 `line-height` 等于盒子高度。

### 10. 字间距 letter-spacing

```css
.box {
  letter-spacing: 5px;
}
```

控制文字之间的间距。

### 11. font复合写法

```css
body {
  font: italic 700 18px/1.5 "宋体";
}
```

顺序通常是：

```css
font: font-style font-weight font-size/line-height font-family;
```

注意：

- `font-size` 和 `font-family` 不能省略。
- `line-height` 写在 `font-size` 后面，用 `/` 分隔。

## 十一、页面基础布局

示例中的基础布局：

```css
* {
  margin: 0;
  padding: 0;
}

.header {
  height: 80px;
  background-color: black;
}

.nav {
  width: 80%;
  height: 60px;
  background-color: skyblue;
}

.main {
  width: 80%;
  height: 1000px;
  background-color: green;
}

.footer {
  height: 150px;
  background-color: purple;
}

.center {
  margin: 0 auto;
}
```

关键点：

- `width` 设置宽度。
- `height` 设置高度。
- `background-color` 设置背景色。
- `margin: 0 auto` 让有固定宽度或百分比宽度的块级盒子水平居中。

## 十二、盒子模型

每个 HTML 元素都可以看成一个盒子。盒子由四部分组成：

- 内容区：`width`、`height`
- 内边距：`padding`
- 边框：`border`
- 外边距：`margin`

### 1. 宽高 width 和 height

```css
.box {
  width: 200px;
  height: 200px;
}
```

常见取值：

- `px`：固定像素。
- `%`：相对父元素比例。
- `100%`：常用于图片、视频撑满父盒子宽度。

### 2. 边框 border

```css
.box1 {
  border: 1px solid red;
}

.box2 {
  border: 1px dashed red;
}

.box3 {
  border: 1px dotted red;
}

.box4 {
  border-right: 5px double pink;
}
```

复合写法：

```css
border: 边框宽度 边框样式 边框颜色;
```

常见边框样式：

| 取值 | 作用 |
| --- | --- |
| `solid` | 实线 |
| `dashed` | 虚线 |
| `dotted` | 点线 |
| `double` | 双线 |
| `none` | 无边框 |

单独设置某一边：

```css
border-top: 3px solid #ff8400;
border-bottom: 1px solid #d9e0ee;
border-left: 1px solid #d9e0ee;
border-right: 1px solid #d9e0ee;
```

### 3. 圆角 border-radius

```css
.circle {
  width: 100px;
  height: 100px;
  border-radius: 50%;
}

.btn {
  width: 200px;
  height: 40px;
  border-radius: 20px;
}
```

常见取值：

| 写法 | 作用 |
| --- | --- |
| `border-radius: 10px;` | 四个角都是 10px |
| `border-radius: 50%;` | 正方形变圆形 |
| `border-radius: 20px;` | 高度 40px 的盒子可变胶囊形 |
| `border-radius: 10px 30px;` | 左上/右下为 10px，右上/左下为 30px |
| `border-radius: 10px 30px 50px 70px;` | 顺序为左上、右上、右下、左下 |

### 4. 内边距 padding

```css
.box {
  padding: 10px;
  padding-left: 10px;
}
```

复合写法规律：

| 写法 | 含义 |
| --- | --- |
| `padding: 10px;` | 上右下左都是 10px |
| `padding: 10px 20px;` | 上下 10px，左右 20px |
| `padding: 10px 20px 30px;` | 上 10px，左右 20px，下 30px |
| `padding: 10px 20px 30px 40px;` | 上 10px，右 20px，下 30px，左 40px |

单独方向：

```css
padding-top: 10px;
padding-right: 20px;
padding-bottom: 30px;
padding-left: 40px;
```

### 5. 外边距 margin

```css
.box1 {
  margin-bottom: 10px;
}

.box {
  margin: 10px 20px 30px 40px;
}
```

`margin` 的复合写法规则和 `padding` 一样。

单独方向：

```css
margin-top: 10px;
margin-right: 20px;
margin-bottom: 30px;
margin-left: 40px;
```

### 6. 块级盒子水平居中

```css
.box {
  width: 150px;
  margin-left: auto;
  margin-right: auto;
}
```

也可以简写为：

```css
.box {
  width: 150px;
  margin: 0 auto;
}
```

前提：盒子必须有宽度，并且是块级盒子。

### 7. 行内盒子的特点

示例中 `span` 是行内盒子：

```css
span {
  width: 100px;
  height: 100px;
  margin: 10px 20px;
  padding: 10px;
  border: 5px solid red;
}
```

需要记住：

- 行内元素设置 `width`、`height` 通常无效。
- 行内元素上下 `margin` 通常不影响布局。
- 行内元素的左右 `margin`、`padding`、`border` 更容易生效。

### 8. 外边距合并和塌陷

相邻块级元素的上下外边距可能会合并：

```css
.box1 {
  margin-bottom: 100px;
}

.box2 {
  margin-top: 50px;
}
```

实际距离通常不是 `150px`，而是取较大的 `100px`。

父子元素也可能发生外边距塌陷：

```css
.father {
  width: 150px;
  height: 150px;
  background-color: pink;
  overflow: hidden;
}

.son {
  width: 80px;
  height: 80px;
  background-color: purple;
  margin-top: 20px;
}
```

常见解决方式：

- 给父元素加边框。
- 给父元素加内边距。
- 给父元素设置 `overflow: hidden`。

### 9. box-sizing

`box-sizing` 决定盒子尺寸如何计算。

```css
.box1 {
  width: 200px;
  height: 200px;
  box-sizing: content-box;
  border: 10px solid red;
  padding: 20px;
}

.box2 {
  width: 200px;
  height: 200px;
  box-sizing: border-box;
  border: 10px solid red;
  padding: 20px;
}
```

常见取值：

| 取值 | 说明 |
| --- | --- |
| `content-box` | 默认值，`width` 和 `height` 只包含内容区 |
| `border-box` | `width` 和 `height` 包含内容、内边距、边框 |

项目中常用初始化：

```css
* {
  box-sizing: border-box;
}
```

## 十三、背景相关属性

### 1. 背景颜色和背景图片

```css
.box {
  background-color: pink;
  background-image: url(./img/ldh.png);
}
```

### 2. 背景平铺 background-repeat

```css
.box {
  background-repeat: no-repeat;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `repeat` | 默认，水平和垂直都平铺 |
| `no-repeat` | 不平铺 |
| `repeat-x` | 水平方向平铺 |
| `repeat-y` | 垂直方向平铺 |

### 3. 背景位置 background-position

```css
.box {
  background-position: 50px 10px;
  background-position: 50% 50%;
  background-position: left top;
  background-position: right bottom;
  background-position: center center;
}
```

常见取值：

- 方位词：`top`、`bottom`、`left`、`right`、`center`
- 像素：`50px 10px`
- 百分比：`50% 50%`

如果只写一个值，另一个值默认是 `center`。

### 4. 背景缩放 background-size

```css
.box {
  background-size: cover;
  background-size: contain;
  background-size: 300px;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `cover` | 等比例缩放，铺满盒子，可能裁剪图片 |
| `contain` | 等比例缩放，完整显示图片，可能留空 |
| `200px` | 指定宽度，高度等比缩放 |
| `200px 100px` | 指定宽高 |

### 5. 背景固定 background-attachment

```css
.box {
  background-attachment: scroll;
  background-attachment: fixed;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `scroll` | 默认，背景跟随元素滚动 |
| `fixed` | 背景相对于视口固定 |

视差滚动示例：

```css
.section1 {
  width: 100%;
  height: 100%;
  background: url(./img/bg1.jpg) no-repeat fixed center center/cover;
}
```

### 6. background复合写法

```css
.box {
  background: pink url(./img/ldh.png) no-repeat center center/200px;
}
```

常见顺序可以理解为：

```css
background: 背景色 背景图 是否平铺 背景位置/背景尺寸;
```

## 十四、渐变、阴影、过渡和变形

### 1. 背景渐变

```css
.box {
  background: linear-gradient(to right, pink, purple);
}
```

也可以用角度：

```css
.box {
  background: linear-gradient(45deg, pink 50%, purple 50%);
}
```

常见写法：

| 写法 | 作用 |
| --- | --- |
| `to right` | 从左到右 |
| `to bottom` | 从上到下 |
| `45deg` | 45 度方向 |
| `pink 50%` | 色标位置 |

### 2. 文字渐变

```css
.text {
  font-size: 30px;
  font-weight: 700;
  background-image: linear-gradient(97deg, #0096FF, #BB64FF 42%, #F2416B 74%, #EB7500);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
```

核心思路：先给文字所在元素加渐变背景，再把背景按文字形状裁剪，最后让文字本身透明。

### 3. 鼠标指针 cursor

```css
.gradient-btn {
  cursor: pointer;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `default` | 默认箭头 |
| `pointer` | 小手，常用于按钮和链接 |
| `text` | 文本选择 |
| `move` | 移动 |
| `not-allowed` | 禁止 |

### 4. 盒子阴影 box-shadow

```css
.box:hover {
  box-shadow: 0 0 10px 10px rgba(0, 0, 0, 0.1);
}
```

语法：

```css
box-shadow: 水平偏移 垂直偏移 模糊距离 扩散距离 阴影颜色;
```

示例：

```css
.card:hover {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}
```

### 5. 过渡 transition

```css
.ai-button {
  transition: all 0.3s ease;
}
```

常见写法：

```css
transition: 属性名 持续时间 运动曲线;
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `all` | 所有可过渡属性 |
| `.3s` / `0.3s` | 过渡持续时间 |
| `ease` | 先快后慢，默认常用 |
| `linear` | 匀速 |

### 6. 变形 transform

```css
.ai-button:hover {
  transform: translateY(-2px);
}

.ai-button:active {
  transform: translateY(0);
}
```

示例中用 `translateY()` 让按钮悬停时向上移动一点，点击时回到原位。

## 十五、表格样式

### 1. 合并相邻边框 border-collapse

```css
table {
  width: 300px;
  border-collapse: collapse;
}

table,
td {
  border: 1px solid red;
}
```

常见取值：

| 取值 | 作用 |
| --- | --- |
| `separate` | 默认，边框分离 |
| `collapse` | 相邻边框合并 |

### 2. 表格隔行变色

```css
.data {
  border-collapse: collapse;
  border: 1px solid #f1f1f1;
  width: 400px;
  margin: 0 auto;
  text-align: center;
  font-size: 14px;
}

.data tr {
  height: 35px;
}

.data tr:first-child {
  background-color: #8fbcf1 !important;
  color: #fff;
}

.data tr:nth-child(odd) {
  background-color: #f9f9f9;
}

.data tr:hover {
  background-color: #f1f1f1;
}
```

这个案例综合使用了：

- `border-collapse`
- `border`
- `width`
- `margin: 0 auto`
- `text-align`
- `font-size`
- `height`
- `:first-child`
- `:nth-child(odd)`
- `:hover`

## 十六、CSS初始化

浏览器会给很多标签默认样式，例如 `body` 默认有外边距，`ul` 默认有小圆点，`a` 默认有下划线。

常见简单初始化：

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

ul,
ol {
  list-style: none;
}

a {
  text-decoration: none;
}

input,
button {
  outline: none;
}
```

相关属性：

| 属性 | 常见取值 | 作用 |
| --- | --- | --- |
| `margin` | `0` | 清除默认外边距 |
| `padding` | `0` | 清除默认内边距 |
| `box-sizing` | `border-box` | 让盒子尺寸更好计算 |
| `list-style` | `none` | 清除列表项目符号 |
| `text-decoration` | `none` | 清除链接下划线 |
| `outline` | `none` | 清除输入框/按钮焦点轮廓线 |

## 十七、文本溢出省略号

### 1. 单行文本省略

小米卡片案例中使用了单行省略：

```css
.desc {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

三个属性通常一起出现：

| 属性 | 取值 | 作用 |
| --- | --- | --- |
| `white-space` | `nowrap` | 强制文字不换行 |
| `overflow` | `hidden` | 隐藏溢出内容 |
| `text-overflow` | `ellipsis` | 溢出显示省略号 |

### 2. 多行文本省略

```css
.box {
  width: 200px;
  height: 48px;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

关键点：

- `-webkit-line-clamp: 2` 限制显示 2 行。
- 盒子高度要刚好容纳指定行数。
- 这是一种常见兼容写法，移动端和现代浏览器中很常用。

## 十八、字体图标

字体图标把图标当成字体使用，可以像文字一样设置大小和颜色。

引入图标字体 CSS：

```html
<link rel="stylesheet" href="./iconfont/iconfont.css">
```

使用：

```html
<span class="iconfont icon-good"></span>
<i class="iconfont icon-xihuan"></i>
```

修改样式：

```css
.icon-good {
  font-size: 140px;
  color: red;
}
```

`iconfont.css` 中的核心结构：

```css
@font-face {
  font-family: "iconfont";
  src: url("iconfont.ttf") format("truetype");
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
}

.icon-good::before {
  content: "\e673";
}
```

理解方式：

- `@font-face` 引入字体文件。
- `.iconfont` 指定使用这套图标字体。
- `.icon-good::before` 通过 `content` 显示某个图标编码。

## 十九、精灵图

精灵图就是把多个小图标放在一张图片里，通过背景定位显示其中一小块。

好处：减少图片请求次数。

```css
.box {
  width: 27px;
  height: 26px;
  background: url(./img/wz.webp) no-repeat;
}

.box1 {
  background-position: 0 -169px;
}

.box2 {
  background-position: -90px -170px;
}
```

关键点：

- 精灵图通常作为背景图使用。
- 盒子的 `width` 和 `height` 决定显示区域大小。
- `background-position` 调整图片位置，露出想要的图标。

## 二十、综合案例复盘

### 1. 登高赏析页面

示例中的《登高》页面综合使用了页面居中、背景色、内边距、图片/视频自适应、文本排版。

```css
* {
  margin: 0;
  padding: 0;
}

body {
  font: 14px/1.5 Helvetica Neue, Helvetica, Arial, Microsoft Yahei, sans-serif;
}

.box {
  width: 677px;
  background-color: #badef5;
  margin: 0 auto;
}

.content {
  width: 610px;
  background-color: #fff;
  margin: 0 auto;
  padding: 20px;
}

.content img,
.content video {
  width: 100%;
}

.content > p {
  line-height: 27px;
  text-indent: 2em;
  text-align: justify;
}

.shi {
  text-align: center;
  line-height: 27px;
  font-size: 16px;
  font-weight: bold;
}
```

这个案例适合练习：

- 页面整体居中：`margin: 0 auto`
- 内容留白：`padding`
- 图片和视频撑满容器：`width: 100%`
- 正文排版：`line-height`、`text-indent`、`text-align`
- 分组选择器：`.content img, .content video`
- 子代选择器：`.content > p`

### 2. 小米卡片

```css
.cart {
  width: 234px;
  height: 300px;
  background-color: #fff;
  margin: 100px auto;
  text-align: center;
  line-height: 25px;
}

.cart:hover {
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.cart img {
  width: 160px;
  height: 160px;
  margin-top: 20px;
}

.cart .desc {
  font-size: 12px;
  color: #999;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.cart .price {
  font-size: 14px;
  color: #ff4444;
}
```

这个案例适合练习：

- 卡片尺寸：`width`、`height`
- 卡片居中：`margin`
- 图片大小：`width`、`height`
- 鼠标悬停阴影：`:hover`、`box-shadow`
- 单行省略号：`white-space`、`overflow`、`text-overflow`

### 3. 淘宝侧边栏

```css
.service {
  width: 256px;
  height: 400px;
  background-color: #f7f7f7;
  border-radius: 12px;
  margin: 100px auto;
}

.service-bd li {
  height: 32px;
  line-height: 32px;
  margin: 0 8px;
  border-radius: 4px;
  transition: all .5s;
  padding-left: 10px;
}

.service-bd li:first-child,
.service-bd li:first-child a {
  color: #ff5000;
}

.service-bd li:hover {
  background-color: #fff;
}

.service-bd li a:hover {
  color: #ff5000;
}
```

这个案例适合练习：

- 列表初始化：`list-style: none`
- 链接初始化：`text-decoration: none`
- 圆角容器：`border-radius`
- 悬停效果：`:hover`
- 过渡动画：`transition`
- 字体图标：`iconfont`
- 后代选择器和分组选择器

## 二十一、常用CSS属性速查表

| 分类 | 属性 | 常见取值 | 作用 |
| --- | --- | --- | --- |
| 颜色 | `color` | `red`、`#333`、`rgb()`、`rgba()` | 文字颜色 |
| 颜色 | `background-color` | `pink`、`#fff`、`transparent` | 背景颜色 |
| 字体 | `font-family` | `"微软雅黑"`、`Arial`、`sans-serif` | 字体族 |
| 字体 | `font-size` | `12px`、`16px`、`1em` | 字体大小 |
| 字体 | `font-style` | `normal`、`italic` | 字体倾斜 |
| 字体 | `font-weight` | `normal`、`bold`、`400`、`700` | 字体粗细 |
| 字体 | `font` | `14px/1.5 Arial` | 字体复合写法 |
| 文本 | `text-align` | `left`、`center`、`right`、`justify` | 文本水平对齐 |
| 文本 | `text-indent` | `2em`、`20px` | 首行缩进 |
| 文本 | `line-height` | `26px`、`1.5` | 行高 |
| 文本 | `letter-spacing` | `5px` | 字间距 |
| 文本 | `text-decoration` | `none`、`underline`、`line-through` | 文本装饰 |
| 文本 | `white-space` | `nowrap` | 空白和换行处理 |
| 文本 | `overflow` | `hidden`、`visible` | 溢出处理 |
| 文本 | `text-overflow` | `ellipsis` | 文本溢出省略 |
| 盒子 | `width` | `200px`、`80%`、`100%` | 宽度 |
| 盒子 | `height` | `200px`、`100%` | 高度 |
| 盒子 | `border` | `1px solid red` | 边框 |
| 盒子 | `border-radius` | `10px`、`50%` | 圆角 |
| 盒子 | `padding` | `10px`、`10px 20px` | 内边距 |
| 盒子 | `margin` | `10px`、`0 auto` | 外边距 |
| 盒子 | `box-sizing` | `content-box`、`border-box` | 盒子尺寸计算方式 |
| 盒子 | `box-shadow` | `0 0 10px rgba(...)` | 阴影 |
| 背景 | `background-image` | `url(...)` | 背景图片 |
| 背景 | `background-repeat` | `repeat`、`no-repeat`、`repeat-x`、`repeat-y` | 背景平铺 |
| 背景 | `background-position` | `center`、`left top`、`50% 50%` | 背景位置 |
| 背景 | `background-size` | `cover`、`contain`、`200px` | 背景尺寸 |
| 背景 | `background-attachment` | `scroll`、`fixed` | 背景是否固定 |
| 背景 | `background` | `pink url(...) no-repeat center/cover` | 背景复合写法 |
| 列表 | `list-style` | `none` | 列表样式 |
| 表格 | `border-collapse` | `separate`、`collapse` | 表格边框是否合并 |
| 表单 | `outline` | `none` | 轮廓线 |
| 交互 | `cursor` | `pointer`、`default`、`not-allowed` | 鼠标指针 |
| 动画 | `transition` | `all .3s ease` | 过渡 |
| 变形 | `transform` | `translateY(-2px)` | 元素变形 |
| 透明度 | `opacity` | `0` 到 `1` | 元素整体透明度 |
| 显示 | `display` | `-webkit-box` | 显示模式，多行省略中用到 |

## 二十二、复习清单

学完这部分，可以用下面的问题自测：

- CSS 的基本语法是什么？
- 内联、内部、外部样式表分别怎么写，适合什么场景？
- 标签选择器、类选择器、ID 选择器、通配符选择器分别怎么写？
- 后代选择器和子代选择器有什么区别？
- `+` 和 `~` 两种兄弟选择器有什么区别？
- 属性选择器 `[attr]`、`[attr=value]`、`^=`、`$=`、`*=` 分别是什么意思？
- `:hover`、`:focus`、`:checked`、`:disabled` 分别在什么场景使用？
- `nth-child(odd)`、`nth-child(even)`、`nth-child(n+4)` 分别选中哪些元素？
- `::before` 和 `::after` 为什么必须写 `content`？
- CSS 权重中，行内样式、ID、类、标签谁更高？
- 哪些文字样式可以被继承？
- `color` 常见取值有哪些？
- `font` 复合写法的顺序是什么？
- 如何让单行文字在盒子中水平和垂直居中？
- 盒子模型包含哪四部分？
- `padding` 和 `margin` 的四值写法顺序是什么？
- 块级盒子如何水平居中？
- `content-box` 和 `border-box` 有什么区别？
- `background-size: cover` 和 `contain` 有什么区别？
- 如何写背景复合属性？
- 如何做单行和多行文本省略？
- 字体图标的基本使用步骤是什么？
- 精灵图为什么要配合 `background-position`？

## 二十三、建议练习

建议不看源码，自己完成一个“商品卡片 + 分类侧边栏”的练习页面：

- 使用外部 CSS 文件。
- 写一份 CSS 初始化。
- 使用类选择器、后代选择器、伪类选择器。
- 商品卡片包含图片、名称、描述、价格、原价。
- 描述文字单行省略。
- 卡片鼠标悬停时出现阴影。
- 侧边栏使用 `ul`、`li`、`a`，清除列表默认样式。
- 侧边栏每一行鼠标悬停时改变背景色。
- 使用 `border-radius` 做圆角。
- 使用 `transition` 让 hover 效果更自然。

能独立写出这个页面，CSS 入门阶段的常用选择器、盒子模型和基础视觉样式就基本稳了。
