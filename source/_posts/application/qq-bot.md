---
title: QQ Bot
date: 2026-09-24
updated: 2026-09-24
top_img: /img/series/application.jpg
cover: /img/covers/application/qq-bot.jpg
series: application
categories:
  - 应用实例
---

# 快速阅读

记录一个基于 OneBot 11 通信标准的 QQ 机器人应用的完整实现过程。

## 功能概述

1. 通过 `/upload` 指令上传群文件到服务器；
2. 通过 `/remote <指令>` 发送 RCON 命令到 `Left 4 Dead 2` 服务器，并回传执行结果；
3. 通过 `/help` 指令获取所有插件的使用说明；
4. 直接和大模型进行对话，支持输入文本、图片、视频、文件等内容。

## 核心节点

1. NapCat 协议层：负责把 QQ 底层消息封装成 OneBot 标准，并与应用层进行通信；
2. OneBot Server 应用层：负责解析协议层推送的事件并调用相应的插件进行处理；
3. LLM API Server 接口层：提供通用的大模型对话服务。

## 运行效果

![多轮会话](/img/captures/application/qq-bot/ch1-01.jpg)
![媒体识别](/img/captures/application/qq-bot/ch1-02.jpg)
![引用解析](/img/captures/application/qq-bot/ch1-03.jpg)
![联网搜索](/img/captures/application/qq-bot/ch1-04.jpg)
![远程控制](/img/captures/application/qq-bot/ch1-05.jpg)
![文件上传](/img/captures/application/qq-bot/ch1-06.jpg)

# 协议层

协议层为 **NapCat**，是一个基于 NTQQ（PC 新版 QQ）的无头 Bot 协议端。

{% note info %}
你也可以选择 **OpenShamrock** 等其他协议端，一般都是 OneBot 通信标准，它们的区别主要在于底层平台，例如 **OpenShamrock** 的宿主系统是 Android，依赖安卓 QQ 运行（PC 需要安卓模拟器）。
{% endnote %}

## 核心功能

**NapCat** 可以通过 IPC（进程间通信技术） 调用 NTQQ 原生能力，核心作用是把 QQ 底层消息封装成 **OneBot 标准**，并与你的应用层进行通信。

{% note info %}
**OneBot** 是一套通信标准，定义了协议层和应用层之间的通信规范，它的核心价值是解耦协议层和应用层，只要按照统一的通信标准实现，机器人业务就可以无代价接入不同的协议端。
{% endnote %}

