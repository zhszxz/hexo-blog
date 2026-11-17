---
title: Vue3
tags:
  - Vue3
  - 组合式API
  - Pinia
categories:
  - 前端
  - Vue
date: 2026-11-13 17:00:00
---

# Vue3 学习笔记

Vue3 相比 Vue2 最大的变化是引入**组合式 API（Composition API）**，并把响应式从 `Object.defineProperty` 升级为 `Proxy`。同时生态配套升级为 **Pinia**（状态管理）和 **vue-router 4**（路由）。

这一篇把 Vue3 的核心内容合并讲完：组合式 API、ref/reactive、computed/watch、生命周期、Proxy 响应式、组件通信、新特性（Fragment/Teleport/组合式函数）、Pinia、vue-router 4。

> 对比 Vue2（Options API）：Vue2 按"选项"组织（data/methods/computed 分散），Vue3 按"功能"组织（一个功能的代码聚在一起）。

<!--more-->

## 一、Vue3 相比 Vue2 的变化

| 变化 | Vue2 | Vue3 |
| --- | --- | --- |
| 组织方式 | Options API（选项式） | Composition API（组合式）+ 兼容选项式 |
| 响应式 | `Object.defineProperty` | `Proxy` |
| 根节点 | 组件只能**一个根节点** | 支持**多根节点**（Fragment） |
| 状态管理 | Vuex | Pinia |
| 路由 | vue-router 3 | vue-router 4 |
| TypeScript | 支持一般 | 原生友好 |
| 性能 | 一般 | 更快、更小 |

## 二、创建 Vue3 项目

```bash
npm create vue@latest   # 官方脚手架（Vite 驱动）
cd my-vue3-app
npm install
npm run dev
```

## 三、组合式 API：ref 与 reactive

`<script setup>` 是 Vue3 的语法糖，无需写 `export default`，顶层代码直接是组件逻辑。

```vue
<script setup>
import { ref, reactive } from "vue";

// ref：包装基本类型（也可对象），访问/修改要用 .value
const count = ref(0);
count.value++;          // 修改要 .value

// reactive：包装对象，直接访问（无需 .value）
const user = reactive({ name: "Tom", age: 18 });
user.age++;             // 直接改
</script>

<template>
  <p>{{ count }}</p>      <!-- 模板里 ref 自动解包，不用 .value -->
  <p>{{ user.name }}</p>
</template>
```

### ref vs reactive 对比

| | `ref` | `reactive` |
| --- | --- | --- |
| 适用 | 基本类型（也可对象） | 对象/数组 |
| 访问/修改 | 要 `.value` | 直接访问 |
| 模板中 | 自动解包（不用 `.value`） | 直接访问 |

> 记忆点：**基本类型用 `ref`，对象用 `reactive`**；`ref` 在 JS 里要 `.value`，模板里不用。

## 四、计算属性 computed

```vue
<script setup>
import { ref, computed } from "vue";

const count = ref(0);
const double = computed(() => count.value * 2);

// 可写 computed
const full = computed({
  get: () => count.value,
  set: (val) => { count.value = val; }
});
</script>
```

- 有缓存，依赖不变不重新计算（与 Vue2 一致）。

## 五、侦听器 watch / watchEffect

```vue
<script setup>
import { ref, watch, watchEffect } from "vue";

const count = ref(0);
const user = reactive({ name: "Tom" });

// watch：监听指定数据
watch(count, (newVal, oldVal) => {
  console.log(newVal, oldVal);
});

// 监听 reactive 对象（默认深度监听）
watch(user, (newVal) => console.log(newVal));

// watchEffect：自动追踪依赖，立即执行一次
watchEffect(() => {
  console.log(count.value);   // 用了 count，count 变化就执行
});
</script>
```

| | `watch` | `watchEffect` |
| --- | --- | --- |
| 监听目标 | 显式指定 | 自动追踪用到的依赖 |
| 立即执行 | 否（可 `immediate: true`） | 是 |
| 旧值 | 有 | 无 |

## 六、生命周期钩子（Vue3 变化）

Vue3 生命周期钩子**从 `vue` 导入**，在 `setup` 里使用：

