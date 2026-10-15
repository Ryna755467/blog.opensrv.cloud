---
title: HTTP 请求
date: 2026-10-10
updated: 2026-10-10
top_img: /img/series/browser.jpg
cover: /img/covers/browser/http-request.jpg
series: browser
categories:
  - 浏览器
---

## 快速阅读

| 方法      | 安全 | 幂等 | 用途         |
| --------- | ---- | ---- | ------------ |
| `GET`     | 是   | 是   | 获取资源     |
| `HEAD`    | 是   | 是   | 获取元信息   |
| `POST`    | 否   | 否   | 提交数据     |
| `PUT`     | 否   | 是   | 全量替换资源 |
| `PATCH`   | 否   | 否   | 局部更新资源 |
| `DELETE`  | 否   | 是   | 删除目标资源 |
| `OPTIONS` | 是   | 是   | CORS 预检    |
| `TRACE`   | 是   | 是   | 调试诊断     |
| `CONNECT` | 否   | 否   | HTTPS 代理   |

## GET

### 用途

`GET` 是 HTTP 协议中最常用的方法，作用是 **从服务器获取资源**，规范语义上属于 **只读操作**，不能修改服务器的数据。

`GET` 的使用场景有：

1. 获取静态资源：HTML 页面、CSS、JS、图片、视频、音频等，可被浏览器 / CDN 缓存，提升访问速度；
   例如：`GET /index.html HTTP/1.1`
2. 查询接口，读取业务数据：列表查询、详情查询、搜索功能等；
   例如：`GET /api/article?id=1024 HTTP/1.1`
3. 页面跳转和分享链接：`GET` 的查询参数可以完整保留在 URL 中，可用于链接跳转。
   例如：`https://example.com/share?invite=9sK7g2pR5xQ8zD2`

### 特点

**通过查询字符串传递参数**

`GET` 不使用请求体，而是通过查询字符串传递参数，这些字符串会出现在浏览器历史记录和书签等场景，**不适合携带敏感信息**（密码、Token 等）。

URL 中 `?` 后面是查询参数部分，多个参数用 `&` 分隔，中文和特殊符号需要进行 **URL 编码**：

- `A-Z a-z 0-9 - _ . ~` 属于 **未保留字符**，不需要编码；
- `: / ? # [ ] @ ! $ & ' ( ) * + , ; =` 属于 **保留字符**，不会自动编码；
- 其他所有字符（中文、空格、控制字符等），属于 **非法字符**，都需要编码。

在浏览器地址栏填入 URL 会自动编码，例如填入 `https://example.com/api?note=项目&说明=版本1.0`，地址栏页面上依旧显示为中文，但实际的请求已经编码过。

{% note warning %}
URL 解析器对输入的 URL 进行 **整体编码** 时，只会编码 **非法字符**，不会编码 **保留字符**，如果保留字符刚好属于 **内容部分**，会导致服务端解析错误。
{% endnote %}

通过 `fetch` 等方法发送请求，URL 会自动编码：

