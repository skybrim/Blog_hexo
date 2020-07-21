---
title: UIFont
date: 2019/1/1
tags: iOS
comments: true
---

iOS 系统中，用户可以通过 设置->显示与亮度->文字大小 来调节系统字号的大小。  
此时，我们也需要跟进适配。
<!--more-->

## 固定字号

字号不会跟随系统改变，适合一些复杂的 UI 效果，防止因为字号改变，而产生 UI 错乱的 BUG。
```
//自定字号，字体跟随系统
func systemFont(ofSize: CGFloat) -> UIFont

//自定字号，自定字体
init?(name fontName: String, size fontSize: CGFloat)
```

## Dynamic Type

使用 Dynamic Type 字号可以跟随系统设置改变，但是不支持自定义字体，只能使用系统字体。    
支持 iOS7 及以上 

//iOS10 以下，需要监控通知消息来进行更改 UIContentSizeCategoryDidChangeNotification   
```objectivec
//在通知方法中，更新 font 
label.font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
// 注意这里需要调用setNeedsLayout然后在layoutSubview或者viewDidLayoutSubview里面更新label的frame
[self.view setNeedsLayout]; 
```

iOS10 及以上可以直接设置
```swift
label.adjustsFontContentSizeCategory = true
label.font = UIFont.preferredFont(forTextStyle: .Title1) 
```

下面是系统支持的相关字号
![](https://raw.githubusercontent.com/skybrim/AllImages/dev/font.png)


## UIFontMetrics

iOS11，apple 加入了 UIFontMetrics ，终于可以支持自定义字体了。  
```swift
label.adjustsFontForContentSizeCategory = true
let font = UIFont.systemFont(ofSize: 20)
label.font = UIFontMetrics.default.scaledFont(for: font)
```


