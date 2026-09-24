---
title: 更新日志
date: 2026-06-01
updated: 2026-09-24
top_img: /img/series/default.jpg
toc: false
---

## 变更记录

### v1.2.1 - 2026.09.24

**docs: QQ Bot 应用实例文档**
{% note modern no-icon %}

- 新增 QQ Bot 应用实例文档
  {% endnote %}

### v1.2.0 - 2026.09.20

**feat: 新增谷歌分析 GA4 代码埋点**
{% note modern no-icon %}

- 更新日志增加时间节点
- 新增谷歌分析 GA4 代码埋点
- 站内链接统一修改为 **根相对路径**
- 写作规范增加 `Button` 标签外挂
- 新增侧边栏最新评论卡片
- 修改文末分页逻辑
  {% endnote %}

{% note info %}
**GSC** 由搜索引擎爬虫采集站点数据，用于监控网站抓取、页面收录与搜索曝光；
**GA4** 由访客浏览器主动上报数据，用于分析用户在站点内的浏览与交互行为。
{% endnote %}

{% note primary no-icon %}
**链接路径的写法**

1. 绝对路径 `https://xxx.com/path/`
   固定链接，更换目录或域名时需要全部替换。

2. 根相对路径 `/path/`
   从域名根目录开始寻址，不受当前页面位置影响。

3. 文档相对路径 `./path/`
   `./` 代表当前目录，`../` 代表上一级目录，可以省略 `./`，例如 `path/`。

4. 协议相对路径 `//xxx.com/path/`
   自动复用当前页面的 `http` / `https` 协议头。
   {% endnote %}

### v1.1.9 - 2026.09.19

**build: 修改 Nginx 缓存策略**
{% note modern no-icon %}

- Nginx 缓存策略从 **强缓存 + Hash 参数** 改为 **全量协商缓存**
- 移除 hexo-hash 插件
- 修复 LaTeX 字符报错问题
- 增加 `ads.txt` 声明文件
  {% endnote %}

{% note info %}
**强缓存 + Hash 参数** `?v=xxx` 可以避免新文件命中浏览器本地缓存，但需要构建流程支持资源哈希；**全量协商缓存** 每次访问页面都会发送校验请求，对服务器资源有更大的压力。
{% endnote %}

### v1.1.8 - 2026.09.18

**build: 新增站点地图**
{% note modern no-icon %}

- 新增 `sitemap.xml` 站点地图
- 修改 Adsense 发布商 ID
  {% endnote %}

{% note info %}
**站点地图** 是一份 XML 格式的网页链接清单，记录了网站内所有页面的网址、最后修改时间等信息，专门提供给搜索引擎爬虫读取，加快网页收录，同时便于在 GSC 查看抓取与收录状态。
{% endnote %}

### v1.1.7 - 2026.09.18

**feat: 新增谷歌搜索 GSC 网站验证**
{% note modern no-icon %}

- 修改更新日志展现样式
- 修改收藏页面按钮排序
- 新增谷歌搜索 GSC 网站验证
- 新增 Google AdSense 自动广告
  {% endnote %}

{% note info %}
**网站验证** 用于确认网站的所有权。获取搜索引擎平台提供的验证标识后，将其嵌入到网站的 HTML 文本中，然后在平台提交验证请求，通过后就可以访问网站的后台管理数据，例如曝光、点击、关键词展现等。
{% endnote %}

{% note warning %}
**网站验证** 不是被搜索引擎收录的前提条件，就算不做验证，搜索引擎爬虫也会自发查找并收录陌生网站。
{% endnote %}

{% note info %}
**自动广告** 是 AdSense 提供的 AI 自动投放方案。在网页中嵌入一段 JS 代码，它会自动扫描页面结构、内容长度、设备类型，由机器学习算法自主选择广告位置与广告样式，无需手动在文章内逐个插入广告单元。
{% endnote %}

{% note warning %}
**自动广告** 出现的位置由 AI 算法决定，无法精准控制，页面刷新时有概率发生变化。
{% endnote %}

### v1.1.6 - 2026.09.17

