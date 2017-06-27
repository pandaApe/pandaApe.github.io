---
title: Xcode中项目的多环境管理
date: 2016-07-10 17:25:54
categories: iOS
---

在项目开发过程中，通常我们有不同的版本，例如：生产版本和公开版本。这些版本之间在代码上会有一些差别，例如生产版我们要使用生产版的Server和数据库，到了公开版我们会换成公开版的Server和数据库，与之相类似的区别还有API Version，API Key等等。

<!-- more -->

## 源起
以前，我们想在不同版本之间切换时，需要手动地更改 Server URL, API Key这些参数

```Swift
// MARK: - Development
let APIEndpointURL = "http://mysite.com/dev/api"
let APIKey = "lihln2wewensf1sfnxapw"
        
// MARK: - Production
// let APIEndpointURL = "http://mysite.com/prod/api"
// let APIKey = "fnxapwlinsf1shln2wewe"
```
需要使用哪个版本就取消注释哪个版本，接着再把别外一个注释掉。。。很麻烦。后来发现了一个高级一点的办法：
```Swift
 		enum environmentType {
            case development, production
        }
        
        let environment:environmentType = .production
        
        switch environment {
            
        case .development:
            let APIEndpointURL = "http://mysite.com/dev/api"
            let APIKey = "lihln2wewensf1sfnxapw"
            print("It's for development")
        case .production:
             let APIEndpointURL = "http://mysite.com/prod/api"
             let APIKey = "fnxapwlinsf1shln2wewe"
            print("It's for production")
        }
```

通过修改枚举值来使用不同的参数。然而，并没有什么卵用啊，一样需要修改代码，而且这种方法
+ 对于不同的版本只能使用同一个Bundle ID ，不能在同一个设备上安装两个版本
+ 有可能忘记修改，有把开发版上传到应用商店的风险
都是重复的劳动，既没有技术含量，又花费时间，而且还容易出错。

本文将安利一个方法解决上述问题，好用到没朋友。

## Step by Step
为了方便讲解，已经新建了一个工程名为MultiEnviSample。
![](http://ww3.sinaimg.cn/large/8c0dc373jw1f5uxcf12ffj214m0sa42u.jpg)

### 新建Build Configuration
点击左侧工程图标，再选中PROJECT中的工程，点击Info tab下的"+"号，根据需要选择 

![](http://ww3.sinaimg.cn/large/8c0dc373jw1f5uxcxxzyfj21k60ra464.jpg)

在"Release"或"Debug"基础上新建，我们这里选择"Debug", 最后把新建的Configuration重命名为"DebugDev"
### 新建Scheme

点击"Manage Schemes..."。
![](http://ww1.sinaimg.cn/large/8c0dc373jw1f5uxcd42n2j20p006i76a.jpg)

再点击左下角的"+"号，我们新建一个名为"MultiEnviSampleDev"的Scheme。
![](http://ww1.sinaimg.cn/large/8c0dc373jw1f5uxce17wbj216o0pwdjr.jpg)

选择右侧的Shared复选框，这样就可以把Scheme添加到Git仓库。
![](http://ww4.sinaimg.cn/large/8c0dc373jw1f5uxcdyrz5j21620pgadu.jpg)

### 编辑Scheme
选中我们刚刚新建的Scheme，点击左下角的"Edit"。
![](http://ww2.sinaimg.cn/large/8c0dc373jw1f5uxcg4e2uj21680pmtcq.jpg)

在Info Tab下的Build Configuration中选择之前新建"DebugDev"。
![](http://ww4.sinaimg.cn/large/8c0dc373jw1f5uxciyif5j21kw0lrq8g.jpg)
### 设置Flag
在Build Setting里找到 "Swift Compiler-Custom Flag"，如图设置，加入"-DDevelopment"。
![](http://ww2.sinaimg.cn/large/8c0dc373jw1f5uxccdvtoj20x207kjso.jpg)
### 代码中使用
```Swift

 		#if DEVELOPMENT
            let SERVER_URL = "http://dev.server.com/api/"
            let API_TOKEN = "DI2023409jf90ew"
        #else
            let SERVER_URL = "http://prod.server.com/api/"
            let API_TOKEN = "71a629j0f090232"
        #endif
        
        print(SERVER_URL+API_TOKEN)

```
选择不同的Scheme，即可在不同环境之间切换。

### 多个APP（可选）
为了很好的区分开发版和生产版，我们可以通过对不同的环境使用不同的Bundle ID，App Icon，和启动页，这样方便QA更清晰地知道正在测的是哪个版本。

![](http://ww4.sinaimg.cn/large/8c0dc373jw1f5uxchopfoj20uu0bwae2.jpg)

在"Build Setting"下找到"Product Bundle Indentifier"和"Product Name",在 "DebugDev"后的Bundle ID加上"Dev",

## Objective-C项目
对于Objective-C的项目，去到"Build Settings"下"Apple LLVM 7.0 - Preprocessing"。拓展"Preprocessor Macros"在Rebug和Release区域添加一个变量。对于开发"Build Configuration"，将该值设置为"DEVELOPMENT = 1"。另一个，将值设为"DEVELOPMENT=0"来表示生产版本。
![](http://ww1.sinaimg.cn/large/8c0dc373jw1f5uxcd0d5xj20ug07sjti.jpg)

## 注意
上述做法只是一个Sample。触类旁通，我们可以使用同样的思路，做其他一些好玩的事情：
1. 使用模拟器运行，还是真机运行。
2. 是否清空数据库
3. 使用真实API登录，还是模拟登录
4. 等等...

## 参考资料
[如何使用Xcode的Targets来管理开发和生产版本的构建 ](http://mp.weixin.qq.com/s?__biz=MjM5OTM0MzIwMQ==&mid=2652546114&idx=1&sn=67e479d82e0d0a662b05082fe74f731b&scene=2&srcid=0627ccdTvSBjeEDlKwmSXPgo#wechat_redirect)
[XCode: 7 Steps to Easily Switch Between Multiple Environments ](http://www.teratotech.com/blog/xcode-7-steps-to-easily-switch-between-multiple-environments/)
[Xcode tricks. Deal with multiple environments easily ](http://limlab.io/swift/2016/02/22/xcode-working-with-multiple-environments.html)






