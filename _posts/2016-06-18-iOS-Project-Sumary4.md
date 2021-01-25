---
layout:		post
title:		"iOS项目总结(四)"
description	"让你的应用回光返照-signal"
date:		2016-06-18
author:		"PfCStyle"
header-img:	"img/post/2016-06-18/head.jpg"
categories: "iOS"
keywords
    - iOS
    - OC
    - signal
    - runloop
    - exception
---

> 不积跬步，无以至千里；不积小流，无以成江海；

今天在应用里添加一个异常捕获类，以方便测试人员在没有xcode的情况下也可以直接看到错误日志。先说一下原理：

# Signal是什么

iOS SDK中提供了一个现成的函数 NSSetUncaughtExceptionHandler 用来做异常处理，但功能非常有限，而引起崩溃的大多数原因如：内存访问错误，重复释放等错误就无能为力了。因为这种错误它抛出的是Signal，所以必须要专门做Signal处理。

在计算机科学中， 信号 （ 英语： Signals）是Unix、类Unix以及其他POSIX兼容的操作系统中进程间通讯的一种有限制的方式。它是一种异步的通知机制，用来提醒进程一个事件已经发生。当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，此时，任何非原子操作都将被中断。如果进程定义了信号的处理函数，那么它将被执行，否则就执行默认的处理函数。

信号处理函数可以通过  signal() 系统调用来设置。如果没有为一个信号设置对应的处理函数，就会使用默认的处理函数，否则信号就被进程截获并调用相应的处理函数。在没有处理函数的情况下，程序可以指定两种行为：忽略这个信号  SIG_IGN 或者用默认的处理函数  SIG_DFL 。但是有两个信号是无法被截获并处理的：  SIGKILL、SIGSTOP 。

### 信号的类型
- SIGABRT--程序中止命令中止信号
- SIGALRM--程序超时信号
- SIGFPE--程序浮点异常信号
- SIGILL--程序非法指令信号
- SIGHUP--程序终端中止信号
- SIGINT--程序键盘中断信号
- SIGKILL--程序结束接收中止信号
- SIGTERM--程序kill中止信号
- SIGSTOP--程序键盘中止信号
- SIGSEGV--程序无效内存中止信号
- SIGBUS--程序内存字节未对齐中止信号
- SIGPIPE--程序Socket发送失败中止信号

# 如何实现

废话不多说了，直接上代码，不懂的看注释就可以了。

{% raw %}

```Objective-C
//UncaughtExceptionHandl.h
#import <UIKit/UIKit.h>

@interface UncaughtExceptionHandl : NSObject{
	BOOL dismissed;
}

@end
void HandleException(NSException *exception);
void SignalHandler(int signal);

void InstallUncaughtExceptionHandler(void);
```

{% endraw %}

{% raw %}

