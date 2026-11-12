---
title: Vuex 状态管理与项目实战
tags:
  - Vue
  - Vue2
  - Vuex
  - 状态管理
  - axios
categories:
  - 前端
  - Vue
date: 2026-09-13 14:00:00
---

# Vuex 状态管理与项目实战

Vuex 是 Vue 的**状态管理库**，集中管理多个组件共享的数据，解决"跨组件传值太麻烦"的问题。

这一篇讲 Vuex 的五个核心概念、模块化、辅助函数，以及项目实战中的 **axios 封装、api 模块化、移动端 vant、mixins** 等工程化要点。

> Java 对照：Vuex ≈ 一个"全局共享的内存数据容器"，类似后端的单例 Service + 缓存；`mutations` 像"同步 setter"，`actions` 像"异步业务方法"。

<!--more-->

## 一、为什么需要 Vuex

- 多组件共享同一份状态（如登录用户、购物车），用 props/$emit 层层传递太繁琐。
- Vuex 把共享状态**集中到一个 store**，任何组件都能访问和修改。

## 二、Vuex 五个核心概念

| 概念 | 作用 | 特点 |
| --- | --- | --- |
| `state` | 数据源（共享数据） | 只读 |
| `getters` | 派生数据（类似 computed） | 有缓存 |
| `mutations` | 修改 state 的唯一途径 | **必须同步** |
| `actions` | 处理异步，再提交 mutation | 可异步 |
| `modules` | 模块化拆分 store | 大型项目用 |

```js
// store/index.js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    count: 0
  },
  getters: {
    doubleCount(state) {          // 派生值
      return state.count * 2;
    }
  },
  mutations: {
    add(state, payload) {         // 同步改 state
      state.count += payload;
    }
  },
  actions: {
    asyncAdd(context, payload) {  // 异步，再 commit
      setTimeout(() => {
        context.commit("add", payload);
      }, 1000);
    }
  }
});
```

### 组件里使用

```js
this.$store.state.count;             // 读 state
this.$store.getters.doubleCount;     // 读 getter
this.$store.commit("add", 1);        // 提交 mutation
this.$store.dispatch("asyncAdd", 1); // 派发 action
```

### 数据流（核心，必记）

```
组件 dispatch → actions（异步）→ commit → mutations（同步改 state）→ 视图自动更新
```

> ⚠️ **核心原则**：**改 state 只能通过 mutation，且 mutation 必须同步**。异步操作放 action，action 里再 commit mutation。

## 三、模块化 modules

大型项目按功能拆分成多个模块：

```js
// store/modules/cart.js
export default {
  namespaced: true,        // 开启命名空间
  state: { list: [] },
  getters: {},
  mutations: {},
  actions: {}
};
```

```js
// store/index.js
import cart from "./modules/cart.js";
export default new Vuex.Store({
  modules: { cart }
});
```

命名空间下访问：

```js
this.$store.state.cart.list;
this.$store.commit("cart/add", data);   // 前面加模块名
```

## 四、辅助函数（简化写法）

```js
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";

export default {
  computed: {
    ...mapState(["count"]),
    ...mapGetters(["doubleCount"])
  },
  methods: {
    ...mapMutations(["add"]),
    ...mapActions(["asyncAdd"])
  }
};
// 之后可以直接 this.count、this.add(1)
```

## 五、axios 封装与 api 模块化（项目实战）

### 1. 封装 request.js（统一 baseURL、拦截器）

```js
// utils/request.js
import axios from "axios";

const request = axios.create({
  baseURL: "http://localhost:8080",   // 统一后端地址
  timeout: 5000
});

// 请求拦截器：统一加 token
request.interceptors.request.use(config => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = token;
  return config;
});

// 响应拦截器：直接返回 data，统一处理错误
request.interceptors.response.use(
  res => res.data,
  err => {
    console.log("请求失败", err);
    return Promise.reject(err);
  }
);

export default request;
```

### 2. api 模块化（按业务拆分接口）

```js
// api/user.js
import request from "../utils/request.js";

export const getUser = (id) => request.get(`/user/${id}`);
export const login = (data) => request.post("/login", data);
```

```js
// 组件里调用
import { login } from "../api/user.js";
login({ username, password }).then(res => { /* ... */ });
```

## 六、项目实战其他要点

### 1. mixins（复用组件逻辑）

多个组件共用的逻辑抽成 mixin：

```js
// mixins/loginConfirm.js
export default {
  methods: {
    checkLogin() {
      if (!this.$store.state.user.token) {
        this.$router.push("/login");
        return false;
      }
      return true;
    }
  }
};
```

```js
// 组件里引入
import loginConfirm from "../mixins/loginConfirm.js";
export default {
  mixins: [loginConfirm]
  // 组件里就能直接用 this.checkLogin()
};
```

### 2. 移动端 UI 库 vant（简要）

```js
// 引入 vant
import Vant from "vant";
import "vant/lib/index.css";
Vue.use(Vant);
```

### 3. utils 工具函数

把常用工具（本地存储、表单校验）抽到 `utils/`：

```js
// utils/storage.js
export const getToken = () => localStorage.getItem("token");
export const setToken = (t) => localStorage.setItem("token", t);
```

### 4. 典型项目结构（整合路由 + Vuex + axios）

```
src/
├── api/            # 接口封装（按模块）
├── store/          # Vuex（按模块）
├── router/         # 路由
├── views/          # 页面组件
├── components/     # 通用组件
├── mixins/         # 复用逻辑
└── utils/          # 工具（request、storage、validate）
```

## 七、速查表

| 需求 | 写法 |
| --- | --- |
| 创建 store | `new Vuex.Store({ state, getters, mutations, actions, modules })` |
| 读 state | `this.$store.state.count` |
| 读 getter | `this.$store.getters.doubleCount` |
| 同步改 | `this.$store.commit("add", 1)` |
| 异步改 | `this.$store.dispatch("asyncAdd", 1)` |
| 模块化 | `modules: { cart }` + `namespaced: true` |
| 辅助函数 | `mapState` / `mapMutations` / `mapActions` / `mapGetters` |
| axios 实例 | `axios.create({ baseURL, timeout })` |
| 请求拦截 | `request.interceptors.request.use(fn)` |
| 响应拦截 | `request.interceptors.response.use(fn)` |
| 复用逻辑 | `mixins: [loginConfirm]` |

## 八、复习要点

1. Vuex 集中管理**多组件共享**的状态；跨组件传值麻烦时用它。
2. 五概念：`state`（数据）、`getters`（派生）、`mutations`（同步改）、`actions`（异步）、`modules`（拆分）。
3. **数据流**：dispatch → action → commit → mutation → 改 state → 视图更新。
4. **mutation 必须同步，改 state 只能走 mutation**；异步放 action。
5. 模块化用 `namespaced: true`，访问要加模块名前缀。
6. axios 封装：`axios.create` + 请求/响应拦截器（统一 token、错误处理）。
7. api 模块化：按业务拆接口，组件里按需引入。
8. mixins 复用组件逻辑，utils 放工具函数。
9. 忘了某条，回本文对应小节看示例即可。

> 至此，Vue2 四篇笔记全部完成：基础与核心 → 组件化开发 → 路由 → Vuex 与项目实战，覆盖了 Vue2 框架开发的全部常用知识点。
