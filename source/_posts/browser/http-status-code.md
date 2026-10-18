---
title: HTTP 状态码
date: 2026-10-15
updated: 2026-10-15
top_img: /img/series/browser.jpg
cover: /img/covers/browser/http-status-code.jpg
series: browser
categories:
  - 浏览器
---

## 快速阅读

HTTP 状态码是服务器返回给客户端的 **三位数字标识**，代表本次请求的处理结果。

状态码按首位数字划分为 **五个类别**。

{% note info %}
HTTP 状态码的含义是有标准规定的，对应的原因短语通常也是固定的，但原因短语 **不作为逻辑依赖**，理论上只要符合字符规范就可以随便写。
{% endnote %}

**信息类**

`1xx` 状态码表示 **服务器已收到请求，正在处理**。它们是 **临时响应**，不是最终结果。

| 状态码 | 原因短语            | 说明                                             |
| ------ | ------------------- | ------------------------------------------------ |
| `100`  | Continue            | 服务器已收到完整请求头，愿意继续接收请求体       |
| `101`  | Switching Protocols | 服务器同意将当前连接从 HTTP 协议切换到另一种协议 |

**成功类**

`2xx` 状态码表示 **服务器已成功处理了客户端的请求**，这是最常见的状态码类别。

| 状态码 | 原因短语        | 说明                                                    |
| ------ | --------------- | ------------------------------------------------------- |
| `200`  | OK              | 服务器成功接收并处理了请求                              |
| `201`  | Created         | 服务器创建了一个新资源，`Location` 字段指向新资源的地址 |
| `204`  | No Content      | 请求已被成功处理，但没有响应体                          |
| `206`  | Partial Content | 客户端请求了资源的一部分，服务器成功返回了这部分内容    |

**重定向类**

`3xx` 状态码表示 **重定向**，告诉客户端 **请求的资源不在当前位置**，需要进一步操作。

| 状态码 | 原因短语           | 说明                           |
| ------ | ------------------ | ------------------------------ |
| `301`  | Moved Permanently  | 永久重定向，可能改变方法       |
| `302`  | Found              | 临时重定向，可能改变方法       |
| `304`  | Not Modified       | 资源未修改，直接使用缓存       |
| `307`  | Temporary Redirect | 临时重定向，保持原方法和请求体 |
| `308`  | Permanent Redirect | 永久重定向，保持原方法和请求体 |

**客户端错误**

`4xx` 状态码表示 **客户端错误**，即 **请求本身有问题**，服务器无法处理。

| 状态码 | 原因短语          | 说明               |
| ------ | ----------------- | ------------------ |
| `400`  | Bad Request       | 请求格式错误       |
| `401`  | Unauthorized      | 需要身份认证       |
| `403`  | Forbidden         | 认证成功，但无权限 |
| `404`  | Not Found         | 资源不存在         |
| `429`  | Too Many Requests | 客户端被限流       |

**服务器错误**

`5xx` 状态码表示 **服务器错误**，即 **服务器在处理请求时出了问题**。

| 状态码 | 原因短语              | 说明           |
| ------ | --------------------- | -------------- |
| `500`  | Internal Server Error | 服务器内部错误 |
| `502`  | Bad Gateway           | 上游服务器错误 |
| `503`  | Service Unavailable   | 服务暂时不可用 |
| `504`  | Gateway Timeout       | 上游服务器超时 |

## 信息类

### 100 Continue

客户端准备发送一个较大的请求体（比如上传文件），但 **不确定服务器是否愿意接收**（未登录 / 权限不足 / 文件太大）。

客户端可以先发送请求头（**必须包含空行**，否则服务器会一直等待头部字段结束）：

```http
POST /upload HTTP/1.1
Host: example.com
Content-Length: 1048576
Expect: 100-continue
```

服务器如果愿意接收请求体，就返回：

```http
HTTP/1.1 100 Continue
```

客户端收到后，再继续发送请求体；服务器处理完后，再返回最终响应。这样可以 **避免浪费带宽**。

