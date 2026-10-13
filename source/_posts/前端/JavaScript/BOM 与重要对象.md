---
title: BOM
tags:
  - BOM
  - window
  - localStorage
categories:
  - 前端
  - JavaScript
date: 2026-09-30 21:00:00
---

# BOM

BOM（Browser Object Model，浏览器对象模型）是浏览器提供的、用来**操作浏览器本身**的对象集合，`window` 是它的顶层对象。

这一篇讲 BOM 的重要对象：`window`、`location`、`history`、`navigator`、`screen`，以及本地存储 `localStorage / sessionStorage`（重点）。

<!--more-->

## 一、window 对象

`window` 是**全局对象**，所有全局变量和全局函数都是 `window` 的属性和方法。

```js
var name = "Tom";
console.log(window.name);   // "Tom"（全局变量挂在 window 上）
console.log(window === window.window);  // true
```

### 1. 常用方法

```js
alert("提示框");          // 弹窗提示
confirm("确定吗？");      // 确认框，返回 true / false
prompt("请输入");         // 输入框，返回输入字符串或 null

window.open("https://example.com");  // 打开新窗口
window.close();                       // 关闭窗口
window.scrollTo(0, 100);              // 滚动到指定位置（x, y）
```

### 2. 视口尺寸

```js
window.innerWidth;    // 视口宽度（不含滚动条）
window.innerHeight;   // 视口高度
```

### 3. 定时器（回顾）

```js
const id = setTimeout(() => { /* 延迟执行一次 */ }, 1000);
const id2 = setInterval(() => { /* 每隔一段时间重复 */ }, 1000);
clearTimeout(id);
clearInterval(id);
```

> 注意：`setTimeout`/`setInterval` 本质是 `window` 的方法，可省略 `window.`。

## 二、location 对象（URL 相关）

```js
location.href;           // 完整 URL
location.href = "https://example.com";   // 跳转页面
location.reload();       // 刷新页面

location.host;           // 主机名（含端口）
location.pathname;       // 路径
location.search;         // ? 后面的查询参数
location.hash;           // # 后面的锚点
```

**获取 URL 查询参数**（常用技巧）：

```js
// 当前 URL: https://a.com/page?id=123&name=tom
const params = new URLSearchParams(location.search);
params.get("id");    // "123"
params.get("name");  // "tom"
```

## 三、history 对象（前进后退）

```js
history.back();       // 后退一页
history.forward();    // 前进一页
history.go(-1);       // 后退一页（go(-2) 后退两页，go(1) 前进）
```

SPA 单页应用修改地址栏（不刷新页面）：

```js
history.pushState(null, "", "/new-page");  // 修改 URL 但不刷新
```

## 四、navigator 对象（浏览器信息）

```js
navigator.userAgent;   // 浏览器 UA 字符串（含浏览器、系统信息）
navigator.language;    // 浏览器语言，如 "zh-CN"
navigator.platform;    // 操作系统平台
navigator.onLine;      // 是否在线（布尔值）
```

## 五、screen 对象（屏幕信息）

```js
screen.width;    // 屏幕宽度（物理像素）
screen.height;   // 屏幕高度
```

## 六、本地存储（重点）

### 1. localStorage 与 sessionStorage

```js
// 存（只能存字符串）
localStorage.setItem("name", "Tom");
sessionStorage.setItem("name", "Tom");

// 取
localStorage.getItem("name");     // "Tom"

// 删
localStorage.removeItem("name");  // 删单个
localStorage.clear();             // 清空所有
```

两者的区别：

| | `localStorage` | `sessionStorage` |
| --- | --- | --- |
| 生命周期 | **持久**，手动删才消失 | 关闭标签页/浏览器即清除 |
| 作用域 | 同源所有标签页**共享** | 仅当前标签页 |
| 容量 | 约 5MB | 约 5MB |

### 2. 存对象必须 JSON 序列化

```js
const user = { name: "Tom", age: 18 };

// 存：对象 → 字符串
localStorage.setItem("user", JSON.stringify(user));

// 取：字符串 → 对象
const obj = JSON.parse(localStorage.getItem("user"));
console.log(obj.name);   // "Tom"
```

> ⚠️ **核心坑**：`localStorage` 只能存**字符串**，直接存对象会变成 `"[object Object]"`。所以存对象必须 `JSON.stringify`，取出来必须 `JSON.parse`。

### 3. 应用场景

- `localStorage`：记住登录状态、主题偏好、购物车（长期保存）。
- `sessionStorage`：表单临时数据、单次会话内的状态（关页即清）。

### 4. cookie 对比（了解）

| | cookie | localStorage |
| --- | --- | --- |
| 是否发给服务器 | 每次请求**自动携带** | 不自动发 |
| 容量 | 约 4KB | 约 5MB |
| 生命周期 | 可设过期时间 | 持久（手动删） |

> 记忆点：需要**服务器能读到**的小数据用 cookie；纯前端本地存大数据用 localStorage。

## 七、速查表

| 需求 | 写法 |
| --- | --- |
| 弹窗提示 | `alert("msg")` / `confirm("确定?")` / `prompt("输入")` |
| 视口尺寸 | `window.innerWidth` / `innerHeight` |
| 跳转页面 | `location.href = "url"` |
| 刷新页面 | `location.reload()` |
| 获取 URL 参数 | `new URLSearchParams(location.search).get("id")` |
| 后退/前进 | `history.back()` / `history.forward()` / `history.go(-1)` |
| 浏览器信息 | `navigator.userAgent` / `navigator.language` |
| 本地存储 | `localStorage.setItem("k", "v")` / `getItem("k")` |
| 删除存储 | `localStorage.removeItem("k")` / `clear()` |
| 存对象 | `setItem("k", JSON.stringify(obj))` |
| 取对象 | `JSON.parse(localStorage.getItem("k"))` |

## 八、复习要点

1. **window 是全局对象**，全局变量/函数都挂在它上面；定时器是 window 的方法。
2. **location** 管 URL：跳转 `href`、刷新 `reload`、参数 `search` + `URLSearchParams`。
3. **history** 管前进后退：`back/forward/go`，`pushState` 用于 SPA。
4. **navigator** 提供浏览器信息：`userAgent`、`language`、`onLine`。
5. **localStorage 持久、跨标签共享**；**sessionStorage 会话级、仅当前标签**。
6. 本地存储只能存**字符串**，存/取对象必须 `JSON.stringify` / `JSON.parse`。
7. 需要服务器读取的小数据用 cookie，纯前端大数据用 localStorage。
