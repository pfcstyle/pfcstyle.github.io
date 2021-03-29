---
layout:		post
title:		"iOS-OC-Runtime"
description: "常用方法"
date:		2017-01-07
author:		"Yawei"
categories: ["iOS", "OC", "Runtime"]
keywords:
    - iOS
    - OC
    - Runtime
---

# 常用方法

## Class 反射创建
通过字符串创建类：Class

```objc
// 方式1
NSClassFromString(@"NSObject");

// 方式2 
objc_getClass("NSObject");
```

## SEL 反射创建
通过字符串创建方法 selector

```objc
// 方式1
@selector(init);

// 方式2
sel_registerName("init");

// 方式3
NSSelectorFromString(@"init");
```

## 方法替换/交换
- 方法替换：`class_replaceMethod`
- 方法交换：`method_exchangeImplementations`

```objc
// 方法替换
- (void)methodReplace
{
    Method methodA = class_getInstanceMethod(self.class, @selector(myMethodA));
    IMP impA = method_getImplementation(methodA);
    class_replaceMethod(self.class, @selector(myMethodC), impA, method_getTypeEncoding(methodA));
    
    // print: myMethodA
    [self myMethodC];
}

// 方法交换
- (void)methodExchange
{
    Method methodA = class_getInstanceMethod(self.class, @selector(myMethodA));
    Method methodB = class_getInstanceMethod(self.class, @selector(myMethodB));
    method_exchangeImplementations(methodA, methodB);
    
    // print: myMethodB
    [self myMethodA];
    
    // print: myMethodA
    [self myMethodB];
}

- (void)myMethodA
{
    NSLog(@"myMethodA");
}

- (void)myMethodB
{
    NSLog(@"myMethodB");
}

- (void)myMethodC
{
    NSLog(@"myMethodC");
}
```

## 新增类
通过字符串动态新增一个类

1. 首先创建新类：`objc_allocateClassPair`
2. 然后注册新创建的类：`objc_registerClassPair`

这里有个小知识点，为什么类创建的方法名是`objc_allocateClassPair`，而不是`objc_allocateClass`呢？这是因为它同时创建了一个类(class)和元类(metaclass)。关于元类可以看这篇文章：[What is a meta-class in Objective-C?](https://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [self addNewClassPair];
    
    Class MyObject = NSClassFromString(@"MyObject");
    NSObject *myObj = [[MyObject alloc] init];
    [myObj performSelector:@selector(sayHello)];

    return YES;
}

- (void)addNewClassPair
{
    Class myCls = objc_allocateClassPair([NSObject class], "MyObject", 0);
    objc_registerClassPair(myCls);
    [self addNewMethodWithClass:myCls];
}
```

## 新增方法

新增方法：`class_addMethod`

这里也有个小知识点，就是使用特定字符串描述方法返回值和参数，例如：`v@:`。其具体映射关系请移步：[Type Encodings](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)

```objc
void sayHello(id self, SEL _cmd)
{
    NSLog(@"%@ %s", self, __func__);
}

- (void)addNewMethodWithClass:(Class)targetClass
{
    class_addMethod(targetClass, @selector(sayHello), (IMP)sayHello, "v@:");
}
```

## 消息转发

当给对象发送消息时，如果对象没有找到对应的方法实现，那么就会进入正常的消息转发流程。其主要流程如下：

```objc
// 1.运行时动态添加方法
+ (BOOL)resolveInstanceMethod:(SEL)sel 
 
// 2.快速转发
- (id)forwardingTargetForSelector:(SEL)aSelector
 
// 3.构建方法签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector

// 4.消息转发
- (void)forwardInvocation:(NSInvocation *)anInvocation

```

其中最后的`forwardInvocation:`会传递一个`NSInvocation`对象（**Ps：NSInvocation 可以理解为是消息发送`objc_msgSend(void id self, SEL op, ...  )`的对象**）。NSInvocation 包含了这个方法调用的所有信息：selector、参数类型、参数值和返回值类型。此外，你还可以去更改参数值和返回值。

**除了上面的正常消息转发，我们还可以借助`_objc_msgForward`方法让消息强制转发。**

```objc
Method methodA = class_getInstanceMethod(self.class, @selector(myMethodA));
IMP impA = method_getImplementation(methodA);
IMP msgForwardIMP = _objc_msgForward;

// 替换 myMethodA 的实现后，每次调用 myMethodA 都会进入消息转发
class_replaceMethod(self.class, @selector(myMethodC), msgForwardIMP, method_getTypeEncoding(methodA));
```

## Method 调用方式

1. 常规调用
2. 反射调用
3. objc_msgSend 
4. C 函数调用
5. NSInvocation 调用

```objc
@interface People : NSObject

- (void)helloWorld;

@end

// 常规调用
People *people = [[People alloc] init];
[people helloWorld];

// 反射调用    
Class cls = NSClassFromString(@"People");
id obj = [[cls alloc] init];
[obj performSelector:NSSelectorFromString(@"helloWorld")];

// objc_msgSend
((void(*)(id, SEL))objc_msgSend)(people, sel_registerName("helloWorld"));

// C 函数调用
Method initMethod = class_getInstanceMethod([People class], @selector(helloWorld));
IMP imp = method_getImplementation(initMethod);
((void (*) (id, SEL))imp)(people, @selector(helloWorld));

// NSInvocation 调用
NSMethodSignature *sig = [[People class] instanceMethodSignatureForSelector:sel_registerName("helloWorld")];
NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
invocation.target = people;
invocation.selector = sel_registerName("helloWorld");
[invocation invoke];
```

第五种 **`NSInvocation 调用`** 是热修复调用任意OC方法的核心基础。通过 NSInvocation 不但可以自定义函数的参数值和返回值，而且还可以自定义方法：`selector` 和消息接收对象：`target`。因此，我们可以通过字符串的方式构建任意OC方法调用。
