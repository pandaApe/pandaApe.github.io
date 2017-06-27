title: Swift可选绑定多个可选变量
date: 2015-12-05 22:19:09
categories: iOS
---

最近一个已经开始的项目客户要求用Swift写，已经写好的代码要转成Swift，这个是这次“翻译”工作中的一个小知识，本想一句话放在笔记里的，但想想还是拿出来分享给大家，分享也是一种学习嘛。

<!--more-->
使用可选绑定（optional binding）来判断可选类型是否包含值，如果包含就把值赋给一个临时常量或者变量.
单个可选绑定我们这样用：
```swift
var str1:String? 
if let s1 = str1{
    print(s1)
}
```

 如果是多个,就用逗号隔开，第一个之后let可以省略，默认是let
```swift
var str1:String?
var str2:String?

str1 = "-"
str2 = "."

if let s1 = str1, s2 = str2{
    print(s1 + s2)
}
```

注意：_多个可选绑定之间是“并且”（&&）关系，如果有一个可选变量没有值，则整个条件语句的结果为false._