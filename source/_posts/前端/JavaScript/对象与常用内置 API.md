---
title: 对象与常用内置 API
tags:
  - JavaScript
  - 对象
  - 内置API
  - String
  - Date
  - JSON
categories:
  - 前端
  - JavaScript
date: 2026-09-12 18:00:00
---

# 对象与常用内置 API

这一篇分两块：**对象**（JS 的核心数据载体，与 Java 的"类"差异很大）和**常用内置 API**（String、Number/Math、Date、JSON、定时器、console、正则等日常工具箱）。

其中**原型与 class**、**Date 的月份从 0 开始**、**JSON 序列化的坑**是重点差异；`Array`、`Set/Map` 的方法已在第 2 篇覆盖，本篇不再重复。

<!--more-->

## 一、对象基础

### 1. 对象字面量与属性访问

```js
const user = {
  name: "Tom",
  age: 18,
  "full-name": "Tom Hanks"   // 含特殊字符的键要加引号
};

user.name;           // 点访问
user["name"];        // 方括号访问（键是变量时用这个）
user["full-name"];   // 特殊键必须用方括号
```

与 Java 的差异：Java 要 `new 类`，JS 直接用 `{ }` 写字面量，且键值可任意类型。

### 2. 动态增删属性、in 判断

```js
user.email = "a@b.com";   // 动态添加属性（Java 字段是编译期定死的）
delete user.age;          // 删除属性
"name" in user;           // 判断属性是否存在，返回 true
```

### 3. 解构与展开

```js
const { name, age } = user;        // 提取属性
const { name: n } = user;          // 提取并重命名
const { name, ...rest } = user;    // 剩余属性装进 rest

const copy = { ...user };          // 浅拷贝
const merged = { ...a, ...b };     // 合并（后者覆盖前者）
```

### 4. 方法简写与 this

```js
const obj = {
  name: "Tom",
  say() {                          // 方法简写，等价 say: function() {}
    console.log(this.name);        // this 指向调用者（第 4 篇已讲）
  }
};
obj.say();  // "Tom"
```

## 二、原型与 class（重点差异）

### 1. 原型 prototype 概念

JS 对象访问属性时，先找自身，找不到就沿**原型链**向上找：

```js
const parent = { greet() { return "hello"; } };
const child = Object.create(parent);   // child 的原型是 parent
child.greet();   // "hello"（自身没有，沿原型链找到 parent）
```

> 记忆点：**原型 = 对象的"爹"**，属性查找是"子找不到就找爹"。Java 没有这个概念（Java 是类继承，编译期确定）。

### 2. class 语法（语法糖）

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  say() {
    console.log(`我叫${this.name}`);
  }
}
const p = new Person("Tom", 18);
p.say();  // 我叫Tom
```

### 3. 继承 extends / super

```js
class Student extends Person {
  constructor(name, age, grade) {
    super(name, age);   // 必须且必须先调用 super（调父类构造）
    this.grade = grade;
  }
}
```

### 4. 与 Java class 的本质区别

| 特性 | Java | JavaScript |
| --- | --- | --- |
| 本质 | 真正的类，编译期类型 | `class` 是语法糖，底层是原型链 |
| 访问修饰符 | `public/private/protected` | 无（ES2022 有 `#` 私有字段） |
| 方法重载 | 支持 | 不支持 |
| 字段 | 编译期固定 | 可运行时动态增删 |
| `this` | 当前实例 | 运行时动态绑定 |

> ⚠️ **核心认知**：JS 的 `class` 和 Java 的 `class` 只是名字相同，机制完全不同。JS 里"继承"的本质是原型链，`extends` 只是把原型链接起来的语法糖。

## 三、Object 静态方法

```js
Object.keys(obj);        // 键数组 ["name", "age"]
Object.values(obj);      // 值数组 ["Tom", 18]
Object.entries(obj);     // [键,值] 数组 [["name","Tom"],["age",18]]

Object.assign(target, src);        // 把 src 浅拷贝/合并到 target
const copy = Object.assign({}, obj); // 常用浅拷贝写法

Object.freeze(obj);     // 冻结：不能增、删、改（浅冻结）
Object.seal(obj);       // 密封：不能增删，但可改已有属性值

obj.hasOwnProperty("name");   // 是否自有属性（不含原型链）
Object.hasOwn(obj, "name");   // 现代推荐写法
```

## 四、String 常用方法

### 1. 查找

```js
"hello".indexOf("l");        // 2（找不到返回 -1）
"hello".lastIndexOf("l");    // 3
"hello".includes("ell");     // true（是否存在）
"hello".startsWith("he");    // true（是否以...开头）
"hello".endsWith("lo");      // true（是否以...结尾）
```

### 2. 截取（slice / substring / substr 的区别）

