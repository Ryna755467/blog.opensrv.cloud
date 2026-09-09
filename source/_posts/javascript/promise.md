---
title: 异步编程
date: 2026-07-06
updated: 2026-07-06
top_img: /img/series/javascript.jpg
cover: /img/covers/javascript/promise.jpg
series: javascript
categories:
  - JavaScript
---

## 异步编程体系

{% note primary %}
JS 是 **单线程 + 事件循环**，所有耗时操作丢给浏览器 / Node 底层多线程处理，完成后把回调推入 **任务队列**，主线程同步代码执行完再取队列执行。所有异步方案，全部基于 **回调函数入队执行** 这套底层机制搭建。
{% endnote %}

### 回调函数模式

回调函数模式是 JS 异步编程 **最原始、最通用** 的实现方式，所有更高层的异步方案本质都是 **对回调的封装**。

**核心逻辑**
异步操作执行完成后，调用预先传入的回调函数来传递结果，通常遵循 **错误优先** 的约定（Node.js 生态的统一规范）。

{% note info %}
**错误优先**：第一个参数永远是 **错误对象**，成功时 `error` 为 `null`，后续为结果。
{% endnote %}

{% tabs error_first %}

<!-- tab 错误优先回调 -->

```javascript
// Node 文件读取 API
fs.readFile("./a.txt", function (err, data) {
  if (err) {
    // 单独处理当前层错误
    console.error(err);
    return;
  }
  console.log(data.toString());
});
```

<!-- endtab -->

<!-- tab 成功 / 失败双回调 -->

```javascript
// 模拟原生请求
function getUser(success, fail) {
  const xhr = new XMLHttpRequest();
  xhr.open("GET", "/user");
  xhr.onload = () => success(JSON.parse(xhr.responseText));
  xhr.onerror = () => fail(new Error("网络错误"));
  xhr.send();
}
```

<!-- endtab -->

{% endtabs %}

**局限性**

1. 回调地狱：多个串行异步操作会导致 **嵌套层级过深**，代码可读性、可维护性极差。
2. 并发计数：框架本身 **没有并发能力**，需要开发者自己写计数、数组存储结果。
3. 控制反转：回调函数交给 **第三方异步API执行**，无法保证次数、时机、参数的正确性。
4. 错误处理分散：每个回调都要 **单独判断错误**，无法统一捕获。

{% tabs limitations %}

<!-- tab 回调地狱 -->

```javascript
getUser(
  (user) => {
    getOrder(
      user.id,
      (order) => {
        getGoods(
          order.gid,
          (goods) => {
            render(goods);
          },
          (err3) => {},
        );
      },
      (err2) => {},
    );
  },
  (err1) => {},
);
```

<!-- endtab -->

<!-- tab 并发计数 -->

```javascript
const ids = [1, 2, 3];
let result = [];
let finishCount = 0;
ids.forEach((id) => {
  getUser(
    id,
    (data) => {
      result.push(data);
      finishCount++;
      if (finishCount === ids.length) {
        // 全部完成再处理
        console.log(result);
      }
    },
    (err) => {},
  );
});
```

<!-- endtab -->

{% endtabs %}

### 发展历程

{% timeline 1995 年 12 月，JavaScript 对外发布。 %}

<!-- timeline 1995 - 2009 年 -->

**原始回调模式**
浏览器的 `setTimeout`、`XMLHttpRequest` 方法。

<!-- endtimeline -->

<!-- timeline 2009 - 2015 年 -->

**流程控制库**
`Async.js`、`jQuery.Deferred`、事件发布订阅、Thunk 模式。

<!-- endtimeline -->

<!-- timeline 2015 - 2017 年 -->

**规范过渡阶段**
原生 Promise 正式成为语言标准（ES6），**Promise 开始普及**。

<!-- endtimeline -->

<!-- timeline 2017 年 - 至今 -->

**标准异步范式**
引入了 `async / await` 语法糖（ES8），形成了 **标准异步范式**。

<!-- endtimeline -->

{% endtimeline %}

所有 Promise 之前的异步方案，都没有解决最核心的 **标准化** 问题：

