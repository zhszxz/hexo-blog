---
title: vue-router 路由
tags:
  - Vue
  - Vue2
  - vue-router
  - 路由
categories:
  - 前端
  - Vue
date: 2026-09-13 13:00:00
---

# vue-router 路由

路由是 SPA（单页应用）的核心：**根据 URL 的变化，切换显示不同的组件**，页面不刷新。

这一篇讲 vue-router 的基本使用、路由高亮、导航传参、重定向与 404、路由模式、编程式导航。

> Java 对照：路由 ≈ 后端 Controller 的 URL 映射（`@RequestMapping`），但作用在**前端**，切换的是组件而不是后端接口。

<!--more-->

## 一、路由基本使用（5+2 步骤）

安装：`npm install vue-router@3`（Vue2 用 3.x）。

```js
// main.js
import Vue from "vue";
import VueRouter from "vue-router";
import App from "./App.vue";
import Find from "./views/Find.vue";
import My from "./views/My.vue";

Vue.use(VueRouter);                       // 1. 安装插件

const router = new VueRouter({            // 2. 创建路由实例
  routes: [                               // 3. 配置路由表
    { path: "/find", component: Find },
    { path: "/my", component: My }
  ]
});

new Vue({
  router,                                 // 4. 挂载到实例
  render: h => h(App)
}).$mount("#app");
```

```vue
<!-- App.vue：两个"出入口" -->
<template>
  <div>
    <router-link to="/find">发现</router-link>   <!-- 5. 导航链接 -->
    <router-link to="/my">我的</router-link>
    <router-view></router-view>                  <!-- 6. 组件渲染出口 -->
  </div>
</template>
```

> "5+2"：5 个步骤（安装、use、创建路由、配置 routes、挂载）+ 2 个组件（`router-link` 导航、`router-view` 出口）。

## 二、路由模块拆分

路由表放单独文件，`main.js` 只引入：

```js
// router/index.js
import Vue from "vue";
import VueRouter from "vue-router";
import Find from "../views/Find.vue";
import My from "../views/My.vue";

Vue.use(VueRouter);

export default new VueRouter({
  routes: [
    { path: "/find", component: Find },
    { path: "/my", component: My }
  ]
});
```

```js
// main.js
import router from "./router";
new Vue({ router, render: h => h(App) }).$mount("#app");
```

## 三、路由高亮与匹配

`router-link` 激活时自动加 `router-link-active`（模糊匹配）和 `router-link-exact-active`（精确匹配）类。

自定义高亮类名：

```js
new VueRouter({
  routes: [...],
  linkActiveClass: "active",              // 模糊匹配的高亮类
  linkExactActiveClass: "exact-active"    // 精确匹配的高亮类
});
```

- **模糊匹配**：`/find` 能匹配 `/find/xxx`（前缀匹配）。
- **精确匹配**：只有完全相等才匹配。

## 四、导航传参

### 1. query 传参（? 参数）

```html
<router-link to="/search?keyword=vue">搜索</router-link>
<!-- 对象写法 -->
<router-link :to="{ path: '/search', query: { keyword: 'vue' } }">搜索</router-link>
```

接收：

```js
this.$route.query.keyword;   // "vue"
```

### 2. params 动态路由传参（路径参数）

```js
// 路由表：冒号占位
{ path: "/search/:keyword", component: Search }
```

```html
<router-link to="/search/vue">搜索</router-link>
```

接收：

```js
this.$route.params.keyword;   // "vue"
```

### 3. 可选符 `?`

```js
{ path: "/search/:keyword?", component: Search }   // keyword 可省略
```

### query vs params 区别

| | query | params |
| --- | --- | --- |
| 传参方式 | `?keyword=vue` | 路径参数 `/search/vue` |
| 路由表配置 | 不需要改 | 需要 `:keyword` |
| 接收 | `$route.query` | `$route.params` |

## 五、重定向与 404

```js
routes: [
  { path: "/", redirect: "/home" },      // 重定向
  { path: "/home", component: Home },
  { path: "*", component: NotFound }     // 404 兜底（放最后）
]
```

## 六、路由模式 hash / history

```js
new VueRouter({
  mode: "history",   // 默认是 "hash"
  routes: [...]
});
```

| 模式 | URL | 特点 |
| --- | --- | --- |
| `hash` | `/#/find` | 兼容好，无需服务器配置 |
| `history` | `/find` | 干净美观，但刷新需服务器配置（否则 404） |

## 七、编程式导航

不用 `router-link`，用 JS 跳转：

```js
// 跳转（可后退）
this.$router.push("/find");
this.$router.push({ path: "/find" });
this.$router.push({ name: "find", params: { id: 1 } });

// 替换（不可后退）
this.$router.replace("/find");

// 前进后退
this.$router.back();
this.$router.go(-1);
```

> ⚠️ **$route vs $router 别搞混**：
> - `$route`：**当前路由信息**，用来**取参数**（`$route.query` / `$route.params`）。
> - `$router`：**路由实例**，用来**做跳转**（`$router.push` / `back`）。

## 八、速查表

| 需求 | 写法 |
| --- | --- |
| 创建路由 | `new VueRouter({ routes: [...] })` |
| 导航链接 | `<router-link to="/find">` |
| 渲染出口 | `<router-view>` |
| query 传参 | `:to="{ path:'/x', query:{k:v} }"` |
| params 传参 | `{ path:'/x/:id' }` + `/x/1` |
| 取参数 | `this.$route.query.k` / `this.$route.params.id` |
| 重定向 | `{ path:'/', redirect:'/home' }` |
| 404 | `{ path:'*', component: NotFound }` |
| history 模式 | `mode: "history"` |
| 编程式跳转 | `this.$router.push("/x")` |
| 后退 | `this.$router.back()` |

## 九、复习要点

1. 路由 = 根据 URL 切换组件，SPA 核心；Vue2 用 vue-router 3.x。
2. "5+2"：5 个步骤 + `router-link` / `router-view` 两个组件。
3. 路由表拆到 `router/index.js`，`main.js` 挂载。
4. **query 是 `?` 参数（不用改路由表），params 是路径参数（要 `:id`）**。
5. 404 用 `path: "*"` 兜底，放路由表**最后**。
6. `hash` 模式兼容好，`history` 模式干净但要服务器配置。
7. **`$route` 取参数，`$router` 做跳转**，别搞混。
8. 忘了某条，回本文对应小节看示例即可。

下一篇预告：**Vuex 状态管理与项目实战**——state/getters/mutations/actions、模块化、axios 封装、项目整合。
