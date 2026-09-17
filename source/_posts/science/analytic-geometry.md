---
title: 解析几何
date: 2026-09-17
updated: 2026-09-17
top_img: /img/series/science.jpg
cover: /img/covers/science/analytic-geometry.jpg
series: science
categories:
  - 理科散文
katex: true
---

## 内容概要

写于 2020 年 5 月，文章介绍了一个关于高中数学 **解析几何** 相关题目的 **快速解法**。

此方法源于一本课外书上的速解公式，核心意义在于快速计算，能够避免在过于冗长的推导过程中写错某个参数导致整个计算过程出错。

原公式是用 $Ax+By$ 直线式联立椭圆方程解出的，有些复杂。我做了一些优化，用 $y=kx+m$ 联立，并扩展了抛物线公式，得出的结论经过化简后很容易记忆，所有解析几何类题目都可以使用。

## 推导过程

### 椭圆

以椭圆方程为例，推导过程如下：

{% note modern no-icon %}
联立

$$
\begin{cases}
\dfrac{x^2}{a^2}+\dfrac{y^2}{b^2}=1 \\
y=kx+m \quad(\text{如果是 }y=kx-k，\text{那么 }m=-k)
\end{cases}
$$

得

$$
(a^2k^2+b^2)x^2+2a^2kmx+a^2m^2-a^2b^2=0
$$

$$
\Delta=4a^2b^2(a^2k^2+b^2-m^2)>0
$$

由韦达定理得

$$
\begin{cases}
x_1+x_2=\dfrac{-2a^2km}{a^2k^2+b^2} \\
x_1\cdot x_2=\dfrac{a^2m^2-a^2b^2}{a^2k^2+b^2}
\end{cases}
$$

联立 $y=kx+m$，得

$$
\begin{cases}
y_1+y_2=\dfrac{2b^2m}{a^2k^2+b^2} \\
y_1\cdot y_2=\dfrac{b^2m^2-a^2b^2k^2}{a^2k^2+b^2}
\end{cases}
$$

$$
\begin{aligned}
|AB|&=\sqrt{1+k^2}|x_1-x_2|=\sqrt{1+k^2}\sqrt{(x_1+x_2)^2-4x_1x_2} \\
&=\dfrac{2ab\sqrt{1+k^2}\sqrt{a^2k^2+b^2-m^2}}{a^2k^2+b^2}
\end{aligned}
$$

$$
\begin{aligned}
S_{\triangle AOB}&=\dfrac12 d\cdot|AB|=\dfrac12\cdot\dfrac{|m|}{\sqrt{1+k^2}}\cdot 2ab\sqrt{1+k^2}\dfrac{\sqrt{a^2k^2+b^2-m^2}}{a^2k^2+b^2} \\
&=|abm|\dfrac{\sqrt{a^2k^2+b^2-m^2}}{a^2k^2+b^2}
\end{aligned}
$$

{% endnote %}

得到的快速计算公式有：

- $\Delta$
- $x_1+x_2/x_1\cdot x_2$
- $y_1+y_2/y_1\cdot y_2$
- $|AB|$
- $S_{\triangle AOB}$

### 抛物线

对于抛物线也有一套计算公式，可以联立

$$
\begin{cases}
y^2=cx \\
y=kx+m
\end{cases}
$$

按同样步骤得出上述结论。

### 双曲线

双曲线和椭圆的推导过程是一样的，只需要把 $b^2$ 替换为 $-b^2$ 就可以了，化简后公式的正负符号会发生改变。

## 结论整理

### 椭圆

对于

$$
\begin{cases}
\dfrac{x^2}{a^2}+\dfrac{y^2}{b^2}=1 \\
y=kx+m
\end{cases}
$$

联立得到

$$
(a^2k^2+b^2)x^2+2a^2kmx+a^2m^2-a^2b^2=0
$$

$$
\Delta=4a^2b^2(a^2k^2+b^2-m^2) \quad (>0)
$$

$$
\begin{cases}
x_1+x_2=\dfrac{-2a^2km}{a^2k^2+b^2}\\[6pt]
x_1x_2=\dfrac{a^2m^2-a^2b^2}{a^2k^2+b^2}
\end{cases}
\quad
\begin{cases}
y_1+y_2=\dfrac{2b^2m}{a^2k^2+b^2}\\[6pt]
y_1y_2=\dfrac{b^2m^2-a^2b^2k^2}{a^2k^2+b^2}
\end{cases}
$$

