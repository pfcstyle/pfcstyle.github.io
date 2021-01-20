---
layout:		post
title:		"iOS项目总结(九)"
subtitle:	"iOS监听网络状态变化"
date:		2016-07-04
author:		"PfCStyle"
header-img:	"img/post/2016-07-04/head.jpg"
tags:
    - iOS
    - 网络状态监听
    - Reachability
    - NetWork
---

> 不积跬步，无以至千里；不积小流，无以成江海；

苹果官方提供了[Reachbility](https://developer.apple.com/library/ios/samplecode/Reachability/Listings/Reachability_Reachability_m.html#//apple_ref/doc/uid/DTS40007324-Reachability_Reachability_m-DontLinkElementID_8)可以直接用来判断网络状态，这里分析一下。

Reachability主要使用的是`<SystemConfiguration/SystemConfiguration.h>`，所以你在使用的时候应该添加SystemConfiguration.framework文件依赖。我们倒着分析：

```C
//很明显这里是作为一个回调函数，其功能是在网络状态发生变化的时候就发出通知
static void ReachabilityCallback(SCNetworkReachabilityRef target, SCNetworkReachabilityFlags flags, void* info)
{
#pragma unused (target, flags)
	NSCAssert(info != NULL, @"info was NULL in ReachabilityCallback");
	NSCAssert([(__bridge NSObject*) info isKindOfClass: [Reachability class]], @"info was wrong class in ReachabilityCallback");

    Reachability* noteObject = (__bridge Reachability *)info;
    // Post a notification to notify the client that the network reachability changed.
    [[NSNotificationCenter defaultCenter] postNotificationName: kReachabilityChangedNotification object: noteObject];
}
```

接下来我们按图索骥，找到这个回调是在哪里执行。

```Objective-C
//这里是开始通知的函数，可以看到，上面的回调就是在这里设置的
- (BOOL)startNotifier
{
	BOOL returnValue = NO;
	SCNetworkReachabilityContext context = {0, (__bridge void *)(self), NULL, NULL, NULL};
//第一个参数是主机地址的句柄，用于监听主机状态
//第二个参数就是回调了
	if (SCNetworkReachabilitySetCallback(_reachabilityRef, ReachabilityCallback, &context))
	{
	//使监听在mainrunloop中运行  异步
		if (SCNetworkReachabilityScheduleWithRunLoop(_reachabilityRef, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode))
		{
			returnValue = YES;
		}
	}
    
	return returnValue;
}
```

好了，流程就是这么简单，我们再看一下初始化

```Objective-C
+ (instancetype)reachabilityWithHostName:(NSString *)hostName
{
	Reachability* returnValue = NULL;
	//这里是创建主机地址的操作句柄，第一个参数是分配器，null表示使用默认，第二个参数就是主机地址了
	SCNetworkReachabilityRef reachability = SCNetworkReachabilityCreateWithName(NULL, [hostName UTF8String]);
	if (reachability != NULL)
	{
		returnValue= [[self alloc] init];
		if (returnValue != NULL)
		{
			returnValue->_reachabilityRef = reachability;
			returnValue->_alwaysReturnLocalWiFiStatus = NO;
		}
	}
	return returnValue;
}
```

调用当前状态


```Objective-C
//这里主要是分析flag的去向
- (NetworkStatus)currentReachabilityStatus
{
	NSAssert(_reachabilityRef != NULL, @"currentNetworkStatus called with NULL SCNetworkReachabilityRef");
	NetworkStatus returnValue = NotReachable;
	SCNetworkReachabilityFlags flags;
    
	if (SCNetworkReachabilityGetFlags(_reachabilityRef, &flags))
	{
		if (_alwaysReturnLocalWiFiStatus)
		{
			returnValue = [self localWiFiStatusForFlags:flags];
		}
		else
		{
			returnValue = [self networkStatusForFlags:flags];
		}
	}
    
	return returnValue;
}
```

flag的去向，我们找一个讲解

```Objective-C
//可以看到，这里都是与kSCNetworkReachabilityFlags进行与操作，判断是否是对应状态。
- (NetworkStatus)localWiFiStatusForFlags:(SCNetworkReachabilityFlags)flags
{
	PrintReachabilityFlags(flags, "localWiFiStatusForFlags");
	NetworkStatus returnValue = NotReachable;

	if ((flags & kSCNetworkReachabilityFlagsReachable) && (flags & kSCNetworkReachabilityFlagsIsDirect))
	{
		returnValue = ReachableViaWiFi;
	}
    
	return returnValue;
}
```









