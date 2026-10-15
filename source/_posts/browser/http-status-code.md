---
title: HTTP 状态码
date: 2026-10-03
updated: 2026-10-03
top_img: /img/series/browser.jpg
cover: /img/covers/browser/http.jpg
series: browser
categories:
  - 浏览器
---

## 快速阅读

HTTP 状态码是服务器返回给客户端的 **三位数字标识**，代表本次请求的处理结果。

状态码按首位数字划分为 **五个类别**。

**信息类**

`1xx` 状态码表示 **服务器已收到请求，正在处理**。它们是 **临时响应**，不是最终结果。

| 状态码 | 原因短语            | 说明                                             |
| ------ | ------------------- | ------------------------------------------------ |
| `100`  | Continue            | 服务器已收到完整请求头，愿意继续接收请求体       |
| `101`  | Switching Protocols | 服务器同意将当前连接从 HTTP 协议切换到另一种协议 |

**成功类**

`2xx` 状态码表示 **服务器已成功处理了客户端的请求**，这是最常见的状态码类别。

| 状态码 | 原因短语        | 说明                                                   |
| ------ | --------------- | ------------------------------------------------------ |
| `200`  | OK              | 请求成功，返回内容                                     |
| `201`  | Created         | 资源创建成功，响应头通常携带 `Location` 指向新资源地址 |
| `204`  | No Content      | 请求成功，无返回内容                                   |
| `206`  | Partial Content | 范围请求成功，返回部分资源，用于断点续传、分片下载     |

**重定向类**

`3xx` 状态码表示 **重定向**，告诉客户端 **请求的资源不在当前位置**，需要进一步操作。

| 状态码 | 原因短语           | 说明                   |
| ------ | ------------------ | ---------------------- |
| `301`  | Moved Permanently  | 永久重定向             |
| `302`  | Found              | 临时重定向             |
| `304`  | Not Modified       | 资源未修改，使用缓存   |
| `307`  | Temporary Redirect | 临时重定向，保持原方法 |
| `308`  | Permanent Redirect | 永久重定向，保持原方法 |

**客户端错误**

`4xx` 状态码表示 **客户端错误**，即 **请求本身有问题**，服务器无法处理。

| 状态码 | 原因短语          | 说明             |
| ------ | ----------------- | ---------------- |
| `400`  | Bad Request       | 请求格式错误     |
| `401`  | Unauthorized      | 未认证           |
| `403`  | Forbidden         | 无权限           |
| `404`  | Not Found         | 资源不存在       |
| `429`  | Too Many Requests | 请求过多（限流） |

**服务器错误**

`5xx` 状态码表示 **服务器错误**，即 **服务器在处理请求时出了问题**。

| 状态码 | 原因短语              | 说明           |
| ------ | --------------------- | -------------- |
| `500`  | Internal Server Error | 服务器内部错误 |
| `502`  | Bad Gateway           | 网关错误       |
| `503`  | Service Unavailable   | 服务不可用     |
| `504`  | Gateway Timeout       | 网关超时       |

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

`200` 并不代表业务层面的成功，例如 `200` 状态码的响应体返回了 `{"code": 500, "msg": "登录失败"}`。HTTP 状态码和业务 `code` 解决的是两个不同层面的问题，`状态码 + 原因短语` 表示 **HTTP 请求层面的处理结果**，而 `code + msg` 表示 **业务层面的处理结果**，它们不能互相替代。

处理响应时应该通过状态码判断 HTTP 层结果，通过约定的业务 `code` 判断业务层结果。原因短语和 `msg` 只是提示信息，HTTP 解析器只会依赖状态码进行程序判断，**JS 代码也必须依赖 `code` 处理业务层结果**。

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

`201` 表示请求成功，并且 **服务器创建了一个新的资源**，通过 `Location` 响应头告诉客户端新资源的位置。

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

服务器创建成功，并返回 `201`：

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

## 重定向类

## 客户端错误

## 服务器错误
