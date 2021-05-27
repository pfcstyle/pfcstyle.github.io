---
layout:		post
title:		"SwiftUI Property Wrapper使用注意"
description: "坑不少"
date:		2021-05-26
author:		"Yawei"
categories: ["swift", "SwiftUI"]
keywords:
    - iOS
    - swift
    - SwiftUI
    - "@Binding"
---

1. @Binding改变不会触发SwiftUI的更新, 这只取决于根变量是否是类似@State, Publisher或者ObservableObject等
2. @State可以再init中初始化
```
//这俩暂未发现不同
_stateVar = State(initialValue: settingItem.wrappedValue)
_stateVar = State(wrappedValue: settingItem.wrappedValue)
```
3. 使用@Binding希望内部组件更新
```
@Binding var settingItem: SparkFormItem{
    willSet{
        settingItemState = newValue // settingItemState是@State变量
    }
}
```
4. @Binding定义的变量，在初始化时，不会改变wrapper值
```
@Binding var settingItem: SparkFormItem
init(settingItem: Binding<SparkFormItem>) {
    _settingItem = settingItem // 这里_settingItem的初始化不会改变self.settingItem的值
}
```

5. @environment和@environmentObject同样不会触发ui的刷新，需要Publisher或者ObservableObject等支持
@environment和@environmentObject可以在整个view结构中共享，但是遇到navigation link时就会断掉