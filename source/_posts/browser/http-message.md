---
title: HTTP 消息
date: 2026-10-01
updated: 2026-10-01
top_img: /img/series/browser.jpg
cover: /img/covers/browser/http-message.jpg
series: browser
categories:
  - 浏览器
---

## 基本结构

HTTP 消息分为 **请求消息** 和 **响应消息**，都由 `4` 个部分组成：

1. 起始行：分为请求行和状态行，用来描述请求或响应的基本信息；
2. 头部字段：若干行键值对，用于传递元信息，不承载主体数据；
3. 空行：用 `\r\n` 分隔头部字段与消息正文，即使无正文也必须存在；
4. 消息正文：承载业务主体数据的部分，位于空行之后，可选。

{% note info %}
**HTTP 消息** 是标准术语，很多场景也称作 **HTTP 报文**，含义是一致的。
{% endnote %}

## 请求消息

### 请求行

请求行是请求消息的第一行，用来告诉服务器执行什么操作。

格式为 `方法 SP 请求目标 SP HTTP版本 CRLF`，例如：

```http
GET /index.html HTTP/1.1
```

{% note info %}
`SP` 和 `CRLF` 是描述消息格式的记法，代表不可见的控制字符。`SP` 是 `Space` 的缩写，表示一个空格字符；`CRLF` 是 `Carriage Return` 和 `Line Feed` 的缩写，表示回车符与换行符的组合 `\r\n`。
{% endnote %}

**方法** 表示对目标资源执行的操作，必须大写。

**请求目标** 表示请求的资源位置，有 `4` 种形式：

1. origin-form：最常见，路径 + 查询字符串，服务器依靠请求头里的 `Host` 字段确定目标主机；
   例如 `/search?q=http&page=1`
2. absolute-form：完整URL，客户端向 HTTP 代理服务器发送请求时使用；
   例如 `http://www.example.com/index.html`
3. authority-form：没有协议和路径，只有域名 + 端口，用于建立一条 TCP 隧道，仅 `CONNECT` 方法使用；
   例如 `www.example.com:443`
4. asterisk-form：用于查询服务器整体支持的 HTTP 方法，仅 `OPTIONS` 方法使用。
   例如 `*`

**HTTP 版本** 表示客户端使用的 HTTP 版本。

### 请求头

请求头是客户端发给服务器的一组键值对元信息，位于请求行之后。

格式为 `字段名: 字段值 CRLF`，例如：

```http
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Content-Type: application/json
Content-Length: 27
Cookie: sessionid=8sKp9Rz...
```

字段名不区分大小写，但字段值不一定，例如 `Content-Type: application/JSON` 和 `application/json` 等效，但 `Cookie` 的值和 `Authorization` 令牌等严格区分大小写。

**核心字段**

| 字段名          | 说明                     | 示例值                                     |
| --------------- | ------------------------ | ------------------------------------------ |
| Host            | 目标服务器的主机名和端口 | `Host: www.example.com`                    |
| User-Agent      | 客户端软件标识           | `User-Agent: Mozilla/5.0...`               |
| Accept          | 可接受的响应类型         | `Accept: text/html,application/json,*/*`   |
| Accept-Language | 可接受的自然语言         | `Accept-Language: zh-CN,zh;q=0.9,en;q=0.8` |
| Accept-Encoding | 可接受的内容编码         | `Accept-Encoding: gzip, deflate, br`       |
| Connection      | 连接管理选项             | `Connection: keep-alive`                   |
| Cookie          | 发送给服务器的 Cookie    | `Cookie: sessionid=8sKp9Rz...; theme=dark` |

{% note info %}
`Cookie` 属于 **请求头字段**，浏览器会自动携带，用于向服务器传递客户端存储的键值对数据。跨域请求默认不会携带 `Cookie`，需要客户端和服务器额外开启凭证。
{% endnote %}

**请求内容**

| 字段名         | 说明               | 示例值                           |
| -------------- | ------------------ | -------------------------------- |
| Content-Type   | 请求正文的媒体类型 | `Content-Type: application/json` |
| Content-Length | 请求正文的字节数   | `Content-Length: 27`             |

