---
title: 遍历
date: 2026-08-04
updated: 2026-08-04
top_img: /img/series/javascript.jpg
cover: /img/covers/iteration.png
series: javascript
categories:
  - JavaScript
---

# 数组遍历

## 全量遍历

{% label forEach blue %}
遍历数组，对每一项执行回调函数，**无返回值**，除非抛出错误否则 **无法中断遍历**。

`arr.forEach(callback(currentValue, index, array), thisArg)`

**参数列表**

| 参数         | 说明                         |
| ------------ | ---------------------------- |
| currentValue | 当前遍历的元素               |
| index        | 当前元素索引（可选）         |
| array        | 调用方法的原数组（可选）     |
| thisArg      | 指定回调内的this指向（可选） |

**调用示例**

```javascript
const fruits = ["苹果", "香蕉", "橙子"];

const context = {
  prefix: "我喜欢的水果：",
  printFruit: function (name) {
    console.log(this.prefix + name);
  },
};

fruits.forEach(function (currentValue, index, arr) {
  console.log("当前元素：", currentValue);
  console.log("当前索引：", index);
  // 调用方法的原数组（可以用来校验/修改原数组）
  console.log("原数组：", arr);
  this.printFruit(currentValue);
}, context);
// 指定回调内的this为context
```

**注意事项**

1. 函数参数是 **按位置匹配** 的，如果要拿到第三个参数 `array`，前面的两个参数必须占位置，不能直接跳过第二个写第三个，不需要的参数可以用 `_` 占位。
2. 如果要让 `thisArg` 生效，回调函数就 **不能用箭头函数**；如果回调函数内部是一个复用的代码块，那么可以用 `thisArg` 参数设置 `this`，避免在复用代码块内部硬编码。

---

{% label map pink %}
对每一项执行回调，把回调的返回值收集为新数组返回，**原数组保持不变**。

`const newArr = arr.map(callback(currentValue, index, array), thisArg)`

语法和参数与 `forEach` 方法完全一致。

**调用示例**

{% tabs map, 2 %}

<!-- tab 1. 把数字数组转为字符串数组 -->

```javascript
const nums = [1, 2, 3];
const strs = nums.map((num) => String(num)); // ['1','2','3']
```

<!-- endtab -->

<!-- tab 2. 提取对象数组的某个属性 -->

```javascript
const users = [
  { name: "张三", age: 18 },
  { name: "李四", age: 20 },
];
const names = users.map((user) => user.name); // ['张三','李四']
```

<!-- endtab -->

{% endtabs %}

**注意事项**

1. map **必须有返回值**，如果回调里没有写 `return`，新数组的对应项会是 `undefined` 。
2. 如果不需要生成新数组，只是想遍历执行操作，直接用 `forEach` 即可，`map` 会额外创建新数组 **浪费内存**。

## 条件遍历

以下所有方法都是 **原型方法**，且都 **不会修改原数组**。

**通用参数规则**

| 参数         | 说明                     |
| ------------ | ------------------------ |
| currentValue | 当前遍历的元素           |
| index        | 当前元素索引（可选）     |
| array        | 调用方法的原数组（可选） |

**条件遍历方法**

{% label filter blue %}
提取 **所有满足条件** 的元素，返回新数组。

```javascript
const goods = [
  { name: "耳机", price: 99 },
  { name: "键盘", price: 299 },
  { name: "鼠标", price: 159 },
];
// 回调函数返回 true 则保留元素，false 则过滤
const highPriceGoods = goods.filter((item) => item.price > 100);
```

{% label find orange %}
返回 **第一个符合条件** 的元素，找到后立即停止遍历，找不到则返回 `undefined`。

```javascript
const users = [{ name: "李四" }, { name: "张三" }, { name: "张三" }];

const targetUser = users.find((item) => item.name === "张三");
// 结果：{name: '张三'}（只返回第一个匹配项）
```

{% label findIndex green %}
和 `find` 逻辑一致，返回 **第一个符合条件** 的元素索引，找不到则返回 `-1`。

```javascript
const nums = [10, 20, 30, 40];

const index = nums.findIndex((num) => num > 25);
// 结果：2（对应元素30的索引）
```

{% label some red %}
判断数组中是否存在 **至少一个符合条件** 的元素，找到就停止遍历并返回 `true`，否则返回 `false`。

```javascript
const scores = [59, 60, 85];

const hasFail = scores.some((score) => score < 60);
// 结果：true（存在59分的不及格项）
```

{% label every pink %}
判断数组中是否 **所有元素都符合条件**，有一个不符合就停止遍历并返回 `false`，否则返回 `true`。