```javascript
const note = "项目&模块A";
fetch(`https://example.com/api?content=${note}`);
```

URL 会自动进行整体编码，但是 URL 解析器无法识别在整个字符串中 `&` 是作为内容还是分隔符，实际上它属于合法的保留字符不会被编码，会导致服务端解析错误。

最标准的写法是使用 `URL + URLSearchParams`：

```javascript
const url = new URL("https://example.com/api");
url.searchParams.set("content", "项目&模块A");
fetch(url);
```

{% note info %}
`url.searchParams.set` 会覆盖同名 `key`；`url.searchParams.append` 不会覆盖而是追加一条，很多后端框架会把 **多值参数** 整合为一个数组。
{% endnote %}

手动构造原始 HTTP 报文时不会自动编码：

```http
GET /api?content=项目&模块A HTTP/1.1
Host: example.com
```

服务器会识别为两个参数：`content='项目'`，`模块A=''`。

如果发送到服务端的 URL 中包含非法字符，例如空格：

```javascript
GET /search?q=hello world HTTP/1.1
Host: example.com
```

服务器会解析为 `方法: GET; 请求目标: /search?q=hello; HTTP 版本: world`，返回 `400 Bad Request`，请求格式错误。

如果 URL 包含中文字符，服务器不做严格校验时会按原始字节解析，可能出现乱码；其他非法字符严格服务器会返回 `400 Bad Request`，宽松服务器会按字面解析，但可能出现安全漏洞（**CRLF 注入** 等）。

**安全与幂等**

`GET` 规范语义上是 **安全方法**，不改变服务器状态。浏览器、爬虫等可以自动发起 `GET` 请求，不会破坏服务器数据。

{% note info %}
安全方法强调 **不修改数据、无副作用**，是否可以自动触发。
{% endnote %}

`GET` 规范语义上也是 **幂等方法**，多次执行效果相同。网络出错时，客户端可以自动重试 `GET` 请求。

{% note info %}
幂等方法强调 **重复执行，结果不变**，是否可以自动重试。
{% endnote %}

**缓存特性**

`GET` 是 **默认可缓存** 的方法，允许浏览器、CDN、代理服务器存储该请求的响应，后续相同请求直接复用缓存，不用再次请求源服务器。

{% note info %}
`GET`、`HEAD` 默认可缓存；`POST` 通常不可缓存，但在特定条件下可缓存；其他方法都不可缓存。`POST` 语义是 **提交数据**，通常有副作用，缓存一般只用于 `GET` 和 `HEAD` 方法。
{% endnote %}

### 示例

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```

### 响应

```http
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 12 Oct 2026 08:10:00 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 226
Connection: keep-alive

<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>示例网页</title>
</head>
<body>
<div>GET请求获取到的HTML页面</div>
</body>
</html>
```

## HEAD

### 用途

`HEAD` 用于 **获取资源的元信息**，但不获取资源本身，可以节省流量和时间。`HEAD` 方法和 `GET` 几乎完全相同，区别仅在于服务器只返回响应头，不返回响应体。

`HEAD` 的使用场景有：

1. 检查资源是否存在和有效，只看状态码，不下载正文；
2. 获取资源元信息，通过响应头识别资源类型、大小和修改时间；
3. 预检重定向，检查 URL 是否需要跳转，客户端可以据此更新链接；
4. 预检范围下载，下载大文件前，先发送 `HEAD` 请求检查是否支持断点续传。

### 特点

**无请求体**

`HEAD` 方法通常无请求体，参数放在 URL 查询字符串中，与 `GET` 相同。

**无响应体**

规范上服务器对 `GET` 和 `HEAD` 的处理逻辑是相同的，唯一差别是服务器对 `HEAD` 只返回响应头，不返回响应体。

**安全与幂等**

`HEAD` 是 **安全** 方法，不修改服务器状态；`HEAD` 也是 **幂等** 方法，多次执行效果相同。

**缓存特性**

HEAD 响应 **默认可缓存**，但缓存的是头部信息，没有正文。

### 示例

```http
HEAD /file.zip HTTP/1.1
Host: example.org
User-Agent: curl/8.5.0
Accept: */*
Connection: close
```

### 响应

```http
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 12 Oct 2026 09:35:00 GMT
Content-Type: application/zip
Content-Length: 102400
Accept-Ranges: bytes
Last-Modified: Sun, 11 Oct 2026 10:00:00 GMT
ETag: "19000-51f8d23456"
Connection: close
```

## POST

### 用途

`POST` 用于 **向服务器提交数据**，可能会改变服务器的状态。与 `GET` 的 **只读** 形成对比，它是 Web 应用中最常用的 **写操作**。

`POST` 的使用场景有：

1. 提交表单数据：HTML 表单默认用 `POST` 方法提交，数据在请求体中，不暴露在 URL；
2. 上传文件：通过网页表单上传文件，标准做法是 `POST` + `multipart/form-data`；
3. 创建新资源：在服务端新增一条记录，例如发表一篇文章、新建用户账号、创建订单等；
4. 提交复杂数据给接口：前端把结构化、多层嵌套的数据放在 `POST` 请求体中，以 `JSON` 格式发送到后端接口。

### 特点

**可携带大量数据**

`POST` 请求的数据放在请求体中，不暴露在 URL，因此它 **不可书签、不可分享、不被历史记录**。请求体长度受服务器配置限制，但不受 URL 长度限制，可以携带大量数据。

