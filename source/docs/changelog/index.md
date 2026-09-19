---
title: 更新日志
date: 2026-06-01
updated: 2026-09-17
top_img: /img/series/default.jpg
toc: false
---

## 变更记录

### v1.1.9

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

### v1.1.8

**build: 新增站点地图**
{% note modern no-icon %}

- 新增 `sitemap.xml` 站点地图
- 修改 Adsense 发布商 ID
  {% endnote %}

{% note info %}
**站点地图** 是一份 XML 格式的网页链接清单，记录了网站内所有页面的网址、最后修改时间等信息，专门提供给搜索引擎爬虫读取，加快网页收录，同时便于在 GSC 查看抓取与收录状态。
{% endnote %}

### v1.1.7

**feat: 新增谷歌 GSC 网站验证**
{% note modern no-icon %}

- 修改更新日志展现样式
- 修改收藏页面按钮排序
- 新增谷歌 GSC 网站验证
- 新增谷歌 Adsense 自动广告
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

### v1.1.6

**docs: 更新日志增加提交前缀规范**
{% note modern no-icon %}

- 理科散文相关文档
- MD 参考文档增加 Mermaid 图表示例
- 更新日志增加提交前缀规范
- 目录结构调整
  {% endnote %}

### v1.1.5

**refactor: 规范文档结构**
{% note modern no-icon %}

- 规范文档增加 Mermaid、Timeline 标签外挂
- 修改 docs 目录结构，文章访问路径统一使用文件夹模式
- 图片资源统一存放在根目录 /img 文件夹，使用绝对路径引入
  {% endnote %}

### v1.1.4

**feat: 新增 Giscus 评论**
{% note modern no-icon %}

- 新增 Giscus 评论
  {% endnote %}

### v1.1.3

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

### v1.1.2

**fix: 修复静态资源浏览器缓存不更新问题**
{% note modern no-icon %}

- 侧边栏增加社交图标
- 静态资源 URL 增加 hash 查询参数
- MD 参考文档
  {% endnote %}

### v1.1.1

**feat: 新增系列文章和最新文章卡片**
{% note modern no-icon %}

- 文章封面图调整
- 应用实例相关文档
- 侧边栏增加系列文章和最新文章卡片
  {% endnote %}

### v1.1.0

**docs: 程序设计相关文档**
{% note modern no-icon %}

- 目录结构调整
- 程序设计相关文档
  {% endnote %}

### v1.0.9

**docs: 算法原理与程序设计文档**
{% note modern no-icon %}

- 算法原理相关文档
- 程序设计相关文档
- 规范文档补充内容
  {% endnote %}

### v1.0.8

**docs: 规范文档补充内容**
{% note modern no-icon %}

- 规范文档补充内容
  {% endnote %}

### v1.0.6

**docs: 写作规范**
{% note modern no-icon %}

- 新增写作规范
  {% endnote %}

### v1.0.5

**refactor: 目录结构调整**
{% note modern no-icon %}

- 目录结构调整
- 文档排版优化
  {% endnote %}

### v1.0.4

**docs: JavaScript 相关文档**
{% note modern no-icon %}

- 新增简繁转换按钮
- 新增页面锚点
- JavaScript 相关文档
  {% endnote %}

### v1.0.3

**feat: 主题配置**
{% note modern no-icon %}

- Butterfly 相关主题配置
- 新增数学公式和流程图渲染
  {% endnote %}

### v1.0.2

**fix: 修复隐式依赖问题**
{% note modern no-icon %}

- 包管理器替换为默认的 yarn
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
