---
title: 缓存机制
date: 2026-10-22
updated: 2026-10-22
top_img: /img/series/browser.jpg
cover: /img/covers/browser/cache.jpg
series: browser
categories:
  - 浏览器
---

## 快速阅读

浏览器的缓存机制是浏览器为了 **减少网络请求、加速页面访问** 而设计的多层体系，主要包括 HTTP 缓存、`SW Cache` 和 `bfcache` 三类。

当一个请求从页面发出时，如果页面注册了 `Service Worker` 且其 `fetch` 事件拦截了该请求，开发者编写的缓存策略会先决定是从 `SW Cache` 返回还是发送网络请求。

当请求未被 `Service Worker` 拦截或不符合条件时，会进入 HTTP 缓存层，由浏览器根据 `Cache-Control` 和 `ETag` 等响应头自动判断能否复用本地的响应副本。

`bfcache` 不在这个请求链路上，它是浏览器在用户离开页面时把整个页面快照冻结在内存中的机制，用户点击前进或后退按钮返回页面时可以瞬间恢复，不用重新请求资源。

{% note info %}
`SW Cache` 的优先级最高，由前端 JS 代码控制；HTTP 缓存位于其后，由浏览器自动执行；`bfcache` 独立于请求链路，用于管理整个页面实例的往返恢复。
{% endnote %}

## HTTP 缓存

### 基本概念

HTTP 缓存是浏览器缓存机制最核心的部分，**浏览器会在本地存储服务器的响应副本，以便后续请求直接复用**，从而减少延迟和网络流量。

HTTP 缓存的核心逻辑是通过新鲜度（`max-age` / `Expires`）判断能否直接复用缓存，若缓存过期再通过验证机制（`ETag` / `Last-Modified`）确认资源是否变更，未变更则返回 `304` 并更新过期时间，变更则返回新的资源。

HTTP 缓存按缓存校验策略分为 **强制缓存和协商缓存**，缓存位置既可能在内存中，也可能在磁盘上，具体位置由浏览器根据资源特性和当前状态决定，不受业务代码控制。

{% note info %}
HTTP 缓存存储的是 **一次完整的响应报文**，以请求 `URL` 为索引键，存放格式为浏览器封装的 **二进制数据**，不能直接双击查看。
{% endnote %}

### 强制缓存

强制缓存的核心逻辑是在缓存有效期内，浏览器直接使用本地副本，**不向服务器发送请求**。

**控制字段**

强制缓存通过 `Expires` 和 `Cache-Control` 两个响应头控制，同时存在时 `Expires` 会被忽略。

`Expires` 是 `HTTP/1.0` 时期用来控制强制缓存的响应头，采用 **绝对时间**（即 `UTC` 时间，但记为 `GMT` 格式），例如 `Expires: Wed, 28 May 2026 10:00:00 GMT`，超过这个时间强制缓存就会失效。`Expires` **依赖客户端本地时间**，缓存判断可能会出错。

{% note info %}
`UTC` 是 **现代国际基准时间**，世界各地的本地时间都是用这个基准加上对应的时区偏移得到的，例如北京时间 `UTC+8 = UTC 基准 + 8h`。`Expires` 使用 `UTC` 时间，本身 **不会产生跨时区问题**。
{% endnote %}

`Cache-Control` 是 `HTTP/1.1` 时期用来控制强制缓存的响应头，优先级比 `Expires` 更高，采用 **相对时间**，单位为秒，例如 `Cache-Control: max-age=86400`，代表收到响应后 `86400` 秒强制缓存就会失效。`Cache-Control` **不依赖本地时钟**，准确性更高。

{% note modern no-icon %}
**Cache-Control**

`Cache-Control` 最常用的指令有 `max-age` / `no-cache` / `no-store` / `public` / `private`。

`max-age` / `no-cache` / `no-store` 在语义上是互斥的，不应该同时设置。

- `max-age` 是最常用的指令，用于设置一个强制缓存过期的 **相对时间**；
- `no-cache` 代表 **跳过强制缓存** 校验，直接进入协商缓存流程；
- `no-store` 代表 **禁用 HTTP 缓存**，避免响应中的敏感数据泄漏。