**认证与安全**

| 字段名        | 说明                | 示例值                                  |
| ------------- | ------------------- | --------------------------------------- |
| Authorization | 认证凭据            | `Authorization: Bearer eyJhbGci...`     |
| Origin        | 请求来源，用于 CORS | `Origin: https://www.example.com`       |
| Referer       | 来源页面 URL        | `Referer: https://www.example.com/page` |

{% note info %}
浏览器通过 `Origin` 字段主动告诉服务器这个请求是从哪个源发起的，服务器可以据此拒绝不可信的来源。`Origin` 由浏览器自动添加，页面无法伪造，主要应用于 **CORS 校验** 和 **CSRF 防护**。
{% endnote %}

{% note warning %}
所有跨域请求都会携带 `Origin` 字段，同源的 `POST/PUT` 请求通常也会携带 `Origin`，但同源的 `GET/HEAD` 通常不带 `Origin`。`GET/HEAD` 约定为只读、无副作用，而 CSRF 一般用 `POST` 修改数据。
{% endnote %}

**缓存与条件请求**

| 字段名            | 说明                               | 示例值                              |
| ----------------- | ---------------------------------- | ----------------------------------- |
| Cache-Control     | 缓存指令                           | `Cache-Control: no-cache`           |
| If-Modified-Since | 条件请求，资源在此时间后修改才返回 | `If-Modified-Since: Mon, 23 Oct...` |
| If-None-Match     | 条件请求，ETag 不匹配才返回        | `If-None-Match: "8f29d412b7c"`      |
| If-Match          | 条件请求，ETag 匹配才返回          | `If-Match: "2e417c909a3"`           |
| Range             | 请求部分内容，用于断点续传         | `Range: bytes=0-1023`               |

{% note info no-icon %}
`Range` 字段用于范围请求，可以实现 **断点续传**。客户端告知服务器需要读取的字节范围，服务器返回对应片段；传输中断后，客户端再次请求时从该位置继续获取剩余字节，不用重新下载全部资源，需要服务器支持 `Accept-Ranges: bytes`。
{% endnote %}

只有 `Host` 字段在 `HTTP/1.1` 中是 **强制携带** 的请求头，其余字段都是可选的，你也可以设置自定义的请求头字段，自定义头部不属于 HTTP 协议标准，需要客户端与服务端自行约定。旧惯例中自定义请求头字段以 `X-` 开头，现代规范不再推荐加前缀。

### 空行

头部字段结束后，必须有一个空行，即单独的 `\r\n`，例如：

```http
...最后一个头部字段\r\n
\r\n
消息正文...
```

即使没有正文也必须存在，否则接收方无法判断头部何时结束。

{% note info %}
在底层 HTTP 客户端中，报文并不一定是一次性发送的，也可能通过 TCP 连接 **分批发送**。
{% endnote %}

### 请求体

请求体是 **可选** 部分，承载客户端发给服务器的载荷数据。

`GET`、`HEAD`、`DELETE`、`OPTIONS` 通常没有正文，`POST`、`PUT`、`PATCH` 通常有正文。

{% note warning %}
`GET` 请求语法上可以携带消息正文，但 HTTP 规范没有定义 `GET` 的正文语义，很多服务端框架会直接丢弃它的请求体。因此 `GET` 请求通常不携带消息正文，**只通过查询字符串传参**。
{% endnote %}

请求体的正文格式由头部字段 `Content-Type` 描述。

| Content-Type                        | 说明            |
| ----------------------------------- | --------------- |
| `text/plain`                        | 纯文本          |
| `text/html`                         | HTML 文档       |
| `text/css`                          | CSS 样式表      |
| `text/javascript`                   | JavaScript 代码 |
| `application/json`                  | JSON 字符串     |
| `application/pdf`                   | PDF 文档        |
| `application/zip`                   | ZIP 压缩包      |
| `application/octet-stream`          | 二进制流        |
| `application/x-www-form-urlencoded` | 普通文本表单    |
| `multipart/form-data`               | 文件上传表单    |
| `image/svg+xml`                     | SVG 矢量图      |
| `image/png`                         | PNG 图片        |
| `audio/mpeg`                        | MP3 音频        |
| `video/mp4`                         | MP4 视频        |

