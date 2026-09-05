---
title: 媒体播放器
date: 2026-06-01
updated: 2026-06-01
top_img: /img/series/traces.jpg
cover: /img/covers/traces/media-player.jpg
series: traces
categories:
  - 程序设计
---

## 快速阅读

### 功能概述

这是一个媒体播放器，用于播放视频和音频，可以修改播放速度、显示歌词、修改轮播逻辑、切换主题、缩放页面。

**界面划分**

1. 上栏：包括白天/夜间模式切换、时间显示、媒体搜索框、消息提醒。
2. 侧栏：包括主页、电影页、MV 页、音频页、自定义页、资源库 ABC。
3. 主栏：显示选中的页面内容。
4. 下栏：包括控制区、进度条、功能区。

**按键控制**

- `W` `A` `S`：显示 / 隐藏上、侧、下栏
- `D`：进入 / 退出主栏全屏
- `←` `→`：播放回退 / 播放快进
- `↑` `↓`：切换到上一个页面 / 下一个页面
- `F`：进入 / 退出画中画
- `G`：进入 / 退出视频全屏
- `Q`：进入 / 退出缩放屏状态
- `R`：打开 / 关闭主题窗口
- `T`：打开 / 关闭模式窗口
- `V`：打开 / 关闭词条窗口
- `B`：打开 / 关闭视频窗口
- `1` `2` `3` `4` `5` `6` `7` `8`：切换到第 1 / 2 / 3 / 4 / 5 / 6 / 7 / 8 个页面

### 核心节点

1. 轮播图：保存元素序列到数组，所有的元素横向排列，通过设定整体偏移距离实现轮播图效果。
2. 缩略图：设置页面栈保存当前打开的页面，对不同的页面数进行不同方式的排版。
3. 滑动栏：设定两个按键左移和右移，改变元素的位置和层序，设定标签栈保存当前的元素序列。
4. 搜索框：搜索框获得焦点时按下 `Enter` 键查找媒体，根据后缀判断资源类型，打开对应的媒体界面，输入有误时清空内容并闪烁。
5. 词条控制：播放媒体时自动更新词条的内容，对每行绑定点击事件为修改当前媒体进度。
6. 自定义窗体：对拖拽事件和对象定义一系列函数，确保可拖拽的对象能被正确归并。
7. 画中画：修正画中画 API 在此程序中视觉效果不合理以及进入退出画中画时媒体资源设置不同步的问题，增加对画中画、全屏与其他状态切换的限定条件。

### 运行效果

![主界面](/img/captures/media-player/ch1-01.jpg)
![检索面板](/img/captures/media-player/ch1-02.jpg)
![倍速播放](/img/captures/media-player/ch1-03.jpg)
![电影页面](/img/captures/media-player/ch1-04.jpg)
![播放模式](/img/captures/media-player/ch1-05.jpg)
![视频页面](/img/captures/media-player/ch1-06.jpg)
![系列菜单](/img/captures/media-player/ch1-07.jpg)
![音频页面](/img/captures/media-player/ch1-08.jpg)
![自定义页面 - 视频栏目](/img/captures/media-player/ch1-09.jpg)
![自定义页面 - 音频栏目](/img/captures/media-player/ch1-10.jpg)
![画中画播放](/img/captures/media-player/ch1-11.jpg)
![视频播放](/img/captures/media-player/ch1-12.jpg)
![缩略图](/img/captures/media-player/ch1-13.jpg)
![歌词面板](/img/captures/media-player/ch1-14.jpg)

## 实现过程

{% timeline 2021 年, blue %}

<!-- timeline 8 月 1 日 -->

C语言做得比较差，用前端语言重写一次，反正也没什么新构思...

{% gallery %}
![](/img/captures/media-player/ch2-01.jpg)
![](/img/captures/media-player/ch2-03.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 2 日 -->

启动UI动画 + 布局：控制栏、词条、模式、主题 + 主栏、侧栏动态缩放

{% gallery %}
![](/img/captures/media-player/ch2-04.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 6 日 -->

切换动画 + 分屏功能（一二三四分屏）

{% gallery %}
![](/img/captures/media-player/ch2-05.jpg)
![](/img/captures/media-player/ch2-06.jpg)
![](/img/captures/media-player/ch2-07.jpg)
![](/img/captures/media-player/ch2-08.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 10 日 -->

窗口缩放 + 画中画分屏

{% gallery %}
![](/img/captures/media-player/ch2-09.jpg)
![](/img/captures/media-player/ch2-10.jpg)
![](/img/captures/media-player/ch2-11.jpg)
![](/img/captures/media-player/ch2-12.jpg)
![](/img/captures/media-player/ch2-13.jpg)
![](/img/captures/media-player/ch2-14.jpg)
![](/img/captures/media-player/ch2-15.jpg)
![](/img/captures/media-player/ch2-16.jpg)
![](/img/captures/media-player/ch2-17.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 13 日 -->

三天做了主体八分之一的六分之一…

{% gallery %}
![](/img/captures/media-player/ch2-18.jpg)
![](/img/captures/media-player/ch2-19.jpg)
![](/img/captures/media-player/ch2-20.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 16 日 -->

背景图动态切换和四个主题

{% gallery %}
![](/img/captures/media-player/ch2-21.jpg)
![](/img/captures/media-player/ch2-22.jpg)
![](/img/captures/media-player/ch2-23.jpg)
![](/img/captures/media-player/ch2-24.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 20 日 -->

响应式布局

{% gallery %}
![](/img/captures/media-player/ch2-25.jpg)
![](/img/captures/media-player/ch2-26.jpg)
![](/img/captures/media-player/ch2-27.jpg)
![](/img/captures/media-player/ch2-28.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 8 月 25 日 -->

视频和音频栏

{% gallery %}
![](/img/captures/media-player/ch2-29.jpg)
![](/img/captures/media-player/ch2-30.jpg)
![](/img/captures/media-player/ch2-31.jpg)
![](/img/captures/media-player/ch2-32.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 9 月 4 日 -->

试着录了两首歌的歌词

{% gallery %}
![](/img/captures/media-player/ch2-33.jpg)
![](/img/captures/media-player/ch2-34.jpg)
![](/img/captures/media-player/ch2-35.jpg)
![](/img/captures/media-player/ch2-36.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 10 月 20 日 -->

播放速率、时间轴、搜索栏、自定义栏

{% gallery %}
![](/img/captures/media-player/ch2-37.jpg)
![](/img/captures/media-player/ch2-38.jpg)
![](/img/captures/media-player/ch2-39.jpg)
![](/img/captures/media-player/ch2-40.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 10 月 25 日 -->

终于做完了，动画效果还是超出开始的预期的，毕竟6月那时候只是想复刻...

{% gallery %}
![](/img/captures/media-player/ch2-41.jpg)
![](/img/captures/media-player/ch2-42.jpg)
![](/img/captures/media-player/ch2-43.jpg)
![](/img/captures/media-player/ch2-44.jpg)
![](/img/captures/media-player/ch2-45.jpg)
![](/img/captures/media-player/ch2-46.jpg)
![](/img/captures/media-player/ch2-47.jpg)
![](/img/captures/media-player/ch2-48.jpg)
![](/img/captures/media-player/ch2-49.jpg)
{% endgallery %}

<!-- endtimeline -->

{% endtimeline %}
