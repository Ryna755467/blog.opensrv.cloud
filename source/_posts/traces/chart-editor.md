---
title: 图表编辑器
date: 2026-08-05
updated: 2026-08-05
top_img: /img/series/traces.jpg
cover: /img/covers/traces/chart-editor.jpg
series: traces
categories:
  - 应用实例
---

## 快速阅读

记录上学时做过的一个图表编辑器软件。

### 功能概述

基于 ECharts 实现的图表编辑器，通过点击按钮的方式选择相关配置项，自动编码并生成图表。

1. 对于主要配置项提供不同类型的交互按钮：

- 输入框：data 类需要手动输入的数据，会自动错误检测。
- 单选框：有限个固定值的配置项。
- 多选框：可选用的工具类组件。

2. 对于次要配置项，提供编码预览功能，可以在预览界面添加次要配置项代码。

### 核心节点

页面双向锚定、自适应图表尺寸、配置项转化为 JavaScript 代码以及错误检测的过程实现。

### 运行效果

![主界面-柱状图](/img/captures/chart-editor/ch1-01.jpg)
![配置项-堆叠面积图通用配置](/img/captures/chart-editor/ch1-02.jpg)
![配置项-堆叠面积图数据系列](/img/captures/chart-editor/ch1-03.jpg)
![编码预览-堆叠面积图](/img/captures/chart-editor/ch1-04.jpg)
![生成图表-堆叠面积图折线图类型](/img/captures/chart-editor/ch1-05.jpg)
![生成图表-堆叠面积图柱状图类型](/img/captures/chart-editor/ch1-06.jpg)

---

{% series traces %}
