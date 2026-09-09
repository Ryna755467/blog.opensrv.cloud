---
title: 模块
date: 2026-07-03
updated: 2026-07-03
top_img: /img/series/javascript.jpg
cover: /img/covers/javascript/module.jpg
series: javascript
categories:
  - JavaScript
---

在 JavaScript 中，**模块** 的作用是：将代码分割成 **独立、可复用** 的文件，每个文件拥有独立私有作用域，通过导出、导入实现文件间代码复用。

主要有两大规范：**CommonJS（CJS，Node.js 旧规范）、ES Module（ESM，ES6 新标准）**。

## CommonJS

CommonJS 中，模块的导出本质是给 `module.exports` 对象赋值，所有需要对外暴露的内容都挂载在这个对象上。

{% note warning %}
CommonJS 是 Node.js 默认模块规范，浏览器不原生支持，**运行时同步加载**。
{% endnote %}

### 导出和导入

**1. 导出单个内容（函数、对象、变量、类都可以）**

```javascript
// utils.js 导出一个工具函数
function add(a, b) {
  return a + b;
}
// 直接把整个模块导出为add函数
module.exports = add;

// 导入的add就是utils.js导出的函数
const add = require("./utils.js");
console.log(add(1, 2)); // 3
```

**2. 导出多个内容（挂载到 `module.exports` 对象上）**

```javascript
// user.js 导出多个工具方法和变量
const name = "张三";
function getUserInfo() {
  return { name, age: 18 };
}
function updateUser(newName) {
  name = newName;
}
// 多个导出挂在module.exports对象上
module.exports = {
  name,
  getUserInfo,
  updateUser,
};

// 整体导入
const user = require("./user.js");
console.log(user.name); // 张三
console.log(user.getUserInfo()); // {name: '张三', age: 18}

// 解构导入
const { getUserInfo } = require("./user.js");
console.log(getUserInfo());
```

**3. 导出构造函数 / 类**

```javascript
// Person.js
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`Hi, ${this.name}`);
  }
}
module.exports = Person;

// 导入后实例化
const Person = require("./Person.js");
const p = new Person("张三");
p.sayHi(); // Hi, 张三
```

### 动态导出

`require` 是 **运行时同步执行** 的，所以支持 **动态路径** 和 **条件导入**，这是 CommonJS 相比 ESModule 非常灵活的一点：

```javascript
// 动态路径
const fileName = "utils.js";
const utils = require("./" + fileName);

// 条件导入
if (process.env.NODE_ENV === "dev") {
  const devTools = require("./dev.js");
  devTools.init();
}
```

### 对象引用

CommonJS 提供了 `exports` 变量，它是 `module.exports` 对象的引用，相当于在每个模块顶部默认执行了 `const exports = module.exports`。

所以也可以直接给 `exports` 变量加属性来导出，和 `module.exports` 效果完全一致：

```javascript
// user.js 用exports简化写法
const name = "张三";
function getUserInfo() {
  return { name, age: 18 };
}

// 直接给exports加属性，等价于给module.exports加属性
exports.name = name;
exports.getUserInfo = getUserInfo;
```

{% note warning no-icon %}
不能直接给 `exports` 赋值，这样会切断 `const exports = module.exports` 的引用：

```javascript
// 错误写法：直接赋值exports
exports = { name: "张三" };
```

{% endnote %}

导出什么，导入就拿到什么，完全一一对应。

```javascript
// a.js 导出函数
module.exports = () => console.log("a");
// b.js 导入拿到的就是这个函数
const a = require("./a.js");
a(); // 打印'a'
```

### 注意事项

1. 如果只需要执行模块的代码，不需要接收导出内容，可以直接 `require` 不需要赋值：

   ```javascript
   // 导入直接执行，不需要变量接收
   require("./init.js"); // 打印 '模块初始化'
   ```

