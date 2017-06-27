---
title: 第26周开发Tips
date: 2016-06-22 21:00:02
categories: Tips
---

## 安装Node.js时，提示 **nvm: command not found**

解决办法是： 执行 nvm 命令前，先执行 `source ~/.nvm/nvm.sh` 命令。载入配置命令使nvm命令生效。

可以把 `source ~/.nvm/nvm.sh` 命令 写进 `~/.profile` 或者 `~/.bashrc` 文件中, 下次打开Terminal 时会自动执行这个命令。 
<!--more-->
有可能 `~/.profile` 文件不存在（比如新安装系统的电脑），使用 

>touch ~/.profile 
>open -e !$ 

创建并打开，粘贴完保存就可以了。 对没错，`~/.profile` 文件一般就是用来设置环境变量的地方。

参考链接：
[1]: [Node Version Manager install - nvm command not found ](http://stackoverflow.com/questions/16904658/node-version-manager-install-nvm-command-not-found)
[2]: [Can't find .profile file in OS X ](http://superuser.com/questions/473087/cant-find-profile-file-in-os-x)

## Terminal 格式化 JSON
方法1：
> echo '{"foo": "lorem", "bar": "ipsum"}' | python -m json.tool

方法2(如果在文件里)：
> python -m json.tool my_json.json

Terminal 里查看结果：
![](http://ww3.sinaimg.cn/large/8c0dc373jw1f55hls0bp0j20us0l4agc.jpg)

参考链接：
[1]: [How can I pretty-print JSON? ](http://stackoverflow.com/questions/352098/how-can-i-pretty-print-json)

## 真机调试CocoaPods提示LibraryNotLoaded
使用Cocoapods管理依赖，用真机运行时，控制台打印：
>dyld: Library not loaded: @rpath/Pods.framework/Pods
>Referenced from: /private/var/mobile/Containers/Bundle/Application/366AF813-0739-48B2-BF1D-5FA6E90A29CA/Fetch.app/Fetch
>Reason: image not found

Step 1:
>disable bit code

Step 2:
>The issue was the Pods.framework was set to required and not optional. It wasn't being copied to device by the .sh script and so the app crashed. Setting this to optional fixed the issue.
![](https://cloud.githubusercontent.com/assets/1169297/7726958/9156c376-fefd-11e4-90c8-2412f67f52c2.png)

参考链接：
[1]: [CocoaPods issue#3586 ](https://github.com/CocoaPods/CocoaPods/issues/3586)
[2]: [The solution on sack overflow ](http://stackoverflow.com/questions/30053144/dyld-library-not-loaded-with-cocoapods-0-37-and-xcode-6-3/30166310#30166310)

