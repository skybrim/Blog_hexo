---
title: View
date: 2017/1/1
tags: iOS
comments: true
---

记录
<!--more-->

* storyboard 分模块使用，尽量一个模块一个 storyboard， 不要整个工程混杂在一个里面。不仅可以尽量减少冲突，也有助于清晰整个工程的结构。

* 勾选 Is Initial View Controller 确定 storyboard 的主VC入口，主入口的左侧会有一个箭头

* 三大容器（container view controller，Tab Bar Controller，Navigation Controller，Split View Controller）。

* 容器与其他VC关联： relationship segue。

* TableViewController 中 Dynamic Propotype 与 Static Cells 混用：   
在 Storyboard 中，设置类型为 Static Cells ，在需要使用 Dynamic Propotype 的 section 中，留一个 cell ，并设置好 Identifier 。  
代码中需要实现 UITableView 的代理方法。  
以下方法必须实现。
	```swift
	tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell  
	tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int  
	tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat  
	// *** ***
	tableView(tableView: UITableView, indentationLevelForRowAtIndexPath indexPath: NSIndexPath) -> Int
	```

* Static Cell，不重用cell，比如个人信息页面。可以直接在 storyboard 里设置，更方便。

* Dynamic Prototypes，可重用cell。需要确定 cell 的 identifier，并在在代码中设置代理。

* Selection Segue 的意思是当用户点击table view cell的任何部分，都会产生反应。  
Accessory Action 的意思是只有当用户点击table view cell右边的圆圈箭头按钮时，才会产生的反应。

* apple建议使用segue来进行跳转，项较于push等操作，支持形式更多，并且在不同设备（iPad和iPhone），会有更好的体验。  
	Segue的类型  
	* Show  
		>使用方法 showViewController: sender:  
		>该方法为VC提供了自适应、灵活的呈现方式；  
		>用在 UINavigationController 堆栈视图时，presentedViewController 进入时由右向左，退出时由左向右。新压入的VC有返回按钮，单击可以返回；  
		>用在 UIViewController 实例时，和 presentViewController: animated: completion: 效果一致。  
	* Show Detail   
		>使用方法 showDetailViewController: sender:；  
		>只适用于嵌入在 UISplitViewController 对象内的VC，分割控制器用以替换详细控制器（DetailViewController)；  
		>不提供返回按钮；  
		>用于 UISplitViewController 以外的控制器时，和 showViewController:sender用法一样。  
	* Present Modally  
		>使用方法 presentViewController: animated: completion:；  
		>有多种不同呈现方式，可根据需要设置。在 iPhone 中，一般以动画的形式自下向上覆盖整个屏幕，用户无法与上一个VC交互，除非关闭当前VC；在 iPad 中，常见呈现为一个中心框，中心框以动画形式自下向上弹出，同时使底层VC变暗；  
		>不提供返回按钮。  
	* Present as Popover  
		>在 iPad 中，目标VC以浮动窗样式呈现，点击目标VC以外区域，目标VC消失；在 iPhone 中，默认目标VC以模态覆盖整个屏幕。  
	* Exit Segue  
		>在需要返回到的界面实现方法 @IBAction func xxxx(segue: UIStoryboardSegue)，方法名字任意，参数类型必须为 UIStoryboardSegue，一定要加上 @IBAction 关键字。  
		>右键返回界面上方的 Exit，弹出菜单中可以看到刚才在返回到的界面中加的那个方法的名称；   
		>连线，选择实现动作的控件，或者连接到当前界面选择手动代码实现返回（需要指定 identifier）；  
		>使用 exit segue 的好处是可以跳转到任意打开过的界面比如从3->1，而不是只能返回上级界面从2->1。
	* Embed Segue
		>Container View，将容器 MVC 的 View 放入到另一个 MVC 中。

* 获取 storyboard 的实例：UIStoryboard(name: <#T##String#>, bundle: <#T##Bundle?#>)。

* 获取 storyboard 中 VC 的实例，通过 VC 的 Storyboard ID：  
	```swift
	UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "");
	```

* 通过 storyboard reference，可以拆分成多个 storyboard，方便开发。  
选中需要拆分的 VC，菜单栏干中 Editor->Refactor to Storyboard。

* 加载其他 storyboard 的指定 VC：  
拖入控件 Storyboard Reference；  
填写 storyboard 的名字；  
填写 Reference ID，即目标 storyboard 中目标VC的 Storyboard ID。  

* UIStackView，快速进行水平或者垂直布局。

* 如何对两个已经和边界拉好约束的相邻控件进行约束：  
先设置约束距离为0，然后设置约束  greater than or equal。

* 关键字 IBDesignable，可以将 code 实现的 view 显示到 storyboard 上。  
如果有 image，需要使用下面的方法，才能正确显示  
	```swift
	UIImage(named: "xxx", in: Bundle(for: self.classForCoder), compatibleWith: traitCollection);
	```

* 关键字 IBInspectable，可以将代码写好的 property 显示到 storyboard 右边的 inspect 上。

* setNeedsDisplay()，当需要重新绘制的时候，需要主动调用这个方法，不要调用 UIView 的 draw() 方法。

* setNeedsLayout()，当需要 subView 重新布局的时候，调用这个方法。

* 如果 UIView 需要重新绘制，content mode 需要设置为 Redraw 。

* 当 UITraitCollection 改变时，系统会调用此方法，用来更新UI。在方法里实现 setNeedsDisplay() 和 setNeedsLayout()。  
	```objectivec
	//To be overridden as needed to provide custom behavior when the environment's traits change.
	-(void)traitCollectionDidChange:(nullable UITraitCollection *)previousTraitCollection;
	```
	e.g. UIImage(named: "name", in: Bundle(for: self.classForCoder), compatibleWith: traitCollection)，并使用draw绘制图片，此时屏幕翻转，需要重新绘制，系统会调用上面方法。

* 子视图的布局写在此方法中，当视图需要重新绘制布局时，调用 setNeedsLayout()，会自动调用此方法。  
	```objectivec
	//override point. called by layoutIfNeeded automatically.
	// As of iOS 6.0, when constraints-based layout is used the base implementation applies the constraints-based layout, otherwise it does nothing.
	-(void)layoutSubviews;
	```

* 自定义绘制实现的方法，当视图需要重新绘制（Redraw）的时候调用 setNeedsDisplay()，会自动调用此方法。
	```objectivec
	-(void)drawRect:(CGRect)rect;
	```

* Core Graphics属于更底层  
	UIKit依赖于Core Graphics框架  
	UIBezierPath属于UIKit，是对Core Graphics框架中的CGPath的封装
	补充：UIGraphicsGetCurrentContext()画出的线，被使用了就被消耗掉。

* 三种动画方式：  
	* UIViewPropertyAnimator(iOS 10.0)  
		>iOS 10.0，apple 封装好的动画API，更精细的控制动画，可以通过 startAnimation、 pauseAnimation 和 stopAnimation 控制动画。
	* Transitions  
	* Dynamic Animator  
		>给对象添加各种物理属性（重力、加速度等）  

* Auto Layout
	* 通过 Interface Builder 来实现是好的选择；  
	* 通过代码实现 Auto Layout；  
		>在 -init 或者 -viewDidLoad 方法中，实现固定不变的约束  
		>在 -updateConstraints 中，实现程序运行时可能改变的约束  
		>当需要改变约束的时候，按顺序调用下面方法  
		```swift
		setNeedsUpdateConstraints()  
		updateConstraintsIfNeeded()  
		```  
	* view 的子类布局同理