2. Node.js 中如果要在 CommonJS 里导入 ESModule 模块，不能用 `require`，必须用 `import()` 动态导入；而 ESModule 可以直接导入 CommonJS 模块。

   ```javascript
   async function loadESM() {
     const esModule = await import("./b.js"); // b.js是ESM模块
     console.log(esModule.default); // 取ESM的默认导出
     console.log(esModule.name); // 取ESM的命名导出
   }
   ```

   {% note info %}
   `import()` 是 JavaScript 中用于 **异步、按需加载模块** 的语法：传入模块的路径，它会返回一个 `Promise` 对象，ES Module 和 CommonJS 都支持。
   {% endnote %}

3. CommonJS 是 **同步加载模块**。Node.js 中的模块都在本地磁盘，加载速度快；浏览器环境下模块需要网络请求，同步加载会阻塞渲染，所以浏览器端一般用 ES Module，或者把 CommonJS 打包成兼容浏览器的代码。

## ES Module

ES Module 是 ES6 推出的标准模块化规范，目前已经成为浏览器和 Node.js 通用的模块化方案，彻底统一了前端和服务端的模块化标准，逐步取代了早期的 CommonJS、AMD 等非官方规范。

### 导出和导入

**1. 命名导出**

```javascript
// 写法1：声明时直接导出
export const name = "张三";
export function getUserInfo() {
  return { name, age: 18 };
}
export class Person {
  constructor(name) {
    this.name = name;
  }
}

// 写法2：统一导出（和上面等价，更清晰）
const name = "张三";
function getUserInfo() {
  /*...*/
}
class Person {
  /*...*/
}

export { name, getUserInfo, Person };
// 导出时重命名
export { name as userName }; // 对外导出的名称是userName
```

**2. 默认导出**

一个模块只能有一个默认导出，适合模块只有一个核心功能的场景。

```javascript
// 默认导出一个函数
export default function add(a, b) {
  return a + b;
}

// 也可以导出对象、类、变量等
export default {
  name: "工具模块",
  add: (a,b) => a+b
};
```

{% note success %}
一个模块可以 **同时存在默认导出和命名导出**。
{% endnote %}

**3. 导入的写法**

```javascript
// 导入命名导出
import { name, getUserInfo, Person } from "./user.js";
// 导入时重命名
import { name as userName } from "./user.js";
// 批量导入所有内容到一个对象上
import * as user from "./user.js";
console.log(user.name);

// 导入默认导出，名称随便写
import add from "./utils.js";
import myAdd from "./utils.js"; // 和上面等价

// 同时导入默认导出和命名导出
import add, { PI } from "./math.js";

// 如果只需要执行模块的初始化逻辑（比如全局样式注入、全局变量挂载），不需要接收导出
import "./init.js";
```

{% note info %}
一个模块的所有命名导出，会被整合为一个 **导出对象**，导入时加 `{}` 的写法，就是从这个导出对象里 **解构** 出需要的属性。
{% endnote %}

### ESM 特性

**1. 静态解析**

`import/export` 必须写在模块顶层，**不能用在条件判断、函数内部，也不支持动态路径**。这个限制让打包工具可以在编译阶段就分析出哪些模块代码没有被使用，直接删除（Tree Shaking 摇树优化），大幅减少打包体积。

如果需要动态导入，可以使用 `import()` 语法，返回 `Promise`，支持动态路径和条件加载：

```javascript
if (process.env.NODE_ENV === "dev") {
  import("./dev.js").then((devModule) => {
    devModule.init();
  });
}

// async/await 写法
async function loadUtils() {
  const utils = await import("./utils.js");
  utils.add(1, 2);
}
```

**2. 只读引用**

ESM 导出的是值的 **只读引用**，模块内部修改导出的值，外部导入的结果会同步更新，而 CommonJS 是 **值拷贝**。

```javascript
// count.js ESM导出
export let count = 1;
export function add() {
  count++;
}

// 导入count.js
import { count, add } from "./count.js";
console.log(count); // 1
add();
console.log(count); // 2（模块内部修改，外部值同步更新）

// 引用是只读的，外部不能直接修改导出的值，只能通过模块内部暴露的方法修改
import { count } from "./count.js";
count = 2; // 报错：Assignment to constant variable
```

**3. 模块缓存**

和 CommonJS 一样，ESM 同一个模块只会被加载一次，**连续导入直接复用缓存**，不会重复执行模块代码。