`public` / `private` 在语义上是互斥的，但可以和 `max-age` 等并存。

- `public` 允许响应被任何缓存存储，包括代理和 CDN 等共享缓存；
- `private` 只允许存入私有缓存（例如用户浏览器），**禁止共享缓存存储**。

`public` 和 `private` 用于显式设置是否允许共享缓存存储，默认情况下取决于响应本身是否 **默认可缓存**。
{% endnote %}

**状态码**

强制缓存命中时的状态码是 `200 OK (from memory cache)` 或 `200 OK (from disk cache)`，该请求 **不会发送到服务器**，服务器日志也不会有这次访问记录。

**注意事项**

强制缓存可以最大限度地减少服务器压力，但在 `max-age` 到期之前，即使服务器已经更新，客户端也 **不会获取新的响应**。如果静态资源的文件名携带了哈希参数，就可以放心设置较长的 `max-age`，服务器更新后静态资源的 `URL` 会立刻改变，缓存自然失效。

{% note warning %}
`HTML` 文件通常不建议设置过长的强制缓存，因为它是 **资源入口**，一旦被锁死，用户就无法获取到指向新资源的引用路径。
{% endnote %}

### 协商缓存

当强制缓存过期后，浏览器必须 **向服务器发送请求进行验证**，由服务器决定继续使用本地缓存还是返回新资源。

**控制字段**

`Last-Modified` / `If-Modified-Since` 是 `HTTP/1.0` 时期用来控制协商缓存的头部字段。服务器在首次响应中返回资源的 **最后修改时间** `Last-Modified` 响应头，下次请求时，浏览器通过 `If-Modified-Since` 请求头携带该时间，由服务器比对最后修改时间，返回 `304` 或新资源。

{% note info %}
`Last-Modified` 的精度只到秒级，一秒内的多次修改以及内容没变但修改时间变了都会导致缓存失效。
{% endnote %}

`ETag` / `If-None-Match` 是 `HTTP/1.1` 时期用来控制协商缓存的头部字段。服务器在首次响应中返回资源的 **唯一标识** `ETag` 响应头，浏览器再次请求时通过 `If-None-Match` 请求头带回，由服务器比对资源的唯一标识，返回 `304` 或新资源。

{% note info %}
`ETag` 比时间验证更精确，当 `ETag` 和 `Last-Modified` 同时存在时，`ETag` 的校验优先级更高。
{% endnote %}

**状态码**

协商缓存命中时的状态码是 `304 Not Modified`，浏览器会发送完整的 HTTP 请求到服务器，但 **可以省掉响应体的传输**。

**注意事项**

协商缓存适用于 `HTML` 入口文件和需要即时更新的接口。`HTML` 文件不应该设置长期的强制缓存，而应设置为 `no-cache`，让浏览器每次都走协商验证，保证用户能拿到指向新资源的引用路径。

强制缓存相关的 `Cache-Control` / `Expires` 通常需要业务代码 **显式设置**，而协商缓存相关的 `ETag` / `Last-Modified` 以及请求侧的 `If-None-Match` / `If-Modified-Since` 通常由浏览器或服务端底层框架 **自动处理**。

## Service Worker Cache

### 基本概念

`Service Worker Cache` 是一个由 JS 完全控制的独立缓存层（即 `Cache Storage`），让开发者能以编程的方式完全接管网络请求，从而实现 HTTP 缓存无法做到的精细控制和离线体验。`SW Cache` 通过全局的 `caches` 对象访问（即 `Cache Storage API`），实际存放的是 `Request - Response` 对，以请求为键、响应为值。

当一个受 `Service Worker` 控制的页面发起请求时，缓存查找顺序如下：

1. Service Worker 缓存：首先被 `Service Worker` 的 `fetch` 事件拦截，可以在这里编写自定义逻辑；
2. HTTP 缓存：如果 `Service Worker` 放行请求，才会落到浏览器的 HTTP 缓存。

如果前两层都没有找到可用资源，请求才会发往服务器。

{% note modern no-icon %}
**Service Worker**
`SW` 通过主线程的 `navigator.serviceWorker.register()` 方法注册，是一个运行在浏览器后台的独立 `worker` 线程，负责监听 `install` / `activate` / `fetch` / `push` 等事件，执行前端自定义的代码逻辑。

