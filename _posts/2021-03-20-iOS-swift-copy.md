---
layout:		post
title:		"-[** copyWithZone:] unrecognized selector sent to instance"
description: ""
date:		2021-03-20
author:		"Yawei"
categories: "iOS"
keywords:
    - iOS
    - copyWithZone
    - NSCopying
---

很多OC的class, 内部属性声明使用的@property (nonatomic, **copy**),导致在赋值使用时，自动调用copyWithZone:方法，如果这时传入的对象没有实现这个方法，就会报找不到的错误。

# 解决方法

## 第一种方法：实现copyWithZone

```
class SparkAppService: NSCopying{
    var vendorId: NSCopying
    func copy(with zone: NSZone? = nil) -> Any {
        let c = SparkAppService()
        c.vendorId = self.vendorId.copy(with: zone) as! NSCopying
        return c
    }
}
```

## 第二种方法：继承NSObject
```
@objc class SparkAppService: NSObject{
    override init() {
        super.init()
    }
}
```