{% note warning %}
即使客户端带了 `Expect: 100-continue`，服务器也不一定会回复 `100 Continue`，客户端等待一小段时间后如果没有收到响应，应该继续发送请求体。
{% endnote %}

`Expect: 100-continue` 是底层 HTTP 客户端（cURL、Node.js http）才有的能力，它们可以 **直接控制报文是否继续发送**；浏览器需要保持对底层 HTTP 传输的控制权，防止网页脚本干扰协议层面的行为（连接管理、传输编码等），因此 **JS 不能直接控制 TCP 字节流**，它会把 `header` 和 `body` 同时交给浏览器，由浏览器决定是否启用 `Expect: 100-continue`。

### 101 Switching Protocols

客户端在 HTTP 请求头写入 `Connection: Upgrade` 表示 **本次连接需要切换协议**，同时在 `Upgrade` 字段中列出希望切换到的协议（必须包含空行，代表请求头结束）。

```http
GET /chat HTTP/1.1
Host: example.com
Connection: Upgrade
Upgrade: websocket
```

如果服务器同意，会返回 `101 Switching Protocols`，并在响应头中携带一个 `Upgrade` 字段，指明已切换到的协议。

```http
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
```

{% note info %}
`Upgrade` 支持用逗号分隔多个协议，按从高到低优先级排列。
{% endnote %}

客户端收到 `101` 响应后，双方开始使用新协议进行通信。这样可以 **复用同一条连接**，省去重新建立 TCP 的开销。

{% note warning %}
`101 Switching Protocols` 切换的不是 TCP 连接，而是在这条 TCP 连接上 **解释字节流的协议**。
{% endnote %}

在浏览器环境中，`101` 状态码由浏览器底层接管，专用于 WebSocket 协议升级，**JS 无法直接控制**。执行 `new WebSocket` 时，浏览器底层会自动构造一个带有 `Upgrade: websocket` 和 `Connection: Upgrade` 字段的 HTTP 请求，服务器如果同意，就返回 `101`。此时同一条 TCP 连接的控制权会从 HTTP 模块移交给 WebSocket 模块，JS 也会收到一个 `open` 事件。

## 成功类

### 200 OK

`200` 是最常见的 HTTP 状态码，表示 **服务器成功接收并处理了请求**。

`200` 并不代表业务层面的成功，例如 `200` 状态码的响应体返回了 `{"code": 500, "message": "登录失败"}`。HTTP 状态码和业务 `code` 解决的是两个不同层面的问题，`状态码 + 原因短语` 表示 **HTTP 请求层面的处理结果**，而 `code + message` 表示 **业务层面的处理结果**，它们不能互相替代。

处理响应时应该通过状态码判断 HTTP 层结果，通过约定的业务 `code` 判断业务层结果。原因短语和 `message` 只是提示信息，HTTP 解析器只会依赖状态码进行程序判断，**JS 代码也必须依赖 `code` 处理业务层结果**。

{% note info %}
HTTP 的状态码是有协议标准的，**它会被浏览器底层解析**，而 `code` 是业务自定义的，可以随意使用。
{% endnote %}

标准的业务响应体一般是这样的：

```json
{
  "code": 0,
  "message": "success",
  "data": {...},
  "traceId": "7f3a9c2e1b8d4f6a"
}
```

- `code` 是逻辑判断的唯一依据，必须是稳定、可枚举的唯一标识，**一旦确定就不应该再被更改**；
- `message` 是写给用户或开发者看的提示信息，不应包含任何敏感信息，**不能用于逻辑判断**；
- `data` 是业务数据的载体，结构随接口而定，失败时统一为 `null`，**只有在 `code` 表示成功时才可信**；
- `traceId` 是链路追踪标识，用于把一次请求在不同模块中的日志串联起来，**不参与任何业务逻辑**。

### 201 Created

`201` 表示请求成功，并且 **服务器创建了一个新资源**，通过 `Location` 响应头告诉客户端新资源的位置。

