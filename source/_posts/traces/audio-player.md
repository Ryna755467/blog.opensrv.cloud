---
title: 音频播放器
date: 2026-09-01
updated: 2026-09-01
top_img: /img/series/traces.jpg
cover: /img/covers/traces/audio-player.jpg
series: traces
categories:
  - 程序设计
---

## 快速阅读

记录大一寒假做过的一个音频播放器软件。

### 功能概述

基于 C语言 和 EasyX图形库 实现的音频播放器，完成了音频控制、轮播、搜索、收藏等功能。

**界面划分**

1. 上栏：包括时间显示和搜索框。
2. 侧栏：包括主页、最近播放序列、最近更新序列、音频库 ABCD。
3. 主栏：显示选定的页面内容。
4. 下栏：包括控制区、进度条、轮播设置。

### 核心节点

这是用绘图代码制作的应用程序，所有的视觉元素和视觉效果都是通过在给定坐标处重复绘制基本几何体实现的，由于绘图代码没有控件，时间轴、拖动条和所有的悬停点击交互效果都需要单独设计对应的绘图逻辑。

{% note info %}
这种程序的优点是自由度高，可以严格实现自己的想要的效果，缺点是代码量大，很难实现平滑的过渡动画。
{% endnote %}

### 运行效果

![检索面板](/img/captures/audio-player/ch1-01.jpg)
![播放模式](/img/captures/audio-player/ch1-02.jpg)
![更新记录](/img/captures/audio-player/ch1-03.jpg)
![系列页面](/img/captures/audio-player/ch1-04.jpg)
![主页面](/img/captures/audio-player/ch1-05.jpg)
![自定义条目](/img/captures/audio-player/ch1-06.jpg)

## 实现过程

{% timeline 2021 年, blue %}

<!-- timeline 1 月 27 日 -->

一个基于图形库的音频播放器，看看多久可以做完...

{% gallery %}
![](/img/captures/audio-player/ch2-01.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 1 月 30 日 -->

为什么别人的掩码图可以达到目的，而我的掩码图却没有效果，起码可以证明单掩码思路是对的吧？
两天时间，分别去做局部滚动栏和mtv，结果是都没有解决emmm...

{% gallery %}
![](/img/captures/audio-player/ch2-02.jpg)
![](/img/captures/audio-player/ch2-03.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 2 月 3 日 -->

关于局部动画的算法

本来是想用掩码图来达到目的，然后就花了几天时间各种博客贴吧官网查资料，算法是很明确，但代码的参数还是搞不准（因为CSDN的资料是很多年前的，差不多算过期版本） 睡觉的时候突然就想到解决办法了，爬起来测试过之后效果和预期完全一致。

只需要调整一下绘图顺序，简单说就是先全局绘图，然后用背景色绘制一个填充矩形来覆盖需要隐藏的部分，接下来再一个模块一个模块开始绘制，这样视觉上就达到了局部动画的效果，被掩码误导了几天...

{% gallery %}
![](/img/captures/audio-player/ch2-04.jpg)
![](/img/captures/audio-player/ch2-05.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 2 月 6 日 -->

滚动条部分，总算是解决了...

{% gallery %}
![](/img/captures/audio-player/ch2-06.jpg)
![](/img/captures/audio-player/ch2-07.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 3 月 1 日 -->

播放模式与历史记录

{% gallery %}
![](/img/captures/audio-player/ch2-08.jpg)
![](/img/captures/audio-player/ch2-09.jpg)
{% endgallery %}

<!-- endtimeline -->

<!-- timeline 3 月 7 日 -->

花了一个月时间才完成，读档存档的相关代码没生效，应该是版本有区别...

对播放器而言很重要的精确计时，原先是通过特殊方法测定系统延迟，然后再以精度为微秒的增量赋值给时间变量...
在做上栏实时钟表的时候突然想到用系统时间的变动作为信号传递给时间变量 - 这样也会有误差（小于等于1秒），如果系统秒数变动临界的时候激活计时，会立即返回一个信号使得时间增量增加1秒，但是设定3秒播放真空就可以完全抵消产生的影响，这样直接避免了由于不同面板系统误差不同造成的计时错误。

拖动条的算法原先是一个无限循环，以按键弹起为信号跳出，这样必须在循环里绘制整个画面，后来通过定义全局变量设计的另一个算法就简化了很多。

最复杂的是自定义栏，因为输入和删除都涉及相关信息的整体变动...
还有像以按键弹起为信号的结果被无限次执行等好多bug，都已经修正了。

{% gallery %}
![](/img/captures/audio-player/ch2-10.png)
![](/img/captures/audio-player/ch2-11.png)
![](/img/captures/audio-player/ch2-12.jpg)
![](/img/captures/audio-player/ch2-13.jpg)
![](/img/captures/audio-player/ch2-14.jpg)
![](/img/captures/audio-player/ch2-15.jpg)
![](/img/captures/audio-player/ch2-16.jpg)
![](/img/captures/audio-player/ch2-17.jpg)
![](/img/captures/audio-player/ch2-18.jpg)
{% endgallery %}

<!-- endtimeline -->

{% endtimeline %}

## 备注信息

1. Save栏
   - 滚动栏图片可以直接在PNG文件夹同名更替。
   - 鼠标位于滚动栏左侧暂停滚动，位于右侧并左键点击可无延迟切换，正常状态的切换延迟设定为720（启动时为1440）。
   - 两个Custom区域为自定义栏，以左键弹起为信号激活键盘输入，以右键弹起为信号删除已添加信息，每个自定义栏元素上限分别为998。

2. Recent栏
   - 每次播放的信息会被收录到这里。
   - 排列顺序为倒序，即最近播放的排在最后前，元素上限为998。

3. 控制栏
   - 对于左下侧三个图标，在响应区域内按左键以激活，以左键弹起为信号执行对应的结果。
   - 在右下侧设置播放模式。

4. 搜索栏
   - 输入格式：英文单词间有间隔，采用正常拼写方式，中文字符间无间隔，采用驼峰拼写方式，第一字符为大写识别为英文，第一字符为小写识别为中文，例如：追光者 - 应输入：zhuiGuangZhe；Good Goodbye - 应输入：Good Goodbye。
   - 搜索结果面板：左键点按关闭并清除信息，以清除搜索结果，否则不清除当前信息；左键点按保存到音频序列，若为序号类型为L，会被保存到自定义B栏，若为S则保存到自定义A栏。

5. 其他
   - 鼠标滑轮滚动速度为2，拖动条滚动速度为4。
   - 时间轴的误差小于等于1秒，误差来源于系统临界时刻变动，但播放默认为3秒，不会产生影响。
   - PNG文件夹有两个，应用文件夹ID为PNG（另一个ID为PNG B），可更改ID以替换应用。
   - 文件夹内置图片可以替换为相同ID的任意图片，同理MP3文件夹资源也可自行替换。

---

{% series traces %}