1. 没有统一的规范：不同库的异步API设计不一致，**跨项目学习成本高**；
2. 没有解决控制反转问题：回调的执行权还是在第三方库，仍然可能出现回调 **被多次调用、不调用** 的问题；
3. 没有统一的错误处理机制：每个方案的错误捕获逻辑都不一样，无法形成通用的 **错误冒泡机制**。

{% note success %}
Promise 从规范层面 **统一了异步操作的接口**，解决了上述所有问题，后来的 `async/await` 也是基于 Promise 实现的，最终形成了现在的异步编程标准。
{% endnote %}

## Promise 实现原理

一个异步操作，对应三种状态：**正在执行 / 执行成功 / 执行失败**，当异步操作结束时，执行对应的 **成功 / 失败** 回调。

1. 需要有一个属性保存 **异步任务的状态**；
2. 需要有一个属性保存 **异步任务的执行结果**；
3. 注册回调时，如果异步任务已经结束，那么立刻执行回调；如果异步任务正在执行，需要有一个数组 **保存回调**。

### 实例属性

| 属性      | 类型                                   | 初始值      |
| --------- | -------------------------------------- | ----------- |
| state     | `'pending' / 'fulfilled' / 'rejected'` | `'pending'` |
| result    | 任意类型                               | `undefined` |
| callbacks | 对象数组                               | `[]`        |

{% note success %}
`state` 是 **不可逆的**，不允许多次调用 `resolve/reject` 改变状态。这个设定符合现实逻辑，并且 **确保链式调用的可靠性**。
{% endnote %}

{% note primary %}
`callbacks` 的存储单元是包含了一组回调的对象，而不是零散独立的两个函数，因为成功和失败的回调总是成对注册的，它们 **关联同一个异步任务**。
{% endnote %}

### 构造函数

创建一个 Promise 实例，需要传入一个执行器函数来 **启动异步任务**。

执行器函数是用户定义的，Promise 自身 **无法感知** 异步操作什么时候完成、成功还是失败。因此需要定义 `resolve` 和 `reject` 方法，让用户在异步逻辑完成时，**主动通知** Promise：任务成功 / 任务失败。

{% note info no-icon %}
**必须在构造函数内部调用执行器函数**
Promise 实现了状态控制器、状态机、回调队列的 **封装**。只有在 Promise 内部启动异步任务，才能保证任务完成的通知只通过受控的 `resolve/reject` 触发，**避免外部绕过状态规则**。
{% endnote %}

{% note warning %}
`resolve` 和 `reject` 不是原型方法。它是构造函数内部的闭包函数，**只允许执行器调用**，不对外暴露。
{% endnote %}

构造函数需要 **初始化属性**，并执行 `executor` 方法 **启动异步任务**：

```javascript
  constructor(executor) {
    this.state = 'pending';
    this.result = undefined;
    this.callbacks = [];

    const resolve = (value) => {
      if (this.state !== 'pending') return;
      this.state = 'fulfilled';
      this.result = value;
      this.callbacks.forEach((item) => item.onFulfilled(value));
    };
    const reject = (reason) => {
      if (this.state !== 'pending') return;
      this.state = 'rejected';
      this.result = reason;
      this.callbacks.forEach((item) => item.onRejected(reason));
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
```

**`resolve/reject` 中的 this 指向问题**

`class` 内部自动开启 **严格模式**，`resolve()` 被执行器调用时 `this = undefined`，解决方案有：

{% tabs this_binding, 1 %}

<!-- tab 箭头函数锁定 -->

```javascript
const resolve = (value) => {
  if (this.state !== "pending") return;
  this.state = "fulfilled";
  this.result = value;
  this.callbacks.forEach((item) => item.onFulfilled(value));
};
```

<!-- endtab -->

<!-- tab 绑定当前实例 -->

```javascript
const resolve = function (value) {
  if (this.state !== "pending") return;
  this.state = "fulfilled";
  this.result = value;
  this.callbacks.forEach((item) => item.onFulfilled(value));
}.bind(this);
```

<!-- endtab -->

<!-- tab 闭包捕获 -->