**docs: 更新日志增加提交前缀规范**
{% note modern no-icon %}

- 理科散文相关文档
- MD 参考文档增加 Mermaid 图表示例
- 更新日志增加提交前缀规范
- 目录结构调整
  {% endnote %}

### v1.1.5 - 2026.09.14

**refactor: 规范文档结构**
{% note modern no-icon %}

- 规范文档增加 Mermaid、Timeline 标签外挂
- 修改 docs 目录结构，文章访问路径统一使用文件夹模式
- 图片资源统一存放在根目录 /img 文件夹，使用绝对路径引入
  {% endnote %}

### v1.1.4 - 2026.09.13

**feat: 新增 Giscus 评论**
{% note modern no-icon %}

- 新增 Giscus 评论
  {% endnote %}

### v1.1.3 - 2026.09.13

**feat: 新增系统功能**
{% note modern no-icon %}

- 新增顶部加载进度条
- 新增文章字数统计功能
- 新增文字左右对齐功能
- 新增图片大图查看模式
- 新增收藏页面列表
- 新增本地搜索功能
- 新增底部评论功能
- 新增 404 页面
  {% endnote %}

### v1.1.2 - 2026.09.12

**fix: 修复静态资源浏览器缓存不更新问题**
{% note modern no-icon %}

- 侧边栏增加社交图标
- 静态资源 URL 增加 hash 查询参数
- MD 参考文档
  {% endnote %}

### v1.1.1 - 2026.09.10

**feat: 新增系列文章和最新文章卡片**
{% note modern no-icon %}

- 文章封面图调整
- 应用实例相关文档
- 侧边栏增加系列文章和最新文章卡片
  {% endnote %}

### v1.1.0 - 2026.09.08

**docs: 程序设计相关文档**
{% note modern no-icon %}

- 目录结构调整
- 程序设计相关文档
  {% endnote %}

### v1.0.9 - 2026.09.06

**docs: 算法原理与程序设计文档**
{% note modern no-icon %}

- 算法原理相关文档
- 程序设计相关文档
- 规范文档补充内容
  {% endnote %}

### v1.0.8 - 2026.08.30

**docs: 规范文档补充内容**
{% note modern no-icon %}

- 规范文档补充内容
  {% endnote %}

### v1.0.7 - 2026.08.29

**docs: 修改内容**
{% note modern no-icon %}

- 修改写作规范
  {% endnote %}

### v1.0.6 - 2026.08.29

**docs: 写作规范**
{% note modern no-icon %}

- 新增写作规范
  {% endnote %}

### v1.0.5 - 2026.08.27

**refactor: 目录结构调整**
{% note modern no-icon %}

- 目录结构调整
- 文档排版优化
  {% endnote %}

### v1.0.4 - 2026.08.27

**docs: JavaScript 相关文档**
{% note modern no-icon %}

- 新增简繁转换按钮
- 新增页面锚点
- JavaScript 相关文档
  {% endnote %}

### v1.0.3 - 2026.05.15

**feat: 主题配置**
{% note modern no-icon %}

- Butterfly 相关主题配置
- 新增数学公式和流程图渲染
  {% endnote %}

### v1.0.2 - 2026.05.13

**fix: 修复隐式依赖问题**
{% note modern no-icon %}

- 包管理器替换为默认的 yarn
  {% endnote %}

### v1.0.1 - 2026.05.12

**feat: Butterfly 主题**
{% note modern no-icon %}

- 引入 Butterfly 主题
  {% endnote %}

### v1.0.0 - 2026.05.11

**build: 初始化结构**
{% note modern no-icon %}

- 初始化结构
- 目录划分
  {% endnote %}

## 提交规范

{% note info %}
feat: 新增功能
{% endnote %}

{% note info %}
docs: 新增文档
{% endnote %}

{% note warning %}
fix: 修复 Bug
{% endnote %}

{% note warning %}
refactor: 结构调整
{% endnote %}

{% note success %}
perf: 性能优化
{% endnote %}

{% note success %}
build: 系统构建
{% endnote %}

{% note default %}
chore: 其他杂项
{% endnote %}