```javascript
const ages = [18, 22, 25];

const isAllAdult = ages.every((age) => age >= 18);
// 结果：true（所有年龄都成年）
```

{% label indexOf purple %}
查找指定元素在数组中 **第一次出现** 的索引，本质是按 `===` 条件匹配，找不到则返回 `-1`。

```javascript
const arr = ["a", "b", "c", "b"];

const index = arr.indexOf("b");
// 结果：1
```

{% label includes blue %}
判断数组中是否 **包含指定元素**，也是按 `===` 条件匹配，返回布尔值。

```javascript
const arr = [1, 2, 3, NaN];

const hasNaN = arr.includes(NaN);
// 结果：true（比indexOf更友好，能正确识别NaN）
```

{% label reduce pink %}
对数组中的每个元素执行一个 **归约函数** `reducer`，将其结果汇总为 **单个任意类型** 的返回值。

**参数列表**

1. callback (必填)：对每个元素执行的 **归约函数**，包含四个参数：

| 参数               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| accumulator (acc)  | 累计器 - 上一次调用 `callback` 后的返回值，或 `initialValue` |
| currentValue (cur) | 当前正在处理的元素                                           |
| index (可选)       | 当前元素的索引                                               |
| array (可选)       | 调用 `reduce` 方法的数组本身                                 |

2. initialValue (可选)：第一次调用 `callback` 时 `accumulator` 的 **初始值**。

**调用示例**
{% tabs reduce, 3 %}

<!-- tab 1. 数字计算 -->

```javascript
const nums = [1, 2, 3, 4];

const product = nums.reduce((acc, cur) => acc * cur, 1);
console.log(product); // 24
```

<!-- endtab -->

<!-- tab 2. 数组去重 -->

```javascript
const arr = ["a", "b", "a", "c", "b"];

const unique = arr.reduce((acc, cur) => {
  if (!acc.includes(cur)) {
    acc.push(cur);
  }
  return acc;
}, []);
console.log(unique); // ['a', 'b', 'c']
```

<!-- endtab -->

<!-- tab 3. 聚合遍历 -->

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

// 传统写法：先过滤偶数，再乘以 2（遍历两次）
const result1 = numbers.filter((n) => n % 2 === 0).map((n) => n * 2);

// reduce 写法：一次遍历完成（性能更优）
const result2 = numbers.reduce((acc, cur) => {
  if (cur % 2 === 0) {
    acc.push(cur * 2);
  }
  return acc;
}, []);
console.log(result2); // [4, 8, 12]
```

<!-- endtab -->

{% endtabs %}

**注意事项**

1. 如果提供了 `initialValue`：`accumulator` 第一次等于 `initialValue` ，`callback` 从数组的 **第一个元素** 开始执行。
2. 如果没有提供 `initialValue`：`accumulator` 第一次等于数组的第一个元素，`callback` 从数组的 **第二个元素** 开始执行。

{% note primary no-icon %}
**reduce 与 map、forEach 的区别**
`map` 强制返回和原数组长度相等的新数组；`forEach` 强制返回 `undefined`；而 `reduce` 的返回值完全自定义，如果返回数组，**长度可以任意控制**。
{% endnote %}

# 对象遍历

## 静态方法

以下方法只处理对象自身的属性，**不会遍历到原型链**：

```javascript
const obj = { name: "Alice", age: 25 };

// 获取键列表
Object.keys(obj).forEach((key) => {
  console.log(key); // 'name', 'age'
});

// 获取值列表
Object.values(obj).forEach((value) => {
  console.log(value); // 'Alice', 25
});

// 获取键值对
Object.entries(obj).forEach(([key, value]) => {
  console.log(key, value); // 'name' 'Alice', 'age' 25
});
```

{% note primary no-icon %}
**静态方法 和 原型方法 的区别**
静态方法的 `this` 指向类本身（即构造函数），只能访问类的静态属性和其他静态方法，**拿不到实例的属性**。

1. 不需要依赖具体实例数据、属于整个类的通用工具能力，就定义为 **静态方法**；
2. 需要依赖具体实例的数据、属于实例自身的行为能力，就定义为 **原型方法**。

如果 `push` 是静态方法，每次调用都要手动把数组实例传进去：`Array.push(arr, 1)`，远不如 `arr.push(1)` 简洁。
{% endnote %}

## for...in

遍历对象 **自身以及原型链上** 的所有可枚举属性。

```javascript
for (const 键名 in 遍历目标) {
  // 每次迭代拿到的是「键名」
}

const obj = { name: "张三", age: 18 };
for (const key in obj) {
  console.log(key); // 输出 'name'、'age'
  // 要取值需要用：obj[key]
}
```

{% note info no-icon %}
**设置属性不可枚举的标准方法：`Object.defineProperty()`**

```javascript
const obj = { name: "Alice" };