```vue
<script setup>
import { onMounted, onUnmounted } from "vue";

onMounted(() => {
  console.log("DOM 已挂载");
});

onUnmounted(() => {
  console.log("组件销毁");
});
</script>
```

对应关系（Vue2 → Vue3 组合式）：

| Vue2 选项式 | Vue3 组合式 |
| --- | --- |
| `created` | `setup()` 本身（无需钩子） |
| `mounted` | `onMounted` |
| `updated` | `onUpdated` |
| `beforeDestroy` | `onBeforeUnmount` |
| `destroyed` | `onUnmounted` |

> ⚠️ Vue3 里没有 `beforeCreate`/`created` 的组合式写法，因为 `setup()` 本身就在创建阶段执行，逻辑直接写 `setup` 顶层即可。

## 七、响应式原理：Proxy vs Object.defineProperty

Vue2 用 `Object.defineProperty` 劫持每个属性，有两大缺陷；Vue3 用 `Proxy` 代理整个对象解决：

| | Vue2 `Object.defineProperty` | Vue3 `Proxy` |
| --- | --- | --- |
| 劫持方式 | 逐个属性 getter/setter | 代理整个对象 |
| 新增/删除属性 | ❌ 不响应（要 `$set`） | ✅ 响应 |
| 数组索引/长度 | ❌ 要特殊处理 | ✅ 响应 |
| 性能 | 遍历所有属性，较慢 | 更快 |

> 这就是 Vue3 最大的改进之一：**后添加属性、数组下标修改，Vue3 都能自动响应**，不再需要 `$set`。

## 八、toRefs（解构保持响应式）

`reactive` 对象解构会**丢失响应式**，用 `toRefs` 解决：

```vue
<script setup>
import { reactive, toRefs } from "vue";

const user = reactive({ name: "Tom", age: 18 });

// 解构后 name、age 仍是响应式的
const { name, age } = toRefs(user);
</script>
```

> 记忆点：`reactive` 对象直接解构会失去响应式，用 `toRefs` 把每个属性转成 `ref` 再解构。

## 九、组件通信

### 1. defineProps（父传子）

```vue
<!-- 父组件 -->
<Son :msg="message" :count="10" />
```

```vue
<!-- 子组件 -->
<script setup>
const props = defineProps({
  msg: String,
  count: { type: Number, default: 0 }
});
// 模板直接用 msg、count，脚本里用 props.msg
</script>

<template>
  <p>{{ msg }} - {{ count }}</p>
</template>
```

### 2. defineEmits（子传父）

```vue
<!-- 子组件 -->
<script setup>
const emit = defineEmits(["add"]);
</script>

<template>
  <button @click="emit('add', 1)">+1</button>
</template>
```

```vue
<!-- 父组件 -->
<Son @add="handleAdd" />
```

### 3. defineExpose（暴露给父组件）

父组件用 `ref` 拿子组件实例时，子组件需 `defineExpose` 暴露方法：

```vue
<!-- 子组件 -->
<script setup>
const reset = () => { /* ... */ };
defineExpose({ reset });
</script>
```

```vue
<!-- 父组件 -->
<script setup>
import { ref } from "vue";
import Son from "./Son.vue";
const sonRef = ref(null);
sonRef.value.reset();   // 调用子组件方法
</script>

<template>
  <Son ref="sonRef" />
</template>
```

> ⚠️ Vue3 的 `<script setup>` 组件默认是"封闭"的，父组件拿不到内部方法，必须用 `defineExpose` 显式暴露。

### 4. provide / inject（跨层级）

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from "vue";
provide("msg", ref("hello"));
</script>
```

```vue
<!-- 任意后代组件 -->
<script setup>
import { inject } from "vue";
const msg = inject("msg");
</script>
```

### 5. v-model（Vue3 支持多个）

Vue3 组件上 `v-model` 默认用 `modelValue` prop + `update:modelValue` 事件：

```vue
<!-- 父组件 -->
<MyInput v-model="msg" />
```

```vue
<!-- 子组件 -->
<script setup>
defineProps(["modelValue"]);
defineEmits(["update:modelValue"]);
</script>

<template>
  <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" />