### 使用场景

**1. 浏览器中使用 ESM**

只需要给 script 标签加 `type="module"` 属性即可。

{% hideToggle script 标签加载 %}
普通 script 标签的加载逻辑是：

1. 浏览器解析 HTML 碰到 `<script>` 时，暂停 HTML 解析，立刻去请求 `a.js`；
2. 请求到之后立刻执行 `a.js`，执行完才会继续请求下一个 `b.js`；
3. 所有 `<script>` 标签加载执行完，才会继续解析后面的 HTML、渲染页面。

这种模式下，script 会 **阻塞页面渲染**，如果脚本大 / 网络慢，页面会长时间空白，这就是 **同步阻塞**。

当用 `<script type="module">` 引入 ESM 模块时，默认行为和普通 script 完全不同，相当于自带 `defer` 属性：

1. 浏览器解析 HTML 碰到 ESM script，不暂停 HTML 解析，后台 **异步请求** `main.js`；
2. 拿到 `main.js` 后，不会立刻执行，而是先静态分析代码里的 `import` 依赖，继续异步并行请求所有依赖的子模块（比如 `main.js` 里 `import` 了 `utils.js`、`component.js`，这些请求是同时发的，不需要等前一个加载完）；
3. 所有依赖模块都加载完成，且 **HTML 解析完毕** 后，再按模块的依赖顺序依次执行代码（先执行被依赖的子模块，再执行入口 `main.js`）。

{% endhideToggle %}

**2. Node.js 中使用 ESM**

- 修改文件后缀为 `.mjs` ：只要文件后缀是 `.mjs`，Node.js 就会自动当做 ESM 解析。
- 在 `package.json` 中设置 `"type": "module"` ：所有 `.js` 文件都会默认当做 ESM 解析，不需要改后缀，如果想要个别文件用 CommonJS 解析，把后缀改成 `.cjs` 即可。

**3. ESM 中导入 CommonJS 模块**

ESM 可以直接导入 CommonJS 模块，CommonJS 的 `module.exports` 会被当做 ESM 的默认导出。

```javascript
// a.js 是CommonJS模块
module.exports = { name: "张三" };

// ESM中导入CommonJS
import commonjsModule from "./a.js";
console.log(commonjsModule.name); // 张三
// 也可以解构导入（但不推荐，因为CommonJS是动态导出，编译时无法确定结构，可能导致Tree Shaking失效）
import { name } from "./a.js";
```

## 核心区别

| 特性      | ES Module                                    | CommonJS                                     |
| --------- | -------------------------------------------- | -------------------------------------------- |
| 加载阶段  | 编译时加载（代码执行前就确定导入导出关系）   | 运行时加载（执行 `require` 时才加载模块）    |
| 加载方式  | 异步加载（并行加载模块，不阻塞渲染）         | 同步加载（适合本地磁盘加载，浏览器端会阻塞） |
| 导出特性  | 导出值的引用（内部修改，外部导入会同步变化） | 导出值的拷贝（导出后内部修改不影响外部）     |
| this 指向 | `undefined`（模块独立作用域）                | 指向 `module.exports`                        |
| 动态导入  | `import()` 动态导入（返回 `Promise`）        | 原生支持动态 `require`                       |
| 适用场景  | 浏览器 和 Node.js 通用                       | 仅 Node.js 原生支持                          |

**最佳实践**

1. **优先使用 ESM 作为项目的模块化标准**：目前前端工程化工具都默认优先支持 ESM，Node.js 也已经全面兼容，ESM 是未来的主流。
2. **尽量用命名导出代替默认导出**：命名导出可以强制名称一致，避免多人开发时自定义名称导致的混乱，同时更利于 Tree Shaking 优化。
3. **避免混合使用 ESM 和 CommonJS**：混用会增加构建和运行时的复杂度，容易出现兼容性问题，新项目尽量统一用 ESM。
4. **动态导入用来做代码分割**：`import()` 非常适合做路由懒加载、按需加载（比如点击按钮才加载某个模块），可以大幅减少首屏包体积。
