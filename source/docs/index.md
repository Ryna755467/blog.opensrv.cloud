---
title: 写作规范
date: 2026-05-12
updated: 2026-08-28
top_img: /img/series/default.jpg
---

## 基本规则

站点内的文档分为 **分类文档** 和 **标签文档** 两种类型。

### 分类文档

用于存放多个 **同系列** 的文档，一个分类中文档的个数必须大于 `1`：

1. `front-matter` 必须填写 `series` 字段，用于在文档末尾 **展示系列文档**。
2. 归属同一个分类的文档使用 **统一的顶部图**。
3. 归属同一个分类的文档使用 **单独的封面图**，封面的 **主体色调** 需要保持一致。

### 标签文档

用于存放 **零散或聚合类** 的文档，这些文档与其他文档没有归属关系：

1. `front-matter` 必须填写 `tags` 字段，用于在文档末尾 **展示标签**。
2. 每个标签文档使用 **单独的顶部图和封面图**。

## Markdown

此章节介绍 **站点内的文档** 会用到的部分 MD 语法，完整的内容请阅览 [MD 参考文档](https://blog.opensrv.cloud/markdown/ "MD 参考文档")。

### 标题

**MD 语法**

```markdown
# Heading level 1

## Heading level 2

### Heading level 3
```

**渲染效果**

参考本文的标题渲染效果。

**使用场景**

用于 **划分文章结构** 和 **渲染目录**。

**注意事项**

1. 长文档用三级标题 `# ## ###`，短文档用二级标题 `## ###`，如果没有分级目录，统一用 `###` 标题。
2. 标题层级不要超过 `3` 层，不要使用 `####`，避免 **目录嵌套过深**。

### 粗体

**MD 语法**

```markdown
I just love **bold text**.
```

**渲染效果**

I just love **bold text**.

**使用场景**

1. 用于强调关键信息或概念，**突出长文本的重点内容**，例如：
   {% note modern no-icon %}
   程序在运行过程中，已经不再需要使用的内存，没有被操作系统或垃圾回收机制正确释放，导致 **可用内存持续减少** 的现象即 **内存泄露**。
   {% endnote %}

2. 用于 **短段落的标题**，如果内容区域较短，不要使用 `###` 标题，用 `**` 代替，合理规划目录层级，例如：
   {% note modern no-icon %}
   **参数列表**

   **调用示例**

   **注意事项**
   {% endnote %}

**注意事项**

1. 用作标题时，如果带有列举含义，可以 **加上序号前缀**，例如：
   {% note modern no-icon %}
   **1. 外层函数返回内层函数，同时保留对上层变量的引用**
   {% endnote %}

2. 用作标题时，可以选中行内代码；用作内容时，不要选中代码，**避免过于突出的视觉效果**，例如：
   {% note modern no-icon %}
   **`resolve/reject` 中的 this 指向问题**
   {% endnote %}

3. 粗体文字两侧需要有一个 **空格缩进**，如果某一侧紧邻标点符号则 **省略该侧缩进**。
4. 选中的文本应 **尽量简短**，避免出现超过一行的粗体文字。

### 引用

**MD 语法**

```markdown
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

**渲染效果**

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

**使用场景**

为了更好的渲染效果，避免在 **站点内的文档** 中使用 `>` 引用，用 `{% note %}` **标签外挂** 代替。

### 图片

**MD 语法**

```markdown
![默认封面](/img/covers/default.jpg)
```

**渲染效果**

![默认封面](/img/covers/default.jpg)

**使用场景**

用于在页面中插入一张 **通栏图片**，例如：

![首屏大图](/img/index.jpg)

**注意事项**

`{% image %}` 标签也可以插入通栏图片，例如：`{% image /img/covers/default.jpg 默认封面 %}`，但是 `![]()` 语法有 **更合适的上下间距**，因此所有通栏图片都要使用 `![]()` 语法。

### 链接

**MD 语法**

```markdown
[Link text](https://www.example.com "Link Title")
```

**渲染效果**

[Link text](https://www.example.com "Link title")

**使用场景**

1. 用于 **指向另一篇文档**，例如：
   {% note modern no-icon %}
   此章节介绍 **站点内的文档** 会用到的部分 MD 语法，完整的内容请阅览 [MD 参考文档](https://blog.opensrv.cloud/markdown/ "MD 参考文档")。
   {% endnote %}

2. 用于 **跳转外部链接**，例如：
   {% note modern no-icon %}
   1. [Hexo - 快速、简洁且高效的博客框架](https://hexo.io/zh-cn/ "Hexo 中文文档")
   2. [Butterfly - A Simple and Card UI Design theme for Hexo](https://butterfly.js.org/ "Butterfly 主题文档")
   3. [Rinokit - A react library developed with dumi](https://rinokit.opensrv.cloud/ "Rinokit Library")
   4. [Markdown教程 - 最简明的Markdown语法入门指南](https://markdown.com.cn/ "Markdown 教程")
      {% endnote %}

**注意事项**

必须填写链接的标题，应 **尽量简短**。

### 代码

**MD 语法**

```markdown
At the command prompt, type `nano`.
```

**渲染效果**

At the command prompt, type `nano`.

**使用场景**

1. 用于变量、方法、类名、取值、关键字等 **行内代码**，例如：
   {% note modern no-icon %}
   `state` / `resolve()` / `Promise` / `undefined` / `async`。
   {% endnote %}

2. 用于 **单行代码**，例如：
   {% note modern no-icon %}
   `Array.from(arrayLike, mapFn?, thisArg?)`
   {% endnote %}

**注意事项**

1. 渲染静态方法时需要写上 **归属的构造函数**，例如 `Promise.all`。
2. 代码两侧需要有一个 **空格缩进**，如果某一侧紧邻标点符号则 **省略该侧缩进**。
3. 如果一个字符在段落内 **频繁出现**，为了方便阅读，不要使用行内代码渲染它。
4. 不要使用行内代码渲染 **中文字符**。

### 代码块

**MD 语法**

````markdown
```javascript
const quickSort = (arr) => {
  if (arr.length <= 1) return arr;
  const p = arr[Math.floor(arr.length / 2)];
  return [
    ...quickSort(arr.filter((x) => x < p)),
    ...arr.filter((x) => x === p),
    ...quickSort(arr.filter((x) => x > p)),
  ];
};
```
````

**渲染效果**

```javascript
const quickSort = (arr) => {
  if (arr.length <= 1) return arr;
  const p = arr[Math.floor(arr.length / 2)];
  return [
    ...quickSort(arr.filter((x) => x < p)),
    ...arr.filter((x) => x === p),
    ...quickSort(arr.filter((x) => x > p)),
  ];
};
```

**使用场景**

用于渲染 **多行代码**。

**注意事项**

1. 在嵌套结构中，**内层的代码块不会被渲染**。
2. 可以用不同数量的点号区分层级，例如：外层用 ` ```` `，内层用 ` ``` `。

### 有序列表

**MD 语法**

```markdown
1. First item
2. Second item
3. Third item
   - Indented item
   - Indented item
4. Fourth item
```

**渲染效果**

1. First item
2. Second item
3. Third item
   - Indented item
   - Indented item
4. Fourth item

**使用场景**

在此站点使用的主题中，有序列表的渲染效果优于无序列表，因此任何列举场景都 **优先使用有序列表**，例如：

{% note modern no-icon %}
分代回收机制中，**堆内存** 主要划分为两个代：

1. 新生代：存放新创建的对象、临时变量，**空间小（通常 1~8MB），GC 频繁，速度极快**。
2. 老生代：存放存活时间长的大对象、全局变量、闭包，**空间大，GC 不频繁，速度较慢**。

{% endnote %}

**注意事项**

1. 不要用作 **段落的标题**。标题为纯行内代码时用 `{% label %}` 标签外挂代替；含有其他字符时用 `**` 代替。
2. 子列表用 **无序列表** 渲染。

### 无序列表

**MD 语法**

```markdown
- First item
- Second item
- Third item
  1. Indented item
  2. Indented item
- Fourth item
```

**渲染效果**

- First item
- Second item
- Third item
  1. Indented item
  2. Indented item
- Fourth item

**使用场景**

仅用于 **有序列表的子列表**，例如：

{% note modern no-icon %}

1. `async` 用来标记函数是 **异步函数**：

- `async function fn() {}` 或者箭头函数 `const fn = async () => {}`；
- 异步函数的返回值会自动包装成 Promise：如果返回普通值，会变成 `Promise.resolve(普通值)`；如果抛出错误，会变成 `Promise.reject(错误)`。

2. `await` 只能在 `async` 函数内部使用：

- `await` 后面一般跟 Promise 对象，作用是暂停当前 `async` 函数的执行，等待状态变更为 `fulfilled` 后，再继续执行后面的代码；
- `await` 的返回值就是 Promise 成功的结果；如果状态变为 `rejected`，会 **抛出错误**，需要外层用 `try/catch` 捕获。

{% endnote %}

**注意事项**

为了更明显的区分，子列表用 **有序列表** 渲染，形成 **有序列表 -> 无序列表 -> 有序列表** 的嵌套结构。

### 分隔线

**MD 语法**

```markdown
---
```

**渲染效果**

---

**使用场景**

1. 用于分隔正文和相关链接，例如：
   {% note modern no-icon %}

   ***

   {% series javascript %}
   {% endnote %}

2. 用于分隔以 `{% label %}` 为主标题、以粗体文字为副标题的 **长段落**，例如：
   {% note modern no-icon %}

   ***

   {% label map pink %}
   对每一项执行回调，把回调的返回值收集为新数组返回，**原数组保持不变**。

   `const newArr = arr.map(callback(currentValue, index, array), thisArg)`

   语法和参数与 `forEach` 方法完全一致。

   **调用示例**
   {% endnote %}

**注意事项**

合理规划文档结构，避免使用分隔线 **划分短段落**。

### 表格

**MD 语法**

```markdown
| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragraph | Text        |
```

**渲染效果**

| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragraph | Text        |

**使用场景**

1.  用于函数的 **参数列表**，参数列不用渲染为行内代码，例如：
    {% note modern no-icon %}
    `Array.from(arrayLike, mapFn?, thisArg?)`

    | 参数      | 说明                                     |
    | --------- | ---------------------------------------- |
    | arrayLike | 要转换的类数组对象或可迭代对象           |
    | mapFn     | 映射函数，相当于对每个元素执行一次 `map` |
    | thisArg   | 执行 `mapFn` 时的 `this` 指向            |

    {% endnote %}

2.  用于带有列举或对比含义的 **短文本列表**。

**注意事项**

默认对齐方式为：标题居中对齐，列表项左对齐，不要使用 `| :---        |    :----:   |          ---: |` 修改对齐方式。

## 标签外挂

此章节介绍 **站点内的文档** 会用到的部分标签外挂，完整的内容请阅览 [Butterfly 文档(四) 标签外挂](https://butterfly.js.org/posts/ceeb73f/ "Butterfly 标签外挂")。

### Label

**MD 语法**

```markdown
{% label bubbleSort blue %}

{% label selectionSort orange %}

{% label insertionSort green %}

{% label mergeSort red %}

{% label quickSort pink %}

{% label heapSort purple %}

{% label countingSort default %}
```

**渲染效果**

{% label bubbleSort blue %}

{% label selectionSort orange %}

{% label insertionSort green %}

{% label mergeSort red %}

{% label quickSort pink %}

{% label heapSort purple %}

{% label countingSort default %}

**使用场景**

**纯行内代码** 用作小标题时，使用更醒目的 `{% label %}` 标签展示，例如：

{% note modern no-icon %}

{% label reduce pink %}
对数组中的每个元素执行一个 **归约函数** `reducer`，将其结果汇总为 **单个任意类型** 的返回值。

**参数列表**

1. callback (必填)：对每个元素执行的 **归约函数**，包含四个参数：
   | 参数 | 说明 |
   | ------------------ | ------------------------------------------------------------ |
   | accumulator (acc) | 累计器 - 上一次调用 `callback` 后的返回值，或 `initialValue` |
   | currentValue (cur) | 当前正在处理的元素 |
   | index (可选) | 当前元素的索引 |
   | array (可选) | 调用 `reduce` 方法的数组本身 |

2. initialValue (可选)：第一次调用 `callback` 时 `accumulator` 的 **初始值**。

{% endnote %}

**注意事项**

1. `color` 可选值为 `blue, orange, green, pink, purple, default`。
2. `{% label %}` 标签两侧需要有一个 **空格缩进**，如果某一侧紧邻标点符号则 **省略该侧缩进**。

### Note

**MD 语法**

```markdown
{% note info %}
Markdown 是一种轻量级 **标记语言**，它允许人们使用易读易写的 **纯文本格式** 编写文档，Markdown 文件的后缀名是 `.md`。
{% endnote %}

{% note warning %}
标签外挂是 Hexo 的标签插件语法，**不是标准的 Markdown 文本**，仅 Butterfly 主题生效，其他平台会原样显示源码，**不会渲染样式**。
{% endnote %}

{% note primary %}
Markdown 应用程序使用一种称为 **Markdown 处理器**（也通常称为 “解析器” 或 “实现”）的东西将获取到的 `Markdown` 格式的文本输出为 `HTML` 格式。
{% endnote %}
```

**渲染效果**

{% note info %}
Markdown 是一种轻量级 **标记语言**，它允许人们使用易读易写的 **纯文本格式** 编写文档，Markdown 文件的后缀名是 `.md`。
{% endnote %}

{% note warning %}
标签外挂是 Hexo 的标签插件语法，**不是标准的 Markdown 文本**，仅 Butterfly 主题生效，其他平台会原样显示源码，**不会渲染样式**。
{% endnote %}

{% note primary %}
Markdown 应用程序使用一种称为 **Markdown 处理器**（也通常称为 “解析器” 或 “实现”）的东西将获取到的 `Markdown` 格式的文本输出为 `HTML` 格式。
{% endnote %}

**使用场景**

1. 用于信息扩充，例如：
   {% note info %}
   `async/await` 本质是 **生成器 + Promise + 自动执行器** 的语法糖。
   {% endnote %}

2. 用于易错提示，例如：
   {% note warning %}
   `resolve` 和 `reject` 不是原型方法。它是构造函数内部的闭包函数，**只允许执行器调用**，不对外暴露。
   {% endnote %}

3. 用于关键延伸内容，例如：
   {% note primary %}
   正因为Scavenge算法把存活对象都复制到了新区域，因此它可以**避免一次完整的内存遍历**，同时完全**避免产生碎片化内存**；但它需要两倍的内存区域，不适合大对象GC。
   {% endnote %}

4. 用于背景机制说明，例如：
   {% note success %}
   `entries()` 返回的对象同时满足 **迭代器** 和 **可迭代对象** 的定义，既实现了 `next` 方法，又实现了 `Symbol.iterator` 方法（**返回它自身**），因此它可以被 `for...of` 遍历。
   {% endnote %}

5. 用于举例说明，例如：
   选择排序是 **不稳定排序**，因为远距离交换可能改变相等元素的相对顺序。
   {% note modern no-icon %}
   数组 `[2ₐ, 2ᵦ, 1]`，第一轮找到最小值 `1`，和第一个 `2ₐ` 交换，数组变成 `[1,2ᵦ,2ₐ]`。
   原本 `2ₐ` 在 `2ᵦ` 前面，交换后反过来，相等元素的相对顺序改变。
   {% endnote %}

**注意事项**

1. 属于正文的内容不要使用 `{% note %}` 标签。

2. `class` 属性可选值为 `info, warning, primary, success, danger, default`，其中 `danger` 过于醒目，不要在 **站点内的文档** 中使用。

3. 当文本超过两行时，使用 `no-icon` 参数隐藏图标，**让内容区域更紧凑**，例如：
   {% note success no-icon %}
   **临时包装对象**
   字符串、数字、布尔值都是基本数据类型，本质不是对象，是不能调用方法、访问属性的。JS 为了让基本类型也能方便地使用配套能力，设计了 **临时包装对象机制**。

   对基本数据类型调用方法、访问属性时（比如 `str.length`），JS 会自动做3件事：
   1. 临时创建一个和字符串值对应的 **引用类型实例**（也就是包装对象）；
   2. 在这个 **临时对象** 上执行对应的操作（比如调用方法、访问属性）；
   3. 操作完成后立刻销毁这个临时对象，不留下任何痕迹。
      {% endnote %}

### Tabs

**MD 语法**

````markdown
{% tabs sorting-algorithms, 1 %}

<!-- tab 冒泡排序 -->

```javascript
function bubbleSort(arr) {
  const n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    for (let j = 0; j < n - 1 - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}
```

<!-- endtab -->

<!-- tab 选择排序 -->

```javascript
function selectionSort(arr) {
  for (let i = 0; i < arr.length - 1; i++) {
    let minIdx = i;
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < arr[minIdx]) minIdx = j;
    }
    [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
  }
  return arr;
}
```

<!-- endtab -->

<!-- tab 插入排序 -->

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}
```

<!-- endtab -->

{% endtabs %}
````

**渲染效果**

{% tabs sorting-algorithms, 1 %}

<!-- tab 冒泡排序 -->

```javascript
function bubbleSort(arr) {
  const n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    for (let j = 0; j < n - 1 - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}
```

<!-- endtab -->

<!-- tab 选择排序 -->

```javascript
function selectionSort(arr) {
  for (let i = 0; i < arr.length - 1; i++) {
    let minIdx = i;
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < arr[minIdx]) minIdx = j;
    }
    [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
  }
  return arr;
}
```

<!-- endtab -->

<!-- tab 插入排序 -->

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}
```

<!-- endtab -->

{% endtabs %}

**使用场景**

仅用于 **带有列举含义** 的代码块，例如：

{% note modern no-icon %}
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
{% endnote %}

**注意事项**

如果每个列举项的代码块都较短，直接 **聚合为一个代码块** 即可，不要使用 `{% tabs %}` 标签，例如：

{% note modern no-icon %}

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

{% endnote %}

### Gallery

**MD 语法**

```markdown
{% gallery %}
![](/img/covers/javascript/iteration.png)
![](/img/covers/javascript/closure.png)
![](/img/covers/javascript/module.png)
![](/img/covers/javascript/garbage-collection.png)
![](/img/covers/javascript/promise.png)
![](/img/covers/javascript/event-loop.png)
{% endgallery %}
```

**渲染效果**

{% gallery %}
![](/img/covers/javascript/iteration.png)
![](/img/covers/javascript/closure.png)
![](/img/covers/javascript/module.png)
![](/img/covers/javascript/garbage-collection.png)
![](/img/covers/javascript/promise.png)
![](/img/covers/javascript/event-loop.png)
{% endgallery %}

**使用场景**

用于在页面中插入一个 **相册图库**，例如：

{% gallery %}
![](/img/captures/media-player/ch2-21.jpg)
![](/img/captures/media-player/ch2-22.jpg)
![](/img/captures/media-player/ch2-23.jpg)
![](/img/captures/media-player/ch2-24.jpg)
{% endgallery %}

**注意事项**

当图片数量超过 `10` 张时，必须设置 `{% gallery true %}` 开启 **加载更多** 按钮，例如：

{% gallery true %}
![](/img/captures/media-player/ch1-01.jpg)
![](/img/captures/media-player/ch1-02.jpg)
![](/img/captures/media-player/ch1-03.jpg)
![](/img/captures/media-player/ch1-04.jpg)
![](/img/captures/media-player/ch1-05.jpg)
![](/img/captures/media-player/ch1-06.jpg)
![](/img/captures/media-player/ch1-07.jpg)
![](/img/captures/media-player/ch1-08.jpg)
![](/img/captures/media-player/ch1-09.jpg)
![](/img/captures/media-player/ch1-10.jpg)
![](/img/captures/media-player/ch1-11.jpg)
![](/img/captures/media-player/ch1-12.jpg)
![](/img/captures/media-player/ch1-13.jpg)
![](/img/captures/media-player/ch1-14.jpg)
{% endgallery %}

### Hide Toggle

**MD 语法**

```markdown
{% hideToggle 为什么使用 Markdown %}

当你可以通过按下界面中的按钮来设置文本格式时，为什么还要使用 Markdown 来书写呢？使用 Markdown 而不是 Word 类编辑器的原因有：

1. Markdown 无处不在。StackOverflow、CSDN、掘金、简书、GitBook、有道云笔记、V2EX、光谷社区等。主流的代码托管平台，如 GitHub、GitLab、BitBucket、Coding、Gitee 等等，都支持 Markdown 语法，很多开源项目的 README、开发文档、帮助文档、Wiki 等都用 Markdown 写作。

2. Markdown 是纯文本可移植的。几乎可以使用任何应用程序打开包含 Markdown 格式的文本文件。如果你不喜欢当前使用的 Markdown 应用程序了，则可以将 Markdown 文件导入另一个 Markdown 应用程序中。这与 Microsoft Word 等文字处理应用程序形成了鲜明的对比，Microsoft Word 将你的内容锁定在专有文件格式中。

3. Markdown 是独立于平台的。你可以在运行任何操作系统的任何设备上创建 Markdown 格式的文本。

4. Markdown 能适应未来的变化。即使你正在使用的应用程序将来会在某个时候不能使用了，你仍然可以使用文本编辑器读取 Markdown 格式的文本。当涉及需要无限期保存的书籍、大学论文和其他里程碑式的文件时，这是一个重要的考虑因素。

{% endhideToggle %}
```

**渲染效果**

{% hideToggle 为什么使用 Markdown %}

当你可以通过按下界面中的按钮来设置文本格式时，为什么还要使用 Markdown 来书写呢？使用 Markdown 而不是 Word 类编辑器的原因有：

1. Markdown 无处不在。StackOverflow、CSDN、掘金、简书、GitBook、有道云笔记、V2EX、光谷社区等。主流的代码托管平台，如 GitHub、GitLab、BitBucket、Coding、Gitee 等等，都支持 Markdown 语法，很多开源项目的 README、开发文档、帮助文档、Wiki 等都用 Markdown 写作。

2. Markdown 是纯文本可移植的。几乎可以使用任何应用程序打开包含 Markdown 格式的文本文件。如果你不喜欢当前使用的 Markdown 应用程序了，则可以将 Markdown 文件导入另一个 Markdown 应用程序中。这与 Microsoft Word 等文字处理应用程序形成了鲜明的对比，Microsoft Word 将你的内容锁定在专有文件格式中。

3. Markdown 是独立于平台的。你可以在运行任何操作系统的任何设备上创建 Markdown 格式的文本。

4. Markdown 能适应未来的变化。即使你正在使用的应用程序将来会在某个时候不能使用了，你仍然可以使用文本编辑器读取 Markdown 格式的文本。当涉及需要无限期保存的书籍、大学论文和其他里程碑式的文件时，这是一个重要的考虑因素。

{% endhideToggle %}

**使用场景**

用于 **和文档相关的边缘内容** 的介绍，例如：

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

**注意事项**

如果内容区域较短，应该使用 `{% note %}` 标签展示。

### Series

**MD 语法**

```markdown
---

{% series javascript %}
```

**渲染效果**

---

{% series javascript %}

**使用场景**

仅在 **分类文档** 的末尾使用。

**注意事项**

确保文档的 `front-matter` 中包含 `series` 字段，例如：

```markdown
---
series: javascript
...
---
```

## 相关链接

1. [Hexo - 快速、简洁且高效的博客框架](https://hexo.io/zh-cn/ "Hexo 中文文档")
2. [Butterfly - A Simple and Card UI Design theme for Hexo](https://butterfly.js.org/ "Butterfly 主题文档")
3. [Rinokit - A react library developed with dumi](https://rinokit.opensrv.cloud/ "Rinokit Library")
4. [Markdown教程 - 最简明的Markdown语法入门指南](https://markdown.com.cn/ "Markdown 教程")
