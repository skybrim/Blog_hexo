---
title: 语音播报
date: 2018/11/24
tags: iOS
comments: true
---

iOS 12 及以上，需要使用本地音频拼接，或者申请VoIP。
iOS 10 及以上，使用 Notification Service Extension 实现远程推送语音播报。
iOS 10 以下，只能播放固定音频。
<!--more-->

需求：无论应用在前台、后台还是完全杀死，都能语音播放通知带来的文本。

原理：在收到远程推送的时候，Service Extension将远程推送的信息拦截下来，此时将文字信息转换为语音，进行播报，播报完毕之后再展示弹框信息。
UNNotificationServiceExtension，可以拦截到远程推送的信息，调用self.contentHandler(self.bestAttemptContent); 之后，就会进行弹框显示了，此时UNNotificationServiceExtension的结束。

## 添加 Notification Service Extension

打开 Xcode ，File -> New -> Target  
![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/Screen%20Shot%202019-06-20%20at%2014.05.54.png)  
自定 Product Name  
Xcode会为你的项目生成新的 target。 
![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/Screen%20Shot%202019-06-20%20at%2014.07.00.png)  
在NotificationService 中，-didReceiveNotificationRequest:withContentHandler:,此方法可以获取到推送相关信息。  
注意，需要在推送的格式中，写入参数 mutable-content ，示例如下：
```
{
 "aps":{
  "alert":{
   "title":"title",
   "subtitle":"subtitle",
   "body":"body",
  },
 "category":"category",
 "mutable-content":1,
 "badge":1,
 }
}
```
## 添加文字转语音功能

方法有很多，系统自带的文字转语音功能、科大讯发的SDK等等第三方。  
我这里使用系统自带的文字转语音功能：AVSpeechSynthesizer ，相关方法以及设置可以参考 [demo](https://github.com/skybrim/VoicePush) 。 

## 真机运行

真机运行时，请注意，除了运行原工程 target ，也要选择新的 target 重新运行一遍。
![](https://cdn.jsdelivr.net/gh/skybrim/AllImages@dev/Screen%20Shot%202019-06-20%20at%2014.15.49.png)

## iOS 12.1 及之后版本

iOS 12.1 apple官方给出说明，推送扩展无法在后台进行播放。  

* 申请VoIP  
VoIP 会唤醒 app ，然后进行语音播报。  
缺点是审核需要提供相关演示，表示需要申请 VoIP 。  

* 播放本地音频   
本地存放拆分好的音频文件，接受到远程通知是，利用本地通知来播放，缺点是本地通知每通知播放一次就会震动一次。  

## 打包 

打包时，需要给新的 target 设置好证书和 profile 文件，同 app 打包类似。 
Developer 网站，Certificates,Identifiers & Profiles ，app 和 extension 都需要AppID，通过 App Groups 进行关联。

