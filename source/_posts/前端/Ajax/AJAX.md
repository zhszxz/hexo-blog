---
title: AJAX
tags:
  - AJAX
  - axios
  - fetch
  - Promise
categories:
  - 前端
  - Ajax
date: 2026-10-14 09:00:00
---

# AJAX

AJAX（Asynchronous JavaScript And XML）的核心思想：浏览器用 JS **异步**地发 HTTP 请求，拿到数据后**局部更新页面，不刷新整页**。

这一篇完整覆盖 AJAX 的整套知识：**Promise（异步基础）→ XMLHttpRequest（经典实现）→ fetch（现代原生）→ async/await（优雅写法）→ axios（最常用库）**。

> Java 对照：`axios` ≈ 后端的 `RestTemplate` / `OkHttp` / `HttpClient`；`Promise` ≈ `CompletableFuture`。

<!--more-->

## 一、AJAX 概述

### 1. 为什么需要异步

```js
// 同步（阻塞）：请求期间页面卡死
console.log("1");
同步请求();     // 假设要等 3 秒
console.log("2");

// 异步（不阻塞）
console.log("1");
异步请求(() => console.log("2"));  // 完成后才执行
console.log("3");
// 输出顺序：1 → 3 → 2
```

### 2. 一次请求的流程

```
客户端 JS 发请求 → 服务器处理 → 返回响应（JSON）→ 前端解析并更新页面
```

> 数据格式：现在统一用 **JSON**，不再用 XML（AJAX 里的 "X" 只是历史遗留的名字）。

## 二、Promise（异步基础）

fetch 和 axios 都返回 Promise，所以先理解它。

### 1. 为什么需要 Promise：回调地狱

```js
getData(function (a) {
  getData2(a, function (b) {
    getData3(b, function (c) {
      console.log(c);   // 嵌套越来越深
    });
  });
});
```

### 2. 基本用法

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("成功");   // 成功时调用
    // reject("失败"); // 失败时调用
  }, 1000);
});

p.then(result => console.log(result))   // "成功"
 .catch(err => console.log(err))        // "失败"
 .finally(() => console.log("无论成败都执行"));
```

### 3. 三种状态

`pending`（进行中）→ `fulfilled`（成功）/ `rejected`（失败），状态一旦改变**不可逆**。

### 4. 链式调用与组合

```js
fetch("/user")
  .then(res => res.json())            // 返回新 Promise
  .then(user => fetch(`/user/${user.id}`))
  .then(res => res.json())
  .then(detail => console.log(detail))
  .catch(err => console.log(err));    // 一个 catch 兜底整条链

Promise.all([p1, p2]).then(results => { /* 全部成功 */ });
Promise.race([p1, p2]).then(result => { /* 谁先完成用谁 */ });
```

## 三、XMLHttpRequest（XHR，经典实现）

XHR 是 AJAX 最基础的实现，回调风格。

### 1. GET 请求（四步）

```js
// 1. 创建对象
const xhr = new XMLHttpRequest();

// 2. 配置请求（方法 + URL）
xhr.open("GET", "https://api.example.com/users");

// 3. 注册回调
xhr.onload = function () {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.onerror = function () {
  console.log("请求失败");
};

// 4. 发送
xhr.send();
```

### 2. POST 请求

```js
const xhr = new XMLHttpRequest();
xhr.open("POST", "https://api.example.com/users");
xhr.setRequestHeader("Content-Type", "application/json");   // 设请求头
xhr.onload = function () {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.send(JSON.stringify({ name: "Tom", age: 18 }));          // 请求体
```

### 3. 关键属性

| 属性 | 说明 |
| --- | --- |
| `xhr.readyState` | 状态 0~4，4 表示完成 |
| `xhr.status` | HTTP 状态码（200 成功） |
| `xhr.responseText` | 响应文本（需 JSON.parse） |

> XHR 是回调风格、写法繁琐，现已被 `fetch` 取代，但理解它有助于明白请求的本质。

## 四、fetch（现代原生 API）

`fetch` 基于 Promise，是现代浏览器的标准请求方式。

### 1. GET 请求

```js
fetch("https://api.example.com/users")
  .then(res => res.json())      // 解析响应为 JSON
  .then(data => console.log(data));
```

### 2. POST 请求

```js
fetch("https://api.example.com/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Tom", age: 18 })
})
  .then(res => res.json())
  .then(data => console.log(data));
```

### 3. 响应对象

| 属性/方法 | 说明 |
| --- | --- |
| `res.json()` | 解析为 JSON 对象 |
| `res.text()` | 解析为文本 |
| `res.status` | 状态码 |
| `res.ok` | 是否成功（200~299） |

> ⚠️ **fetch 最大的坑**：HTTP 错误（404、500）**不会自动 reject**，`fetch` 只在网络故障时 reject，所以要手动检查 `res.ok`：

```js
fetch(url)
  .then(res => {
    if (!res.ok) throw new Error("请求失败 " + res.status);
    return res.json();
  })
  .then(data => console.log(data))
  .catch(err => console.log(err));