客户端向 `/users` 提交注册信息：

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}
```

服务器创建成功并返回 `201`：

```http
HTTP/1.1 201 Created
Location: https://api.example.com/users/8f3a2c7e
Content-Type: application/json

{
  "id": "8f3a2c7e",
  "name": "Alice",
  "email": "alice@example.com",
  "createdAt": "2026-10-15T10:00:00Z"
}
```

客户端拿到 `Location` 后，就可以访问新资源：

```http
GET /users/8f3a2c7e HTTP/1.1
Host: api.example.com
```

{% note info %}
`200` 不表示是否创建资源，但 `201` 表示 **一定创建了新资源**，并且通常都会携带 `Location` 响应头告诉客户端新资源的位置。
{% endnote %}

### 204 No Content

`204` 表示请求已被成功处理，但 **没有响应体**。

客户端发送请求删除一篇已存在的文章，成功后不需要返回任何内容：

```http
DELETE /articles/8f3a2c7e HTTP/1.1
Host: api.example.com
```

服务器删除成功并返回 `204`：

```http
HTTP/1.1 204 No Content
Date: Wed, 15 Oct 2026 10:00:00 GMT
```

{% note info %}
`200` 不表示是否存在响应体，但 `204` 表示 **一定没有响应体**，它们的区别不在于有没有响应体，而是客户端应不应该读响应体。
{% endnote %}

### 206 Partial Content

`206` 表示 **客户端请求了资源的一部分**，服务器成功返回了这部分内容。

播放器一般不会直接下载整个文件，而是先请求一部分数据：

```http
GET /videos/movie.mp4 HTTP/1.1
Host: example.com
Range: bytes=0-1023
```

服务器如果支持范围请求，会返回 `206`：

```http
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/10485760
Content-Length: 1024
Content-Type: video/mp4

[前 1024 字节的二进制数据]
```

播放器会根据播放进度，继续 **按需请求对应的部分**。当用户拖动进度条时，播放器不再按顺序下载，而是直接跳到目标节点对应的字节位置继续请求。

{% note info %}
播放器下载过的字节，通常会 **缓存在内存中**，**由播放器自己控制**，关闭就清空；而 HTTP 缓存通常是 **在磁盘中持久缓存**，**由浏览器底层控制**。
{% endnote %}

## 重定向类

### 301 Moved Permanently

`301` 表示 **请求的资源已被永久移动到新位置**，以后都应该用新地址访问。

网站的旧域名是 `example.com`，现在换成了 `example.net`，当用户访问旧地址时，应该被永久重定向到新地址：

```http
GET /products/8f3a2c7e HTTP/1.1
Host: example.com
```

服务器如果配置了重定向规则，就会返回 `301`：

```http
HTTP/1.1 301 Moved Permanently
Location: https://example.net/products/8f3a2c7e
Content-Length: 0
```

浏览器收到 `301` 状态码后，会读取 `Location` 并 **自动跳转**，同时更新地址栏。

{% note info %}
页面内部的资源请求如果返回了 `301`，也会 **自动跟随重定向** 到对应的新资源。
{% endnote %}

返回 `301` 状态码是业务层自己实现的逻辑，**不是服务器的默认行为**。例如在 `Nginx` 中配置重定向规则：

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://example.net$request_uri;
}
```

### 302 Found

`302` 表示 **请求的资源被临时移动到另一个位置**，但以后应该用旧地址访问。

用户在网站 `example.com` 上访问订单详情页，但还没有登录：

```http
GET /orders/8f3a2c7e HTTP/1.1
Host: example.com
```

服务器检测到用户未登录，会返回 `302`，**临时把用户带到登录页**：

```http
HTTP/1.1 302 Found
Location: https://example.com/login?redirect=/orders/8f3a2c7e
```

浏览器收到 `302` 状态码后，会读取 `Location` 并自动跳转：

```http
GET /login?redirect=/orders/8f3a2c7e HTTP/1.1
Host: example.com
```

