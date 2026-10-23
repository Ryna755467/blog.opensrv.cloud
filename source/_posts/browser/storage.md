---
title: 存储机制
date: 2026-10-24
updated: 2026-10-24
top_img: /img/series/browser.jpg
cover: /img/covers/browser/storage.jpg
series: browser
categories:
  - 浏览器
---

## 快速阅读

浏览器的存储机制主要包括 `Cookie`、`Web Storage`、`CacheStorage` 和 `IndexedDB`。

`Cookie` 是最早的存储机制，单条 `Cookie` 容量约 `4KB`，会 **随每个同域请求自动发送**，主要用于会话管理和身份识别，可以通过 `HttpOnly`、`Secure`、`SameSite` 等属性提高安全性。会话 `Cookie` 存储在内存中，关闭浏览器后会自动清除；持久 `Cookie` 存储在磁盘中，到期前一直有效。

`Web Storage`（即 `localStorage` 和 `sessionStorage`）是简单的 **同步键值对存储**，键值对的总和不超过 `5MB`，数据不会随请求发送。`localStorage` 在磁盘中持久存储、跨标签页共享；`sessionStorage` 主要存储在内存中，绑定单个标签页、关闭即清除。

`CacheStorage` 是 **代码驱动的缓存接口**，每个 `Cache` 对象下存放的是 `Request/Response` 对，数据在磁盘中持久化存储，不会自动过期，通常在 `Service Worker` 中使用，以实现离线缓存和请求拦截。

`IndexedDB` 是 **浏览器内置** 的面向对象的 `NoSQL` 数据库，数据在磁盘中持久化存储，不会自动过期，适合离线应用、大量结构化数据和需要检索的场景。

## Cookie

### 基本概念

`Cookie` 是服务器通过 `Set-Cookie` 响应头发送给浏览器并 **保存在客户端本地** 的一小段文本数据（单条 `Cookie` 通常不超过 `4KB`），会随着每一次同域的 HTTP 请求自动发送回服务器。

`Cookie` 的设计初衷是为了 **让服务器记住用户**，使用场景有：

1. 会话管理：用户登录后，服务器在 `Set-Cookie` 中下发一个会话 ID，后续请求通过会话 ID 识别对应的用户；
2. 个性化设置：记录用户的偏好设置，例如语言选择、界面主题等；
3. 追踪与分析：用于记录和分析用户行为，例如广告商利用第三方 `Cookie` 跨站追踪用户的浏览习惯。

{% note info %}
登录请求（通常是 `POST` + `HTTPS` 请求）会 **在请求体中携带账号密码**，服务器验证通过后利用 `Set-Cookie` 下发生成的 `Session ID`，后续请求都用这个会话 ID 识别登录状态。
{% endnote %}

{% note warning %}
`Session ID` 本身也是敏感凭证，通常会设置一个较短的过期时间。
{% endnote %}

### 生命周期

HTTP 协议本身是无状态的，服务器会把每次请求都当作全新请求处理，不知道两次请求是否来自同一用户，而 `Cookie` 机制让服务器能够 **维持连续的会话状态**。

浏览器向服务器发起请求后，服务器可以在响应头中添加一个或多个 `Set-Cookie` 字段，每个字段定义一个键值对，
例如 `Set-Cookie: session_id=a3f9c2e7b1d8406f; Path=/; HttpOnly; Secure; SameSite=Lax`，
浏览器收到响应后，会解析并存储这些 `Cookie`。

JavaScript 也可以通过 `document.cookie` 读写存储（带有 `HttpOnly` 属性除外），格式为 `document.cookie = "名称=值; 属性1=值1; 属性2=值2";`，每次赋值只能操作一条 `Cookie`，不会覆盖其他 `Cookie`：

```javascript
document.cookie =
  "session=Xk3pQ9vLm2Rt7YwZ8aB4nC; path=/; domain=example.com; max-age=3600; secure; samesite=lax";
```

