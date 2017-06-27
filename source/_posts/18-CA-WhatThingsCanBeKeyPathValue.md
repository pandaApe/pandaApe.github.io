---
title: Core Animation(核心动画) 中KeyPath的取值
date: 2016-08-15 22:20:24
categories: iOS
---

在Core Animation中， 借助***Core Animation*** 提供的几个动画类我们可以对一个Layer做各种酷炫的动画，在实例 CAPropertyAnimation 的时候（ ```CABasicAnimation``` 和 ```CAKeyFrameAnimation```均继承自该类），通过构造方法 ```convenience init(keyPath path: String?)```，需要传入一个字符串。 那么问题来了，这个参数到底可以取哪些值呢？

<!-- more -->

我们知道 ```CALayer``` 实现了 ```NSKeyValueCoding``` 协议，因此我们可以使用 KVC 对 CALayer的实例中的属性进行取值和赋值，而在 ***Core Animation*** 提供的几个动画类影响的对象便是 ```CALayer```的实例，我们可以推测在 CAPropertyAnimation 类中构造方法 ```convenience init(keyPath path: String?)``` 的keyPath参数的取值 应该是 CALayer 中的属性。 带着这个推测，参考苹果的官方文档，笔者把可以用作keyPath的值列举如下。

##  一、可动画属性
### 1. 几何属性 (Geometry Properties)
|Field Key Path|Remark| En Description
|-------|-------|------|
|transform.rotation.x|按x轴旋转的弧度|Set to an NSNumber object whose value is the rotation, in radians, in the x axis.|
|transform.rotation.y|按y轴旋转的弧度|Set to an NSNumber object whose value is the rotation, in radians, in the y axis.|
|transform.rotation.z|按z轴旋转的弧度|Set to an NSNumber object whose value is the rotation, in radians, in the z axis.|
|transform.rotation|按z轴旋转的弧度, 和transform.rotation.z效果一样|Set to an NSNumber object whose value is the rotation, in radians, in the z axis. This field is identical to setting the rotation.z field.|
|transform.scale.x|在x轴按比例放大缩小|Set to an NSNumber object whose value is the scale factor for the x axis.|
|transform.scale.y|在x轴按比例放大缩小|Set to an NSNumber object whose value is the scale factor for the y axis.|
|transform.scale.z|在z轴按比例放大缩小|Set to an NSNumber object whose value is the scale factor for the z axis.|
|transform.scale|按比例放大缩小|Set to an NSNumber object whose value is the average of all three scale factors.|
|transform.translation.x|沿x轴平移|Set to an NSNumber object whose value is the translation factor along the x axis.|
|transform.translation.y|沿y轴平移|Set to an NSNumber object whose value is the translation factor along the y axis.|
|transform.translation.z|沿z轴平移|Set to an NSNumber object whose value is the translation factor along the z axis.|
|transform.translation| x,y 坐标均发生改变|Set to an NSValue object containing an NSSize or CGSize data type. That data type indicates the amount to translate in the x and y axis.|
|transform| CATransform3D 4*4矩阵||
|bounds|layer大小||
|position|layer位置||
| ~~frame~~ |不支持 frme 属性|***computed from the bounds and position and is NOT animatable***|
|anchorPoint|锚点位置||
|cornerRadius|圆角大小||
|zPosition|z轴位置| ||

### 2.背景属性 (Background Properties)

|Field Key Path|Remark| En Description
|-------|-------|------|
|backgroundColor|背景颜色 | ||

### 3.Layer内容 (Layer Content)

|Field Key Path|Remark| En Description
|-------|-------|------|
|contents|Layer内容，呈现在背景颜色之上| |
|contentsRect| |The rectangle, in the unit coordinate space, that defines the portion of the layer’s contents that should be used. 
|masksToBounds||setting the layer’s masksToBounds property to YES does cause the layer to clip to its corner radius

### 4.子Layer内容 (Sublayers Content)

|Field Key Path|Remark| En Description
|-------|-------|------|
|sublayers|子Layer数组|
|sublayerTransform|子Layer的Transform|Specifies the transform to apply to sublayers when rendering.|

### 5.边界属性 (Border Attributes)

|Field Key Path|Remark| En Description
|-------|-------|------|
|borderColor|||
|borderWidth||||


### 6.阴影属性 (Shadow Properties)

|Field Key Path|Remark| En Description
|-------|-------|------|
|shadowColor| 阴影颜色||
|shadowOffset| 阴影偏移距离||
|shadowOpacity| 阴影透明度||
|shadowRadius| 阴影圆角||
|shadowPath| 阴影路径|||

### 7.透明度 (Opacity Property)

|Field Key Path|Remark| En Description
|-------|-------|------|
|opacity|透明度 ||
|hiden|  |||

### 8.遮罩 (Mask Properties)
|Field Key Path|Remark| En Description
|-------|-------|------|
| mask | |||

### 9.ShapeLayer属性 (ShapeLayer)
|Field Key Path|Remark| En Description
|-------|-------|------|
| fillColor | ||
| strokeColor | ||
| strokeStart | 从无到有 ||
| strokeEnd | 从有到无||
| lineWidth | 路径的线宽||
| miterLimit | 相交长度的最大值||
| lineDashPhase | 虚线样式|||



## 二、注意
这里特别需要注意的是 layer的 ```frame``` 是不支持动画的，我们可以通过改变```position```和````bounds``` 变通实现


## 三、参考资料

- [Apple Key-Value Coding Extensions](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html#//apple_ref/doc/uid/TP40004514-CH12-SW2)

- [Layer Animatable Properties](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW1)

- [How can I know the values in CABasicAnimation keyPath?](https://stackoverflow.com/questions/5459673/how-can-i-know-the-values-in-cabasicanimation-keypath)

- [What is the role of key path in Core Animation?](https://stackoverflow.com/questions/9717397/what-is-the-role-of-key-path-in-core-animation)
