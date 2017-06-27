title: 在Xcode和AS中查找中文字符
date: 2016-01-04 21:05:13
categories: Tips
---
我们的代码里出现中文汉字，基本上是因为项目时间紧，为了快速开发，先Hard Code 代码里。

但是后期项目要进行国际化，必须要把汉字替换回来。然而如果一个一个地找然后替换，费时又费力，即使再仔细，也难免会有漏网之鱼。

<!--more-->
懒人总有懒方法。---在搜索框里使用正则表达式

**Xcode:**

>1. 打开”Find Navigator” , 快捷键 Command+Shift+F
>2. 切换搜索模式到 “Find > Regular Expression”
>3. 输入    **```"[^"]*[\u4E00-\u9FA5]+[^"\n]*?"```** 

![](http://ww1.sinaimg.cn/large/8c0dc373gw1f1d01ytrmsj20b409wq4h.jpg)


**Android Studio:**

>1. 使用快捷键 Command+Shift+F 打开
>2. 勾选 Regular expression 复选框
>3. 输入   **```"[^"]*[\u4E00-\u9FA5]+[^"\n]*?" ```**

![](http://ww4.sinaimg.cn/large/8c0dc373gw1f1d02r81nzj20b40eb0ub.jpg)

重点在于第三步的能找到中文字符的正则表达式。你Get到了么？