{% note info %}
浏览器的历史记录保存的主要是 **用户主动发起的、可导航的** 请求，本质是 **可重新访问的 URL**，并不是完整的 HTTP 报文，符合条件的一般都是 `GET` 方法。
{% endnote %}

**安全与幂等**

`POST` 不是安全方法，可能会修改服务器的状态；`POST` 也不是幂等方法，多次执行，效果可能不同。

`POST` 不必然改变服务器状态，它只是语义上 **不保证安全**，即客户端、代理、缓存不能假设 `POST` 是安全的，具体是否修改数据，由服务器实现决定，**服务器完全可以用 `POST` 做只读操作**。

{% note warning %}
爬虫通常不会主动发起 `POST` 请求，避免触发副作用。
{% endnote %}

**缓存特性**

`POST` 响应 **默认不缓存**，除非显式设置缓存头。

当查询参数巨大，无法放到 `GET` 的 URL 中（有长度限制），只能放在 `POST` 的请求体中，并且相同参数的查询结果固定不变时才会考虑使用 `POST` 缓存。服务端可以在响应中显式添加 `Cache-Control` 等缓存头，命中缓存需要 URL 与请求体都完全一致。

{% note info %}
浏览器、CDN、代理服务器都支持 `GET` 缓存，命中条件只匹配 URL；而 `POST` 缓存 **兼容性差**，且要求 URL 和请求体都完全一致时才能命中缓存。
{% endnote %}

### 示例

```http
POST /api/comment HTTP/1.1
Host: api.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 41

articleId=1001&content=HTTP协议&author=user01
```

### 响应

```http
HTTP/1.1 201 Created
Server: nginx
Date: Mon, 12 Oct 2026 10:25:00 GMT
Content-Type: application/json
Content-Length: 70

{"code":0,"msg":"评论发表成功","data":{"commentId":5008}}
```

## PUT

### 用途

`PUT` 用于 **替换目标资源**，必须上传 **完整的资源数据**，覆盖服务端该位置已存在的内容，若资源不存在，则创建。

{% note warning %}
`PUT` 用于整体更新资源，而非部分更新，请求体必须包含资源的完整表示，否则未提供的字段会丢失。
{% endnote %}

`PUT` 的使用场景有：

1. 对象存储上传文件：指定路径上传文件，通过 `PUT` 全量更新已有资源；
2. 静态站点部署：部署工具通常使用 `PUT` 上传静态文件；
3. 覆盖配置文件：通过 `PUT` 多次提交同一配置，结果相同；
4. 幂等状态设置：通过 `PUT` 设置固定状态而非增量操作，网络中断后可安全重试。

{% note modern no-icon %}
**对象存储**

对象存储是一种以 **对象** 为单位存储数据的 **分布式存储架构**，适用于海量非结构化数据（图片、视频、文档、压缩包等），核心概念包括：

- Object 对象：数据本身 + 元数据 + 唯一标识 `Key`
- Bucket 存储桶：对象的容器，有全局唯一名称，并且绑定地域，
  访问格式例如：`<bucket-name>.<endpoint>/<key>`

对象存储没有目录层级，桶是顶层容器，下面直接存放对象，但可以在 `Key` 中用 `/` 模拟目录结构，例如 `/user/avatar/pic.png` 整个字符串作为 `Key`，`/` 只是 `Key` 里面的字符，并不是真实目录。

对象存储的本质是 HTTP 服务，你可以直接调用 `RESTful HTTP API`，或者通过封装的客户端库访问，最终都会转化为 HTTP 请求。

| 操作       | HTTP 方法 | 示例                      |
| ---------- | --------- | ------------------------- |
| 上传对象   | PUT       | `PUT /bucket/key`         |
| 下载对象   | GET       | `GET /bucket/key`         |
| 删除对象   | DELETE    | `DELETE /bucket/key`      |
| 查询元数据 | HEAD      | `HEAD /bucket/key`        |
| 列出对象   | GET       | `GET /bucket?list-type=2` |
| 创建桶     | PUT       | `PUT /bucket`             |
| 删除桶     | DELETE    | `DELETE /bucket`          |