```javascript
const _this = this; // 缓存实例
const resolve = function (value) {
  if (_this.state !== "pending") return;
  _this.state = "fulfilled";
  _this.value = value;
  _this.callbacks.forEach((item) => item.onFulfilled(value));
};
```

<!-- endtab -->

{% endtabs %}

### 回调注册

`then()` 用于注册异步任务结束后的回调。注册时异步任务未结束则 **加入回调数组**，已结束则 **立刻执行**。

为了实现链式调用，代码中固定返回一个新的 Promise：

1. 如果用户在 `onFulfilled / onRejected` 中已经返回了一个 Promise，就让新创建的实例和用户返回的 Promise **状态对齐**（即 `state` 和 `result` 对齐）；
2. 如果用户在 `onFulfilled / onRejected` 中返回了普通值，就把它包装成 `fulfilled` 状态的 Promise，触发下一层的 `onFulfilled` 回调。

{% note info %}
Promise/A+ 规范要求 `then` 注册的回调必须 **异步执行**，可以通过 `queueMicrotask` 实现：`queueMicrotask()` 用于将一个函数推入 **微任务队列**，在同步代码执行完毕后、下一次事件循环前执行。
{% endnote %}

```javascript
then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const safeOnFulfilled =
        typeof onFulfilled === 'function' ? onFulfilled : (v) => v;
      const safeOnRejected =
        typeof onRejected === 'function'
          ? onRejected
          : (err) => {
              throw err;
            };

      const handleFulfilled = (value) => {
        queueMicrotask(() => {
          try {
            const result = safeOnFulfilled(value);
            if (result instanceof MyPromise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (err) {
            reject(err);
          }
        });
      };
      const handleRejected = (reason) => {
        queueMicrotask(() => {
          try {
            const result = safeOnRejected(reason);
            if (result instanceof MyPromise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (err) {
            reject(err);
          }
        });
      };

      if (this.state === 'fulfilled') {
        handleFulfilled(this.result);
      } else if (this.state === 'rejected') {
        handleRejected(this.result);
      } else {
        this.callbacks.push({
          onFulfilled: handleFulfilled,
          onRejected: handleRejected,
        });
      }
    });
  }
```

{% note info %}
**回调的执行时机**：一次是 `then()` 注册时状态已经改变，立刻执行；另一次是 `then()` 注册时状态为 `pending`，先存到回调数组，等 `resolve / reject` 执行后触发。
{% endnote %}

{% note warning %}
**用户自定义函数** 执行的地方都需要用 `try/catch` 捕获，也就是 `executor` 和 `onFulfilled / onRejected` 执行的地方。
{% endnote %}

{% note primary no-icon %}
`then、catch、finally` 等方法都允许不传参数，需要考虑 **空值兜底**。

`then` 本质是消费上一层产生的值，可以选择不处理，直接透传给下一层，它的空值兜底逻辑是 **原样传递**：

```javascript
const safeOnFulfilled =
  typeof onFulfilled === "function" ? onFulfilled : (v) => v;

const safeOnRejected =
  typeof onRejected === "function"
    ? onRejected
    : (err) => {
        throw err;
      };
```

如果 `then` 省略了 `onRejected` 方法，那么错误会向后传递直到被捕获，因此它的空值兜底是抛出错误。

{% endnote %}

**注意事项**

1. `handleFulfilled` 和 `handleRejected`：**执行处理函数拿到结果，并透传给下一层**。
   这个返回的 Promise 是类自身创建的，`resolve` 和 `reject` 由类自身控制，只有当处理函数执行失败时（被 `try/catch` 捕获）才调用 `reject`。
2. 在 `handleRejected` 中对于非 Promise 类型的 `result` 用的是 `resolve` 而非 `reject`。
   `then().then()` 链式调用时，如果第一个 `then()` 触发了 `onRejected` 并返回普通值，那么第二个 `then()` 会触发 `onFulfilled`。

### 工具方法

ES6 Promise 是 **Promises/A+** 规范的一个 **具体实现**，它额外扩展了 `catch` / `finally` / `static resolve` / `static reject` / `static all` / `static race` 等工具方法。

{% note info %}
**Promises/A+** 规范是一个 **核心标准**，它只规定了 `then` 方法应该如何工作，其他所有方法都是 ES6 中基于 `then` 构建的便捷方法。
{% endnote %}

