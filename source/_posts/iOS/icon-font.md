---
title: icon font
date: 2019/1/1
tags: iOS
comments: true
---

icon font  
<!--more-->

优点：  
1、减少包的体积  
2、无需2x或者3x  
3、颜色修改方便准确  
缺点：  
1、只有单色  
2、使用时需要使用 unicode 字符  
3、维护 ttf 字体库  

<!--more-->

## 获取 iconfont

* 1 [iconfont.cn](https://www.iconfont.cn/)  
	Iconfont阿里的矢量图标库，可以直接使用里面的免费的iconfont，设计师也可以将自己的矢量图 svg 上传到 iconfont，在线管理。
	开发人员将需要的 iconfont 加入购物车，下载代码，找到 ttf 文件；  
* 2 本地管理  
	使用第三方工具，如 iconfount、FontForge，通过 svg 矢量图，生成 ttf 文件。

## iOS 开发使用 iconfont

* 1 注册字体  
	iOS中加入字体文件，需要先把 ttf 文件加入项目，然后注册字体。  
	注册字体分两种：  
	1、info.plist 中， Fonts provided by application  
	2、使用 API 注册  
	建议使用 API 注册，方便下载字体注册  
	```swift
	extension UIFont {
    		static func registerFont(fontName: String) {
        		if let url = Bundle.main.url(forResource: fontName, withExtension: "ttf"),
            			let fontDataProvider = CGDataProvider(url: url as NSURL),
            			let newFont = CGFont(fontDataProvider) {
            				CTFontManagerRegisterGraphicsFont(newFont, nil)
        			}
    			}
	}
	```

* 2 使用 iconfont  
	一般采用 UIImage 的形式使用
	```swift
	extension UIImage {
    		static func image(iconCode: String, fontName: String, size: CGFloat, color: UIColor) -> UIImage? {
        		let imageSize = CGSize(width: size, height: size)
        		UIGraphicsBeginImageContextWithOptions(imageSize, false, UIScreen.main.scale)
        		let label = UILabel(frame: CGRect(x: 0, y: 0, width: size, height: size))
        		label.font = UIFont(name: fontName, size: size)
        		label.text = iconCode
        		label.textColor = color
        		label.layer.render(in: UIGraphicsGetCurrentContext()!)
        		let retImage = UIGraphicsGetImageFromCurrentImageContext()
        		return retImage
    		}		
	}
	```

	iconfont 使用时，需要使用 Unicode 编码，打开从 iconfont.cn 下载的文件夹，找到 html 文件，打开，会看到 &#xXXXX 的 Unicode 编码。  
	在 Swift 中，使用 \u{XXXX} ,将上面的 XXXX 替换一下即可。  
	建议在工程中，维护一个 enum ，方便使用。  
	在 iconfont.cn 的项目页面，打开 chrome 开发者工具的 console ，执行如下 JavaScript 脚本，获取枚举结构。  
	```swift
	function camelCase(text, separator) {
    		var arr = text.split(separator);
    		for(var i = 1; i < arr.length; i++) {
        		  var s = arr[i].slice(0, 1).toUpperCase(); 
        		  var h = arr[i].slice(1);
        		  arr[i] = s + h;
    		}
    		return arr.join('')
	}

	var items = document.getElementsByClassName('icon-item')
	var result = ""
	for(let i = 0, len = items.length; i < len; i++) {
    		let item = items[i]
    		var name = item.getElementsByClassName('icon-name')[0].innerHTML
    		name = camelCase(name, ' ')
    		name = camelCase(name, '-')
    		var code = item.getElementsByClassName('icon-code')[0].innerHTML
    		code = code.replace('&amp;#x', '')
    		code = code.replace(';', '')
    		result = result + "case " + name + " = " + "\"\\u{" + code + "}\"" + "\n"
	}
	console.log(result)
	```

## EFIconFont

在 GitHub 上，找到了一个第三方库 EFIconFont ，可以辅助开发在工程中使用 iconfont。  
[EFIconFont](https://github.com/XiangWuShuo/EFIconFont) 
