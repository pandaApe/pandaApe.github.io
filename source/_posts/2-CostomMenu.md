title: 自定义 选择内容后 的菜单
date: 2015-08-22 16:45:53
tags: IOS菜单
categories: iOS
---

iOS长按菜单的使用

<!-- more-->

## 微信长按菜单的联想
	
![](http://ww4.sinaimg.cn/large/8c0dc373gw1evbu9lnabjj20a00a9dge.jpg)

阅读类应用必备的功能有木有。不单是可以加入分享功能，用浏览器搜索，颜色标记选中内容都可以做到撒。

## 效果图

![](http://ww4.sinaimg.cn/large/8c0dc373gw1evbkys8npyj209109gjsh.jpg)

## 示例代码

````swift
override func viewDidLoad(){
	super.viewDidload()

	//1.自定义选择后的菜单
	var itemMail = UIMenuItem(title: "邮件分享", action: "mailShare:")
	var itemWeibo = UIMenuItem(title: "微博分享", action: "weiboShare:")

	//2.取menu实例
	var menu = UIMenuController.sharedMenuController()
	//3.增加到menu中
	menu.menuItems = [itemMail, itemWeibo]
}

//点击时的Action
func weiboShare(sender:UIMenuItem){
	println("weiboShare:")
}

func mailShare(sender:UIMenuItem){
	println("mailShare:")
}
````