REST（Representational State Transfer，表述性状态转移）是一套 **软件架构设计风格**，常用来设计基于 HTTP 的接口，遵循这套风格写出来的接口就叫 `RESTful API`，它的核心思想是把后端所有业务都抽象成 **资源**，用 URL 定位资源，使用 HTTP 方法表达对资源的操作。

常见的对象存储服务有 AWS S3（Simple Storage Service）、百度云 BOS（Baidu Object Storage）等，S3 协议是 AWS S3 定义的一套基于 HTTP / HTTPS 的 `RESTful` **对象存储接口规范**，原本是 AWS S3 的专用 API，现在已经成为对象存储领域的行业标准。
{% endnote %}

### 特点

**请求体是完整表示**

`PUT` 的请求体必须包含资源的完整表示，不能是部分字段。

**客户端指定 URI**

`PUT` 方法需要客户端明确指定资源位置，而 `POST` 通常由服务器决定新资源的 URI。

{% note info %}
**URI**（统一资源标识符）用来唯一标识一个资源，例如：`urn:isbn:9780123456789`；**URL**（统一资源定位符）用来定位一个资源，它是 URI 的子集，必须包含 **访问地址**，例如：`https://opensrv.cloud/`。
{% endnote %}

**安全和幂等**

`PUT` 不是安全方法，会修改服务器的状态；`PUT` 是幂等方法，网络中断后可安全重试，这是 `PUT` 与 `POST` 的核心区别。

**缓存特性**

`PUT` 响应 **默认不缓存**，缓存是为查询获取资源而设计的，`PUT` 语义是修改资源，实际几乎不会用到缓存。

### 示例

```http
PUT /my-bucket/images/pic.png HTTP/1.1
Host: my-bucket.s3.us-east-1.amazonaws.com
Date: Mon, 23 Oct 2023 12:00:00 GMT
Content-Type: image/png
Content-Length: 4096
Authorization: AWS4-HMAC-SHA256 Credential=AKIAIOSFODNN7EXAMPLE/20231023/us-east-1/s3/aws4_request, SignedHeaders=host;x-amz-date, Signature=...

[PNG 二进制数据]
```

### 响应

```http
HTTP/1.1 200 OK
Date: Mon, 23 Oct 2023 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"
Content-Length: 0
Server: AmazonS3
```

## PATCH

### 用途

`PATCH` 用于对资源进行 **部分修改**，只发送需要修改的部分，而不是整个资源。

`PATCH` 的使用场景有：

1. 局部更新资源：通过 `PATCH` 方法修改对象的少数字段，只传需要变更的字段即可，传输体积更小；
2. 结构化补丁：`JSON Patch` 是一种用 JSON 文档描述修改操作的格式，是 `PATCH` 请求最常用的格式之一。

{% note modern no-icon %}
**JSON Patch**

`JSON Patch` 是专门用来描述 JSON 文档局部修改的标准格式，通过 HTTP `PATCH` 方法使用，它不直接传修改后的片段，而是传一组 **操作指令数组**，服务器按顺序执行这些指令修改原始 JSON。

`JSON Patch` 指令整体是一个 **数组**，数组的每一项是一个 **操作对象**，每个操作包含：

- `op`：操作类型（必填）
- `path`：目标路径（必填）
- `value`：操作值（可选）
- `from`：源路径（可选）

`JSON Patch` 一共有 `6` 种操作类型：

1. `add`：添加对象字段或数组元素；
   例如 `{"op":"add","path":"/tags","value":["dev"]}`
2. `remove`：删除对象字段或数组元素；
   例如 `{"op":"remove","path":"/avatar"}`
3. `replace`：替换字段值，字段必须已存在；
   例如 `{"op": "replace", "path": "/age", "value": 30}`
4. `move`：把一个字段移到另一个位置；
   例如 `{"op": "move", "from": "/oldName", "path": "/newName"}`
5. `copy`：复制一个字段到另一个位置；
   例如 `{"op": "copy", "from": "/name", "path": "/displayName"}`
6. `test`：校验字段值，不匹配则整个 `PATCH` 全部不执行。
   例如 `{"op": "test", "path": "/name", "value": "Alice"}`