用户登录成功后，服务器会根据 `redirect` 参数，把用户带回原页面：

```http
HTTP/1.1 302 Found
Location: https://example.com/orders/8f3a2c7e
```

浏览器收到 `302` 后再次自动跳转，进入订单详情页。

{% note info %}
中间页通常会 **携带原地址**，处理完成后再跳转回原页面。`302` 跳转是由浏览器自动发起的 `GET` 请求，不携带请求体，所以一般会把原地址 **放在查询参数中**。
{% endnote %}

### 304 Not Modified

`304` 表示 **客户端缓存的资源仍然有效**，服务器不需要返回内容，直接使用缓存即可。

用户首次访问页面中的一张图片：

```http
GET /images/pic.png HTTP/1.1
Host: example.com
```

图片属于静态资源，服务器给它配置了 `ETag`：

```http
HTTP/1.1 200 OK
Content-Type: image/png
Content-Length: 4096
ETag: "7e2f9a4c"
Cache-Control: max-age=3600

[图片数据]
```

浏览器会把静态资源存进本地缓存。强制缓存到期后，浏览器不确定图片有没有改变，于是带上 `ETag` 请求服务器：

```http
GET /images/pic.png HTTP/1.1
Host: example.com
If-None-Match: "7e2f9a4c"
```

服务器检查图片，发现 `ETag` 没有改变，于是返回 `304`：

```http
HTTP/1.1 304 Not Modified
ETag: "7e2f9a4c"
Cache-Control: max-age=3600
```

浏览器收到 `304` 后，会自动从本地缓存读取资源，这样可以 **大幅减少重复下载**。

{% note info %}
`3xx` 代表为了完成请求，客户端 **需要采取进一步动作**。`304` 不属于重定向，但语义上符合 `3xx` 的定义。
{% endnote %}

`304` 的缓存校验方式有两种：**基于时间** 和 **基于内容指纹**。

**基于时间**

首次发送请求时，服务器返回 `Last-Modified`，代表资源的最后修改时间：

```http
HTTP/1.1 200 OK
Content-Type: image/png
Last-Modified: Wed, 15 Oct 2026 10:00:00 GMT

[图片数据]
```

强制缓存过期后，浏览器会发送请求并自动携带 `If-Modified-Since` 校验头：

```http
GET /images/pic.png HTTP/1.1
Host: example.com
If-Modified-Since: Wed, 15 Oct 2026 10:00:00 GMT
```

服务器对比资源的最后修改时间，返回 `304` 或 `200` + 新内容。

{% note warning %}
即使服务器的响应头没有设置强制缓存，浏览器也会走 **启发式缓存**，自己推算一个强制缓存时间。
{% endnote %}

**基于内容指纹**

首次发送请求时，服务器返回 `ETag`，代表资源的内容指纹：

```http
HTTP/1.1 200 OK
Content-Type: image/png
ETag: "7e2f9a4c"

[图片数据]
```

强制缓存过期后，浏览器会发送请求并自动携带 `If-None-Match` 校验头：

```http
GET /images/pic.png HTTP/1.1
Host: example.com
If-None-Match: "7e2f9a4c"
```

服务器对比资源的 **内容指纹**，返回 `304` 或 `200` + 新内容。

{% note modern no-icon %}
**强制缓存和协商缓存**

浏览器的缓存机制分为两层，校验头 `If-None-Match` / `If-Modified-Since` 只和第二层有关：

- 强制缓存由 `Cache-Control: max-age` 和 `Expires` 响应头控制，不会向服务器发送请求；
- 协商缓存由 `ETag` 和 `Last-Modified` 响应头控制，**强制缓存过期后**，客户端需要发送携带校验头的请求，才有可能命中缓存返回 `304`。
  {% endnote %}

{% note info %}
服务器通常会同时返回 `ETag` 和 `Last-Modified`，但校验时 `ETag` 优先级更高，只要请求头带了 `If-None-Match`，就会忽略 `If-Modified-Since`。
{% endnote %}

