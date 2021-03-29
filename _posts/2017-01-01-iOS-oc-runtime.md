---
layout:		post
title:		"iOS-OC-Runtime"
description: "What's a meta-class"
date:		2017-01-01
author:		"Yawei"
categories: ["iOS", "OC", "Runtime"]
keywords:
    - iOS
    - OC
    - Runtime
    - meta-class
---


# 先解释几个名词（区分大小写）

* object: 对象,class的实例，每一个object都有一个自己的class
```C
// object的基础定义长这样
typedef struct objc_object {
    // Class定义了class，因为每一个object的Class结构包含了不同的属于这个object的函数（消息）列表
    // isa作为指针告诉runtime去哪里找到这个object的class定义，为什么isa是指针看下面Class的定义。
    Class isa;
} *id;
```
* class: 类，作为面向对象的定义，一个object就应该有对应的自己的class。这个class应该包含属性(但oc中属性也是函数，或者说消息)、函数等
```C
// class的基础定义是这样
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    /* followed by runtime specific details... */
};
```
* Class: Class定义了class，Class包含了这个object的函数（消息）列表。Class的定义也在上面了，Class本身也是一个object，所以必须和objc_object一样以`Class isa;`开始。然后接下来为了提供继承能力，提供了`super_class`属性。
* meta-class: Class object的class。Class object的isa也必须指向一个Class来提供函数列表供我们可以调用。meta-class也是一个object(下面解释）。

> 所以，这里我可以得出一个结论，如果我么给object发消息，那么会去object's class中寻找函数列表；如果我们向class发消息，那么回去meta-class中寻找函数列表

下面开始概念上划等号： Class = class，object' class = class, class's class = meta-class

也就是: object的函数信息存储在class中，class的函数信息存储在meta-class中。

# meta-class's class?

前面既然说，meta-class也是对象，那也应该和Class一样有`isa, super_class`。

## meta-class的isa指向哪里？
runtime将所有的meta-class的class都统一使用`base class's meta-class`，也就是其继承层次结构中最上级的Class的meta-class。

所以，在OC中，这意味着所有从NSObject派生出来的class的meta-class的class都是NSObject的meta-class。任何base class's meta-class的class其实都是其本身，比如NSObject的meta-class的class也是NSObject的meta-class，也就是说NSObject的meta-class的isa指向自身


## meta-class的super_class指向哪里？

class的super-class指向其父类，同样，meta-class的super-class指向class的父类的meta-class.而base class的meta-class的super-class指向base class本身。

这导致所有的object、class、meta-class都有一个共同的基类。大部分都是NSObject。

![class-diagram](/img/post/2017-01-01/runtime-class-diagram.jpg)