整个 `JSON Patch` 数组是一个 **原子事务**，只会全部执行或者全部不执行，任意一条失败则所有操作都不生效。

服务器识别到 `JSON Patch` 对应的数据类型 `Content-Type: application/json-patch+json` 时，必须按照 `JSON Patch` 格式解析请求体，大多数服务端都会通过第三方库来自动识别和应用补丁。
{% endnote %}

### 特点

**部分修改**

`PATCH` 的核心语义是 **对目标资源应用部分修改**，与 `PUT` 不同，`PATCH` 不会丢失未提供的字段。

**通过指令修改**

`PATCH` 的请求体是描述如何修改的指令，格式不唯一，由 `Content-Type` 约定，例如 `application/json-patch+json` 或 `application/merge-patch+json`。

**安全与幂等**

`PATCH` 会修改服务器的状态，不是安全方法；`PATCH` 可以通过 `JSON Patch` 的 `add` 操作向数组末尾追加元素，它也不是幂等方法。

**缓存特性**

`PATCH` 响应 **默认不缓存**。

### 示例

```http
PATCH /resources/10 HTTP/1.1
Host: api.example.com
Content-Type: application/json-patch+json
If-Match: "5"
Content-Length: 316

[
  {"op": "test", "path": "/revision", "value": 5},
  {"op": "replace", "path": "/status", "value": "active"},
  {"op": "remove", "path": "/meta/priority"},
  {"op": "add", "path": "/tags/1", "value": "review"},
  {"op": "add", "path": "/tags/-", "value": "published"}
]
```

**原始资源**

```json
{
  "id": 42,
  "status": "pending",
  "meta": {
    "token": "s9k-7gx-p2q",
    "priority": "low"
  },
  "tags": ["draft", "internal"],
  "revision": 5
}
```

### 响应

```http
HTTP/1.1 204 No Content
Date: Tue, 13 Oct 2026 06:20:00 GMT
ETag: "5"
Cache-Control: no-cache
```

**修改后的资源**

```json
{
  "id": 42,
  "status": "active",
  "meta": {
    "token": "s9k-7gx-p2q"
  },
  "tags": ["draft", "review", "internal", "published"],
  "revision": 5
}
```

## DELETE

### 用途

`DELETE` 用于 **删除目标资源**，操作对象是整个资源，不是资源里的某个字段。

`DELETE` 的使用场景有：

1. 删除指定资源，例如 `DELETE /resources/10`；
2. 删除用户账号，例如 `DELETE /users/alice`；
3. 删除文章、评论，例如 `DELETE /articles/01/comments/201`；
4. 取消订单、订阅，例如 `DELETE /orders/6001`。

### 特点

**无请求体**

`DELETE` 通常不携带请求体。

**URI 指向具体资源**

`DELETE` 的 URI 应该指向具体资源，而不是集合。

**安全与幂等**

`DELETE` 会修改服务器的状态，不是安全方法；但它是幂等方法，资源已被删除时返回 `404`。

**缓存特性**

`DELETE` 响应 **默认不缓存**。

### 示例

```http
DELETE /api/orders/1001 HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
Connection: keep-alive
```

### 响应

```http
HTTP/1.1 204 No Content
Date: Mon, 23 Oct 2023 12:00:00 GMT
Server: nginx/1.24.0
```

## OPTIONS

### 用途

`OPTIONS` 用于 **查询目标资源支持哪些 HTTP 方法**，它是查询服务器能力的请求，不修改资源、不读写业务数据。

`OPTIONS` 的使用场景有：

1. 查询某个端点支持的 HTTP 方法，服务器会返回 `Allow` 响应头；
   例如 `Allow: GET, POST, PUT, DELETE, OPTIONS`
2. 查询服务器整体能力，但不代表每一个接口都支持全部方法；
   例如 `OPTIONS * HTTP/1.1`
3. CORS 预检，浏览器在发送跨域复杂请求前，会先发一个 `OPTIONS` 预检。

{% note modern no-icon %}
**CORS 预检**

浏览器在发送 **跨域复杂请求** 前，会先发一个 `OPTIONS` 请求询问服务器是否允许这个跨域请求，只有预检通过，浏览器才会发送实际请求。这个预检请求是浏览器自动发起的，不是真实的业务请求。