</template>
```

支持**多个 v-model**（给 v-model 加名字）：

```vue
<MyForm v-model:title="title" v-model:content="content" />
```

```vue
<!-- 子组件：分别对应 title、content -->
<script setup>
defineProps(["title", "content"]);
defineEmits(["update:title", "update:content"]);
</script>
```

> Vue2 里一个组件只能一个 `v-model`（或 `.sync`）；Vue3 直接支持多个具名 v-model，`.sync` 已废弃。

## 十、新特性

### 1. Fragment（多根节点）

Vue3 组件模板**可以写多个根节点**，不再需要包一层 div：

```vue
<template>
  <h1>标题</h1>
  <p>内容</p>
</template>
```

> Vue2 要求组件模板只能一个根节点；Vue3 的 Fragment 特性解除了这个限制。

### 2. Teleport（传送门）

把组件内容渲染到指定 DOM 位置（如 `body`），常用于弹窗、模态框避免被父容器裁剪：

```vue
<template>
  <Teleport to="body">
    <div class="dialog">弹窗内容</div>
  </Teleport>
</template>
```

> `to="body"` 表示把内容渲染到 `body` 下，脱离当前组件的 DOM 层级。

### 3. 组合式函数（hooks）

把可复用的逻辑抽成函数，是组合式 API 的核心价值：

```js
// hooks/useCounter.js
import { ref } from "vue";

export function useCounter() {
  const count = ref(0);
  const add = () => count.value++;
  return { count, add };
}
```

```vue
<script setup>
import { useCounter } from "./hooks/useCounter.js";
const { count, add } = useCounter();   // 复用计数逻辑
</script>

<template>
  <button @click="add">{{ count }}</button>
</template>
```

> hooks 是"带状态的逻辑复用"，比 Vue2 的 mixins 更清晰（不会命名冲突、来源可追踪）。

## 十一、Pinia 状态管理

### 1. 安装与创建

```bash
npm install pinia
```

```js
// main.js
import { createApp } from "vue";
import { createPinia } from "pinia";
import App from "./App.vue";

createApp(App).use(createPinia()).mount("#app");
```

```js
// store/counter.js
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", {
  state: () => ({ count: 0 }),       // 数据
  getters: {
    double: (state) => state.count * 2   // 派生
  },
  actions: {
    add() { this.count++; }              // 方法（可异步）
  }
});
```

### 2. 组件中使用

```vue
<script setup>
import { useCounterStore } from "./store/counter.js";
const store = useCounterStore();
</script>

<template>
  <p>{{ store.count }}</p>      <!-- 读 state -->
  <p>{{ store.double }}</p>     <!-- 读 getter -->
  <button @click="store.add()">+1</button>   <!-- 调 action -->
</template>
```

### 3. Pinia vs Vuex

| | Vuex（Vue2） | Pinia（Vue3） |
| --- | --- | --- |
| mutations | 有（改 state 必须走它） | **没有**，直接改 state |
| API | 繁琐（state/commit/dispatch） | 简单（store.xxx） |
| 模块化 | modules + namespaced | 多个独立 store，天然模块化 |
| TypeScript | 支持一般 | 原生友好 |

> Pinia 核心简化：**去掉了 mutations**，直接 `store.count++` 就能改状态，`actions` 可同步可异步。

### 4. 组合式写法（Setup Store）

```js
import { ref, computed } from "vue";
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", () => {
  const count = ref(0);
  const double = computed(() => count.value * 2);
  const add = () => count.value++;
  return { count, double, add };
});
```

> 选项式写法和组合式写法二选一，组合式更贴近 Vue3 风格。

## 十二、vue-router 4

### 1. 创建路由（与 Vue2 的差异）

```js
// router/index.js
import { createRouter, createWebHistory } from "vue-router";
import Home from "../views/Home.vue";
import About from "../views/About.vue";

const router = createRouter({
  history: createWebHistory(),    // history 模式
  routes: [
    { path: "/", component: Home },
    { path: "/about", component: About }
  ]
});

export default router;
```

```js
// main.js
import { createApp } from "vue";
import router from "./router";
import App from "./App.vue";