HTML `<form>` 标签的 `enctype` 属性决定了表单提交的编码类型，默认值为 `application/x-www-form-urlencoded`，仅发送文本键值对，**不能上传文件**，特殊字符会做 URL 编码；设置为 `multipart/form-data` 时，会自动生成 `boundary` 字符串切分多个独立片段，**支持文件上传**，数据不做 URL 编码。

HTML 的 `form` 表单提交只需要写 `enctype="multipart/form-data"`，浏览器会自动生成 `boundary` 并补全到 `Content-Type` 请求头中；如果是手动构造请求，必须在 `Content-Type` 中写上 `boundary`，并且请求体内的分隔符要和这个字符串对应，否则服务器无法切割请求体，会返回 `400 Bad Request`。

**application/x-www-form-urlencoded**

```html
<form method="POST" action="/api/submit">
  <input name="title" value="note" />
  <input name="content" value="hi http" />
</form>
```

```http
POST /api/submit HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

title=note&content=hi%20http
```

**multipart/form-data**

```html
<form method="POST" action="/api/file" enctype="multipart/form-data">
  <input name="msg" value="upload image" />
  <input type="file" name="pic" />
</form>
```

```http
POST /api/file HTTP/1.1
Host: www.example.com
Content-Type: multipart/form-data; boundary=WebKitFormBoundaryX7s9k2pR6nD1
Content-Length: 246

------WebKitFormBoundaryX7s9k2pR6nD1
Content-Disposition: form-data; name="msg"

upload image
------WebKitFormBoundaryX7s9k2pR6nD1
Content-Disposition: form-data; name="pic"; filename="demo.jpg"
Content-Type: image/jpeg

[JPG二进制数据]
------WebKitFormBoundaryX7s9k2pR6nD1--
```

{% note warning %}
如果 `<form>` 没有设置 `enctype="multipart/form-data"`，`<input type="file" name="pic" />` 就不能正确上传文件内容，服务器收到的 `pic` 会是字符串 `"a.png"`，而不是二进制数据。
{% endnote %}

**正文长度**

接收方如何确定正文结束，是 HTTP 消息解析的核心问题之一。HTTP 是流式协议，数据 **通过 TCP 字节流传输**，没有天然的消息边界，所以必须靠规则来判断正文在哪里结束。

可以设置请求头字段 `Content-Length: 10` 明确指定字节数，接收方读到指定字节数量就认为正文结束，这是最常用的方式。

```http
POST /api/submit HTTP/1.1
Host: www.example.com
Content-Type: text/plain
Content-Length: 10

HelloHTTP!
```

也可以使用 `Transfer-Encoding: chunked` 分块传输，正文由多个块组成，每个块开头标明块大小，读到大小为 `0` 的块就认为正文结束，常用于 **动态输出** 等长度未知的情况。

```http
POST /stream HTTP/1.1
Host: www.example.com
Transfer-Encoding: chunked

14
Server received first part
17
This is the second message
0
```

{% note info %}
分块传输中的 `14` 和 `17` 是 `16` 进制数，分别表示 `20` 和 `23` 个字节。
{% endnote %}

`HTTP/1.0` 无 `Content-Length`，客户端发送完数据后直接断开 TCP 连接，服务器读到连接断开就认为正文结束。`HTTP/1.1` 默认长连接，不再使用这种方式。

## 响应消息

### 状态行

状态行是响应消息的第一行，用来告诉客户端本次请求的处理结果。

格式为 `HTTP版本 SP 状态码 SP 原因短语 CRLF`，例如：

```http
HTTP/1.1 200 OK
```

{% note info %}
客户端解析响应消息应该使用 **状态码**，原因短语仅为可读性说明，可以修改甚至省略。
{% endnote %}

### 响应头

服务器通过响应头向客户端传递本次响应的附加元信息，位于状态行之后。

格式为 `字段名: 字段值 CRLF`，例如：

