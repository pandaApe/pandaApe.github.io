title: Swift的类型兼容和@Objc修饰符
date: 2015-12-10 23:04:21
categories: iOS
---
最近用Swift重写了一个自定义控件（源码已经上传到Github了，点[这里](https://github.com/pandaApe/HLPlaceHolderView)），其中有段代码是要在一个internal的方法里给一个ImageView添加点击手势，响应点击事件的方法是调用当前类里面的一个prevate方法。代码如下
<!--more-->
``` Swift
    func addPlaceholderTapTarget(target:AnyObject!, andAction action:Selector){
        
        self.target = target
        self.action = action
        
        //定义点击手势
        tapGesture = UITapGestureRecognizer(target: self, action: "handleTapAction")
        
        if tapGesture != nil{
            iconImgView.removeGestureRecognizer(tapGesture!)
            msgLabel.removeGestureRecognizer(tapGesture!)
        }
        
        iconImgView.addGestureRecognizer(tapGesture!)
        iconImgView.userInteractionEnabled = true
        
    }
    
    //此方法是私有的,不显露给外面
    private func handleTapAction(){
        self.startAnimating()
        target.performSelector(action)
    }
```
本以为这样写没有问题的，结果一跑起来，抛错了：( 

>-[HLPlaceholderViewDemo.HLPlaceholderView handleTapAction]: unrecognized selector sent to instance 0x7fc94973d470
> *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[HLPlaceholderViewDemo.HLPlaceholderView handleTapAction]: unrecognized selector sent to instance 0x7fc94973d470'


说这个类里面并没有func handleTapAction()这个方法，仔细一想猜测了一下原因大概是：手势类内部使用KVO调用这个Action,然而func handleTapAction()方法是private，在编译时候，不会当成公有方法，因此在KVO调用时候，就会找不到这个方法，接着就会抛错

Google一下，于是在这位大神这里发现的解决方法[Gesture Recognizers](https://thatthinginswift.com/gesture-recognizers-swift/)——在private方法前加上@objc修饰符

虽然解决了问题，但是依然有点不明不白的。顺着大神的思路，顺便查了查Apple官方的《Using Swift with Cocoa and Objective-C》，不查不知道一查吓一跳，原来@objc背后还有这么多说明。

其中找了几段能说明一下：
## Swift的类型兼容性
创建一个Swift的类，如果继承自OC的类，那么这个类的成员，包括属性、方法、下标、构造方法，都自动与OC兼容，可以在OC代码中直接调用。但是有些情况是这个Swift类没有继承OC的类，或者你想修改调用的方法名等，你就需要使用@objc修饰符了。