```js
"hello".slice(1, 3);      // "el"  支持负数（负数从末尾数）
"hello".substring(1, 3);  // "el"  负数当 0 处理
"hello".substr(1, 3);     // "ell" 第二个参数是长度（已废弃，少用）
```

> ⚠️ **坑点**：`slice(start, end)` 和 `substring(start, end)` 都是"前闭后开"，但 `slice` 支持负数、`substring` 不支持；`substr(start, 长度)` 第二个参数是**长度**不是结束位置，且已废弃。

### 3. 转换

```js
"Hello".toUpperCase();           // "HELLO"
"Hello".toLowerCase();           // "hello"
"  hi  ".trim();                 // "hi"（去首尾空格）
"  hi".trimStart();              // "hi"
"hi  ".trimEnd();                // "hi"

"hello world".replace("o", "0");      // "hell0 world"（只替换第一个）
"hello world".replaceAll("o", "0");   // "hell0 w0rld"（全部替换）
"a,b,c".split(",");              // ["a", "b", "c"]（拆成数组）
"hello".charAt(1);               // "e"
```

### 4. 补全 / 重复

```js
"5".padStart(3, "0");   // "005"（左侧补全到长度3）
"5".padEnd(3, "0");     // "500"
"ab".repeat(3);         // "ababab"
"a".concat("b");        // "ab"
```

## 五、Number / Math

### 1. Number

```js
parseInt("12px");         // 12（转整数，直到非数字停止）
parseFloat("3.14");       // 3.14
(3.14159).toFixed(2);     // "3.14"（保留2位小数，注意返回字符串！）

Number.isInteger(3);      // true（是否整数）
Number.isNaN(NaN);        // true（更严格，不做类型转换）
Number.isFinite(1 / 0);   // false（是否有限）
```

> ⚠️ 全局 `isNaN("abc")` 会先转换再判断（返回 true），`Number.isNaN("abc")` 不转换（返回 false）。推荐用 `Number.isNaN`。

### 2. Math

```js
Math.abs(-5);        // 5（绝对值）
Math.ceil(3.1);      // 4（向上取整）
Math.floor(3.9);     // 3（向下取整）
Math.round(3.5);     // 4（四舍五入）
Math.trunc(3.9);     // 3（直接去掉小数）
Math.max(1, 2, 3);   // 3
Math.min(1, 2, 3);   // 1
Math.random();       // [0, 1) 随机数
Math.pow(2, 10);     // 1024
Math.sqrt(16);       // 4
```

随机整数（常用套路）：

```js
Math.floor(Math.random() * 10);      // 0 ~ 9
Math.floor(Math.random() * 10) + 1;  // 1 ~ 10
```

## 六、Date（坑最多）

### 1. 创建

```js
new Date();                    // 当前时间
new Date(0);                   // 时间戳 0（1970-01-01）
new Date("2026-09-12");        // 字符串
new Date(2026, 8, 12);         // ⚠️ 月份 8 表示 9 月！
```

### 2. 获取 / 设置

```js
const d = new Date();
d.getFullYear();   // 2026（年）
d.getMonth();      // 月份 0~11（⚠️ 0 是 1 月）
d.getDate();       // 几号 1~31
d.getDay();        // 星期几（0 是周日）
d.getHours();      // 时
d.getMinutes();    // 分
d.getSeconds();    // 秒
```

### 3. 时间戳

```js
d.getTime();       // 毫秒时间戳
Date.now();        // 当前时间戳（最常用）
```

### 4. 格式化

```js
d.toLocaleString();   // "2026/9/12 18:00:00"（本地格式）
d.toISOString();      // "2026-09-12T10:00:00.000Z"（ISO 标准格式）
```

> ⚠️ **Date 最大的坑**：`getMonth()` 和 `new Date(年, 月, 日)` 的月份都是 **0~11**，`0` 表示一月。所以显示月份要 `+1`，`new Date(2026, 8, 12)` 是 9 月 12 日。

对比 Java：Java 有 `LocalDate`（纯日期）、`LocalDateTime`、`Calendar` 等分工；JS 的 `Date` 一个类同时含日期和时间，且没有不可变副本，用起来更容易踩坑。

## 七、JSON

### 1. 序列化 / 反序列化

```js
const user = { name: "Tom", age: 18 };

const str = JSON.stringify(user);   // '{"name":"Tom","age":18}'
const obj = JSON.parse(str);        // { name: "Tom", age: 18 }
```

对应 Java 的 Jackson / Gson。

### 2. 坑点

```js
JSON.stringify({ a: undefined, b: () => {}, c: Symbol() });
// '{}'  ⚠️ undefined、函数、Symbol 会被直接忽略

JSON.stringify({ d: new Date() });
// '"2026-09-12T..."'  Date 会被转成 ISO 字符串

JSON.parse('{"name": "Tom"}');  // ✅ 键名必须用双引号
```

**深拷贝技巧**（浅拷贝的坑）：