- `install`：浏览器开始安装注册的 `Service Worker` 脚本时触发；
- `activate`：`Service Worker` 完成安装，并且准备好接管页面时触发；
- `fetch`：`Service Worker` 已经运行，浏览器发起网络请求时触发；
- `push`：`Service Worker` 已经运行，浏览器收到推送消息时触发；

`SW` 的默认作用域通常是 `Service Worker` 脚本所在的目录，例如脚本在 `/subdir/sw.js`，默认就控制 `/subdir/` 及其子路径下的所有页面；如果两个页面都在同一个作用域内，例如 `/subdir/a.html` 和 `/subdir/b.html`，它们就会共享同一个 `Service Worker`。

一个文档可能同时落在多个 `Service Worker` 的作用域内，但浏览器不会让它们同时生效，而是选择作用域最具体（路径最深）的那一个来关联该文档，确保每个页面在运行时，**只有一个 `Service Worker` 在控制它**。
{% endnote %}

### 运行机制

**注册与预缓存**

`Service Worker` 脚本需要先在主线程中注册，之后浏览器才会安装它：

```javascript
if ("serviceWorker" in navigator) {
  navigator.serviceWorker
    .register("/sw.js")
    .then((reg) => console.log("SW 注册成功", reg.scope))
    .catch((err) => console.error("SW 注册失败", err));
}
```

{% note warning %}
`Service Worker` 仅能在 **安全上下文** 中运行，例如 `HTTPS` 或 `localhost` 环境。
{% endnote %}

在 `sw.js` 中执行预缓存：

```javascript
const CACHE_NAME = "app-shell-v1";
const PRECACHE_URLS = [
  "/",
  "/index.html",
  "/styles/main.css",
  "/scripts/app.js",
  "/offline.html",
];

self.addEventListener("install", (event) => {
  event.waitUntil(
    caches
      .open(CACHE_NAME)
      .then((cache) => cache.addAll(PRECACHE_URLS))
      .then(() => self.skipWaiting()),
  );
});
```

{% note info %}
`app-shell` 表明这个缓存空间专门存放 **应用外壳资源**，即应用在首次加载时就必须呈现的最小化 UI 骨架，包括 `HTML` / `CSS` / `JavaScript` 等静态资源。
{% endnote %}

`cache.addAll()` 会给数组中的每个 `URL` 都发送 `GET` 请求，并将成功获取的响应存入缓存。

`event.waitUntil()` 能够确保缓存操作完成前，`Service Worker` 不会提前进入下一阶段。如果 `addAll()` 中任何一个资源下载失败，整个安装就会失败，可以防止缓存处于不完整的状态。

**请求拦截与缓存策略**

`Service Worker` 激活后，就能拦截其作用域内的所有网络请求，可以在 `fetch` 事件中根据资源类型，应用不同的策略。

**缓存优先策略** 适合版本化的静态资源（例如携带哈希值的 `CSS` 或 `JS` 文件），直接返回缓存，不发起网络请求。

```javascript
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      return cached || fetch(event.request);
    }),
  );
});
```

{% note warning %}
`event.respondWith()` 会 **终止浏览器默认的网络请求流程**，它必须返回一个 `Response`，可以从 `Cache Storage` 读取缓存响应，或者通过 `fetch(event.request)` 手动发起请求获取响应。
{% endnote %}

{% note info %}
`Service Worker` 内部发起的请求不会被 `Service Worker` 拦截，而是直接进入 HTTP 缓存层。
{% endnote %}

**网络优先策略** 适合对时效性要求高的资源（例如 `API` 数据或 `HTML` 页面），优先尝试网络请求，失败时回退到缓存。

```javascript
self.addEventListener("fetch", (event) => {
  event.respondWith(
    fetch(event.request)
      .then((response) => {
        const clone = response.clone();
        caches
          .open(CACHE_NAME)
          .then((cache) => cache.put(event.request, clone));
        return response;
      })
      .catch(() => caches.match(event.request)),
  );
});
```

**后台更新策略** 适合对时效性要求不高的资源（例如更新频率低的文档），立即返回缓存（可能是旧版本），同时在后台发起网络请求更新缓存。

