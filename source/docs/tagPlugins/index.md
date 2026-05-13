---
title: 标签插件
date: 2026-05-13
---

## Note

{% tabs note %}

<!-- tab 配置项 -->

> 参考链接 - [Note (Bootstrap Callout)](https://butterfly.js.org/posts/ceeb73f/?highlight=tabs#Note-Bootstrap-Callout)

```yaml
# _config.butterfly.yml

note:
  style: flat # 标签样式
  icons: true # 是否显示图标
  border_radius: 3 # 边框圆角半径
  light_bg_offset: 0 # 背景色偏移量
```

<!-- endtab -->

<!-- tab 用法1 -->

> 语法框架

```markdown
{% note [class] [no-icon] [style] %}
Any content (support inline tags too.io).
{% endnote %}
```

> 使用示例

```markdown
{% note default %}
DEFAULT
{% endnote %}

{% note primary %}
PRIMARY
{% endnote %}

{% note success %}
SUCCESS
{% endnote %}

{% note info %}
INFO
{% endnote %}

{% note warning %}
WARNING
{% endnote %}

{% note danger %}
DANGER
{% endnote %}
```

> 渲染效果

{% note default %}
DEFAULT
{% endnote %}

{% note primary %}
PRIMARY
{% endnote %}

{% note success %}
SUCCESS
{% endnote %}

{% note info %}
INFO
{% endnote %}

{% note warning %}
WARNING
{% endnote %}

{% note danger %}
DANGER
{% endnote %}

<!-- endtab -->

<!-- tab 用法2 -->

> 语法框架

```markdown
{% note [color] [icon] [style] %}
Any content (support inline tags too.io).
{% endnote %}
```

> 使用示例

```markdown
{% note default 'fab fa-cc-visa' %}
你是刷 Visa 还是 UnionPay
{% endnote %}

{% note pink 'fas fa-car-crash' %}
小心开车 安全至上
{% endnote %}

{% note red 'fas fa-fan' %}
这是三片呢？还是四片？
{% endnote %}

{% note purple 'far fa-hand-scissors' modern %}
剪刀石头布
{% endnote %}

{% note orange 'fas fa-battery-half' modern %}
电量剩余一半
{% endnote %}

{% note green 'fab fa-internet-explorer' modern %}
前端最讨厌的浏览器
{% endnote %}
```

> 渲染效果

{% note default 'fab fa-cc-visa' %}
你是刷 Visa 还是 UnionPay
{% endnote %}

{% note pink 'fas fa-car-crash' %}
小心开车 安全至上
{% endnote %}

{% note red 'fas fa-fan' %}
这是三片呢？还是四片？
{% endnote %}

{% note purple 'far fa-hand-scissors' modern %}
剪刀石头布
{% endnote %}

{% note orange 'fas fa-battery-half' modern %}
电量剩余一半
{% endnote %}

{% note green 'fab fa-internet-explorer' modern %}
前端最讨厌的浏览器
{% endnote %}

<!-- endtab -->

{% endtabs %}