// 创建并设置属性为不可枚举
Object.defineProperty(obj, "age", {
  value: 25,
  enumerable: false,
});
```

{% endnote %}

如果只需要遍历对象 **自身的属性**，可以用 `hasOwnProperty()` 过滤。

```javascript
for (let key in p) {
  if (p.hasOwnProperty(key)) {
    console.log(key);
  }
}
```

{% note success no-icon %}
**`for...in` 可以遍历数组：数组本质是一个特殊的对象。**

```javascript
const arr = ["a", "b", "c"];

for (let key in arr) {
  console.log(key); // 输出 '0', '1', '2'
  console.log(arr[key]); // 输出 'a', 'b', 'c'
}
```

{% endnote %}

## for...of

遍历 **可迭代对象** 的元素值。

{% note info %}
`for...of` 的设计初衷是统一各种集合类型的遍历方式，替代之前的 `forEach` 等方法，是目前最通用的遍历语法。
{% endnote %}

```javascript
// 语法结构
for (const 元素值 of 可迭代对象) {
  // 遍历的是元素值，不是键
}

// 遍历数组
const arr = ["a", "b", "c"];
for (const item of arr) {
  console.log(item); // 直接输出 'a'、'b'、'c'（数组元素本身）
}

// 遍历字符串
const str = "abc";
for (const char of str) {
  console.log(char); // 输出 'a'、'b'、'c'
}

// 遍历 Set
const set = new Set([1, 2, 3]);
for (const num of set) {
  console.log(num); // 输出1、2、3
}

// 遍历 Map
const map = new Map([
  ["name", "张三"],
  ["age", 18],
]);
for (const [key, value] of map) {
  // map默认返回[key, value]数组，可直接解构
  console.log(key, value); // 输出 'name' '张三' / 'age' 18
}

// 遍历普通对象
for (let [key, value] of Object.entries(obj)) {
  console.log(key, value);
}
```

{% note primary no-icon %}
**`for...of` 可以通过 `entries()` 方法获取索引。**

```javascript
const arr = ["a", "b", "c"];
// entries() 会返回 [索引, 元素] 的迭代器，直接解构赋值即可
for (const [index, value] of arr.entries()) {
  console.log(`索引${index}：值${value}`);
}
```

`entries()` 是所有 **可迭代对象** 都支持的原型方法，作用是返回一个 **迭代器对象**，迭代器每次产生的值都是 `[键/索引, 值]` 形式的数组。

{% endnote %}

{% note success %}
`entries()` 返回的对象同时满足 **迭代器** 和 **可迭代对象** 的定义，既实现了 `next` 方法，又实现了 `Symbol.iterator` 方法（**返回它自身**），因此它可以被 `for...of` 遍历。
{% endnote %}

{% note info %}
**实例方法** 和 **原型方法** 是两个概念，因为实例本身通常不存方法，所以两者高度重合。
{% endnote %}

**注意事项**

1. `for...of` 遍历的是 **元素值**，不是键。
2. 仅遍历可迭代对象自身的属性，**不会遍历原型属性**。
3. 支持 **中断遍历**，可以用 `break` 中断、`continue` 跳过、`return` 退出外层函数（`for...of` 是是一个 **语句** 而非 **方法**，因此它的 `return` 不是对自身起作用的）。

# 迭代器和生成器

**迭代器** 和 **可迭代对象** 是ES6引入的一套统一的遍历机制，让不同类型的数据结构可以用通用的方式去遍历。

## 可迭代对象

任何实现了 `Symbol.iterator` 方法的对象都是 **可迭代对象**。

{% note primary no-icon %}
**为什么用 `Symbol` 类型作为键名？**
`Symbol.iterator` 本质是 `Symbol` 构造函数挂载的一个 **静态属性**，它的值是一个预定义好的全局唯一的 `Symbol` 类型值，避免和用户定义的 `iterator` 命名冲突。
{% endnote %}

{% note info no-icon %}
**内置对象** 是指 ES 规范预先定义好的、可以直接使用的对象，有三种类型：

1. 内置构造函数：类型是 `function`，可以用 `new` 生成实例，例如 `Array Object String Number`；
2. 内置非构造函数：类型是 `function`，不能 `new`，只能直接调用，例如 `Symbol BigInt`；
3. 普通内置对象：类型是 `object`，不是函数，不能 `new`，例如 `Math JSON`。

{% endnote %}

**原生可迭代对象**：数组、字符串、Map、Set 等。

{% note success no-icon %}
**临时包装对象**
字符串、数字、布尔值都是基本数据类型，本质不是对象，是不能调用方法、访问属性的。JS 为了让基本类型也能方便地使用配套能力，设计了 **临时包装对象机制**：

对基本数据类型调用方法、访问属性时（比如 `str.length`），JS 会自动做3件事：

1. 临时创建一个和字符串值对应的 **引用类型实例**（也就是包装对象）；
2. 在这个 **临时对象** 上执行对应的操作（比如调用方法、访问属性）；
3. 操作完成后立刻销毁这个临时对象，不留下任何痕迹。

{% endnote %}

`Symbol.iterator` 是一个特殊的内置标识，它要求返回一个 **迭代器对象**。

## 迭代器

任何实现了 `next` 方法的对象，都是 **迭代器对象**。

`next` 方法要求返回一个固定格式的对象：`{ value: 本次遍历的值, done: 布尔值 }`，其中：

1. `value`：当前遍历到的元素值；
2. `done`：`false` 表示还有后续元素，`true` 表示遍历已经结束。

{% note info %}
可迭代对象是 **被遍历的目标**，迭代器对象是 **执行遍历的工具**。
{% endnote %}

遍历一个 **可迭代对象** 时（比如用 `for...of`），JS 会自动调用它的 `Symbol.iterator` 方法，得到一个迭代器；然后不断调用迭代器的 `next()` 方法，每次拿到 `{ value, done }` ，直到 `done` 为 `true` 就停止遍历。

**模拟数组的迭代过程**

```javascript
const arr = [10, 20, 30];
// 1. 调用可迭代对象的 Symbol.iterator 方法，得到迭代器
const iterator = arrSymbol.iterator;
// 2. 不断调用 next() 遍历元素
console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 20, done: false }
console.log(iterator.next()); // { value: 30, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

