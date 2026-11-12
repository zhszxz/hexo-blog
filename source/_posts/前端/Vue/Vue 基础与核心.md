---
title: Vue 基础与核心
tags:
  - Vue
  - Vue2
  - 指令
  - 计算属性
  - 生命周期
categories:
  - 前端
  - Vue
date: 2026-10-25 11:00:00
---

# Vue 基础与核心

Vue 是渐进式 JS 框架，核心思想是**数据驱动视图**（MVVM）：你只管改数据，视图自动更新，不用再手动操作 DOM。

这一篇讲 Vue 的基础与核心：实例、插值、响应式原理、指令、修饰符、class/style 绑定、计算属性、侦听器、生命周期。

> Java 对照：传统方式（含 jQuery）是"手动查 DOM、改 DOM"；Vue 是"改数据、框架帮你改 DOM"。这是理解 Vue 最重要的思维转变。

<!--more-->

## 一、MVVM 思想

```
Model（数据 data） ⇄ ViewModel（Vue 实例） ⇄ View（模板 DOM）
```

- **Model**：数据（`data`）
- **View**：页面模板（HTML）
- **ViewModel**：Vue 实例，负责双向绑定——数据变了自动更新视图，视图输入自动更新数据。

## 二、创建 Vue 实例与插值

```html
<div id="app">{{ message }}</div>

<script src="vue.js"></script>
<script>
  const vm = new Vue({
    el: "#app",                        // 挂载点
    data: { message: "Hello Vue" },    // 数据
    methods: {}                        // 方法
  });
</script>
```

插值表达式 `{{ }}`：

```html
<p>{{ message }}</p>
<p>{{ 1 + 1 }}</p>
<p>{{ message.toUpperCase() }}</p>
```

> ⚠️ 插值里只能写**表达式**（有返回值），不能写语句（`if`、`for`、`var` 等都不行）。

## 三、响应式原理

Vue2 通过 `Object.defineProperty` 劫持 `data` 里的属性，数据一变，用到它的视图自动更新：

```js
data: { count: 0 }
// 改 vm.count，页面用到 count 的地方自动变
vm.count = 1;
```

> ⚠️ **坑**：`data` 里**后添加的属性不是响应式的**（改它视图不更新），要用 `Vue.set(obj, key, value)` 或 `this.$set` 添加。所以 `data` 里最好**提前声明**所有要用的字段。

## 四、常用指令

| 指令 | 作用 | 简写 |
| --- | --- | --- |
| `v-html` | 解析 HTML 插入 | 无 |
| `v-show` | 切换显示（display:none） | 无 |
| `v-if` / `v-else-if` / `v-else` | 条件渲染（销毁/重建） | 无 |
| `v-on` | 绑定事件 | `@` |
| `v-bind` | 绑定属性 | `:` |
| `v-for` | 列表渲染 | 无 |
| `v-model` | 双向绑定 | 无 |

### 1. v-html（对比插值）

```html
<div v-html="htmlStr"></div>   <!-- 解析成 HTML -->
<p>{{ htmlStr }}</p>           <!-- 原样显示字符串 -->
```

> ⚠️ `v-html` 有 **XSS 风险**，不要渲染用户输入的内容。

### 2. v-show vs v-if

```html
<div v-show="flag">显示/隐藏</div>   <!-- 切换 display:none，频繁切换用它 -->
<div v-if="flag">条件渲染</div>      <!-- 销毁/重建 DOM，切换开销大 -->
```

- `v-show`：始终渲染，只是切换 `display:none`，适合**频繁切换**。
- `v-if`：条件为假时**不渲染/销毁**，适合**不常切换**或初始不渲染。

### 3. v-on 事件绑定

```html
<button v-on:click="count++">点我</button>
<button @click="handleClick">点我</button>   <!-- 简写 -->
```

```js
methods: {
  handleClick(e) {
    console.log(e);          // 事件对象
  }
}
```

传参：`@click="handle(1, $event)"`（`$event` 是事件对象）。

### 4. v-bind 属性绑定

```html
<img :src="imgUrl">          <!-- 绑定属性 -->
<a :href="url">链接</a>
<button :disabled="isDis">按钮</button>
```

### 5. v-for 列表渲染

```html
<li v-for="(item, index) in list" :key="item.id">{{ item.name }}</li>
```

```js
data: {
  list: [{ id: 1, name: "A" }, { id: 2, name: "B" }]
}
```

> ⚠️ **key 很重要**：`v-for` 必须加 `:key`（唯一值，一般用 id，**不要用 index**），Vue 靠 key 做 diff 优化，key 不对会导致列表错乱/复用错误。

### 6. v-model 双向绑定

```html
<input v-model="message">
<!-- 本质是 :value="message" + @input="message = $event.target.value" -->
```

- 数据 → 视图（输入框显示 message）
- 视图 → 数据（输入时自动更新 message）

## 五、指令修饰符

### 1. 事件修饰符

```html
<button @click.stop="fn">阻止冒泡</button>
<a @click.prevent="fn">阻止默认行为（如跳转）</a>
<button @click.once="fn">只触发一次</button>
```

### 2. v-model 修饰符

```html
<input v-model.trim="name">    <!-- 自动去首尾空格 -->
<input v-model.number="age">   <!-- 转成数字 -->
<input v-model.lazy="msg">     <!-- 改为 change 事件（失焦才更新） -->
```