```js
const obj = { a: { b: 1 } };
const copy = JSON.parse(JSON.stringify(obj));  // 深拷贝
```

> ⚠️ 但这种方式会丢失函数、`undefined`，仅适用于纯数据对象。

## 八、定时器（异步基础）

```js
setTimeout(() => {
  console.log("延迟 1 秒执行一次");
}, 1000);

const id = setInterval(() => {
  console.log("每隔 1 秒执行一次");
}, 1000);

clearTimeout(id);     // 清除定时器
clearInterval(id);
```

- `setTimeout`：延迟执行一次；`setInterval`：每隔一段时间重复执行。
- 都返回一个 id，用于清除。
- 对应 Java 的 `ScheduledExecutorService` / `Timer`。

## 九、console（调试）

```js
console.log("普通日志");
console.warn("警告");
console.error("错误");
console.info("信息");

console.table([{ a: 1 }, { a: 2 }]);  // 表格形式打印
console.time("t");                    // 计时开始
console.timeEnd("t");                 // 打印耗时

console.dir(obj);   // 打印对象结构（展开属性）
```

对应 Java 的 `System.out.println`，但功能更丰富。

## 十、正则 RegExp（简要）

```js
const re = /\d+/;                // 字面量
const re2 = new RegExp("\\d+");  // 构造函数（字符串里要转义 \）

re.test("123");          // true（是否匹配）
"a123b".match(/\d+/);    // ["123"]（返回匹配结果）
"abc123".replace(/\d+/, "#");  // "abc#"（替换）
```

常用标志：`g`（全局）、`i`（忽略大小写）、`m`（多行）。

> 正则语法与 Java 高度相似（都是 Perl 风格），Java 程序员可快速上手，本篇只做简要介绍。

## 十一、其他全局函数

```js
encodeURIComponent("a b&c");        // "a%20b%26c"（URL 编码）
decodeURIComponent("a%20b%26c");    // "a b&c"（URL 解码）

isNaN("abc");          // true（先转换再判断）
Number.isNaN("abc");   // false（不转换，更严格）
isFinite(1 / 0);       // false
```

## 十二、速查表

| 需求 | 写法 |
| --- | --- |
| 对象字面量 | `{ name: "Tom" }` |
| 属性访问 | `obj.name` / `obj["name"]` |
| 判断属性存在 | `"key" in obj` |
| 对象解构 | `const { name, age } = obj` |
| 对象合并/浅拷贝 | `{ ...a, ...b }` / `Object.assign({}, obj)` |
| 对象键值数组 | `Object.keys/values/entries` |
| 冻结对象 | `Object.freeze(obj)` |
| 定义类 | `class Person { constructor(){} }` |
| 继承 | `class Student extends Person` + `super()` |
| 字符串查找 | `includes` / `startsWith` / `indexOf` |
| 字符串截取 | `slice(1, 3)` |
| 去空格 | `trim()` |
| 全部替换 | `replaceAll("o", "0")` |
| 拆数组 | `"a,b".split(",")` |
| 保留小数 | `(3.14).toFixed(2)` |
| 取整 | `Math.floor` / `Math.ceil` / `Math.round` |
| 随机数 | `Math.random()` |
| 当前时间戳 | `Date.now()` |
| JSON 序列化 | `JSON.stringify(obj)` / `JSON.parse(str)` |
| 延迟执行 | `setTimeout(fn, 1000)` |
| 定时重复 | `setInterval(fn, 1000)` |
| 打印日志 | `console.log` / `console.table` |

## 十三、复习要点

1. **对象**用字面量创建，属性可动态增删，解构/展开是常用操作。
2. **原型是 JS 的"继承"机制**：属性查找沿原型链向上；`class` 只是语法糖。
3. **JS 的 class ≠ Java 的 class**：无访问修饰符、无重载、this 动态绑定。
4. `Object.assign` 和 `...` 都是**浅拷贝**；深拷贝可用 `JSON.parse(JSON.stringify(obj))`（但丢函数/undefined）。
5. String 截取：`slice` 支持负数、`substr` 已废弃；`replace` 只换第一个、`replaceAll` 换全部。
6. `toFixed` 返回的是**字符串**；`Number.isNaN` 比全局 `isNaN` 更严格。
7. **Date 月份从 0 开始**（`getMonth()` 和构造函数的月参数），显示要 `+1`。
8. `JSON.stringify` 会忽略 `undefined`/函数/Symbol，Date 转 ISO 字符串。
9. `setTimeout` 延迟一次、`setInterval` 定时重复，返回 id 用于清除。
10. 忘了某个 API，回本文对应小节查示例即可。

> 至此，JS 基础五篇（字面量与变量/数据类型/运算符、流程控制与数组集合、函数、对象与内置 API）已全部完成，覆盖了从入门到进阶的核心知识点，可配合复习速查表使用。