{% note warning %}
`Cookie` 的属性名是 **大小写不敏感** 的，`path=/` / `Path=/` / `PATH=/` 是同样的效果，`httponly` / `HttpOnly` / `HTTPONLY` 也一样。
{% endnote %}

修改或删除一条 `Cookie` 时，`Path` 和 `Domain` 必须匹配，其他属性不属于匹配条件：

```javascript
document.cookie = "theme=; path=/account; max-age=0";
```

删除 `Cookie` 的核心代码是 `max-age=0`，浏览器检测到过期，就会把这条 `Cookie` 从存储中移除。`theme=;` 的作用是定位 `Cookie` 而非赋空值，因此不能省略。

`document.cookie` 返回的字符串里只有名称和值，看不到 `Path` / `Domain` / `Expires` 等元数据：

```javascript
console.log(document.cookie);
// "theme=dark; lang=zh-CN"
```

浏览器向同一服务器发起新请求时，会自动在请求头中加入 `Cookie` 字段，将之前存储的、符合发送条件的 `Cookie` 键值对回传，服务器读取该头部即可识别用户身份。

同源请求会默认携带 `Cookie` 字段；跨域请求需要前端设置 `withCredentials = true`，并且服务器响应头设置 `Access-Control-Allow-Credentials` 为 `true` 时才可跨域携带 `Cookie`。

`withCredentials = true` 是前端请求的配置，它会在跨域请求中把符合规则的 `Cookie` 自动放进请求头并发送给服务器，服务器必须同时返回响应头 `Access-Control-Allow-Credentials: true`，否则浏览器会 **丢弃本次响应**，前端拿不到返回的数据。

{% note warning %}
`Access-Control-Allow-Credentials: true` 生效的前提条件是 `Access-Control-Allow-Origin` 不能写 `*`，必须写明确的源。
{% endnote %}

### 核心属性

**存活时间**

`Cookie` 的存活时间由 `Expires` 或 `Max-Age` 属性决定，分为两类：

1. 会话 `Cookie`：未设置过期时间，就会保存在浏览器 **内存** 中，关闭浏览器后即被清除；
2. 持久 `Cookie`：通过 `Expires` 或 `Max-Age` 设置了过期时间，就会保存到 **磁盘** 中，到期前一直有效。

`Cookie` 可以作为会话 `Cookie` 在内存中临时存储，也可以作为持久 `Cookie` 在磁盘中长期保存（**必须设置一个过期时间**）。删除一个 `Cookie`，可以将 `Expires` 设为过去的日期，或者将 `Max-Age` 设为 `0`。

{% note info %}
`Cookie` 的 **过期时间** 有两个属性：`Expires` 是 `UTC` 格式的绝对时间，例如 `Expires=Wed, 23 Oct 2026 22:00:00 GMT`；`Max-Age` 是以秒为单位的相对时间，优先级更高，例如 `Max-Age=3600`。
{% endnote %}

**作用域**

`Cookie` 的作用域由 `Domain` 和 `Path` 属性决定：

1. `Domain` 用于指定 `Cookie` 可以发送的主机。默认仅当前域名会携带，不会发送给子域名；设置为 `Domain=example.com` 后，该域名及其所有子域名（`api.example.com` 等）都会携带；

2. `Path` 用于指定 `Cookie` 可以发送的路径前缀。设置为 `Path=/account` 后，只有请求 `/account` 及其子路径（`/account/profile`）时才会携带 `Cookie`，一般设为 `/` 表示整站可用。

`Domain` 和 `Path` 控制的是单条 `Cookie` 是否符合携带条件，而 `withCredentials` 和 `Access-Control-Allow-Credentials` 控制的是 **整个跨域请求是否允许携带或接收凭证**，包括所有符合条件的 `Cookie`。

{% note info %}
`api.example.com` 中，`.com` 是顶级域名（即一级域名），`example.com` 是注册在顶级域名下的二级域名，`api.example.com` 是 `example.com` 的子域名（三级域名），向子域名发送请求仍然会跨域。
{% endnote %}

**安全属性**

`Cookie` 的安全属性包括 `Secure`、`HttpOnly` 和 `SameSite`：