```javascript
self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      const fetchPromise = fetch(event.request).then((response) => {
        const clone = response.clone();
        caches
          .open(CACHE_NAME)
          .then((cache) => cache.put(event.request, clone));
        return response;
      });
      return cached || fetchPromise;
    }),
  );
});
```

**版本更新与旧缓存清理**

修改 `sw.js` 或需要发布新版本资源时，必须更改缓存名称（例如从 `v1` 改为 `v2`），避免使用旧的缓存空间。新的 `Service Worker` 安装后，旧的缓存不会自动消失，需要在 `activate` 事件中主动清理。

```javascript
const CACHE_NAME = "app-shell-v2";

self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches
      .keys()
      .then((cacheNames) => {
        return Promise.all(
          cacheNames.map((name) => {
            if (name !== CACHE_NAME) {
              return caches.delete(name);
            }
          }),
        );
      })
      .then(() => self.clients.claim()),
  );
});
```

{% note info %}
`self.clients.claim()` 能让新激活的 `Service Worker` 立刻控制当前页面，不需要用户手动刷新。
{% endnote %}

默认情况下，新安装的 `Service Worker` 会进入 **等待中** 状态，直到所有受旧版本控制的页面都关闭后才会激活。

如果希望新版本尽快生效，可以在 `install` 事件中调用 `self.skipWaiting()`，并配合 `activate` 中的 `clients.claim()` 使用，但这样做可能会让正在使用旧页面的用户遇到资源版本不一致的问题。

### 存储位置

`Service Worker` 缓存的数据（主要是 `Cache Storage` 中的数据）位于浏览器的用户数据目录中，以 **磁盘文件** 的形式持久化存储，但在无痕模式下可能变为内存存储。

## bfcache

### 基本概念

`bfcache` 是浏览器的一种 **内存级优化机制**，当用户离开页面时，浏览器不会立即销毁，而是将整个页面冻结在内存中，当用户通过前进或后退按钮返回该页面时，可以近乎瞬间地从内存中恢复，**跳过网络请求和重新渲染的过程**。

{% note info %}
页面从 `bfcache` 中恢复时不会重新请求资源，所有的状态与离开时完全一致。
{% endnote %}

### 运行机制

**缓存内容**

`bfcache` 保存的是页面的快照，包括：

- 完整的 `DOM` 状态：页面当前的 `HTML` 结构、表单中已填写的内容、以及页面元素的实时状态；
- JavaScript 堆内存：页面中所有 JavaScript 对象、变量、闭包以及执行上下文；
- 页面视觉与交互状态：页面的滚动位置、布局信息等，确保恢复后视觉上与离开前完全一致。

**缓存条件**

如果页面触发了某些条件，就无法存入 `bfcache`：

- 页面使用了 `unload` 事件监听；
- `HTML` 页面的响应头设置了 `Cache-Control: no-store`；
- 页面使用了一些独占资源，例如 `IndexedDB` / `WebSocket` 连接等。

{% note warning %}
不能存入 `bfcache` 的页面，本质上都是 **不能安全复原** 的。
{% endnote %}

**生命周期事件**

`pageshow` 在页面每一次显示时都会触发，包括初次加载和从 `bfcache` 中恢复。可以通过 `event.persisted = true` 来确定页面是从 `bfcache` 恢复的。

```javascript
window.addEventListener("pageshow", (e) => {
  if (e.persisted) {
    console.log("页面从 bfcache 恢复");
  }
});
```

`pagehide` 当用户导航离开页面时触发，`event.persisted = true` 表示页面将要被存入 `bfcache` 中。

```javascript
window.addEventListener("pagehide", (e) => {
  if (e.persisted) {
    console.log("页面将要被存入 bfcache 中");
  }
});
```

### 存储位置

`bfcache` 会将网页的完整快照存储在 **内存** 中，以便用户前进或后退时瞬间恢复。

{% note info %}
HTTP 缓存命中的资源可能来自内存，也可能来自磁盘，`SW Cache` 主要在磁盘中缓存，而 `bfcache` 明确在内存中缓存。
{% endnote %}

---

{% series browser %}
