---
title: 虚拟桌面
date: 2026-08-02
updated: 2026-08-02
top_img: /img/series/traces.jpg
cover: /img/covers/traces/virtual-desktop.jpg
series: traces
categories:
  - 应用实例
---

## 快速阅读

记录大一下学期做过的一个虚拟桌面软件。

### 功能概述

基于 Java Swing 实现的一个简易系统平台，可以通过类文件的方式嵌入其他程序，关闭后会保存到系统托盘。

- 桌面顶部的导航栏可以提供适配当前获取焦点的内嵌程序的快捷操作按钮
- 桌面空白区域右键点击可以修改壁纸以及对内嵌程序的快捷操作

内嵌的便笺程序完成了设置字型、颜色、换行等通用功能，桌面顶部的导航栏可以对获取焦点的便笺进行快捷操作。便笺还实现了拖拽功能，可以从外部拖动文件或字符串，也可以用文件选择器打开文件并粘贴内容到便笺中。

### 核心节点

Swing 提供的控件有些情况不符合预期效果，部分区域（例如右键菜单栏等）需要编写自定义控件。

{% note modern no-icon %}
应该还可以实现联网功能：在类文件中用 `socket` 和远程主机建立连接并返回连接状态，这样就可以接入一些联机程序，比如多人小游戏？
{% endnote %}

### 运行效果

![便笺-打开文件功能](/img/captures/virtual-desktop/ch1-01.jpg)
![便笺-顶部导航栏](/img/captures/virtual-desktop/ch1-02.jpg)
![便笺-右键菜单栏](/img/captures/virtual-desktop/ch1-03.jpg)
![桌面-修改壁纸功能](/img/captures/virtual-desktop/ch1-04.jpg)
![桌面-控制功能](/img/captures/virtual-desktop/ch1-05.jpg)

---

{% series traces %}