{% note info %}
`If-Modified-Since` 和 `If-None-Match` 是 **浏览器自动添加** 的请求头；`ETag` 和 `Last-Modified` 对于静态资源通常是服务端框架自动生成的，对于动态接口通常需要业务自己配置。
{% endnote %}

### 307 Temporary Redirect

`307` 表示请求的资源临时被移动到另一个位置，并且 **必须保持原请求方法和请求体**。

{% note info %}
`302` 和 `307` 都是临时重定向，现代浏览器处理 `302` 状态码时可能会改变请求方法，但 `307` 明确要求保持请求方法和请求体不变。
{% endnote %}

用户在页面点击支付，浏览器发出 `POST` 请求：

```http
POST /api/pay HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "orderId": "8f3a2c7e",
  "amount": 19900
}
```

主服务器正在维护，于是返回 `307`，把请求临时重定向到备用服务器：

```http
HTTP/1.1 307 Temporary Redirect
Location: https://backup.example.com/api/pay
Content-Length: 0
```

浏览器收到 `307` 状态码后，会保持原方法和请求体 **自动发送重定向请求**：

```http
POST /api/pay HTTP/1.1
Host: backup.example.com
Content-Type: application/json

{
  "orderId": "8f3a2c7e",
  "amount": 19900
}
```

备用服务器可以正常处理支付，用户对重定向全程无感知：

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "orderId": "8f3a2c7e"
}
```

`nginx` 请求转移的配置：

```nginx
server {
    listen 80;
    server_name example.com;

    return 307 https://backup.example.net$request_uri;
}
```

{% note warning %}
如果服务器返回 `302`，现代浏览器可能会把 `POST` 改成 `GET`，并丢掉 `body`，导致支付失败。
{% endnote %}

### 308 Permanent Redirect

`308` 表示请求的资源被永久移动到新位置，并且 **必须保持原请求方法和请求体**。

{% note info %}
`301` 和 `308` 都是永久重定向，区别在于 `308` 明确要求保持请求方法和请求体不变。
{% endnote %}

用户填写完问卷后，点击提交发送请求：

```http
POST /api/responses HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "surveyId": "8f3a2c7e",
  "answers": [
    { "questionId": "q1", "value": "A" },
    { "questionId": "q2", "value": "B" }
  ]
}
```

接口已经永久迁到了新域名，服务器返回 `308`：

```http
HTTP/1.1 308 Permanent Redirect
Location: https://new-survey.com/api/responses
Content-Length: 0
```

浏览器收到 `308` 后，会保持原方法和请求体 **自动发送重定向请求**：

```http
POST /api/responses HTTP/1.1
Host: new-survey.com
Content-Type: application/json

{
  "surveyId": "8f3a2c7e",
  "answers": [
    { "questionId": "q1", "value": "A" },
    { "questionId": "q2", "value": "B" }
  ]
}
```

新服务器正常保存答卷，并答复用户：

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "responseId": "3b8d1f6a",
  "submittedAt": "2026-10-16T10:00:00Z"
}
```

{% note info %}
浏览器会缓存 `301` 和 `308` 的重定向映射，不会再向旧地址发送请求，而且默认是 **长期缓存**，这是 **永久** 语义的直接结果。
{% endnote %}

## 客户端错误

### 400 Bad Request

`400` 表示 **请求格式或参数有误**，服务器拒绝处理。

{% note info %}
`400` 是通用的兜底客户端错误，它 **不表达具体是哪一类错误**，服务端应该优先使用更精确的状态码。
{% endnote %}

客户端提交注册信息时，写错了 `JSON` 语法（少了一个引号）：

```http
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "name": "Alice,
  "email": "alice@example.com"
}
```