## 六、class 与 style 绑定

### 1. class 对象语法

```html
<div :class="{ active: isActive, red: isRed }"></div>
```

```js
data: { isActive: true, isRed: false }
// 渲染成 <div class="active">
```

### 2. class 数组语法

```html
<div :class="[classA, classB]"></div>
```

### 3. style 绑定

```html
<div :style="{ color: textColor, fontSize: size + 'px' }"></div>
```

## 七、计算属性 computed

计算属性根据已有数据**派生新值**，且有**缓存**。

```js
new Vue({
  data: { num: 1 },
  computed: {
    double() {                 // 简写（只读）
      return this.num * 2;
    },
    full: {                    // 完整写法（可读可写）
      get() { return this.num; },
      set(val) { this.num = val; }
    }
  }
});
```

```html
<p>{{ double }}</p>
```

### computed vs methods

| | computed | methods |
| --- | --- | --- |
| 缓存 | ✅ 依赖不变不重新计算 | ❌ 每次调用都执行 |
| 写法 | `{{ double }}`（不加括号） | `{{ double() }}`（加括号） |

> 记忆点：**有依赖、要缓存、要派生值 → 用 computed**；纯逻辑操作、要传参 → 用 methods。

## 八、侦听器 watch

`watch` 监听数据变化，执行**副作用**（发请求、异步操作、操作 DOM）。

```js
new Vue({
  data: { keyword: "" },
  watch: {
    keyword(newVal, oldVal) {           // 简写
      console.log(newVal, oldVal);
    },
    user: {
      handler(newVal) { /* 处理逻辑 */ },
      deep: true,       // 深度监听（监听对象内部变化）
      immediate: true   // 立即执行一次
    }
  }
});
```

### watch vs computed

| | computed | watch |
| --- | --- | --- |
| 目的 | 派生新值（有返回值） | 执行副作用（无返回值） |
| 缓存 | 有 | 无 |
| 场景 | 根据数据算结果 | 数据变化时发请求、异步 |

> 记忆：**要"算一个新值"用 computed，要"数据变了去做事"用 watch**。

## 九、生命周期（四大阶段、八个钩子）

Vue 实例从创建到销毁会依次触发 8 个钩子：

| 阶段 | 钩子 | 时机与常用场景 |
| --- | --- | --- |
| 创建 | `beforeCreate` | 数据/方法还没初始化（少用） |
| 创建 | `created` | data/methods 已就绪，**DOM 未生成**，常用：**发请求** |
| 挂载 | `beforeMount` | 模板编译完成，未挂载（少用） |
| 挂载 | `mounted` | **DOM 已挂载**，常用：**操作 DOM、初始化图表** |
| 更新 | `beforeUpdate` | 数据更新，DOM 未重渲染（少用） |
| 更新 | `updated` | DOM 已更新（少用） |
| 销毁 | `beforeDestroy` | 销毁前，常用：**清定时器、解绑事件** |
| 销毁 | `destroyed` | 已销毁（少用） |

```js
new Vue({
  el: "#app",
  data: {},
  created() {
    // 发请求拿数据（此时 data 已可用）
  },
  mounted() {
    // 操作 DOM（此时页面已渲染）
  },
  beforeDestroy() {
    // 清理定时器等资源
  }
});
```

> ⚠️ 两个高频场景记住：**created 里发请求**（数据早拿到）、**mounted 里操作 DOM**（DOM 已就绪）。

## 十、速查表

| 需求 | 写法 |
| --- | --- |
| 创建实例 | `new Vue({ el, data, methods })` |
| 插值 | `{{ message }}` |
| 解析 HTML | `v-html="str"` |
| 条件渲染 | `v-if` / `v-else` / `v-show` |
| 事件绑定 | `@click="fn"` |
| 属性绑定 | `:href="url"` |
| 列表渲染 | `v-for="(item, i) in list" :key="item.id"` |
| 双向绑定 | `v-model="msg"` |
| 事件修饰符 | `@click.stop` / `@click.prevent` / `.once` |
| v-model 修饰符 | `.trim` / `.number` / `.lazy` |
| class 对象绑定 | `:class="{ active: isActive }"` |
| 计算属性 | `computed: { double() { return ... } }` |
| 侦听器 | `watch: { key(newVal, oldVal) {} }` |
| 深度/立即监听 | `deep: true` / `immediate: true` |
| 发请求钩子 | `created()` |
| 操作 DOM 钩子 | `mounted()` |

## 十一、复习要点

1. **MVVM**：改数据 → 视图自动更新，不手动操作 DOM（这是 Vue 的核心思想）。
2. `new Vue({ el, data, methods, computed, watch })` 是 Options API 的骨架。
3. **v-show 是 display:none 切换**，**v-if 是销毁/重建**；频繁切换用 v-show。
4. `v-for` 必须加 **`:key`（唯一值，别用 index）**。
5. `v-model` = `:value` + `@input` 的语法糖。
6. **computed 有缓存、用于派生值**；**watch 用于副作用**（发请求/异步）。
7. 生命周期八钩子，重点记 **created（发请求）** 和 **mounted（操作 DOM）**。
8. `data` 里后加属性不是响应式，要用 `$set`；提前声明所有字段。
9. 忘了某条，回本文对应小节看示例即可。

下一篇预告：**组件化开发**——组件注册、props/事件、插槽、自定义指令、$refs、v-model 原理等组件通信全体系。