```http
Content-Type: application/json
Content-Length: 36
Server: nginx
Set-Cookie: sessionid=8sKp9Rz; HttpOnly
Cache-Control: no-cache
Access-Control-Allow-Origin: https://www.example.com
```

**核心字段**

| 字段名     | 说明                        | 示例值                                    |
| ---------- | --------------------------- | ----------------------------------------- |
| Server     | 服务器软件标识              | `Server: nginx`                           |
| Date       | 响应生成时间                | `Date: Tue, 07 Oct 2026 02:00:00 GMT`     |
| Connection | 连接管理选项                | `Connection: keep-alive`                  |
| Location   | 重定向地址，配合 3xx 状态码 | `Location: https://www.example.com/home`  |
| Set-Cookie | 指令浏览器保存 Cookie       | `Set-Cookie: sessionid=8sKp9Rz; HttpOnly` |

浏览器内置的 HTTP 处理器会自动识别 `3xx` 状态码与 `Location` 响应头，并自动跳转，**不依赖 JavaScript**，例如 `<img>` 等图片资源请求返回 `3xx` 和 `Location` 时，会自动替换为新图片。

服务器在 HTTP 响应头里带上 `Set-Cookie` 字段，浏览器收到报文后，会自动保存这条 `Cookie`。可以附加属性 `HttpOnly`，禁止 JS 读取或修改这条 `Cookie`。

{% note info %}
`Set-Cookie` 是 HTTP 协议层面的指令，属于浏览器内置的报文解析逻辑，**不依赖 JavaScript**。JS 是浏览器提供的另一个操作 `Cookie` 的方式，它们读写的是同一份存储。
{% endnote %}

**响应内容**

| 字段名            | 说明               | 示例值                           |
| ----------------- | ------------------ | -------------------------------- |
| Content-Type      | 响应正文的媒体类型 | `Content-Type: application/json` |
| Content-Length    | 响应正文的字节数   | `Content-Length: 36`             |
| Transfer-Encoding | 分块传输编码       | `Transfer-Encoding: chunked`     |
| Content-Encoding  | 正文压缩编码       | `Content-Encoding: gzip`         |

{% note info %}
`Transfer-Encoding` 一般出现在 **响应头** 中，服务器预先不知道响应体总字节大小，就使用 `Transfer-Encoding: chunked`。它可以作为请求头字段，但浏览器不能自动生成，仅自定义客户端可以手动构造。
{% endnote %}

**资源缓存**

| 字段名        | 说明                           | 示例值                               |
| ------------- | ------------------------------ | ------------------------------------ |
| Cache-Control | 缓存指令                       | `Cache-Control: no-cache`            |
| ETag          | 资源唯一标识，用于协商缓存     | `ETag: "v1-8f29d412b7c"`             |
| Last-Modified | 资源最后修改时间，用于协商缓存 | `Last-Modified: Mon, 23 Oct 2023...` |

**跨域与安全**

| 字段名                           | 说明                     | 示例值                    |
| -------------------------------- | ------------------------ | ------------------------- |
| Access-Control-Allow-Origin      | 允许跨域访问的源         | `https://www.example.com` |
| Access-Control-Allow-Credentials | 是否允许携带 Cookie 凭证 | `true`                    |

同源请求会默认携带 `Cookie` 字段；跨域请求需要前端设置 `withCredentials = true`，并且服务器响应头设置 `Access-Control-Allow-Credentials` 为 `true` 时才可跨域携带 `Cookie`。

`withCredentials = true` 是前端请求的配置，它会在跨域请求中把符合规则的 `Cookie` 自动放进请求头并发送给服务器，服务器必须同时返回响应头 `Access-Control-Allow-Credentials: true`，否则浏览器会 **丢弃本次响应**，前端拿不到返回的数据。

{% note warning %}
`Access-Control-Allow-Credentials: true` 生效的前提条件是 `Access-Control-Allow-Origin` 不能写 `*`，必须写明确的源。
{% endnote %}

### 空行

头部字段结束后，必须有一个空行，即单独的 `\r\n`，例如：

```http
...最后一个头部字段\r\n
\r\n
消息正文...
```

即使没有正文也必须存在，否则接收方无法判断头部何时结束。

