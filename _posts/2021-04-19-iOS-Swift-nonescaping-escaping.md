---
layout:		post
title:		"@escaping vs @nonescaping"
description: ""
date:		2021-04-19
author:		"Yawei"
categories: ["iOS", "swift"]
keywords:
    - iOS
    - swift
    - escaping
    - nonescaping
---

# @escaping closure

从字面意思理解，可逃离闭包。即是闭包传入函数，在函数执行完毕后，这个闭包仍然会存在于内存中，直到闭包被执行完毕。

# @nonescaping closure (默认)

相反，函数执行完毕后，闭包会被立刻释放

# 如何使用

异步函数的闭包，就使用`@escaping`，比如网络请求，response应该放在`@escaping`闭包中。同步函数则直接使用`@nonescaping`