服务端尝试解析 `JSON` 字符串，发现有语法错误，返回 `400`：

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "code": 20001,
  "message": "请求体不是合法的 JSON"
}
```

{% note info %}
`4xx` 通常会配合响应体 `code` 使用，状态码表达属于哪一类错误，`code` 表达具体是哪一个错误。实际上很多业务会使用 `200` + `code` 定位错误类型，这样会丢失 HTTP 语义，不是规范做法。
{% endnote %}

### 401 Unauthorized

`401` 表示 **请求需要身份认证**，但客户端未提供有效凭证。

用户点击网页的下载按钮，但 `token` 已经过期：

```http
GET /api/files/8f3a2c7e/download HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

服务器发现 `token` 已过期，无法确认用户身份，于是返回 `401`：

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api"
Content-Type: application/json

{
  "code": 10001,
  "message": "登录已过期，请重新登录"
}
```

**前端业务代码** 识别到 `401` 状态码或对应的响应体 `code`，会调起登录页面：

```javascript
if (res.status === 401) {
  localStorage.removeItem("token");
  redirectToLogin();
}
```

{% note info %}
`401` 通常会携带 `WWW-Authenticate` 响应头，告诉客户端应该用什么方式认证。
{% endnote %}

{% note modern no-icon %}
**Authorization**

`Authorization` 请求头专门用于 **客户端向服务器证明自己的身份**。

格式为：`Authorization: <scheme> <credentials>`

- `scheme`：认证方案，如 `Basic`、`Bearer`、`Digest`；
- `credentials`：认证凭据，格式由 `scheme` 决定。

1. `Basic` 是经过 Base64 编码后的字符串，例如 `Authorization: Basic YWxpY2U6c2VjcmV0`，`YWxpY2U6c2VjcmV0` 是 `<user>:<secret>` 的 Base64 编码，任何人拿到这个字符串都可以直接解码还原，因此 `Basic` 必须配合 HTTPS 使用。
2. `Bearer` 本身没有规定内部结构，它由服务端直接发放，谁持有这个字符串，谁就被认为有权限。
3. `Digest` 是为了解决 `Basic` 明文 Base64 的问题而设计的旧方案，现在很少使用。

`Authorization` 需要客户端手动设置，而 `Cookie` 是浏览器自动携带的，认证凭据既可以放在 `Authorization` 头，也可以放在 `HttpOnly Cookie` 中。
{% endnote %}

### 403 Forbidden

`403` 表示 **服务器理解了请求，但拒绝执行**。

{% note info %}
`401` 代表没有认证，`403` 代表 **认证成功，但无权限**。
{% endnote %}

用户已登录，带着有效的 `token` 请求管理员接口：

```http
GET /api/admin/users HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

服务器校验 `token` 并认证通过，但检查到用户没有访问权限，于是返回 `403`：

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "code": 10003,
  "message": "无权访问该资源"
}
```

前端代码收到 `403` 响应后，执行对应的业务逻辑：

```javascript
if (res.status === 403) {
  showError("没有权限执行此操作");
}
```

{% note warning %}
`token` 本身是 **令牌、凭证** 的意思，但在大模型领域被解释为 **词元**，虽然同名，但含义是完全不一样的。
{% endnote %}

### 404 Not Found

`404` 表示 **服务器找不到请求的资源**。

用户访问一篇已经被作者删除的文章：

```http
GET /api/articles/8f3a2c7e HTTP/1.1
Host: example.com
```

服务器检索 `8f3a2c7e`，发现资源不存在，于是返回 `404`：

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "code": 10004,
  "message": "文章不存在或已被删除"
}
```

{% note info %}
对于 **敏感资源**，有些系统会刻意用 `404` 代替 `403`，隐藏资源存在性。
{% endnote %}

### 429 Too Many Requests

`429` 表示客户端短时间内发送了太多次请求，**已被服务器限流**。

用户登录网站时，多次输错了密码：

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{ "email": "alice@example.com", "password": "k7Xm2pQ9" }
```

服务器短时间内收到多次登录请求，为了防止暴力破解密码，触发了限流并返回 `429`：

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1728986460
Content-Type: application/json

{
  "code": 10005,
  "message": "登录尝试过于频繁，请 60 秒后再试"
}
```

前端收到 `429` 状态码，会读取 `Retry-After` 并告知用户等待后重试。

