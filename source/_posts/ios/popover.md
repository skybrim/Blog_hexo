---
title: Popover
date: 2019/1/1
tags: iOS
comments: true
---

iOS8 之前，Popover 只能在 iPad 上使用，iPhone 上使用效果类似 present。  
iOS8 之后需要实现代理方法，取消设备判断，即可在 iPhone 上实现弹窗效果。  
<!--more-->

* 在 iPhone 上使用 Popover

```swift
//实现 UIPopoverPresentationControllerDelegate 中的以下方法
//取消设备判断
func adaptivePresentationStyle(for controller: UIPresentationController) -> UIModalPresentationStyle {
	return .none
}
```

* 设置 Popover  

```swift
let popover = ViewController()
//设置弹出模式为 popover  
popover.modalPresentationStyle = .popover
//设置size  
popover.preferredContentSize = CGSize(width: 200, height: 400)
//设置代理  
popover.popoverPresentationController?.delegate = self
//设置锚点，下面 1 和 2 任选其一  
//1 依附在 barButtonItem  
popover.popoverPresentationController?.barButtonItem = self.navigationItem.rightBarButtonItem
//2 依附在 view 上，如 button 等  
popover.popoverPresentationController?.sourceView = sender
popover.popoverPresentationController?.sourceRect = sender.bounds
//设置箭头方向  
popover.popoverPresentationController?.permittedArrowDirections = .up
//设置箭头颜色  
popover.popoverPresentationController?.backgroundColor = .red
//显示  
self.present(popover, animated: true, completion: nil)
```