满足以下任意一条即为复杂请求：

- 请求方法不是 `GET` / `HEAD` / `POST`
- `Content-Type` 不属于简单类型
- 请求头包含自定义字段

服务器会返回预检的响应，预检通过后浏览器才会发送实际请求。

| 头部字段                           | 说明             |
| ---------------------------------- | ---------------- |
| `Access-Control-Allow-Origin`      | 允许的源         |
| `Access-Control-Allow-Methods`     | 允许的方法       |
| `Access-Control-Allow-Headers`     | 允许的请求头     |
| `Access-Control-Max-Age`           | 预检结果缓存秒数 |
| `Access-Control-Allow-Credentials` | 是否允许凭证     |

服务端本身没有跨域限制，**跨域只是浏览器的安全策略**，如果不预检直接发送这些请求，服务端本身是不会拦截的，预检的目的并不是为了保护服务器，而是为了保护用户，避免跨域请求已经产生副作用但浏览器又因 CORS 拒绝响应，导致用户处于操作已生效但自己不知情的危险状态。
{% endnote %}

{% note warning %}
浏览器的跨域限制和 CORS 预检的目的都是为了 **保护用户，而非保护服务器**。
{% endnote %}

### 特点

**无请求体**

`OPTIONS` 通常不携带请求体。

**无响应体**

`OPTIONS` 响应通常无正文，信息都在头部字段。

**安全与幂等**

`OPTIONS` 是安全且幂等的方法，属于 **只读请求**。

**缓存特性**

`OPTIONS` 响应 **默认不缓存**，但 CORS 预检的结果可以被缓存，通过 `Access-Control-Max-Age` 控制。

### 示例

```http
OPTIONS /api/comments HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: */*
Connection: keep-alive
```

### 响应

```http
HTTP/1.1 204 No Content
Date: Mon, 23 Oct 2023 12:00:00 GMT
Server: nginx/1.24.0
Allow: GET, POST, PUT, DELETE, OPTIONS, HEAD
Content-Length: 0
```

## TRACE

### 用途

`TRACE` 用于 **调试诊断**，服务器会把收到的请求原样回显给客户端，让客户端能看到请求在传输过程中是否被修改。

`TRACE` 通常用来检查中间代理是否修改了请求。

### 特点

**无请求体**

`TRACE` 通常不携带请求体，响应体是收到的原始请求报文。

**现代浏览器禁用**

为了防止 XST 漏洞，大多数生产服务器禁用 `TRACE` 方法，浏览器也会阻止 JS 发送 `TRACE` 请求。

{% note modern no-icon %}
XST（Cross-Site Tracing，**跨站追踪**）是一种利用 HTTP `TRACE` 方法绕过浏览器安全策略、窃取用户敏感信息的攻击方式，它是 XSS + `TRACE` 的组合攻击。XST 能够绕过 `HttpOnly`，从而获取到原本 JS 无法读取的 `Cookie`。

XST 攻击 **必须配合 XSS 漏洞**，完整的流程是：攻击者在 `example.com` 上找到一个 XSS 漏洞，比如评论区能注入 `<script>`，然后把恶意脚本提交到 `example.com`；受害者访问 `example.com` 的评论页，就会自动执行攻击者注入的脚本。

浏览器会自动填充 `Cookie` 请求头，`TRACE` 方法会回显包含 `Cookie` 的完整请求，而攻击者的脚本能够读取回显。这些恶意脚本会 **运行在受害者的浏览器中**，获取到的是受害者的 `Cookie`，而不是攻击者自己的。
{% endnote %}

**安全与幂等**

`TRACE` 是安全且幂等的方法，无副作用。

**缓存特性**

`TRACE` 响应 **默认不缓存**，`TRACE` 用于诊断，响应是动态回显，缓存无意义。

### 示例

```http
TRACE /api/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: */*
X-Custom-Header: hello
Connection: keep-alive
```

### 响应

```http
HTTP/1.1 200 OK
Date: Mon, 23 Oct 2023 12:00:00 GMT
Server: nginx/1.24.0
Content-Type: message/http
Content-Length: 186

TRACE /api/users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: */*
X-Custom-Header: hello
Connection: keep-alive
```