1. `Secure` 表示 `Cookie` 只能通过 `HTTPS` 请求发送，登录状态相关的 `Cookie` 必须添加该属性；
2. `HttpOnly` 禁止 JS 读取该 `Cookie`，这是防御 `XSS` 攻击窃取会话凭证的关键属性；
3. `SameSite` 控制跨站请求时是否发送 `Cookie`，用于防御 `CSRF` 攻击，有三个取值：

- `Strict`：一律不携带，从外部链接点进来的首次请求会是未登录状态；
- `Lax`：现代浏览器默认，仅顶层导航携带，图片等子资源请求不携带；
- `None`：允许跨站发送，但必须同时设置 `Secure`。

{% note warning %}
`HttpOnly` 只能阻止 JS 读取和覆盖（已存在同名 `HttpOnly Cookie` 时，JS 的写入会被忽略），不会阻止 JS 发起的请求携带该 `Cookie`。
{% endnote %}

### 第三方 Cookie

判断一个 `Cookie` 属于第一方还是第三方，主要看来源域名：

- 第一方 `Cookie`：由正在访问的网站直接设置，通常用于维持登录状态或个性化设置。
- 第三方 `Cookie`：访问的网页中嵌入了来自其他域名的内容（例如广告、`iframe` 框架或分析脚本），这些嵌入内容所属的域名设置的 `Cookie` 就是第三方 `Cookie`。

第三方 `Cookie` 主要用于 **在多个不同域名的网站间共享登录状态** 或者让广告技术平台统计广告曝光并投放相关广告。

其核心争议在于跟踪，同一个第三方域名可能嵌入在成千上万个不同的网站中，可以利用同一个第三方 `Cookie` 跨站识别并逐步构建出个人画像。正因为这种 **跨站跟踪能力**，很多浏览器默认会阻止第三方 `Cookie`。

{% note info %}
可以通过第三方 `Cookie` 让 **不同的网站接入统一的登录系统**，但这不是现代的标准做法，存在兼容性和稳定性问题。
{% endnote %}

## Web Storage

### 基本概念

`Web Storage` 是浏览器提供的一种 **在客户端存储键值对** 的机制，比 `Cookie` 容量更大（在同一个源下 `localStorage` 的所有键值对总和通常不超过 `5MB`，`sessionStorage` 的上限也是 `5MB`），数据不会随每个 HTTP 请求自动发送。

`Web Storage` 包含两种机制：`localStorage` 和 `sessionStorage`，通过 `window.localStorage` 和 `window.sessionStorage` 访问，返回一个 `Storage` 对象实例用于读写数据，它们的核心区别是 **生命周期和作用域不同**。

{% note info %}
`localStorage` 一定会在磁盘中持久化存储，但 `sessionStorage` 主要在内存中短期存储，也可能会写入磁盘，取决于浏览器具体实现。
{% endnote %}

{% note warning %}
`Web Storage API` 都是 **同步方法**，会阻塞浏览器主线程。
{% endnote %}

### localStorage

**生命周期**

`localStorage` 的设计目标就是长期存储，数据 **没有过期时间**，关闭并重新打开浏览器后依然存在，直到被明确清除。

{% note info %}
在无痕模式下，`localStorage` 的数据会存储在内存中，在最后一个标签页关闭时被清除。
{% endnote %}

**作用域**

`localStorage` 的作用域是 **源**（包括协议、主机和端口，任何一项不同都会拥有完全独立的存储空间），**同源的所有标签页和窗口共享同一份数据**。

**基本用法**

`setItem(key, value)` 用于创建或更新数据项，键和值都会被自动转换为字符串：

```javascript
localStorage.setItem("username", "Alice");
localStorage.setItem("age", 25); // 实际存储为字符串 "25"
```

`getItem(key)` 会返回对应的字符串值，键不存在则返回 `null`：

```javascript
const name = localStorage.getItem("username"); // "Alice"
const missing = localStorage.getItem("nonexistent"); // null
```