```Objective-C
//UncaughtExceptionHandl.m

#import "UncaughtExceptionHandl.h"
#include <libkern/OSAtomic.h>
#include <execinfo.h>
#import "AppDelegate.h"

NSString * const UncaughtExceptionHandlerSignalExceptionName = @"UncaughtExceptionHandlerSignalExceptionName";
NSString * const UncaughtExceptionHandlerSignalKey = @"UncaughtExceptionHandlerSignalKey";
NSString * const UncaughtExceptionHandlerAddressesKey = @"UncaughtExceptionHandlerAddressesKey";
//当前处理的异常个数
volatile int32_t UncaughtExceptionCount = 0;
//能够处理的最大异常个数
const int32_t UncaughtExceptionMaximum = 10;

const NSInteger UncaughtExceptionHandlerSkipAddressCount = 4;
const NSInteger UncaughtExceptionHandlerReportAddressCount = 5;
@interface UncaughtExceptionHandl()
//计时器
@property (strong, nonatomic) NSTimer *countDurTimer;

@end


@implementation UncaughtExceptionHandl

+ (NSArray *)backtrace
{
    void* callstack[128];
    int frames = backtrace(callstack, 128);
    char **strs = backtrace_symbols(callstack, frames);
    
    int i;
    NSMutableArray *backtrace = [NSMutableArray arrayWithCapacity:frames];
    for (
         i = UncaughtExceptionHandlerSkipAddressCount;
         i < UncaughtExceptionHandlerSkipAddressCount +
         UncaughtExceptionHandlerReportAddressCount;
         i++)
    {
        [backtrace addObject:[NSString stringWithUTF8String:strs[i]]];
    }
    free(strs);
    
    return backtrace;
}

- (void)alertView:(UIAlertView *)anAlertView clickedButtonAtIndex:(NSInteger)anIndex
{
    if (anIndex == 0)
    {
        dismissed = YES;
    }else{
        dismissed = YES;
        AppDelegate *app = [[UIApplication sharedApplication] delegate];
        app.mainViewController = [[MainTabBarController alloc] init];
        app.window.rootViewController =  app.mainViewController;
        [app.mainViewController exchangeTabbarHightFromSetSystemTextFont:nil];
    }
}

- (void)validateAndSaveCriticalApplicationData
{
    
}
//捕获信号后的回调函数  由HandleException调用
- (void)handleException:(NSException *)exception
{
    [self validateAndSaveCriticalApplicationData];
    NSString *reason = [exception reason];
    NSString *name = [exception name];
    
    UIAlertView *alert =
    [[[UIAlertView alloc]
      initWithTitle:@"tip"
      message:[NSString stringWithFormat:@"CRASH: %@ name:%@,\nReason: %@,\nStack Trace: %@,\n",exception,name,reason,[exception callStackSymbols]]
      delegate:self
      cancelButtonTitle:NSLocalizedString(@"Quit", nil)
      otherButtonTitles:NSLocalizedString(@"Continue", nil), nil]
     autorelease];
    [alert show];
    //或者直接用代码，输入这个崩溃信息，以便在console中进一步分析错误原因
    //当接收到异常处理消息是，让程序开始runloop，防止程序死亡
    CFRunLoopRef runLoop = CFRunLoopGetCurrent();
    CFArrayRef allModes = CFRunLoopCopyAllModes(runLoop);
    while (!dismissed)
    {
        for (NSString *mode in (NSArray *)allModes)
        {
            CFRunLoopRunInMode((CFStringRef)mode, 0.001, false);
        }
    }
    
    CFRelease(allModes);
    
    NSSetUncaughtExceptionHandler(NULL);
    signal(SIGABRT, SIG_DFL);
    signal(SIGILL, SIG_DFL);
    signal(SIGSEGV, SIG_DFL);
    signal(SIGFPE, SIG_DFL);
    signal(SIGBUS, SIG_DFL);
    signal(SIGPIPE, SIG_DFL);
    
    if ([[exception name] isEqual:UncaughtExceptionHandlerSignalExceptionName])
    {
        kill(getpid(), [[[exception userInfo] objectForKey:UncaughtExceptionHandlerSignalKey] intValue]);
    }
    else
    {
        [exception raise];
    }
}

@end

//捕获信号后的回调函数
void HandleException(NSException *exception)
{
    int32_t exceptionCount = OSAtomicIncrement32(&UncaughtExceptionCount);
    if (exceptionCount > UncaughtExceptionMaximum)
    {
        return;
    }

    NSArray *callStack = [UncaughtExceptionHandl backtrace];
    NSMutableDictionary *userInfo =
        [NSMutableDictionary dictionaryWithDictionary:[exception userInfo]];
    [userInfo
        setObject:callStack
        forKey:UncaughtExceptionHandlerAddressesKey];

    [[[[UncaughtExceptionHandl alloc] init] autorelease]
        performSelectorOnMainThread:@selector(handleException:)
        withObject:
            [NSException
                exceptionWithName:[exception name]
                reason:[exception reason]
                userInfo:userInfo]
        waitUntilDone:YES];
}

void SignalHandler(int signal)
{
    int32_t exceptionCount = OSAtomicIncrement32(&UncaughtExceptionCount);
    if (exceptionCount > UncaughtExceptionMaximum)
    {
        return;
    }
    
    NSMutableDictionary *userInfo =
    [NSMutableDictionary
     dictionaryWithObject:[NSNumber numberWithInt:signal]
     forKey:UncaughtExceptionHandlerSignalKey];
    
    NSArray *callStack = [UncaughtExceptionHandl backtrace];
    [userInfo
     setObject:callStack
     forKey:UncaughtExceptionHandlerAddressesKey];
    
    [[[[UncaughtExceptionHandl alloc] init] autorelease]
     performSelectorOnMainThread:@selector(handleException:)
     withObject:
     [NSException
      exceptionWithName:UncaughtExceptionHandlerSignalExceptionName
      reason:
      [NSString stringWithFormat:
       NSLocalizedString(@"Signal %d was raised.", nil),
       signal]
      userInfo:
      [NSDictionary
       dictionaryWithObject:[NSNumber numberWithInt:signal]
       forKey:UncaughtExceptionHandlerSignalKey]]
     waitUntilDone:YES];
}

//SIGABRT--程序中止命令中止信号
//SIGALRM--程序超时信号
//SIGFPE--程序浮点异常信号
//SIGILL--程序非法指令信号
//SIGHUP--程序终端中止信号
//SIGINT--程序键盘中断信号
//SIGKILL--程序结束接收中止信号
//SIGTERM--程序kill中止信号
//SIGSTOP--程序键盘中止信号
//SIGSEGV--程序无效内存中止信号
//SIGBUS--程序内存字节未对齐中止信号
//SIGPIPE--程序Socket发送失败中止信号
void InstallUncaughtExceptionHandler(void)
{
    NSSetUncaughtExceptionHandler(&HandleException);
    signal(SIGABRT, SignalHandler);
    signal(SIGILL, SignalHandler);
    signal(SIGSEGV, SignalHandler);
    signal(SIGFPE, SignalHandler);
    signal(SIGBUS, SignalHandler);
    signal(SIGPIPE, SignalHandler);
}
```

{% endraw %}

使用方法：直接在你的appdelegate的didfinishlaunch函数中添加

{% raw %}

```Objective-C
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
...
InstallUncaughtExceptionHandler();
...
return YES;
}
```

{% endraw %}

注意，这里由于使用了C++的库，所以，如果你是纯OC应用，你的buil settings中应该默认配置是这样的：

![](/img/post/2016-06-18/libc++.png)

那么，你很幸运，什么也不需要调整，直接用就好了。

但是，如果你混编的，而且不得不使用libstdc++,就是build settings中是这样的：

![](/img/post/2016-06-18/libstdc++.png)

请你记得将UncaughtExceptionHandl.m改为UncaughtExceptionHandl.mm，否则会报Undefined symbols for architecture arm64错误，这本该是常识的，但是当你刚刚接触一个新项目，可能还没有反应过来，往往会浪费很多时间，找错方向。