```

## 五、async / await（fetch 的优雅写法）

`async/await` 是 Promise 的语法糖，让异步代码像同步一样写。

```js
async function getUsers() {
  const res = await fetch("https://api.example.com/users");
  if (!res.ok) throw new Error("请求失败");
  const data = await res.json();
  return data;      // async 函数返回的是 Promise
}

getUsers().then(data => console.log(data));
```

错误处理用 `try/catch`：

```js
async function getUsers() {
  try {
    const res = await fetch(url);
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.log("出错", err);
  }
}
```

> ⚠️ **易错点**：`async` 函数**必须 `await` 才能拿到值**；`await` 只能写在 `async` 函数内部。

## 六、axios（最常用的第三方库）

axios 封装了 fetch/XHR，功能更强、写法更简洁，是项目的首选。

### 1. 引入

```html
<!-- CDN -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

```bash
# 或 npm
npm install axios
```

```js
import axios from "axios";
```

### 2. GET 请求（自动 JSON 解析）

```js
axios.get("https://api.example.com/users")
  .then(res => console.log(res.data))   // 直接 .data，无需手动 parse
  .catch(err => console.log(err));      // HTTP 错误会自动 reject
```

带查询参数：

```js
axios.get("https://api.example.com/users", {
  params: { id: 1, page: 2 }
});
// 实际请求 /users?id=1&page=2
```

### 3. POST 请求（自动序列化对象）

```js
axios.post("https://api.example.com/users", {
  name: "Tom",
  age: 18
})  // 对象自动转 JSON，无需手动 stringify
  .then(res => console.log(res.data));
```

### 4. 全局配置与拦截器

```js
// 全局默认配置
axios.defaults.baseURL = "https://api.example.com";
axios.defaults.timeout = 5000;   // 5 秒超时

// 请求拦截器（发请求前统一处理，如加 token）
axios.interceptors.request.use(config => {
  config.headers.Authorization = "Bearer token";
  return config;
});

// 响应拦截器（统一处理响应/错误）
axios.interceptors.response.use(
  res => res,
  err => {
    console.log("请求失败", err);
    return Promise.reject(err);
  }
);
```

### 5. 为什么用 axios（对比 fetch）

| 特性 | fetch | axios |
| --- | --- | --- |
| JSON 解析 | 手动 `res.json()` | 自动 `res.data` |
| 对象序列化 | 手动 `JSON.stringify` | 自动 |
| HTTP 错误 | 不自动 reject，手动查 `res.ok` | 自动 reject |
| 拦截器 | 无 | 有（请求/响应） |
| 超时/取消/进度 | 部分支持 | 内置支持 |

## 七、XHR / fetch / axios 对比与选择

| | XHR | fetch | axios |
| --- | --- | --- | --- |
| 风格 | 回调 | Promise | Promise |
| JSON 解析 | 手动 | 手动 | 自动 |
| 错误处理 | 手动判断 status | 手动查 res.ok | 自动 reject |
| 拦截器 | 无 | 无 | 有 |
| 依赖 | 原生 | 原生 | 第三方库 |

> 选择建议：**学习理解用 fetch（原生），真实项目用 axios（省心）**；XHR 了解即可。

## 八、速查表

| 需求 | 写法 |
| --- | --- |
| 创建 Promise | `new Promise((resolve, reject) => {})` |
| 成功/失败回调 | `p.then(onOk).catch(onErr)` |
| XHR GET | `xhr.open("GET", url)` + `xhr.send()` |
| XHR POST | `xhr.setRequestHeader(...)` + `xhr.send(body)` |
| fetch GET | `fetch(url).then(res => res.json())` |
| fetch POST | `fetch(url, { method:"POST", body: JSON.stringify(obj) })` |
| 检查请求成功 | `if (!res.ok) throw new Error(...)` |
| async 请求 | `const data = await (await fetch(url)).json()` |
| axios GET | `axios.get(url).then(res => res.data)` |
| axios POST | `axios.post(url, obj).then(res => res.data)` |
| axios 拦截器 | `axios.interceptors.request.use(fn)` |

## 九、复习要点

1. **AJAX** = 异步请求 + 无刷新局部更新；数据用 JSON。
2. **Promise** 解决回调地狱：三状态不可逆，`then/catch/finally` 链式，`all/race` 组合。
3. **XHR** 四步（open → 回调 → send），回调式，已被 fetch 取代。
4. **fetch** 返回 Promise，用 `res.json()` 解析；⚠️ HTTP 错误不自动 reject，要查 `res.ok`。
5. **async/await** 是 Promise 语法糖，`try/catch` 捕获错误，`await` 只能在 async 函数内。
6. **axios** 自动 JSON 解析/序列化、自动 reject、有拦截器，是项目首选。
7. 学习用 fetch，项目用 axios，XHR 了解即可。
8. 忘了某段，回本文对应小节看示例即可。