$$
|AB|=2ab\sqrt{1+k^2}\cdot \dfrac{\sqrt{a^2k^2+b^2-m^2}}{a^2k^2+b^2}
$$

$$
x_1y_2+x_2y_1=\dfrac{-2a^2b^2k}{a^2k^2+b^2}
$$

$$
S_{\triangle ABO}=|abm|\cdot \dfrac{\sqrt{a^2k^2+b^2-m^2}}{a^2k^2+b^2}
$$

### 抛物线

**$y^2=cx$ 类型**

对于

$$
\begin{cases}
y^2=cx \\
y=kx+m
\end{cases}
$$

联立得到

$$
k^2x^2+(2km-c)x+m^2=0
$$

$$
\Delta = c^2-4kcm
$$

$$
\begin{cases}
x_1+x_2=\dfrac{c-2km}{k^2}\\[6pt]
x_1x_2=\dfrac{m^2}{k^2}
\end{cases}
\quad
\begin{cases}
y_1+y_2=\dfrac{c}{k}\\[6pt]
y_1y_2=\dfrac{cm}{k}
\end{cases}
$$

$$
|AB|=\sqrt{1+k^2}\cdot \frac{\sqrt{c^2-4kcm}}{k^2}
$$

$$
x_1y_2+x_2y_1=\frac{cm}{k}
$$

$$
S_{\triangle ABO}= \frac{|m|}{2}\cdot \frac{\sqrt{c^2-4kcm}}{k^2}
$$

$$
(S=\dfrac12 dl=\dfrac12 \dfrac{|m|}{\sqrt{1+k^2}}|AB|)
$$

**$x^2=cy$ 类型**

$$
\begin{cases}
x^2=cy \\
y=kx+m
\end{cases}
$$

$$
\Rightarrow x^2=c(kx+m) \Rightarrow x^2-kcx-cm=0
$$

$x^2=cy$ 类型可以直接计算，记公式反而会更麻烦。

### 双曲线

解析几何的解答题不会出双曲线，所以这里就不多写了~

## 方法总结

考试的过程中，只要思路清晰，利用这些公式 5 分钟内就可以做完解析几何，而且不会出错。

### 测试用例

{% note modern no-icon %}
**2015 · 高考全国卷 Ⅱ**

直线 $l:y=kx+m$ 与 $\dfrac{x^2}{4}+y^2=1$ 交于 $A,B$。若 $k_{PA}+k_{PB}=-1$，证明：$l$ 过定点。

解：联立

$$
\begin{cases}
\dfrac{x^2}{4}+y^2=1 \\
y=kx+m
\end{cases}
$$

得：$(4k^2+1)x^2+8kmx+4m^2-4=0$

由韦达定理得

$$
\begin{cases}
x_1+x_2=\dfrac{-8km}{4k^2+1} \\
x_1x_2=\dfrac{4m^2-4}{4k^2+1}
\end{cases}
$$

由 $y_1=kx_1+m,\ y_2=kx_2+m$

联立得

$$
x_1y_2+x_2y_1=\dfrac{-8k}{4k^2+1}
$$

由题意得 $k_{PA}+k_{PB}=-1$，设 $A(x_1,y_1),\ B(x_2,y_2)$，则

$$
\frac{y_1-1}{x_1}+\frac{y_2-1}{x_2}=-1
$$

即

$$
\frac{x_2y_1+x_1y_2-(x_1+x_2)}{x_1x_2}=-1
$$

代入得

$$
\frac{\dfrac{-8k+8km}{4k^2+1}}{\dfrac{4m^2-4}{4k^2+1}}=-1
$$

解得 $m=-1-2k$，故

$$
y = kx-1-2k = k(x-2)-1
$$

故 $l$ 过定点 $(2,-1)$。
{% endnote %}

平时刷题只需要积累一下解题思路就可以了，公式的主要作用是 **快速计算** 和 **避免出错**，实际上解析几何的计算都是 10 以内加减乘除，最容易出错的是推导过程写错参数导致后续计算全部出错。

用公式来计算结果，即便我们在推导过程中写错了参数，只要答案正确，基本是不会被阅卷发现导致扣分的哦~

### 原文链接

文章中的部分手写公式无法通过 `LaTex` 表达，严谨的推导过程请参考原文。

[百度贴吧 - 高考数学解析几何 1分钟解 ~](https://tieba.baidu.com/p/6702507722?fr=personpage)

---

{% series science %}
