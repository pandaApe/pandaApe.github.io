---
title: iOS世界里的贝塞尔曲线(一)：贝塞尔曲线基础
date: 2017-01-25 20:29:54
categories: iOS
---

## 一、前言

本系列文章通过介绍 ***贝塞尔曲线*** 的基础知识，贝塞尔曲线在iOS中的应用以及一些高级技巧，循序渐进，试图让读者对iOS的中贝塞尔曲线知识有一个较系统的认识。

你可能在很多地方听说过***贝塞尔曲线***，但是贝塞尔曲线到底是什么，它有什么特性能让它有这么高的知名度，它到底有什么用呢？

<!-- more -->

## 二、贝塞尔曲线简介
###  1. 贝塞尔曲线 是什么

头一次听，可能以为他和牛顿定律，斐波那契数列一样是由一个名字叫贝塞尔的人发明的，然而并不是。贝塞尔曲线的数学基础最早是1912年就在当世广为人知的[伯恩斯坦多项式](http://en.wikipedia.org/wiki/Bernstein_polynomial)，那什么又是伯恩思坦多项式呢，简单地说，伯恩斯坦多项式是用来证明，在 [a, b]区间上所有的连续函数都可以多项式来逼近，并且一致收敛，再简单地说，就是在一个连续函数，你可以将它写成n个伯恩思坦多项式相加的形式，并且随着n 趋向于无穷大，这个多项式将一致收敛到原函数。

听不懂，没关系，继续往下看。

虽然在1912年就已经被发现，但是其对图形的适用性在半个世纪内者也没有被实现，直到1959年在雪铁龙汽车就职的数学家 [Paul de Casteljau](http://en.wikipedia.org/wiki/Paul_de_Casteljau)，开始对伯恩斯坦多项式进行图形化的尝试，并推出一种新的数值稳定(即在求伯恩斯坦多项式的时候不会引入数值误差)递归算法 [de Casteljau 算法](http://en.wikipedia.org/wiki/De_Casteljau's_algorithm)用来伯恩斯坦多项式。根据这个算法，就可以只通过很少的控制点，去生成复杂的平滑曲线，也就是贝塞尔曲线。

贝塞尔曲线的成名，得益于法国工程师 [Pierre Bézier](http://en.wikipedia.org/wiki/Pierre_B%C3%A9zier) ，他将这种算法用来辅助雷诺汽车的车体工业设计，并且得到广泛宣传。

### 2. 现实应用
正是因为其绘制简便却具有极强的描述能力，贝塞尔曲线在工业设计领域迅速得到了广泛的推广和应用。随后随着计算机技术的发展，在计算机图形学领域，尤其是矢量图形学，贝塞尔曲线也占有了重要的地位。

今天我们使用的绘图软件，Illustrator、CorelDraw 等，无一例外都提供了绘制贝塞尔曲线的功能。甚至像 Photoshop 这样的位图编辑软件，也把贝塞尔曲线作为仅有的矢量绘制工具（钢笔工具）包含其中。

![CorelDraw.png](http://upload-images.jianshu.io/upload_images/172968-a369dd9e0ce1dc94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](http://upload-images.jianshu.io/upload_images/172968-850a7f091b7455ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这里的一个网站可以在线模拟钢笔工具的使用：[http://bezier.method.ac/](http://bezier.method.ac/)

推广到三维空间的[贝塞尔曲面](http://en.wikipedia.org/wiki/B%C3%A9zier_surface)，以及更进一步的[非均匀有理 B 样条（NURBS）](http://en.wikipedia.org/wiki/Non-uniform_rational_B-spline)，早已成为当今计算机辅助设计（CAD）的行业标准，不论是我们平常用到的各种产品，还是在电影院看到的精彩大片，都少不了它们的功劳。

![曲面.png](http://upload-images.jianshu.io/upload_images/172968-90e77721e06237ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![用于汽车设计](http://upload-images.jianshu.io/upload_images/172968-58a1ab918eb38850.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##  二、如何绘制贝塞尔曲线

下面我们就通过例子来了解一下如何用 [de Casteljau 算法](http://en.wikipedia.org/wiki/De_Casteljau's_algorithm)绘制一条贝塞尔曲线。

1. 画线段AC，BC, 相交于C点.
2. 在线段AC取点D，BC上取点E使 ```AD:DC = CE:EB``` 。 如下图所示：

![image.png](http://upload-images.jianshu.io/upload_images/172968-a2c50c63f8635d91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.连接点D、E
4.在线段DE上取点F，使 ```AD:DC = CE:EB = DF:FE```。 如下图：

![image.png](http://upload-images.jianshu.io/upload_images/172968-502dfb5df51970cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此我们就找到了贝塞尔曲线上的点F，这时让选取的点 D 在线段AB上从起点 A 移动到终点 B，找出所有的贝塞尔曲线上的点 F。所有的点找出来之后，我们也得到了这条贝塞尔曲线。如下图：

![image.png](http://upload-images.jianshu.io/upload_images/172968-ee8478efb6f28335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你实在想象不出这个过程，没关系，看动画！

![bezierPath.gif](http://upload-images.jianshu.io/upload_images/172968-82781ddb157c4c6a.gif?imageMogr2/auto-orient/strip)

这样就画出了一条贝塞尔曲线。

贝塞尔曲线的一个比较好的动态演示网站：[http://myst729.github.io/bezier-curve](http://myst729.github.io/bezier-curve)

## 三、多次（阶）贝塞尔曲线
回过头来看这条贝塞尔曲线，
>为了确定曲线上的一个点，需要在线段AC和BC上进行取点的操作，A、B、C三个点被称为控制点，因此我们称得到的曲线为二(控制点个数n-1)次贝塞尔曲线。

根据控制点的个数，贝塞尔曲线被分为一次贝塞尔曲线，二次贝塞尔曲线（3个控制点）、三次贝塞尔曲线（4个控制点）等等，以此类推。

![二次贝塞尔曲线.gif](http://upload-images.jianshu.io/upload_images/3450093-91de929ee26595f4.gif?imageMogr2/auto-orient/strip)

![三次贝塞尔曲线.gif](http://upload-images.jianshu.io/upload_images/3450093-dc67b41b181260ae.gif?imageMogr2/auto-orient/strip)

![四次贝塞尔曲线.gif](http://upload-images.jianshu.io/upload_images/3450093-9c2ddf60cab67586.gif?imageMogr2/auto-orient/strip)

还有只有两个控制点的一次贝塞尔曲线，没错是一条线段，它是贝塞尔曲线的特殊情况：
![](http://upload-images.jianshu.io/upload_images/3450093-fc72aea84a7d25f8.gif?imageMogr2/auto-orient/strip)

### 四、至此
至此我们对贝塞尔曲线的基本有了一个基本的了解。