关于 OneBot 的接口标准，请参考 [OneBot - 官方文档](https://onebot.dev/)

### 网络配置

可以在 WebUI 中修改 NapCat 的 **网络配置**，让它作为客户端或服务器与应用层进行通信。

{% note info %}
HTTP 请求是单向的，但你可以在 NapCat 中同时创建 HTTP 客户端和 HTTP 服务器来实现 **双向收发消息**。
{% endnote %}

**NapCat 作为 HTTP 客户端**

可以发送 HTTP {% label POST pink %} 请求向应用层的 API 地址推送事件。

![](/img/captures/application/qq-bot/ch2-01.jpg)

{% note warning %}
这种配置下，应用层可以接收来自 QQ 用户的消息，但无法连接 NapCat 客户端向 QQ 用户发送消息。
{% endnote %}

**NapCat 作为 HTTP 服务器**

应用层可以向 NapCat 指定的 HTTP 服务器推送事件。

![](/img/captures/application/qq-bot/ch2-02.jpg)

{% note warning %}
这种配置下，应用层可以连接 NapCat 服务器向 QQ 用户发送消息，但无法接收来自 QQ 用户的消息。
{% endnote %}

**NapCat 作为 WebSocket 客户端**

NapCat 通过 WebSocket 连接到应用层的 `ws` 或 `wss` 地址，可以双向发送 `JSON` 格式的事件和处理请求。

![](/img/captures/application/qq-bot/ch2-03.jpg)

**NapCat 作为 WebSocket 服务器**

应用层作为 WebSocket 客户端主动连接 NapCat 的服务器。

![](/img/captures/application/qq-bot/ch2-04.jpg)

{% note info %}
NapCat 可以作为 WebSocket 客户端和服务器，两种方式的通信能力是完全一致的，差异仅在于它和应用层 **谁主动发起连接**。
{% endnote %}

### 消息通信

关于 NapCat 的请求接口和消息格式，请参考 [NapCatQQ - 官方文档](https://napneko.github.io/)

![请求接口](/img/captures/application/qq-bot/ch2-05.jpg)
![消息格式](/img/captures/application/qq-bot/ch2-06.jpg)

### 管理后台

**WebUI** 是 NapCat 自带的网页管理后台，默认端口为 `6099`，可以用浏览器直接访问操作（需要 Token 鉴权），不用手动修改 `JSON` 配置文件。

![WebUI](/img/captures/application/qq-bot/ch2-07.jpg)

**主要功能**

1. QQ 账号管理：支持多账号、查看登录状态、下线账号、快速重登；
2. 网络配置修改：可视化新建、编辑、启用、禁用网络连接；
3. 实时日志查看：查看 NapCat 运行日志、QQ 消息等信息；
4. 进程与版本控制：重启 NapCat、检查更新、管理插件等。

**注意事项**

早期 NapCat 的默认监听地址是 `0.0.0.0`，且低版本 WebUI 默认弱 Token `'napcat'`，大量用户直接把 `6099` 端口暴露到公网并使用默认 Token，攻击者用端口扫描器全网扫描特征端口，找到大量暴露、无鉴权的 NapCat 实例，直接调用 API 控制机器人 QQ 在群里发送违规、违法信息，导致腾讯风控批量触发，大量机器人 QQ、群聊被永久封禁。

这些安全类的配置不能忽略，非 `80` / `443` 端口不需要经过备案就能开放，如果你的云服务器防火墙没有设置端口拦截，那么程序监听 `0.0.0.0` 就意味着它会直接暴露在公网中。

## 系统结构

{% mermaid '{"themeVariables": { "fontSize": "16px" }, "layout": "elk", "look": "handDrawn"}' %}
flowchart LR

    A[腾讯QQ服务器] <--> B[NTQQ客户端运行时]
    subgraph NapCat核心层
        C[NapCat Core]
        D[PacketBackend DLC扩展]
    end
    subgraph 通信模块
        E1[正向WebSocket]
        E2[反向WebSocket]
        E3[HTTP API]
        E4[HTTP WebHook]
    end
    subgraph 管理面板
        F[WebUI :6099]
    end
    subgraph Bot应用层
        G[OneBot Server]
        I[LLM API Server]
    end
    H[浏览器]

    B <-->|IPC| C
    C <--> D
    C --> E1 & E2 & E3 & E4
    C --> F

    E1 <-->|OneBot11| G
    E2 <-->|OneBot11| G
    E3 <-->|OneBot11| G
    E4 -->|推送事件| G
    G -.->|HTTP调用API| E4

    F <--> H
    G --> I

{% endmermaid %}

# 应用层

应用层为 **OneBot Server**，是一个基于 NestJS + WebSocket 实现的 QQ 机器人服务端。

{% note info %}
你也可以选择开源的机器人框架例如 `NoneBot2`、`Koishi` 等与 NapCat 协议层进行对接，但也带来很多约束，自由度会受到限制。
{% endnote %}

## 核心功能

1. 通过 `/upload` 指令上传群文件到服务器；
2. 通过 `/remote <指令>` 发送 RCON 命令到 `Left 4 Dead 2` 服务器，并回传执行结果；
3. 通过 `/help` 指令获取所有插件的使用说明；
4. 直接和大模型进行对话，支持输入文本、图片、视频、文件等内容。

## 系统结构

{% mermaid '{"themeVariables": { "fontSize": "16px" }, "layout": "elk", "look": "handDrawn"}' %}
flowchart TD

    QQ["QQ 客户端"] <--> NC["NapCat<br/>(OneBot 11 协议端)"]
    NC <-- "WebSocket" --> GW["Bot 服务<br/>127.0.0.1:3002"]

    subgraph Server["OneBot Server"]
        GW["NapCatGateway"]
        SVC["NapCatService<br/>协议核心"]
        BOT["BotService<br/>插件调度"]
        P1["UploadPlugin"]
        P2["RconPlugin"]
        P3["HelpPlugin"]
        P4["ChatPlugin"]
        DB[("MySQL")]
        FS["静态资源服务"]
        LLM["LLM API Server"]
    end

    GW --> SVC --> BOT --> P1 & P2 & P3 & P4
    P4 --> DB
    P4 --> LLM
    P2 --> RCON["Left 4 Dead 2 服务端"]
    P1 & P4 --> FS

{% endmermaid %}

### 会话存储

使用 `MySQL` 存储 `uid` → `conversationId` 的映射与 `NapCatEvent` 事件内容，用于多轮会话和引用回复。

**配置文件**

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=******
DB_DATABASE=onebot_server
```

**BotConversation 会话表**

接口层 LLM API Server 使用 `conversationId` 存储会话上下文，在多轮会话场景中请求体需要携带 `prompt` 和 `conversationId`，因此应用层需要把 `uid` → `conversationId` 的映射存进这张表。

`uid` 的设计实现了两级隔离：

- 群聊 `g_{群号}_{QQ号}`：同一个 QQ 群内不同成员各自有独立的上下文，`A` 聊的内容不会串到 `B`；
- 私聊 `p_{QQ号}`：每个用户都有一条独立的上下文。

| 字段           | 类型   | 说明                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| uid            | string | 主键 - 会话隔离维度，群聊 `g_{群号}_{QQ号}`、私聊 `p_{QQ号}` |
| conversationId | string | 接口层返回的多轮会话 ID                                      |
| createdAt      | Date   | 会话创建时间，`TypeORM` 自动填充                             |

**NapCatMessage 消息表**

当用户使用 QQ 的 **引用回复** 功能回复某条消息时，NapCat 推送的事件中只有一个 `reply` 段携带被引用消息的 `message_id`，不带原文内容，因此应用层需要存储每一条消息的原始内容，通过反查这张表，递归还原出被引用消息的完整内容（包括文字、图片、视频等）。

| 字段      | 类型   | 说明                                   |
| --------- | ------ | -------------------------------------- |
| id        | number | 主键 - 直接使用 NapCat 的 `message_id` |
| data      | JSON   | 整条 `NapCatEvent` 事件的原始内容      |
| createdAt | Date   | 消息发送时间，`TypeORM` 自动填充       |

{% note warning %}
引用可能是 **递归** 的，因此必须存储完整的原始事件，才能正确还原引用链。
{% endnote %}

### 资源转存

应用层会启动一个允许公网访问的 **静态资源服务**，把根目录的 `public` 文件夹暴露到公网中。

**核心作用**

NapCat 推送的事件中，图片、音频、视频、文件 URL 是腾讯媒体服务器生成的临时链接，携带过期、`token` 校验、防盗链等，无法被 LLM 直接读取，应用层可以通过静态资源服务将其转存到服务器本地并生成新的公网链接，再拼接到 `prompt` 中即可被 LLM 解析。

{% note warning %}
Bot 服务 **不能直接暴露在公网中**，无法对外提供文件，因此需要在入口文件中单独启动静态资源服务，监听不同的端口。
{% endnote %}

**实现原理**

全局内置统一的下载方法 `download`，可以将 NapCat 推送的临时资源链接下载到根目录的 `public` 文件夹中，用 `UUID` 命名（避免同文件名覆盖），并返回公网链接。

```typescript
let prompt = "";
for (const segment of userMsg) {
  const { type, data } = segment;
  const { text, file, file_id, url, id } = data;

  switch (type) {
    case "text":
      prompt += text;
      break;

    case "image":
      prompt += `[图片名称：${file}，图片链接：${url}]`;
      break;

    case "video": {
      if (!url) continue;
      // 视频可以立刻拿到下载链接 需要转存到服务器才能访问
      const publicUrl = await download(url);
      prompt += `[视频名称：${file}，视频链接：${publicUrl}]`;
      break;
    }

    case "file": {
      if (!file_id) continue;
      let res: NapCatApiResponse;

      if (message_type === "private") {
        res = await napCatService.getPrivateFileUrl(user_id!, file_id);
      } else {
        res = await napCatService.getGroupFileUrl(group_id!, file_id);
      }

      const url = res.data.url;
      if (!url) continue;

      const publicUrl = await download(url);
      prompt += `[文件名称：${file}，文件链接：${publicUrl}]`;
      break;
    }

    case "reply": {
      const prevMessage = await this.messageManager.findMessage(id!);
      if (!prevMessage) continue;

      const replyPrompt = await this.handleSegments(prevMessage, napCatService);

      prompt += `[引用消息："${replyPrompt}"]`;
      break;
    }
  }
}
return prompt;
```

{% note warning %}
NapCat 推送的临时链接后缀名是不可靠的，需要通过二进制识别并追加新的文件名后缀，才能被正确解析。
{% endnote %}

### 内置插件

应用层收到 NapCat 消息后，会调用相应的插件处理，插件按优先级依次匹配：`UploadPlugin` → `RconPlugin` → `HelpPlugin` → `ChatPlugin`。其中 `ChatPlugin` 为兜底插件，无条件命中，放在最后。

1. `UploadPlugin`

用于上传群文件到服务器，指令为 `/upload`。

**配置文件**

```env
UPLOAD_SAVE_DIR=******
UPLOAD_FOLDER_NAME=******
UPLOAD_ALLOWED_GROUPS=ID1,ID2
```

**实现原理**

插件会向 NapCat 发送 `get_group_file_url` 消息，在群文件根目录查找 `UPLOAD_FOLDER_NAME` 指定的文件夹，遍历其中全部文件，发送 `get_group_file_url` 消息逐个获取下载链接并转存到 `UPLOAD_SAVE_DIR` 指定的目录中，插件还会实时推送文件上传进度（已存在的文件自动跳过）。

| 状态       | 说明     |
| ---------- | -------- |
| skipped    | 已经存在 |
| processing | 正在上传 |
| failed     | 上传失败 |
| done       | 上传成功 |

仅 `UPLOAD_ALLOWED_GROUPS` 白名单内的群聊可用。

**测试用例**

![私聊消息](/img/captures/application/qq-bot/ch3-01.jpg)
![实时进度](/img/captures/application/qq-bot/ch3-02.jpg)
![其他状态](/img/captures/application/qq-bot/ch3-03.jpg)

2. `RconPlugin`

用于发送 RCON 命令到 `Left 4 Dead 2` 服务器，并回传执行结果，指令为 `/remote <指令>`。

**配置文件**

```env
RCON_HOST=localhost
RCON_PORT=27015
RCON_PASSWORD=******
RCON_ALLOWED_USERS=ID1,ID2
```

**实现原理**

基于 **起源引擎** 的游戏服务端启动之后，会额外开启一个 `TCP` 监听，用于接收远程发来的控制台命令，控制游戏服务端的运行。

`RconPlugin` 会监听以 `/remote` 开头的 QQ 消息，通过 RCON 协议连接 `Left 4 Dead 2` 服务端，将消息中携带的命令转发给游戏服务器，拿到执行结果后回传给 QQ。

插件内部维护单例 RCON 长连接与异步锁，避免并发重复连接。

{% note info %}
**RCON**（Remote Console）是 Valve 定义的 `TCP` 协议，用于向游戏服务器下发控制台命令，并将执行结果回传。**起源引擎** 实现了这套协议，`Left 4 Dead 2`、`Counter-Strike: Source` 等游戏都原生支持。
{% endnote %}

仅 `RCON_ALLOWED_USERS` 白名单内的用户可用。

**测试用例**

![远程控制](/img/captures/application/qq-bot/ch3-04.jpg)
![错误指令](/img/captures/application/qq-bot/ch3-05.jpg)

3. `HelpPlugin`

用于获取所有插件的使用说明，指令为 `/help`。

4. `ChatPlugin`

用于和大模型对话，不需要指令前缀。

**实现原理**

`ChatPlugin` 会监听所有消息，并调用接口层的大模型服务进行对话。插件会响应所有私聊消息，但群聊仅响应 `@机器人` 的消息。

你可以发送文本、图片、视频、文件等内容，也可以引用或回复一条历史消息，插件内部会将消息段解析后拼接成完整的 `prompt`。会话上下文按照群聊 `g_{群号}_{QQ号}`、私聊 `p_{QQ号}` 的 `uid` 规则进行隔离。

**测试用例**

![联网搜索](/img/captures/application/qq-bot/ch3-06.jpg)
![图片识别](/img/captures/application/qq-bot/ch3-07.jpg)
![引用消息](/img/captures/application/qq-bot/ch3-08.jpg)
![递归引用](/img/captures/application/qq-bot/ch3-09.jpg)

### 协议核心

`NapCatService` 是应用层的协议核心，它从 `WebSocket` 客户端接收 NapCat 事件，并按类型分发到插件模块中。

**通信方法**

`NapCatService` 定义了全局唯一一个和 NapCat 客户端直接通信的私有方法 `sendApiRequest`，并基于此封装了六个对其他模块公开的通信 API。

| 方法                  | 说明                 |
| --------------------- | -------------------- |
| sendPrivateMessage    | 发送私聊消息         |
| sendGroupMessage      | 发送群聊消息         |
| getPrivateFileUrl     | 获取私聊文件真实URL  |
| getGroupFileUrl       | 获取群聊文件真实URL  |
| getGroupRootFiles     | 获取群根目录文件列表 |
| getGroupFilesByFolder | 获取群文件夹文件列表 |

{% note info %}
`sendApiRequest` 可以调用任意的 NapCat `action`，但它不应该直接对外暴露，而是将协议实现细节封装在模块内部，只对外暴露语义化的业务接口。
{% endnote %}

所有通信 API 的返回值统一是 `Promise<NapCatApiResponse>`，业务层可以通过 `await` 拿到 `data` 字段。

**消息匹配**

`WebSocket` 是全双工通信协议，请求和响应不能保证按顺序到达，因此会产生 **消息收发的时序性问题**。OneBot 11 WebSocket 通信约定：请求携带 `echo` 字段时，响应会原样回传这个 `echo`，发送方可以据此匹配。

发送消息时生成一个随机的 `echo` 字符存入 `requestMap`：key = `echo`，value 存这个 Promise 的 `{resolve, reject}`，设置 `5000ms` 的超时定时器。当收到 NapCat 推送回来的消息时，根据 `echo` 从 `requestMap` 中取出存好的 `{resolve, reject}` 并执行对应操作。

- 成功：清除超时计时器，`resolve` 返回响应；
- 失败：清除超时计时器，`reject` 抛出错误。

```typescript
private sendApiRequest(
  request: Omit<NapCatApiRequest, 'echo'>,
): Promise<NapCatApiResponse> {
  return new Promise((resolve, reject) => {
    if (
      !this.activeClient ||
      this.activeClient.readyState !== WebSocket.OPEN
    ) {
      reject(new Error('客户端未连接'));
      return;
    }

    const echo = randomUUID();
    const fullRequest: NapCatApiRequest = { ...request, echo };

    const timeout = setTimeout(() => {
      this.requestMap.delete(echo);
      reject(new Error(`${request.action} - 请求超时`));
    }, 5000);

    this.requestMap.set(echo, {
      resolve: (res: NapCatApiResponse) => {
        clearTimeout(timeout);
        resolve(res);
      },
      reject: (err: Error) => {
        clearTimeout(timeout);
        reject(err);
      },
    });

    const data = JSON.stringify(fullRequest);
    this.activeClient.send(data);
  });
}
```

# 接口层

接口层为 **LLM API Server**，是一个基于 NestJS + LangChain 实现的独立部署的通用大模型对话服务，兼容 OpenAI 接口规范。

## 核心功能

**功能概述**

1. 通过 `SQL` 存储对话上下文，实现了多轮会话；
2. 通过 `Redis` 存储会话向量，实现了语义缓存，加速相似问题查询；
3. 接入视觉模型，实现了 `VisionTool` 工具，支持图片和视频内容识别；
4. 接入联网搜索，实现了 `SearchTool` 工具，支持联网查找最新信息。

**会话接口**

地址：`http://localhost:3001/chat` {% label POST pink %}

请求头：`Content-Type: application/json`

请求体

| 参数名         | 类型   | 必填 | 说明                                                            |
| -------------- | ------ | ---- | --------------------------------------------------------------- |
| prompt         | string | 是   | 用户输入的提示词，作为大模型的用户消息                          |
| conversationId | string | 否   | 会话唯一 `UUID`，传入则读取历史消息开启多轮会话，不传则新建会话 |

响应体

| 参数名         | 类型    | 说明                                                         |
| -------------- | ------- | ------------------------------------------------------------ |
| success        | boolean | 响应标识，`true` 代表接口正常                                |
| content        | string  | 大模型输出的回答内容                                         |
| conversationId | string  | 当前会话的 `UUID`，新建会话时返回新 ID，多轮会话时与入参一致 |

**参数示例**

1. 新建会话

```json
// request
{
  "prompt": "介绍一下 Node.js 是什么"
}

// response
{
  "success": true,
  "content": "Node.js是一个基于Chrome V8引擎的JavaScript运行环境：\n1. 它可以让JavaScript脱离浏览器运行在服务器端，实现后端开发能力\n2. 采用事件驱动、非阻塞I/O模型，轻量高效，适合构建高并发的网络应用\n3. 拥有庞大的包管理生态系统npm（现也称为Node.js包注册表），可以快速调用第三方工具库。",
  "conversationId": "bdc13642-792a-475a-928d-2484edfbc512"
}
```

2. 多轮会话

```json
// request
{
  "prompt": "推测这个女孩现在是什么心情，简单说明理由",
  "conversationId": "f616de75-88a9-43eb-8ca6-8de2da577a2e"
}

// response
{
  "success": true,
  "content": "这个女孩现在是困惑又有点疑惑的心情：\n她盯着报错的笔记本电脑，脸颊泛红、微张着嘴，旁边还有问号特效，看起来完全搞不懂为什么电脑会出故障，正处于对眼前状况摸不着头脑的状态。",
  "conversationId": "f616de75-88a9-43eb-8ca6-8de2da577a2e"
}
```

## 系统结构

{% mermaid '{"themeVariables": { "fontSize": "15px" }, "layout": "elk", "look": "handDrawn"}' %}
flowchart TD

    Client[客户端] -->|POST /chat| C[LlmController]
    C --> S[LlmService]

    S --> Q{携带 conversationId ？}
    Q -->|否，新建会话| Cache[语义缓存查询<br/>相似度 ≥ 0.92 命中]
    Q -->|是，多轮会话| Hist[加载历史消息]

    Cache -->|未命中| Agent[LangChain Agent]
    Hist --> Agent

    Agent -.->|联网搜索| Search[SearchTool]
    Agent -.->|视觉内容识别| Vision[VisionTool]
    Agent --> LLM[ChatModel 生成回答]

    Cache -->|命中| Save
    LLM --> Save[保存会话消息] --> Resp[返回 content + conversationId]

    Cache -.- Redis[(Redis)]
    Save -.- MySQL[(MySQL)]

{% endmermaid %}

### 会话存储

使用 `MySQL` 存储会话与消息上下文，用于多轮会话。

**配置文件**

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=******
DB_DATABASE=llm_api_server
```

**Conversation 会话表**

| 字段      | 类型   | 说明                                           |
| --------- | ------ | ---------------------------------------------- |
| id        | string | 主键 - 会话唯一 `UUID`                         |
| title     | string | 会话标题，截取首次提问的前 `20` 个字符和 `...` |
| createdAt | Date   | 会话创建时间，`TypeORM` 自动填充               |
| updatedAt | Date   | 会话最后修改时间，`TypeORM` 自动填充           |

**Message 消息表**

| 字段           | 类型                    | 说明                                                 |
| -------------- | ----------------------- | ---------------------------------------------------- |
| id             | string                  | 主键 - 消息唯一 `UUID`                               |
| conversationId | string                  | 普通索引、外键 - 所属会话 ID，关联 `conversation.id` |
| role           | `'user' \| 'assistant'` | 消息发送方                                           |
| content        | string                  | 消息正文内容                                         |
| createdAt      | Date                    | 普通索引 - 消息发送时间，`TypeORM` 自动填充          |

{% note info %}
**主键** 要求 **非空 + 整表唯一**，一张表只能有一个主键，用来精准定位单条数据。主键会自带 **主键索引**，按主键查询速度很快。
{% endnote %}

{% note info %}
**外键** 要求当前字段的值，**必须在关联表的主键中存在**，用于维护**多表之间的引用完整性**，外键可以有多个，不会自动创建索引。
{% endnote %}

{% note info %}
**索引** 是数据库为了优化查询额外建立的数据结构，可以避免全表扫描，**提升查询速度**，但会轻微降低写入性能，不校验数据是否合法。
{% endnote %}

**关联特性**

1. 实体关系：会话与消息为 **一对多** 关系，一个会话可包含多条消息，一条消息仅归属一个会话；
2. 关联字段：外键字段 `message.conversationId`，关联主表主键 `conversation.id`；
3. 级联行为：开启 **级联删除**，删除会话记录时，会自动删除该会话下所有关联的消息。

### 语义缓存

使用 `Redis` 存储用户提问的 **向量** 与大模型回答结果，避免语义相似的提问重复调用接口，加快响应速度。

**配置文件**

```env
REDIS_HOST=localhost
REDIS_PORT=6379
```

**存储结构**

| 字段   | 类型     | 说明                                                               |
| ------ | -------- | ------------------------------------------------------------------ |
| vector | number[] | 用户提问经过 `OpenAIEmbeddings` 生成的向量数组，用于余弦相似度计算 |
| answer | string   | 大模型返回的回答文本，余弦相似度达到阈值时直接复用该内容           |

{% note info %}
**向量模型** 可以把文本、图片、视频、音频等内容转换成 **多维数字数组**（即向量），语义相近的内容，在向量空间里距离更近，可用于语义匹配、RAG 检索、内容查重、相似内容推荐等场景。
{% endnote %}

### 模型配置

服务在顶层文件中统一配置了三类模型，**全局复用实例**，避免在子模块中重复初始化。

Agent 作为核心对话执行主体，基于对话模型 `chatModel` 运行，并挂载了视觉内容识别 `VisionTool` 和联网搜索 `SearchTool` 工具。

```typescript
private readonly agent = createAgent({
  model: this.chatModel,
  tools: [VisionTool(this.visionModel), SearchTool],
  systemPrompt: '请简洁明了地回答，关键信息完整，无需多余铺垫和解释。',
});
```

1. 对话模型 `chatModel`

用于 Agent 主体推理，处理普通文本对话、工具调用，根据系统提示词输出简洁完整的回答。

```typescript
private readonly chatModel = new ChatOpenAI({
  model: process.env.CHAT_MODEL,
  apiKey: process.env.CHAT_API_KEY,
  configuration: { baseURL: process.env.CHAT_BASE_URL },
  modelKwargs: { reasoning_effort: 'minimal' },
  temperature: 0.7,
});
```

{% note info %}
**LLM** 仅具备文本理解与生成能力，根据输入的上下文消息直接输出文本；**Agent** 以 **LLM** 作为 “大脑”，额外拥有 **自主决策、调用外部工具、循环执行任务** 的能力。
{% endnote %}

2. 视觉模型 `visionModel`

用于图片和视频内容识别，设置了较低的温度确保识图结果稳定。

```typescript
private readonly visionModel = new ChatOpenAI({
  model: process.env.VISION_MODEL,
  apiKey: process.env.VISION_API_KEY,
  configuration: { baseURL: process.env.VISION_BASE_URL },
  modelKwargs: { reasoning_effort: 'minimal' },
  temperature: 0.1,
});
```

{% note info %}
**temperature** 为模型温度参数，数值越高输出随机性、创造性越强；数值越低结果越稳定、可复现。识图场景使用较低温度，对话场景应适度提高温度确保表达自然。
{% endnote %}

3. 向量模型 `embeddingModel`

将用户提问文本转化为向量，用于余弦相似度计算，实现 **语义缓存**。

```typescript
private readonly embeddingModel = new OpenAIEmbeddings({
  model: process.env.EMBEDDING_MODEL,
  apiKey: process.env.EMBEDDING_API_KEY,
  configuration: { baseURL: process.env.EMBEDDING_BASE_URL },
});
```

{% note warning %}
仅在新会话（未传入 `conversationId`）启用语义缓存，多轮会话依赖历史消息，**不能脱离上下文单独判断语义**，因此不适合用向量语义缓存。
{% endnote %}

### 工具调用

Agent 会根据用户的提问自主判断是否调用工具函数。

系统内置了视觉识别工具 `VisionTool` 和联网搜索工具 `SearchTool`，工具函数使用 LangChain 的 `tool` 构造器结合 `Zod` 参数校验定义入参结构。

1. 视觉识别工具 `VisionTool`

用于解析图片、视频的媒体内容，当用户携带图片或视频链接提问时由 Agent 选择调用。

**入参结构**

| 字段  | 类型   | 说明                                    |
| ----- | ------ | --------------------------------------- |
| list  | Array  | 媒体资源数组                            |
| query | string | 内容识别指令，由 Agent 基于用户提问生成 |

`list` 数组单个对象的数据结构为：

| 字段      | 类型                 | 说明             |
| --------- | -------------------- | ---------------- |
| mediaType | `'image' \| 'video'` | 媒体类型         |
| mediaData | string               | 媒体资源的 `URL` |

**执行逻辑**

接收媒体资源数组与内容识别指令，组装为 **约定格式的消息** 后发送到预先注入的视觉模型 `visionModel`，返回媒体内容识别结果。

2. 联网搜索工具 `SearchTool`

**入参结构**

| 字段  | 类型   | 默认值 | 说明           |
| ----- | ------ | ------ | -------------- |
| query | string | -      | 搜索的关键词   |
| count | number | `10`   | 返回的结果条数 |

**执行逻辑**

通过 `axios` 请求第三方搜索接口，携带鉴权头部、检索词、摘要等参数，返回网页标题、链接、摘要等信息，经过 **序列化** 后返回给 Agent。

{% note info %}
Agent 会自主决策工具调用，只需提供 **入参结构** 即可，它会根据用户的问题，**自动生成** 符合规范的工具调用参数。
{% endnote %}

## 测试用例

![写入语义缓存](/img/captures/application/qq-bot/ch4-01.jpg)
![语义缓存命中](/img/captures/application/qq-bot/ch4-02.jpg)
![图片识别](/img/captures/application/qq-bot/ch4-03.jpg)
![多轮会话](/img/captures/application/qq-bot/ch4-04.jpg)
![视频识别](/img/captures/application/qq-bot/ch4-05.jpg)
![联网搜索](/img/captures/application/qq-bot/ch4-06.jpg)
![会话存储](/img/captures/application/qq-bot/ch4-07.jpg)
![消息存储](/img/captures/application/qq-bot/ch4-08.jpg)
![向量缓存](/img/captures/application/qq-bot/ch4-09.jpg)

# 相关链接

1. [NapCatQQ - 官方文档](https://napneko.github.io/)
2. [OneBot - 官方文档](https://onebot.dev/)
3. [OneBot Server - GitHub](https://github.com/Ryna755467/OneBot-Server)
4. [LLM API Server - GitHub](https://github.com/Ryna755467/LLM-API-Server)

---

{% series application %}