{% label catch blue %}
本质就是 `then(null, onRejected)` 的一个别名，专门用于处理 `rejected` 状态。

```javascript
  catch(onRejected) {
    return this.then(null, onRejected);
  }
```

{% label Promise.resolve orange %}
**将任意变量统一转换成 Promise 对象。**

返回一个 `fulfilled` 状态的 Promise 对象；如果 `Promise.resolve` 的参数是一个 Promise 实例（无论状态是 `fulfilled / rejected / pending`），直接原样返回。

```javascript
  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve) => {
      resolve(value);
    });
  }
```

{% label Promise.reject green %}
**创建一个新的 `rejected` 状态的 Promise 对象。**

```javascript
  static reject(reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason);
    });
  }
```

{% label finally red %}
**无论异步操作结果如何，都需要执行的操作。**

实现思路是创建一个 **Promise 中间层** 处理 `callback`，但是不使用它的 `result`，而是传递上一层的 `result`。如果 `callback` 返回一个 Promise，可以直接用它作为中间层，否则创建一个 Promise 中间层：

1. `static resolve` 可以很方便的实现这个功能：`MyPromise.resolve(cb())`；
2. 也可以直接创建 Promise 作为中间层，但要求当参数为 Promise 时，`resolve` 能够 **等待它状态改变后再执行**（即 `resolve` 需要实现 **thenable 解析**）：`new MyPromise(resolve => resolve(cb()))`。

```javascript
  finally(callback) {
    const cb = typeof callback === 'function' ? callback : () => {};
    return this.then(
      (value) => MyPromise.resolve(cb()).then(() => value),
      (reason) =>
        // 这里只定义了onFulfilled回调
        MyPromise.resolve(cb()).then(() => {
          // 若cb()同步抛错或返回rejected Promise，新错误会覆盖上层reason并向后传递
          throw reason;
        }),
    );
  }
```

{% note primary no-icon %}
**这里一共有三层 Promise：**

1. 调用 `finally` 的 **源 Promise**，它的 `result` 会被透传；
2. `MyPromise.resolve(cb())` 生成的 **中间层 Promise**，它的 `result` 会被抛弃；
3. 最终 `return this.then(...)` 创建并返回的 **新 Promise**。

{% endnote %}

{% note warning %}
当 `cb()` 抛出异常时，实际上是被 `return this.then(...)` 中的 `try catch` 捕获的。
{% endnote %}

{% label Promise.all pink %}
**传入一个 可迭代对象，返回一个 Promise 对象。**

1. 可迭代对象中所有的 Promise 都变为 `fulfilled` 状态时，返回的 Promise 变为 `fulfilled`，它的 `value` 是一个数组，对应传入的可迭代对象的 `value`（**按照传入顺序而非完成顺序**）；
2. 可迭代对象中有一个 Promise 变为 `rejected` 状态时，返回的 Promise 立刻变为 `rejected`，它的 `reason` 为 **第一个抛出错误的 Promise** 对应的 `reason`。

{% note warning %}
如果可迭代对象的长度为 `0`，返回的 Promise 对象会立刻 `resolve([])`。
{% endnote %}

`all()` 传入的是一个可迭代对象，可以通过 `Array.from()` 把它转换为数组。

`Array.from(arrayLike, mapFn?, thisArg?)`

| 参数      | 说明                                     |
| --------- | ---------------------------------------- |
| arrayLike | 要转换的类数组对象或可迭代对象           |
| mapFn     | 映射函数，相当于对每个元素执行一次 `map` |
| thisArg   | 执行 `mapFn` 时的 `this` 指向            |

{% note info no-icon %}
`Array.from()` 用于从 **类数组对象** 或 **可迭代对象** 创建一个新的、浅拷贝的数组实例。

**生成指定长度的序列**

```javascript
// 生成 1~5 的数字序列
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]

// 生成长度为 5 的随机数数组
Array.from({ length: 5 }, () => Math.random());

// 生成长度为 3 的重复值数组
Array.from({ length: 3 }, () => 0); // [0, 0, 0]

// 生成指定范围内的偶数
Array.from({ length: 5 }, (_, i) => i * 2); // [0, 2, 4, 6, 8]
```