### 响应体

响应体是 **可选** 部分，承载服务器返回给客户端的载荷数据。

`HEAD` 请求没有响应体，这是规范强制要求的；其他方法可以有响应体，取决于状态码：`1xx`、`204 No Content`、`304 Not Modified` 禁止携带响应体；其他状态码可以携带响应体。

{% note warning %}
`1xx`、`204 No Content`、`304 Not Modified` 这类状态码，即使服务器强行携带响应体，浏览器的 HTTP 解析器也会忽略这部分载荷。
{% endnote %}

响应体的正文格式由头部字段 `Content-Type` 描述。

| Content-Type                        | 说明            |
| ----------------------------------- | --------------- |
| `text/plain`                        | 纯文本          |
| `text/html`                         | HTML 文档       |
| `text/css`                          | CSS 样式表      |
| `text/javascript`                   | JavaScript 代码 |
| `application/json`                  | JSON 字符串     |
| `application/pdf`                   | PDF 文档        |
| `application/zip`                   | ZIP 压缩包      |
| `application/octet-stream`          | 二进制流        |
| `application/x-www-form-urlencoded` | 普通文本表单    |
| `multipart/form-data`               | 文件上传表单    |
| `image/svg+xml`                     | SVG 矢量图      |
| `image/png`                         | PNG 图片        |
| `audio/mpeg`                        | MP3 音频        |
| `video/mp4`                         | MP4 视频        |

响应体通过头部字段 `Content-Length` 或 `Transfer-Encoding: chunked` 来控制正文长度，与请求体相同。

## 示例代码

### 原始报文

```http
POST /api/article HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Content-Type: application/json
Content-Length: 62

{"title":"HTTP分块传输","content":"演示chunked响应报文"}
```

```http
HTTP/1.1 200 OK
Server: nginx
Content-Type: application/json
Transfer-Encoding: chunked
Access-Control-Allow-Origin: https://example.com

22
{"code":200,"msg":"文章保存成功"}
27
{"articleId":1145,"status":"published"}
0
```

### Node.js

可以直接把完整的 HTTP 请求报文通过 TCP 连接发送给服务器。

`Nest / Express / Nginx` 的 HTTP 服务器底层全部都是 **TCP 服务器**，只是把字节当成 HTTP 报文来解析。你可以直接通过 TCP 连接发送原始的 HTTP 报文，或者通过 `fetch` 等方法组装后发送，本质都是向服务端的 **TCP socket** 发送符合 HTTP 规范的字节流。

```javascript
import net from "net";

const rawRequest = `POST /api/article HTTP/1.1\r\n
Host: 127.0.0.1:3000\r\n
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)\r\n
Accept: application/json\r\n
Content-Type: application/json\r\n
Content-Length: 62\r\n
\r\n
{"title":"HTTP分块传输","content":"演示chunked响应报文"}`;

const socket = net.createConnection(
  {
    host: "127.0.0.1",
    port: 3000,
  },
  () => {
    console.log("TCP连接建立，发送原始HTTP报文");
    socket.write(rawRequest);
  },
);

socket.on("data", (buffer) => {
  console.log("收到原始响应字节");
  console.log(buffer.toString());
});

socket.on("end", () => {
  console.log("\nTCP连接关闭");
});

socket.on("error", (err) => {
  console.error("Socket错误：", err);
});
```

### 浏览器

浏览器环境一般通过 `fetch` 或 `axios` 方法发送 HTTP 请求，它们会把请求自动组装成完整的 HTTP 报文，然后发送给服务端的 TCP 套接字，服务端收到的依然是 TCP 字节流。

```javascript
async function requestChunkedArticle() {
  const response = await fetch("https://example.com/api/article", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      title: "HTTP分块传输",
      content: "演示chunked响应报文",
    }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder("utf-8");

  while (true) {
    const { done, value } = await reader.read();
    if (done) {
      console.log("流读取结束");
      break;
    }
    const chunk = decoder.decode(value, { stream: true });
    console.log("收到分片：", chunk);
  }
}

requestChunkedArticle();
```

---

{% series browser %}