{% note info %}
`429` 通常会携带 `Retry-After` 响应头，并配合其他自定义字段告知客户端限流信息。
{% endnote %}

## 服务器错误

### 500 Internal Server Error

`500` 表示服务器在处理请求时遇到了意外情况，**无法完成请求**。

{% note info %}
和 `400` 类似，`500` 属于通用的服务器兜底错误，仅表示错误在于服务器。
{% endnote %}

用户登录后上传一张图片作为头像：

```http
POST /api/users/avatar HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: multipart/form-data; boundary=----7f3a9c2e

------7f3a9c2e
Content-Disposition: form-data; name="file"; filename="avatar.png"
Content-Type: image/png

[图片二进制数据]
------7f3a9c2e--
```

服务器校验通过后，调用图片处理服务压缩裁剪，但该服务抛出异常，于是返回 `500`：

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "code": 50000,
  "message": "服务器内部错误",
  "traceId": "3b8d1f6a"
}
```

前端收到 `500` 会提示用户错误信息，并记录下 `traceId` 用于问题追踪。

{% note info %}
`5xx` 一般不会返回具体的错误信息，它们属于服务端内部错误，不应该对外暴露。
{% endnote %}

### 502 Bad Gateway

`502` 表示作为网关或代理的服务器，**从上游服务器收到了无效响应**。

{% note info %}
**网关** 是系统的统一入口，客户端不直接访问后端服务，而是先经过网关，再 **路由**（按规则把请求分发给正确的目标）到其他的服务，例如反向代理、API 网关等。
{% endnote %}

用户访问某个页面：

```http
GET /api/orders HTTP/1.1
Host: example.com
```

请求先到 `Nginx` 服务器，它会按配置把请求转发给后端服务：

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080;
}
```

`Nginx` 尝试连接 `127.0.0.1:8080`，但该应用已经停止运行，无法拿到有效响应，于是返回 `502`：

```http
HTTP/1.1 502 Bad Gateway
Content-Type: text/html

<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
</body>
</html>
```

### 503 Service Unavailable

`503` 表示 **服务器暂时无法处理请求**，通常是过载或维护导致的，稍后可能恢复。

{% note warning %}
`503` 不代表服务器彻底故障，客户端能收到响应，说明 **至少还有一层服务器在正常运行**（例如网关等）。如果服务器彻底故障，客户端会连接超时，收不到 HTTP 响应。
{% endnote %}

用户在流量高峰期浏览一个商品：

```http
GET /api/products/8f3a2c7e HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Accept: application/json
```

服务器检测到过载，主动拒绝这个请求，返回 `503`：

```http
HTTP/1.1 503 Service Unavailable
Content-Type: application/json
Retry-After: 30

{
  "code": 50300,
  "message": "服务暂时不可用，请稍后再试",
  "traceId": "3b8d1f6a"
}
```

前端收到 `503` 状态码，会读取 `Retry-After` 并告知用户等待后再试。

{% note info %}
`503` 通常会携带 `Retry-After` 响应头，告诉客户端多久后重试。
{% endnote %}

### 504 Gateway Timeout

`504` 表示作为网关或代理的服务器，**在规定时间内没有等到上游服务器的响应**。

用户通过浏览器提交视频转码请求：

```http
POST /api/transcode/8f3a2c7e HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{
  "format": "mp4",
  "quality": "1080p"
}
```

请求先到 `Nginx` 服务器，`Nginx` 按配置把请求转发给转码服务，但是等待超时了，于是返回 `504`：

```http
HTTP/1.1 504 Gateway Timeout
Content-Type: application/json

{
  "code": 50400,
  "message": "服务响应超时，请稍后再试",
  "traceId": "7e2f9a4c"
}
```

{% note info %}
`504` 的语义是上游服务还在运行，只是等待响应超时了，**网关和上游服务器的 TCP 连接是正常的**；如果连接不到上游服务器，就属于 `502` 错误。
{% endnote %}

---

{% series browser %}
