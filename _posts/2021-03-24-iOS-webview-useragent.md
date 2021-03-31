---
layout:		post
title:		"WKWebview set user agent and JS handler"
description: ""
date:		2021-03-24
author:		"Yawei"
categories: "iOS"
keywords:
    - iOS
    - copyWithZone
    - NSCopying
---

> 注意，下面的代码都不要在webview init中写，不会生效，需要在view did load之后才生效

直接上代码

# 设置JS Handler

```Swift
// 防止循环应用
class LeakAvoider: NSObject {
    weak var delegate: WKScriptMessageHandler?

    init(delegate: WKScriptMessageHandler) {
        super.init()
        self.delegate = delegate
    }
}

extension LeakAvoider: WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        delegate?.userContentController(userContentController, didReceive: message)
    }
}

// 我这里继承的WKWebview, delegate的实现也可以放view controller
extension SparkPageWebView: WKScriptMessageHandler{
    public func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        // message.body.type
    }
}

// 最后为webview设置代理, 注意要在view did load之后设置才会生效
self.configuration.userContentController.add(LeakAvoider(delegate: self), name: "xxx")

```

# 设置user agent

```swift
customUserAgent = Custom_User_Agent
```