{% note warning %}
迭代器遍历到 **最后一个元素** 时，`done` 仍然是 `false`，只有当所有元素都已经被返回、再无元素可遍历时，下一次 `next()` 才会返回 `{ value: undefined, done: true}`。
{% endnote %}

**让一个普通对象可迭代**

```javascript
const myObj = {
  name: '小明',
  age: 18,
  gender: '男',
  // 实现 Symbol.iterator 方法
  Symbol.iterator {
    // 1. 先拿到对象所有的键
    const keys = Object.keys(this);
    let index = 0; // 迭代的指针
    // 2. 返回迭代器对象
    return {
      // 箭头函数绑定外层 this，保证能拿到 myObj 和 keys
      next: () => {
        if (index < keys.length) {
          const key = keys[index];
          index++;
          // 返回当前值，done 为 false 表示还没结束
          return { value: this[key], done: false };
        } else {
          // 迭代结束
          return { value: undefined, done: true };
        }
      }
    }
  }
}
```

## 生成器

**生成器** 是 ES6 新增的一种可以暂停执行、恢复执行的特殊 **函数**。

两个核心标识：

1. 定义时在 `function` 后面加：`function* gen() {}` 或 `function *gen() {}`；
2. 函数内部用 `yield` 关键字标记暂停点。

生成器是创建迭代器的 **语法糖**，生成器执行后返回的对象本身就是迭代器：

```javascript
// 定义生成器函数
function* myGenerator() {
  console.log("第一步执行");
  yield "暂停点1"; // 第一个暂停点
  console.log("第二步执行");
  yield "暂停点2"; // 第二个暂停点
  console.log("第三步执行");
  return "结束";
}

// 调用生成器函数，得到生成器对象，此时函数内部代码一行都没执行
const gen = myGenerator();
// 第一次调用 next()：从开头执行到第一个 yield
const res1 = gen.next();
console.log(res1); // { value: '暂停点1', done: false }
const res2 = gen.next();
console.log(res2); // { value: '暂停点2', done: false }
const res3 = gen.next();
console.log(res3); // { value: '结束', done: true }
```

`next` 方法还可以接收一个参数，这个参数会作为上一次 `yield` 表达式的返回值：

```javascript
function* calcGenerator() {
  const num1 = yield "请输入第一个数"; // 第一个yield暂停，等待外部传值
  const num2 = yield "请输入第二个数"; // 第二个yield暂停，等待外部传值
  return num1 + num2;
}

const gen = calcGenerator();
console.log(gen.next()); // { value: '请输入第一个数', done: false }
// 第二次next()传10，作为第一个yield的返回值，赋值给num1
console.log(gen.next(10)); // { value: '请输入第二个数', done: false }
// 第三次next()传20，作为第二个yield的返回值，赋值给num2
console.log(gen.next(20)); // { value: 30, done: true }
```

{% note info %}
`async/await` 本质是 **生成器 + Promise + 自动执行器** 的语法糖。
{% endnote %}

---

{% series javascript %}
