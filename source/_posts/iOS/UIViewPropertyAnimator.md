---
title: UIViewPropertyAnimator
date: 2019/1/1
tags: iOS
comments: true
---

UIViewPropertyAnimator
<!--more-->

## Demo
[demo](https://github.com/skybrim/iOS_practice/tree/master/Animation)

## 结论
先说一下对别的结论，然后列出代码。  

*  优点：  
	* 1、精细控制动画状态 start pase stop finish  
	* 2、精细控制动画进度 fractionComplete  
	* 3、可以中间加入动画 addAnimations  
	* 4、自定义 curve  
* 缺点：  
	* 1、无法重复动画: .repeat 无效（-iOS12）  
	* 2、无法自动翻转动画： .autoreverse 无效（-iOS12）  

## UIView.animationDuration:...
```Swift
    UIView.animate(
        withDuration: 3,
        delay: 0,
        options: [.curveLinear], // [.repeat, .autoreverse]
        animations: {
            // code...
    }) { (_) in
        print("UIView.animate -- completion")
    }
```

## UIViewPropertyAnimator
iOS 10 以后 Apple 新加入的动画 API，主要用来取代 UIView.animationWithDuration:...

* 类似 [UIView animationWithDuration:...] 的使用形式  
UIViewPropertyAnimator 也可以按照旧式的 UIView.animationWithDuration:... 的形式调用。
```Swift
        UIViewPropertyAnimator.runningPropertyAnimator(
            withDuration: <#T##TimeInterval#>,
            delay: <#T##TimeInterval#>,
            options: <#T##UIView.AnimationOptions#>,
            animations: <#T##() -> Void#>,
            completion: <#T##((UIViewAnimatingPosition) -> Void)?##((UIViewAnimatingPosition) -> Void)?##(UIViewAnimatingPosition) -> Void#>
        )
```

* 创建对象  
Apple 提供了三种快速创建的方法。
```Swift
	// 设置 duration 、 cureve 和 animation， 创建 UIViewPropertyAnimator
    public convenience init(duration: TimeInterval, curve: UIView.AnimationCurve, animations: (() -> Void)? = nil)
	// 通过设置首尾两个点创建动画曲线，创建 UIViewPropertyAnimator
    public convenience init(duration: TimeInterval, controlPoint1 point1: CGPoint, controlPoint2 point2: CGPoint, animations: (() -> Void)? = nil)
	//设置 dampingRatio（阻尼率），创建 UIViewPropertyAnimator
    public convenience init(duration: TimeInterval, dampingRatio ratio: CGFloat, animations: (() -> Void)? = nil)
```  
如果还是未能满足，可以通过自定义创建动画曲线。
```Swift
    public init(duration: TimeInterval, timingParameters parameters: UITimingCurveProvider)
```

* 动画状态
	* Inactive  
	对象初始化或者动画结束后所处的状态。

	* Active  
	在调用 startAnimation() 、 pauseAnimation() 方法后对象就处于激活状态，直到动画完成或者手动调用 stopAnimation() 结束动画。

	* Stopped  
	在 stopAnimation() 被调用之后动画对象就处于停止状态，并且保留当前的所有属性值。如下图所示，停止状态无法反向回到激活态。

	![](https://raw.githubusercontent.com/skybrim/AllImages/dev/UIVIewPropertyAnimator-state.png)

* 添加新动画  
UIViewPropertyAnimator 可以在中途加入新的动画。
```Swift
//The duration of the animation will be (1 - delayFactor) * animator.duration seconds.
//delayFactor，不是动画开始执行的绝对值，因为新动画加入时，animator 可能已经运行一段时间了
//默认为 0
    animator.addAnimations({
    	//code...
    }, delayFactor: 0)
```

* 动画结束
```Swift
//UIViewAnimatingPosition，用于表明动画结束的位置，这个值本身是一个枚举，包含了 starting、end 和 current。通常得到的值是 end。
    animator.addCompletion { (UIViewAnimatingPosition) in
        //code
    }
```

* 控制动画进度  
注意，只能在动画处于 inactive 和 active 的时候，可以控制。
```Swift
    @IBAction func changeAnimator(_ sender: UISlider) {
        animator.fractionComplete = CGFloat(sender.value)
    }
```


