---
title: DOM 基础与节点操作
tags:
  - DOM
categories:
  - 前端
  - JavaScript
date: 2026-09-24 19:00:00
---

# DOM 基础与节点操作

DOM（Document Object Model，文档对象模型）是浏览器把 HTML 解析成的一棵**节点树**，JS 通过 `document` 对象操作这棵树，就能动态改变页面内容。

这一篇把 DOM 的基础操作合并讲完：**获取元素、操作内容、操作属性与样式、节点关系、增删节点、表单操作**。这是 DOM 的地基，后面所有交互都建立在它之上。

<!--more-->

## 一、DOM 概念

```html
<!DOCTYPE html>
<html>
<body>
  <div id="box">hello</div>
</body>
</html>
```

浏览器会把它解析成一棵节点树：`document → html → body → div`，每个标签都是一个"节点（元素节点）"。

## 二、获取元素

```js
// 1. 按 id（返回单个元素，不存在返回 null）
document.getElementById("box");

// 2. 按标签名 / 类名（返回动态 HTMLCollection）
document.getElementsByTagName("div");
document.getElementsByClassName("item");

// 3. CSS 选择器（最常用，推荐）
document.querySelector(".item");        // 第一个匹配的元素
document.querySelectorAll(".item");     // 所有匹配，返回 NodeList
```

| 方法 | 参数 | 返回 | 特点 |
| --- | --- | --- | --- |
| `getElementById` | id | 单个元素 | 找不到返回 `null` |
| `getElementsByTagName/ClassName` | 标签/类名 | HTMLCollection | **动态**集合 |
| `querySelector` | CSS 选择器 | 第一个元素 | 最灵活 |
| `querySelectorAll` | CSS 选择器 | NodeList | **静态**集合 |

> ⚠️ **坑点**：`getElementsBy...` 返回的是**动态集合**（页面变化实时反映）；`querySelectorAll` 返回的是**静态快照**。遍历时动态集合会"越删越乱"，所以现代代码基本只用 `querySelector/querySelectorAll`。

## 三、操作元素内容

```js
el.innerText = "文本";              // 渲染后的文本，不识别标签
el.textContent = "文本";            // 原样文本（含隐藏元素）
el.innerHTML = "<b>加粗</b>";       // 解析并插入 HTML 标签
```

| 属性 | 说明 | 注意 |
| --- | --- | --- |
| `innerText` | 按渲染结果取文本 | 不识别标签，隐藏元素取不到 |
| `textContent` | 原样取所有文本 | 性能好，包含隐藏元素 |
| `innerHTML` | 解析 HTML | ⚠️ 有 **XSS 风险** |

> ⚠️ **XSS 风险**：`innerHTML` 会把字符串当 HTML 解析执行，**绝不要插入未经处理用户输入的内容**。纯文本用 `textContent` 更安全。

## 四、操作属性与样式

```js
// 1. 直接改常用属性
img.src = "1.jpg";
a.href = "https://example.com";
input.value = "hello";

// 2. classList 操作类名（最常用）
el.classList.add("active");
el.classList.remove("active");
el.classList.toggle("active");      // 有则删、无则加
el.classList.contains("active");    // 是否包含

// 3. style 内联样式（属性名用驼峰）
el.style.backgroundColor = "red";
el.style.fontSize = "20px";

// 4. 通用属性读写
el.getAttribute("href");
el.setAttribute("data-id", "1");
el.removeAttribute("data-id");

// 5. 自定义属性 data-*
// <div data-id="1" data-user="tom"></div>
el.dataset.id;    // "1"
el.dataset.user;  // "tom"
```

> ⚠️ `style` 的 CSS 属性名要转成**驼峰**：`background-color` → `backgroundColor`，`font-size` → `fontSize`。

## 五、节点关系

```js
el.parentNode;              // 父节点
el.parentElement;           // 父元素

el.children;                // 子元素集合（HTMLCollection）
el.childNodes;              // 所有子节点（含文本节点）

el.firstElementChild;       // 第一个子元素
el.lastElementChild;        // 最后一个子元素

el.nextElementSibling;      // 下一个兄弟元素
el.previousElementSibling;  // 上一个兄弟元素
```