createApp(App).use(router).mount("#app");
```

与 Vue2 路由的差异：

| | Vue2（vue-router 3） | Vue3（vue-router 4） |
| --- | --- | --- |
| 创建 | `new VueRouter({ mode, routes })` | `createRouter({ history, routes })` |
| 模式 | `mode: "history"` | `createWebHistory()` / `createWebHashHistory()` |
| 挂载 | `new Vue({ router })` | `app.use(router)` |

> 核心变化：`mode` 选项没了，改成 `createWebHistory()`（history 模式）或 `createWebHashHistory()`（hash 模式）。

### 2. 使用（与 Vue2 相同）

```vue
<template>
  <router-link to="/">首页</router-link>
  <router-link to="/about">关于</router-link>
  <router-view></router-view>
</template>
```

```js
// 组合式里用路由
import { useRoute, useRouter } from "vue-router";

const route = useRoute();      // 取参数 route.query / route.params
const router = useRouter();    // 跳转 router.push("/about")
```

> Vue3 组合式里用 `useRoute()`/`useRouter()` 代替 Vue2 的 `this.$route`/`this.$router`。

### 3. 导航传参（与 Vue2 一致）

```js
// query 传参
router.push({ path: "/search", query: { kw: "vue" } });
// 接收：route.query.kw

// params 动态路由
router.push({ name: "search", params: { id: 1 } });
// 路由表：{ path: "/search/:id", name: "search", component: Search }
// 接收：route.params.id
```

## 十三、速查表

| 需求 | 写法 |
| --- | --- |
| 创建项目 | `npm create vue@latest` |
| 基本类型响应式 | `const count = ref(0)` |
| 对象响应式 | `const user = reactive({...})` |
| 计算属性 | `computed(() => ...)` |
| 侦听 | `watch(x, (n, o) => {})` / `watchEffect(() => {})` |
| 挂载后 | `onMounted(() => {})` |
| 解构保持响应式 | `toRefs(reactiveObj)` |
| 父传子 | `defineProps({ msg: String })` |
| 子传父 | `defineEmits(["add"])` + `emit("add", 1)` |
| 暴露方法 | `defineExpose({ reset })` |
| 跨层级传值 | `provide("k", v)` / `inject("k")` |
| 组件 v-model | `defineProps(["modelValue"])` + `emit("update:modelValue", v)` |
| 多个 v-model | `v-model:title` / `v-model:content` |
| 传送门 | `<Teleport to="body">` |
| 复用逻辑 | 组合式函数 `useXxx()` |
| 创建 Pinia store | `defineStore("id", { state, getters, actions })` |
| 用 store | `const store = useXxxStore()` |
| 创建路由 | `createRouter({ history: createWebHistory(), routes })` |
| 取参数/跳转 | `useRoute().query` / `useRouter().push()` |

## 十四、复习要点

1. **Vue3 核心是组合式 API**：`<script setup>` + 按功能组织代码。
2. **基本类型用 `ref`（要 `.value`），对象用 `reactive`**；模板里 `ref` 自动解包。
3. `computed` 有缓存；`watch` 显式监听、`watchEffect` 自动追踪且立即执行。
4. 生命周期从 `vue` 导入（`onMounted` 等），没有 `created` 写法（用 `setup` 顶层）。
5. Vue3 用 **Proxy** 响应式，后加属性、数组下标修改都响应，不再需要 `$set`。
6. `reactive` 解构会失去响应式，用 `toRefs` 保持。
7. 组件通信用 `defineProps`/`defineEmits`；`<script setup>` 组件默认封闭，调子组件方法要 `defineExpose`。
8. **v-model** 默认 `modelValue` + `update:modelValue`，支持多个具名 v-model，`.sync` 废弃。
9. **Fragment** 允许多根节点；**Teleport** 把内容渲染到指定位置；**组合式函数**是逻辑复用的核心。
10. **Pinia** 去掉 mutations、直接改 state、天然模块化；**vue-router 4** 用 `createRouter` + `useRoute/useRouter`。
11. 忘了某条，回本文对应小节看示例即可。

> 至此，Vue3 学习笔记（合并版）完成，覆盖了 Vue3 新语法、新特性、Pinia 与路由等核心内容（大事件项目已按要求忽略）。