`removeItem(key)` 用于删除指定的键值对：

```javascript
localStorage.removeItem("age");
```

`clear()` 会清除 `localStorage` 中属于当前源的所有数据：

```javascript
localStorage.clear();
```

{% note info %}
`localStorage` 只能存储字符串数据，但可以通过 `JSON.stringify()` 和 `JSON.parse()` 间接存储对象或数组。
{% endnote %}

**使用场景**

1. 用户偏好设置：记住用户选择的背景色、字体和装饰图片，关闭页面再打开时偏好依然生效；
2. 简单的表单草稿：填写长表单时可以在 `localStorage` 中定期保存草稿，用于意外刷新或关闭后快速恢复；
3. 轻量级的离线数据缓存：对于不需要复杂查询、数据结构简单的少量数据，可以用 `localStorage` 做一层缓存，减少重复请求。

**跨标签页状态同步**

利用 `storage` 事件，可以实现一个标签页改动、其他标签页 **实时响应** 的效果，例如登录状态同步、主题切换全局生效、购物车数量角标实时更新等。

同源的其他页面修改 `localStorage` 时，当前页面会收到 `storage` 事件（`getItem` 不会触发）：

```javascript
// 标签页 A：用户登录/切换账号
localStorage.setItem("auth-refresh", Date.now());

// 标签页 B：同步刷新登录态
window.addEventListener("storage", (e) => {
  if (e.key === "auth-refresh") location.reload();
});
```

### sessionStorage

**生命周期**

`sessionStorage` **绑定在单个浏览器标签页上**，数据会在标签页或浏览器关闭时被清除，但刷新页面或通过链接跳转后，只要还在同一个标签页内，会话存储仍然会保留。

**作用域**

`sessionStorage` 按 **源和浏览器标签页** 双重分区。在同一个标签页内，主页面和它内部嵌入的同源 `iframe` 共享同一份 `sessionStorage` 数据。

{% note info %}
修改 `sessionStorage` 会在 **同一标签页内的同源上下文** 中（主要是 `iframe`）触发 `storage` 事件。
{% endnote %}

**基本用法**

`setItem(key, value)` 用于创建或更新数据，键和值都会被自动转换为字符串：

```javascript
sessionStorage.setItem("formDraft", "用户输入的草稿内容");
```

`getItem(key)` 会返回对应的字符串值，键不存在则返回 `null`：

```javascript
const draft = sessionStorage.getItem("formDraft");
```

`removeItem(key)` 用于删除指定的键值对：

```javascript
sessionStorage.removeItem("formDraft");
```

`clear()` 会清除 `sessionStorage` 中属于当前标签页和当前源的所有数据：

```javascript
sessionStorage.clear();
```

**使用场景**

1. **自动保存文本框内容**，如果页面意外刷新，输入的内容能被恢复，关闭页面后不留下任何痕迹；
2. 在分步填写、最后统一提交的场景中 **存储多步骤表单的中间数据**，关掉标签页后数据会自动清理。

### Custom Hook

```typescript
import { useState, useEffect } from "react";

const useStorage = <T>(key: string, initialValue: T, storage: Storage) => {
  const [value, setValue] = useState<T>(() => {
    try {
      const item = storage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading from localStorage: ${error}`);
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      storage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error writing to localStorage: ${error}`);
    }
  }, [value, key, storage]);

  useEffect(() => {
    if (storage !== window.localStorage) return;

    const listener = (ev: StorageEvent) => {
      if (ev.key === key) {
        try {
          const newValue = ev.newValue ? JSON.parse(ev.newValue) : null;
          if (newValue !== null) {
            setValue(newValue);
          }
        } catch (error) {
          console.error(`Error parsing localStorage change: ${error}`);
        }
      }
    };

    window.addEventListener("storage", listener);
    return () => {
      window.removeEventListener("storage", listener);
    };
  }, [key]);

  return [value, setValue] as const;
};

export default useStorage;
```

## CacheStorage

### 基本概念