{% endnote %}

```javascript
  static all(iterable) {
    return new MyPromise((resolve, reject) => {
      const promises = Array.from(iterable);
      const result = [];
      let count = 0;

      if (promises.length === 0) {
        resolve(result);
        return;
      }

      promises.forEach((item, index) => {
        MyPromise.resolve(item).then((value) => {
          result[index] = value;
          count++;
          if (count === promises.length) resolve(result);
        }, reject);
      });
    });
  }
```

{% hideToggle 类数组对象 %}
类数组对象是指：拥有 `length` 属性和索引元素（`0, 1, 2...`），但不具备数组的方法（如 `push、forEach、map` 等）的对象。

一个对象要被称为 **类数组对象**，必须满足两个条件：

1. 有 `length` 属性（数字，表示长度）；
2. 有索引属性（`key` 是 `0, 1, 2...` 的数字，且从 `0` 开始连续）。

{% tabs array‑like, 1 %}

<!-- tab arguments -->

```javascript
function test() {
  console.log(arguments); // {0: 'a', 1: 'b', 2: 'c', length: 3}
  console.log(Array.isArray(arguments)); // false
  console.log(arguments.length); // 3
}
test("a", "b", "c");
```

<!-- endtab -->

<!-- tab DOM 集合 -->

```javascript
const divs = document.querySelectorAll("div");
console.log(divs); // NodeList(3) [div, div, div]
console.log(divs.length); // 3
console.log(divs[0]); // 第一个 div 元素
console.log(Array.isArray(divs)); // false
```

<!-- endtab -->

<!-- tab 字符串 -->

```javascript
const str = "hello";
console.log(str.length); // 5
console.log(str[0]); // 'h'
console.log(Array.isArray(str)); // false
```

<!-- endtab -->

<!-- tab 自定义对象 -->

```javascript
const arrayLike = {
  0: "apple",
  1: "banana",
  2: "orange",
  length: 3,
};
```

<!-- endtab -->

{% endtabs %}

{% endhideToggle %}

{% label Promise.race purple %}
**传入一个 可迭代对象，返回一个 Promise 对象。**
返回的 Promise 的最终状态与结果会 **对齐** 可迭代对象中第一个落定的 Promise。

{% note warning %}
如果可迭代对象的长度为 `0`，返回的 Promise 对象会永久 `pending`。
{% endnote %}

```javascript
static race(iterable) {
    return new MyPromise((resolve, reject) => {
      const promises = Array.from(iterable);
      if (promises.length === 0) {
        return;
      }

      promises.forEach((item) => {
        MyPromise.resolve(item).then(resolve, reject);
      });
    });
  }
```

{% note primary %}
`return this.then()` 这种写法要求有一个上游 Promise，目的是 **实现串联调用**。`all` 和 `race` 是静态方法 **不关联实例**，所以需要创建一个新的 Promise。
{% endnote %}

### 完整示例

