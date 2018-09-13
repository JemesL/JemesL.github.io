---
title: WkWebview的使用
date: 2018-09-13 15:47:55
tags:
---

根据项目需求，需要webview和自定义的组件根据 API 返回数据动态混排。
并且需要针对css以及图片懒加载优化。

### 一、添加css
这一点比较简单，webview所显示的内容是通过接口返回获得的字符串。
做一个简单的拼接就可以。

```Swift
let link = "<link rel=\"stylesheet\" href=\"\(String(describing: Bundle.main.url(forResource: "RichStyle", withExtension: "css")!))\">"
let htmlContent = """
<header>
<meta name='viewport' content='width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no' />
\(link)
</header>
<div class='richtext-container'>
\(newContent)
</div>
"""
```
**newContent** 是内容正文
**link** 是css样式
其中由于显示问题，需要在头部添加 **meta** 这一段。
由于css比较多，放在一起不太合适。可以写在另一个文件中，通过 **Bundle** 形式加载。
这里有个需要注意的地方，css文件存放的位置。
此刻RichStyle.css 存放在Classes/Utils/ 下。
要在加载html 时，指定在 **file:///Classes/Utils/** 目录下。
```Swift
webView.loadHTMLString(htmlContent, baseURL: URL(string: "file:///Classes/Utils/") )
```

### 二、图片懒加载
如果是单个webview，且视窗大小固定。可以使用 **lazysizes**，纯js原生，不依赖任何其他库。
**lazysizes** 的逻辑是webview 视窗之外的图片懒加载，滑动到视窗的img 才会加载。
但项目中是多个webview 及其他组件 全展开显示在scrollview里面。
webview 视窗高度根据内容高度来的。导致 **lazysizes**依旧还是在加载同时把所有页面图片加载完，不过加载图片是在 **didFinish**之后。

目前的解决办法就是：
* 在scrollview 中针对滚动代理方法优化，让页面停止下来时才开始触发加载图片的方法。
* 触发的方法 会让scrollview中 所有的webview 调用 判断是否加载图片的js脚本。并且会传入当前scrollview的contentOffset 和 webview的frame.minY
* js脚本拿到当前webview 当中所有的imgs，遍历每一个img，并根据img的topoff 和webview 的偏移参数计算这个img 是否是在scrollview的视窗内。
* 然后判断修改img的src 加载图片。

##### 一、
```Swift
func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
    if !decelerate {
        let dragTpDragStop = scrollView.isTracking && !scrollView.isDragging && !scrollView.isDecelerating
        if dragTpDragStop {
            scrollViewDidEndScroll()
        }
    }
}

func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
    let scrollToScrollStop = !scrollView.isTracking && !scrollView.isDragging && !scrollView.isDecelerating
    if scrollToScrollStop {
        scrollViewDidEndScroll()
    }
}

func scrollViewDidEndScroll() {
    let scrollOffset = scrollView.contentOffset.y
    for rh in allRichText.enumerated() {// rh 是一个包含 webview属性的组件
        rh.element.loadImgsScript(offset:scrollOffset, wkTop: rh.element.frame.minY)
    }
}
```
#####  二、

```Swift
func loadImgsScript(offset: CGFloat, wkTop: CGFloat) {
    let minOffset = offset
    let maxOffset = offset + bmy_screenHeight
    let script = """
        var imgs = document.getElementsByTagName('img');
        var minOffset = \(minOffset)
        var maxOffset = \(maxOffset)
        var wkTop = \(wkTop)
        function lazyload(){
            for(var i=0; i<imgs.length; i++) {
                var img = imgs[i]
                var top = img.offsetTop + wkTop
                var isDisplay = false
                if((top > minOffset - 300) && top < maxOffset + 300) {
                    isDisplay = true
                }
                if(isDisplay){
                    img.src = img.getAttribute('data-src');
                    img.setAttribute('class','lazyloaded bmy-tag bmy-tag_img');
                }
            }
        }
        lazyload()
    """
    webView.evaluateJavaScript(script) { (res, error) in
        if let e = error {
            print(error)
        }
    }
}
```