`CacheStorage` 是浏览器提供的 **一套代码驱动的缓存管理接口**，它管理着多个命名的 `Cache` 对象，每个 `Cache` 内部存放的是 `Request/Response` 对，在磁盘上持久化存储。

`CacheStorage` 通常在 `Service Worker` 中使用，即 `Service Worker Cache`。`CacheStorage API` 和 `Service Worker` 都只能在 **安全上下文** 中运行，例如 `HTTPS` 或 `localhost` 环境。

### 运行机制

**生命周期**

`CacheStorage` 中存储的 `Request/Response` 对长期有效，**没有过期时间**，直到被明确清除。

{% note info %}
在无痕模式下，`CacheStorage` 的数据会存储在内存中，在最后一个标签页关闭时被清除。
{% endnote %}

**作用域**

`CacheStorage` 的作用域是 **源**，同一个源下的所有页面、`Service Worker` 和 `Web Worker` 共享同一份数据。

**基本用法**

`caches` 是一个全局对象，它是 `CacheStorage` 的操作入口。

通过 `caches.open(cacheName)` 打开或创建一个缓存，它会返回一个 `Promise`，解析为对应的 `Cache` 对象：

```javascript
// 打开名为 "v1" 的缓存，不存在则创建
const cache = await caches.open("v1");
```

获取到 `Cache` 对象后，可以通过 `add()` 或 `addAll()` 方法自动发起网络请求（`GET` 方法）并把响应存起来：

```javascript
// 添加单个资源
await cache.add("/styles.css");
// 批量添加多个资源
await cache.addAll(["/", "/app.js", "/logo.svg"]);
```

可以通过 `cache.match(request)` 读取资源缓存，它返回一个 `Promise`，解析为对应的 `Response`，找不到则返回 `undefined`：

```javascript
const response = await cache.match("/app.js");
if (response) {
  // 使用缓存中的响应
}
```

可以通过 `cache.put(request, response)` 存入一个从网络获取的响应：

```javascript
const networkResponse = await fetch(request);
await cache.put(request, networkResponse.clone());
return networkResponse;
```

{% note info %}
`Response` 的 `body` **只能被消费一次**，如果要同时返回给页面和存入缓存，需要先 `clone()`。
{% endnote %}

可以通过 `cache.delete(request)` 删除缓存中的单个条目：

```javascript
const deleted = await cache.delete("/styles.css");
console.log("delete 返回:", deleted); // true
```

除了操作具体的 `Cache` 对象，`CacheStorage` 还提供了管理所有命名缓存的方法：

- 检查缓存是否存在：`caches.has(cacheName)`；
- 删除整个缓存：`caches.delete(cacheName)`；
- 列出所有缓存：`caches.keys()`。

## IndexedDB

### 基本概念

`IndexedDB` 是浏览器内置的一个面向对象的 `NoSQL` 客户端数据库，专为 **存储大量结构化数据** 而设计。

{% note info %}
`NoSQL` 即 **非关系型数据库**，核心特点是不强制使用固定的表结构，也不使用 `SQL` 作为查询语言。
{% endnote %}

### 运行机制

**生命周期**

`IndexedDB` 的数据 **在磁盘中持久化存储**，没有过期时间。

**作用域**

`IndexedDB` 的作用域是 **源**，每个源拥有自己独立的数据库集合。

**基本用法**

浏览器原生的 `IndexedDB API` 基于事件回调，写法较为复杂，实际开发中通常使用轻量的封装库来获得 `Promise` 风格的简洁写法。

### 主要特点

1. `IndexedDB` 的存储容量远大于 `Web Storage`；
2. `IndexedDB` 的所有操作都是异步的，不会像 `Web Storage` 那样阻塞主线程；
3. `IndexedDB` 的所有操作都在事务中执行，如果一系列操作中任何一步失败，整个事务会回滚；
4. `IndexedDB` 可以直接存储对象、文件、`Blob`、`ArrayBuffer` 等二进制数据；
5. `IndexedDB` 可以创建索引，进行高效的范围查询和遍历。

---

{% series browser %}