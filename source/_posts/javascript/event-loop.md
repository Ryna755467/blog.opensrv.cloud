---
title: 事件循环
date: 2026-08-02
updated: 2026-08-02
top_img: /img/javascript_top.jpg
cover: /img/event-loop_cover.png
series: javascript
categories:
  - JavaScript
---

### JavaScript 设计为单线程的原因

JavaScript 在1995年诞生时就是作为浏览器脚本，**让网页拥有交互能力**。

如果允许多线程操作 DOM，需要引入 **锁机制**，但早期网页只需要简单的表单验证和动画，**从简单、安全的角度考虑**，JS被设计成了单线程，从根源上 **避免了 DOM 操作可能产生的并发冲突**。

{% note info %}
Java 和 JavaScript 语言毫无关系，只是为了商业营销而改名。
{% endnote %}

{% note warning %}
`WebWorker` 拥有独立线程，但是不能访问 DOM，DOM 操作 **必须限制在主线程**。
{% endnote %}

{% hideToggle DOM 和 BOM %}
DOM（文档对象模型）提供 **操作文档节点的 API**，`document` 是 DOM 的根节点。
BOM（浏览器对象模型）提供 **与浏览器窗口交互的 API**，`window` 是 BOM 的根结点。
{% endhideToggle %}

### 为什么需要任务队列

JavaScript 的异步 API 是 **由宿主环境提供** 的，定时器、文件读写等异步操作都交给浏览器底层线程处理，它们的回调 **不能立刻执行**，必须先存起来，存放容器就是 **任务队列**。

{% note info %}
JS 采用 **非抢占式调度**，不允许中途打断正在执行的同步代码：一旦允许随时插入回调，执行顺序不可预测，会产生大量难以复现的 Bug。
{% endnote %}

{% hideToggle 宿主环境和运行时 %}
**宿主环境** 指承载 JavaScript 代码运行的外部平台，是完整的大环境：

1. 浏览器（Chrome、Firefox）是一类宿主；
2. Node.js 是另一类宿主；
3. 小程序引擎、Electron 都属于宿主。

宿主环境会给 JS 注入独有的全局 API：

| 宿主    | API                                              |
| ------- | ------------------------------------------------ |
| 浏览器  | `window` / `document` / `setTimeout` / `DOM API` |
| Node.js | `global` / `fs` / `path` / `net` / `process`     |

**运行时** 是宿主内部专门执行 JS 代码的引擎配套体系，核心分为两部分：

1. JS 引擎（V8、SpiderMonkey、JavaScriptCore等，**属于运行时的一部分**）：解析、执行 JS 代码；
2. 运行时底层能力：调用栈、堆内存、任务队列（宏/微任务）、垃圾回收。

**边界区分**

1. **运行时** 只懂 ECMAScript 标准，没有文件、网络、定时器；
2. **宿主环境** 在运行时基础上，额外提供外部能力，拓展了 JS 边界。

{% endhideToggle %}

早期的浏览器只有宏任务队列，事件循环的过程为：

{% mermaid '{"themeVariables":{"fontSize":"15px"},"layout":"elk","look":"handDrawn"}' %}
sequenceDiagram
participant JS主线程
participant 宏任务队列
participant 浏览器渲染

    loop 事件循环轮次
        JS主线程->>宏任务队列: 取出一个宏任务
        宏任务队列-->>JS主线程: 返回待执行任务
        JS主线程->>JS主线程: 执行宏任务，调用栈运行
        JS主线程->>浏览器渲染: 执行页面渲染
    end

{% endmermaid %}

### 为什么需要微任务

微任务是一批 **高优先级、顺序确定、不可插队** 的异步任务：

1. 高优先级：回调执行延迟要理论最小，紧跟同步代码执行；
2. 顺序确定：回调的执行顺序能被精准预测；
3. 不可插队：回调执行过程不会被任何任务打断，状态完全可控。

引入微任务是为了解决：宏任务每执行一个就可能触发渲染，导致 **中间状态可见，并且异步回调延迟偏高** 的问题。

{% note info %}
**浏览器环境** 有多条宏任务队列（timer、用户交互、网络 I/O…），每轮事件循环按优先级挑一条队列取第一个任务执行，因此跨队列时完全可能 **后到先跑**；而微任务队列只有一条，绝对 **先进先出**。
{% endnote %}

### 事件循环的过程

一次完整的事件循环包括：**取一个宏任务并执行 → 清空所有微任务 → 可能的 UI 渲染**。

{% mermaid '{"themeVariables":{"fontSize":"15px"},"layout":"elk","look":"handDrawn"}' %}
sequenceDiagram
participant JS主线程
participant 宏任务队列
participant 微任务队列
participant 浏览器渲染

    loop 事件循环轮次
        JS主线程->>宏任务队列: 取出一个宏任务
        宏任务队列-->>JS主线程: 返回待执行任务
        JS主线程->>JS主线程: 执行宏任务，调用栈运行
        JS主线程->>微任务队列: 循环取出所有微任务
        微任务队列-->>JS主线程: 全部执行完毕
        JS主线程->>浏览器渲染: 执行页面渲染
    end

{% endmermaid %}

### 浏览器中的宏任务和微任务

**宏任务**：

1. `setTimeout`、`setInterval` 定时器回调；
2. `click`、`scroll`、`keydown` 等 DOM 事件回调；
3. `XMLHttpRequest` 等网络请求回调；
4. `<script>` 整体脚本执行。

**微任务**：

1. `Promise.then / .catch / .finally`；
2. `MutationObserver`（监听 DOM 变化）。
3. `queueMicrotask()`；

{% note info %}
同步代码本身不是宏任务，它是 **宏任务执行的内容**。
{% endnote %}

{% note warning %}
UI 渲染既不是宏任务，也不是微任务，它是浏览器宿主行为，不属于 JS 任务队列。
{% endnote %}

---

{% series javascript %}