> ⚠️ **坑点**：`childNodes` 会把**换行、空格也当成文本节点**，所以遍历子元素请用 `children`（只含元素节点）。

## 六、增删改节点

```js
// 1. 创建节点
const div = document.createElement("div");
div.textContent = "新节点";

// 2. 插入节点
parent.appendChild(div);              // 追加到末尾
parent.append(div, "追加文本");       // 现代写法，可追加多个/文本
parent.prepend(div);                  // 插到最前面
parent.insertBefore(div, ref);        // 插到 ref 节点之前

// 3. 删除节点
parent.removeChild(div);              // 删除子节点
div.remove();                         // 现代写法，删除自己

// 4. 克隆节点
const copy = el.cloneNode(true);      // true 深克隆（含子节点）

// 5. 替换节点
el.replaceWith(newEl);                // 用 newEl 替换自己
```

> 记忆点：`appendChild`/`insertBefore`/`removeChild` 是经典 API；`append`/`prepend`/`remove`/`replaceWith` 是更简洁的现代写法，功能等价，优先用后者。

## 七、表单操作

```js
// 获取值
input.value;         // 文本框内容
select.value;        // 下拉框选中的值
checkbox.checked;    // 复选框是否勾选（布尔值）
radio.checked;       // 单选是否选中

// 表单提交事件
form.addEventListener("submit", (e) => {
  e.preventDefault();       // 阻止表单默认提交（避免刷新页面）
  // 手动处理提交逻辑，如校验、发请求
});

// 其他表单事件
input.addEventListener("input", () => {});   // 每次输入触发
input.addEventListener("change", () => {});  // 值改变且失焦触发
input.addEventListener("focus", () => {});   // 获得焦点
input.addEventListener("blur", () => {});    // 失去焦点
```

原生表单验证：

```html
<input required>          <!-- 必填 -->
<input pattern="\d+">     <!-- 正则匹配 -->
<input type="email">      <!-- 邮箱格式 -->
```

> ⚠️ `submit` 事件一定要 `preventDefault()` 才能阻止页面刷新，这是做"无刷新提交"的第一步。

## 八、速查表

| 需求 | 写法 |
| --- | --- |
| 按 CSS 选择器获取 | `document.querySelector(".item")` / `querySelectorAll` |
| 按 id 获取 | `document.getElementById("box")` |
| 改文本内容 | `el.textContent = "x"` |
| 插入 HTML | `el.innerHTML = "<b>x</b>"` |
| 加/删类名 | `el.classList.add/remove("active")` |
| 切换类名 | `el.classList.toggle("active")` |
| 改样式 | `el.style.backgroundColor = "red"` |
| 读写属性 | `el.getAttribute` / `setAttribute` |
| 自定义属性 | `el.dataset.id` |
| 父元素 | `el.parentElement` |
| 子元素集合 | `el.children` |
| 创建节点 | `document.createElement("div")` |
| 追加节点 | `parent.append(el)` |
| 删除节点 | `el.remove()` |
| 克隆节点 | `el.cloneNode(true)` |
| 获取输入值 | `input.value` / `checkbox.checked` |
| 阻止表单提交 | `e.preventDefault()` |

## 九、复习要点

1. **DOM 是节点树**，`document` 是入口，改 DOM 页面立即生效。
2. 获取元素优先用 `querySelector/querySelectorAll`（静态、灵活）；`getElementsBy...` 是动态集合。
3. 改文本用 `textContent`（安全），改 HTML 用 `innerHTML`（⚠️ XSS 风险）。
4. 改类名用 `classList`，改内联样式用 `style`（驼峰），自定义属性用 `dataset`。
5. 遍历子元素用 `children`（不含文本节点），别用 `childNodes`。
6. 增删节点：`createElement` + `append/prepend/insertBefore`，删除用 `remove`。
7. 表单取值 `input.value`、`checkbox.checked`；提交用 `submit` 事件 + `preventDefault()`。
8. 忘了某个 API，回本文对应小节查示例即可。

下一篇预告：**事件**——`addEventListener`、事件对象、事件流（冒泡/捕获）、事件委托、常用事件类型。
