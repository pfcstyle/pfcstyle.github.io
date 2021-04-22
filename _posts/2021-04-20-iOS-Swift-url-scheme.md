---
layout:		post
title:		"iOS URL Scheme 如何配置"
description: "配置第三方登录时，需要配置URL Scheme才能从第三方登录或者分享返回"
date:		2021-04-22
author:		"Yawei"
categories: ["iOS", "swift"]
keywords:
    - iOS
    - swift
    - URL Schemes
---

# info.plist

选择Open As -> Source code, 复制如下内容(dict标签下)：

其中`CFBundleURLName`需要参考接入的第三方要求，比如wechat是'wx+AppId', 有的则可以是任意值; 最重要的是`CFBundleURLSchemes`, 需要配置与第三方后台的RedirectURI一致才可以。

```
<dict>
    <!-- *** ADD *** -->
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>identifyId</string>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>my-app</string>
        </array>
      </dict>
    </array>
```

# AppDelegate.swift
下面的函数会在从第三方登录回到本app时， 并且没有使用SceneDelegate时执行， 如果使用了SceneDelegate， 参考下面的SceneDelegate.swift。判断依据是url是否是你自己配置的url scheme
```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    if let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false),
        AppConfiguration.urlScheme == urlComponents.scheme,
        AppConfiguration.urlAuthPath == urlComponents.host {
        //这里以arcgis runtime为例
        AGSApplicationDelegate.shared().application(app, open: url, options: options)
    }
    return true
}
```

# SceneDelegate.swift
如果使用了SceneDelegate, 则需要在SceneDelegate.swift的回调中处理

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url{
            if let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false),
               AppConfiguration.urlScheme == urlComponents.scheme,
               AppConfiguration.urlAuthPath == urlComponents.host {
                let contextOptions = URLContexts.first?.options
                var options: [UIApplication.OpenURLOptionsKey: Any] = [:]
                if contextOptions?.openInPlace != nil {
                    options[.openInPlace] = contextOptions!.openInPlace
                }
                if contextOptions?.annotation != nil {
                    options[.annotation] = contextOptions!.annotation
                }
                if contextOptions?.sourceApplication != nil {
                    options[.sourceApplication] = contextOptions!.sourceApplication
                }
               AGSApplicationDelegate.shared().application(UIApplication.shared, open: url, options: options)
           }
        }
        
    }
```