```javascript
class MyPromise {
  constructor(executor) {
    this.state = "pending";
    this.result = undefined;
    this.callbacks = [];

    const resolve = (value) => {
      if (this.state !== "pending") return;
      this.state = "fulfilled";
      this.result = value;
      this.callbacks.forEach((item) => item.onFulfilled(value));
    };

    const reject = (reason) => {
      if (this.state !== "pending") return;
      this.state = "rejected";
      this.result = reason;
      this.callbacks.forEach((item) => item.onRejected(reason));
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const safeOnFulfilled =
        typeof onFulfilled === "function" ? onFulfilled : (v) => v;
      const safeOnRejected =
        typeof onRejected === "function"
          ? onRejected
          : (err) => {
              throw err;
            };

      const handleFulfilled = (value) => {
        queueMicrotask(() => {
          try {
            const result = safeOnFulfilled(value);
            if (result instanceof MyPromise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (err) {
            reject(err);
          }
        });
      };

      const handleRejected = (reason) => {
        queueMicrotask(() => {
          try {
            const result = safeOnRejected(reason);
            if (result instanceof MyPromise) {
              result.then(resolve, reject);
            } else {
              resolve(result);
            }
          } catch (err) {
            reject(err);
          }
        });
      };

      if (this.state === "fulfilled") {
        handleFulfilled(this.result);
      } else if (this.state === "rejected") {
        handleRejected(this.result);
      } else {
        this.callbacks.push({
          onFulfilled: handleFulfilled,
          onRejected: handleRejected,
        });
      }
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  finally(callback) {
    const cb = typeof callback === "function" ? callback : () => {};
    return this.then(
      (value) => MyPromise.resolve(cb()).then(() => value),
      (reason) =>
        MyPromise.resolve(cb()).then(() => {
          throw reason;
        }),
    );
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve) => {
      resolve(value);
    });
  }

  static reject(reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason);
    });
  }

  static all(iterable) {
    return new MyPromise((resolve, reject) => {
      const promises = Array.from(iterable);
      const result = [];
      let count = 0;

      if (promises.length === 0) {
        resolve(result);
        return;
      }

      promises.forEach((item, index) => {
        MyPromise.resolve(item).then((value) => {
          result[index] = value;
          count++;
          if (count === promises.length) resolve(result);
        }, reject);
      });
    });
  }

  static race(iterable) {
    return new MyPromise((resolve, reject) => {
      const promises = Array.from(iterable);
      if (promises.length === 0) {
        return;
      }

      promises.forEach((item) => {
        MyPromise.resolve(item).then(resolve, reject);
      });
    });
  }
}
```

## async/await 实现原理

`async/await` 是 ES8 引入的 **语法糖**，作用是把异步逻辑写得和同步代码一样直观。

{% note success %}
`async/await` 是 JS 异步编程的 **主流方案**，所有需要处理异步逻辑的场景都优先用它实现。
{% endnote %}

### 基础规则

1. `async` 用来标记函数是 **异步函数**：

- `async function fn() {}` 或者箭头函数 `const fn = async () => {}`；
- 异步函数的返回值会自动包装成 Promise：如果返回普通值，会变成 `Promise.resolve(普通值)`；如果抛出错误，会变成 `Promise.reject(错误)`。

2. `await` 只能在 `async` 函数内部使用：

- `await` 后面一般跟 Promise 对象，作用是暂停当前 `async` 函数的执行，等待状态变更为 `fulfilled` 后，再继续执行后面的代码；
- `await` 的返回值就是 Promise 成功的结果；如果状态变为 `rejected`，会 **抛出错误**，需要外层用 `try/catch` 捕获。

{% note primary no-icon %}
如果多个异步任务 **没有依赖关系**，不要串行调用 `await`，应该先用 `Promise.all()` **并行发起所有请求**，再统一 `await` 结果：

```javascript
async function fetchAll() {
  // 错误写法：串行执行
  const res1 = await request("/api1");
  const res2 = await request("/api2");
  const res3 = await request("/api3");

  // 正确写法：并行执行
  const promise1 = request("/api1");
  const promise2 = request("/api2");
  const promise3 = request("/api3");
  // 统一等待所有Promise完成
  const [res1, res2, res3] = await Promise.all([promise1, promise2, promise3]);
}
```

{% endnote %}

### 实现原理

`async/await` 是 **生成器 + Promise + 自动执行器** 的语法糖，核心就是 **自动迭代生成器**，把每次 `yield` 出来的 Promise 结果，通过 `next()` 传回生成器，直到生成器执行完毕。

```javascript
async function foo() {
  let a = await Promise.resolve(1);
  let b = await Promise.resolve(2);
  return a + b;
}
foo();

// 等效于
function* fooGenerator() {
  let a = yield Promise.resolve(1);
  let b = yield Promise.resolve(2);
  return a + b;
}
run(fooGenerator);
```

**自动执行器**

```javascript
function run(genFunc) {
  const iterator = genFunc();
  // 外层返回Promise，和原生async行为一致
  return new Promise((resolve) => {
    (function nextStep(value) {
      const result = iterator.next(value);
      if (result.done) {
        resolve(result.value);
        return;
      }
      result.value.then(nextStep);
    })();
  });
}
```

---

{% series javascript %}