## CONNECT

### 用途

`CONNECT` 用于 **建立 TCP 隧道**，客户端通过 HTTP 代理与目标服务器建立一条 TCP 连接，之后的数据不再走 HTTP 协议：`客户端 ──CONNECT──► 代理服务器 ──TCP 连接──► 目标服务器`。

`CONNECT` 通常用于 **HTTPS 代理**，当用户访问 HTTP 网站时，代理服务器会直接转发 HTTP 报文；而 HTTPS 是端到端加密的，访问 HTTPS 网站时，必须通过 `CONNECT` 方法请求代理服务器和目标服务器建立 TCP 连接，代理只会转发加密字节，**不解析 HTTPS 报文内容**。

HTTP 代理工作在 **应用层**，客户端会把完整的 HTTP 请求发给代理，代理必须 **解析这个请求**，并根据解析结果重新构造请求发给目标服务器；而 HTTPS 的 HTTP 报文被 TLS 加密了，目标地址也在加密内容中，只有目标服务器能够解析，所以不能用应用层代理的方式转发，必须用 `CONNECT` 方法建立 TCP 隧道，直接转发 HTTPS 的加密字节（`CONNECT` 一般是明文的 HTTP 请求，不是 HTTPS，它 **必须能被代理服务器解析**）。

{% note info %}
`CONNECT` 是发送给代理服务器的请求，但 **请求行写的是目标服务器的地址**，客户端通过 TCP 协议与代理服务器建立连接，会主动把配置了代理的流量发送给代理服务器。
{% endnote %}

{% note modern no-icon %}
**代理服务器**

代理服务器（Proxy Server）是位于客户端和目标服务器之间的中间服务器，客户端把请求发给代理，代理转发给目标服务器，再把响应返回给客户端，**客户端不直接连接目标服务器**。

- **正向代理** 扮演客户端角色，客户端主动配置代理，通过代理去访问目标服务器，例如 HTTP 代理等；
- **反向代理** 扮演服务器角色，客户端以为自己在直接访问服务器，实际是请求先到代理，再由代理转发给后端，例如 `Nginx`、`CDN` 等。

代理服务器的使用场景有：

1. 限制、审计用户访问，例如企业内网；
2. 隐藏 IP，目标服务器只能看到代理 IP，看不到客户端真实 IP；
3. 突破限制，例如绕过防火墙访问仅限特定地区的内容；
4. 缓存加速，例如 `CDN`，把内容缓存到离用户最近的节点；
5. 负载均衡，例如反向代理、高并发服务，分发请求到多个后端服务器；
6. 统一入口，例如 API 网关，所有请求经过代理统一认证、限流。

VPN 和代理服务器是不同层面的技术。**代理服务器工作在应用层**，代表某个应用发请求，只处理配置了代理的流量；**VPN 工作在网络层**，会把整个设备接入另一个网络，所有流量都走隧道，对应用是透明的。

在企业内网中，VPN 负责接入内网，而代理服务器负责管理内网访问外网。**VPN 是一个网络范围，而代理是一个中间服务器**。
{% endnote %}

{% note warning %}
`Clash Verge` 等应用本质上属于代理客户端，核心功能是把流量转发给配置的代理服务器（机场节点），由代理服务器代为访问目标，**并不属于 VPN**。
{% endnote %}

### 特点

**无请求体**

`CONNECT` 通常不携带请求体。

**请求目标是 `host:port`**

`CONNECT` 的请求目标是 `host:port` 格式，不是路径。

**安全与幂等**

`CONNECT` 会产生连接状态，不是安全方法；`CONNECT` 也不是幂等方法，多次执行会带来不同的连接状态。

**缓存特性**

`CONNECT` 响应 **默认不缓存**，它建立的是有状态的连接，无法缓存。

### 示例

```http
CONNECT www.example.com:443 HTTP/1.1
Host: www.example.com:443
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Proxy-Connection: keep-alive
Proxy-Authorization: Basic dXNlcjpwYXNzd29yZA
```

### 响应

```http
HTTP/1.1 200 Connection Established
```

---

{% series browser %}
