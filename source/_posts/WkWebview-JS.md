---
title: WkWebview 与 js 交互
date: 2018-09-13 11:50:38
tags:
---

最近由于项目的原因，需要wkwebview 与 js 频繁的交互
主要有以下两种
* Content-Blocking Rules
* 调用wkwebview 的js执行脚本的方法
* JS 调用 swift

### 一、Content-Blocking Rules
如果是iOS11 之后的并且需求简单 wkwebview 提供了一个内容过滤规则的功能。可以简单处理一些操作。这个的好处是webview显示内容之前就可以处理。

```Swift
let jsonString = """
[{
    "trigger":{
        "url-filter": ".*",
        "resource-type":["document"]
    },
    "action":{
        "type": "css-display-none",
        "selector": ".js-mp-info"
    }
}]
"""
if #available(iOS 11.0, *) {
    WKContentRuleListStore.default().compileContentRuleList(forIdentifier: "demoRuleList", encodedContentRuleList: jsonString) { (list, error) in
        guard let contentRuleList = list else { return }
        let configuration = self.webView.configuration
        configuration.userContentController.add(contentRuleList)
        if let detailUrl = self.detailUrl {
            self.urlStr = detailUrl
        }
    }
}
```
url-filter 可以用正则匹配url 是否触发规则过滤
resource-type 是适用资源类型
* document
* image
* style-sheet
* script
* font
* raw (Any untyped load, such as XMLHttpRequest)
* svg-document
* media
* popup

action type 是动作类型
* block
Tells the browser engine to abort loading the resource. If the resource was cached, the cache is ignored.

* block-cookies
The engine strips all cookies from the header before sending the request to the server. Safari's own privacy policy takes precedence, so that only cookies that would otherwise be accepted by the privacy policy can be blocked. Combining block-cookies and ignore-previous-rules will not override the browser’s privacy settings.

* css-display-none
Hides elements of the page based on a CSS selector. A second action field, named selector, contains the selector list. Any element matching the selector list has its display property set to none, which hides it.

* ignore-previous-rules
Previously triggered actions are not performed.

* make-https
Changes a URL from http to https before making a server request. URLs with a specified port (other than the default port 80) and links using other protocols are not affected.

[**Content-Blocking Rules** Apple官方文档使用说明](https://developer.apple.com/library/archive/documentation/Extensions/Conceptual/ContentBlockingRules/CreatingRules/CreatingRules.html)

### 二、调用wkwebview 的js执行脚本的方法

如果版本小于ios11，没有Content-Blocking Rules，或者规则不能满足需求。
也可以通过调用 **wk.evaluateJavaScript** 方法, 直接执行js。
```Swift
let str = "$('.js-mp-info').remove();"
webView.evaluateJavaScript(str, completionHandler: nil)
```

JS代码除了直接写成string之外，也可以在另写在文件里，然后读取文件。
例如 JS相关代码写在 **LoginJavaScript.js** 文件里
```Swift
var jsHandler = ""
do {
    jsHandler = try String(contentsOf: Bundle.main.url(forResource: "LoginJavaScript", withExtension: "js")!, encoding: String.Encoding.utf8)
} catch {}
let wkScript = WKUserScript.init(source: jsHandler,
                                  injectionTime: WKUserScriptInjectionTime.atDocumentEnd,
                                  forMainFrameOnly: true)
config.userContentController.addUserScript(wkScript)
let wk = WKWebView.init(frame: .zero, configuration: config)
```

### 三、JS 调用 swift

#### 预定义
wkwebview的 config 可以事先定义一些事件
```Swift
let config = WKWebViewConfiguration()
config.userContentController.add(self, name: "eventOne")
config.userContentController.add(self, name: "eventTwo")
```

#### JS调用
然后在JS 脚本里面调用
```JavaScript
webkit.messageHandlers.eventOne.postMessage(message);
```
其中 **eventOne** 就是之前事先定义好的事件名字。
**message** 就是 js 向 swift 传递的信息

#### Swift 代理方法监听
实现 **WKScriptMessageHandler** 下的一个代理方法，在方法里面监测到JS触发的预定义的事件
```Swift
func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
  switch message.name {
  case "eventOne":
      print("eventOne")
      print(message.body)//js传来的信息
  case "eventTwo":
      print("eventTwo")
  default:
      print("none")
  }
}

```