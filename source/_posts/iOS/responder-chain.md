---
title: iOS 事件响应链
comments: true
date: 2020-05-18 15:09:51
tags: iOS
---

iOS 事件响应链  
事件的传递与响应
<!--more-->


## 基本概念

### UIResponder

* 在 iOS 中，只有继承了 **UIResponder** 的对象才能接受并处理事件

UIApplication (NSObject -> UIResponder -> UIApplication)
UIViewController (NSObject -> UIResponder -> UIViewController)
UIView (NSObject -> UIResponder -> UIView)
UIWindow (NSObject -> UIResponder -> UIView -> UIWindow)
UIButton (NSObject -> UIResponder -> UIView -> UIControl -> UIButton)

* UIResponder 提供以下方法，来处理事件

```objectivec
// UIResponder内部提供了以下方法来处理事件触摸事件
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
// 加速计事件
- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;
// 远程控制事件
- (void)remoteControlReceivedWithEvent:(UIEvent *)event;
```

### UITouch

当用户用一根手指触摸屏幕时，会创建一个与手指相关的UITouch对象

一根手指对应一个UITouch对象

如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象

如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象


```objectivec
//触摸产生时所处的窗口
@property(nonatomic,readonly,retain) UIWindow *window;
//触摸产生时所处的视图
@property(nonatomic,readonly,retain) UIView *view;
//短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多的点击
@property(nonatomic,readonly) NSUInteger tapCount;
//记录了触摸事件产生或变化时的时间，单位是秒
@property(nonatomic,readonly) NSTimeInterval timestamp;
//当前触摸事件所处的状态
@property(nonatomic,readonly) UITouchPhase phase;

// 返回值表示触摸在view上的位置
// 这里返回的位置是针对view的坐标系的（以view的左上角为原点(0, 0)）
// 调用时传入的view参数为nil的话，返回的是触摸点在UIWindow的位置
-(CGPoint)locationInView:(UIView *)view;
// 该方法记录了前一个触摸点的位置
-(CGPoint)previousLocationInView:(UIView *)view;
```

### 以 UIView 为例，重写 UIResponder 方法，说明事件的处理

* 重写以下 4 个方法，处理不同的触摸事件

注意，直接在 UIViewController 的.m文件中重写，重写的是 VC 的方法，并非 VC.view 的

```objectivec
// touches中存放的都是UITouch对象
// 一根或者多根手指开始触摸view，系统会自动调用view的下面方法
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
// 一根或者多根手指在view上移动，随着手指的移动，会持续调用该方法
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
// 一根或者多根手指离开view，系统会自动调用view的下面方法
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
// 触摸结束前，某个系统事件(例如电话呼入)会打断触摸过程，系统会自动调用view的下面方法
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
```

## 事件的产生与传递

### 事件的产生与传递

1. 发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列（先进先出）中。

2. UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）

3. 主窗口会在视图层次结构中**找到一个最合适的视图来处理触摸事件**

触摸事件的传递是从父控件传递到子控件,也就是UIApplication->window->寻找处理事件最合适的view。

**如果父控件不能接受触摸事件，那么子控件就不可能接收到触摸事件**

4. 找到合适的视图控件后，就会调用视图控件的touches方法来作具体的事件处理。

### 寻找最合适的控件处理触摸事件

通过 hitTest:withEvent: 来找到最合适的控件，处理事件。
   
1. 首先判断主窗口（keyWindow）自己是否能接受触摸事件
   
   以下情况不能接收触摸事件
    
    * 不允许交互：userInteractionEnabled = NO
        
    * 隐藏：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件

    * 透明度：如果设置一个控件的透明度<0.01，会直接影响子控件的透明度。alpha：0.0~0.01为透明
  
2. 判断触摸点是否在自己身上
   
   ```
   - (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;
   ```
  
3. 子控件数组中从后往前遍历子控件，重复前面的两个步骤
   
   从后往前遍历子控件，就是首先查找子控件数组中最后一个元素，然后执行1、2步骤

4. view，比如叫做fitView，那么会把这个事件交给这个fitView，再遍历这个fitView的子控件，直至没有更合适的view为止。

5. 如果没有符合条件的子控件，那么就认为自己最合适处理这个事件，也就是自己是最合适的view。

无论控件能否处理事件，只要点击了就会产生事件，区别是最终由哪个控件来处理事件。

综上，hitTest:withEvent: 的实现大致如下

```objectivec
//Returns the farthest descendant of the receiver in the view hierarchy (including itself) that contains a specified point
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {

    // 1.判断当前控件能否接收事件
    if (self.userInteractionEnabled == NO || self.hidden == YES || self.alpha <= 0.01) return nil;
    
    // 2. 判断点在不在当前控件
    if ([self pointInside:point withEvent:event] == NO) return nil;
    
    // 3.从后往前遍历自己的子控件
    NSInteger count = self.subviews.count;
    for (NSInteger i = count - 1; i >= 0; i--) {
        UIView *childView = self.subviews[i];
        
        // 把当前控件上的坐标系转换成子控件上的坐标系
        CGPoint childP = [self convertPoint:point toView:childView];
        // 如果 fitView 为 nil，即 hitTest:withEvent: 返回 nil，当前的 view 及其 subviews 不是合适的view
        UIView *fitView = [childView hitTest:childP withEvent:event];
        
        // 找到最合适的view
        if (fitView) { 
            return fitView;
        }
    }

    // 循环结束,表示没有比自己更合适的view，返回自身
    return self;    
}
```

![](https://raw.githubusercontent.com/skybrim/AllImages/dev/responder-chain-1.png)

### 其他操作

可以通过重写 hitTest:withEvent 来修改响应事件的 view

```objectivec
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    UIView *view = [super hitTest:point withEvent:event];
    if (view == self) {
        return nil;
    }
    return view;
}
```