---
layout:		post
title:		"iOS项目总结(十)"
description	"一个小问题引发的血案--动画冲突"
date:		2016-07-25
author:		"PfCStyle"
header-img:	"img/post/2016-07-25/head.jpg"
categories: "iOS"
keywords
    - iOS
    - UITabBarController
    - TabBar
    - Show
---

> 以小见大，不偏不倚

今天做项目遇到一个奇怪的问题，使用UITabBarController做主控制器，切换控制器然后返回时，下面的tabBar会比view切换慢了几秒，还自带动画，而且这个问题只是在切换特定界面时才会出现。具体描述：

### 1.实例化UITabBarController

```Objective-C
//....
-(YSSJNavigationController *)thirdNav{
    if (!_thirdNav) {
        YSSJLivingViewController *livingViewController = [[YSSJLivingViewController alloc] init];
        [livingViewController setHidesBottomBarWhenPushed:YES];
        _thirdNav = [[YSSJNavigationController alloc] initWithRootViewController:livingViewController];
        _thirdNav.navigationBar.hidden = YES;
        [_thirdNav setHidesBottomBarWhenPushed:YES];

    }
    return _thirdNav;
}
//....
self.thirdNav.tabBarItem = [self  itemWithTitle:[arrayName objectAtIndex:4] image:[UIImage imageNamed:[arrayName objectAtIndex:5]] selectedImage:[UIImage imageNamed:[arrayName objectAtIndex:4]] ];
self.thirdNav.tabBarItem.tag = 2;
//...此处略去了其他nav的实例化
//主控制器是继承的UITabBarViewController
self.viewControllers = @[firstNav,secondNav,self.thirdNav,fourthNav];
```
### 2.点击跳转到thirdNav

```Objective-C
//YSSJLivingViewController.m
//从thirdnav的根控制器present到VLiveStreamViewController
-(void)p_jumpToLivePage{
    NSDictionary *options = @{
                              @"gid" : @(-1),
                              @"gids" : @[],
                              @"shortContent" : @"",
                              @"tags" :@[],
                              @"title" : @"",
                              @"type" : @"live"
                              };
    NSMutableDictionary *liveInfo = [[NSMutableDictionary alloc]initWithDictionary:options];
    
    //网上获取群组信息
    liveInfo = [YSSJTop setLiveInfo:liveInfo];
    NSMutableDictionary *createOption = [liveInfo[@"creator"] mutableCopy];
    NSArray *joinedGroups = liveInfo[@"joinedGroups"];
    
    createOption[@"liveId"]= liveInfo[@"liveId"];
    [createOption setValuesForKeysWithDictionary:liveInfo];
    
    
    // 发起直播
    
    VLiveStreamViewController *liveStreamVC = [[VLiveStreamViewController alloc] initWithNibName:nil bundle:nil];
    liveStreamVC.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
    
    
    liveStreamVC.fromChatGroup = @(-1);
    liveStreamVC.createOption = [createOption copy];
    liveStreamVC.joinedGroups = joinedGroups;
    liveStreamVC.needSelectGroup = YES;
    liveStreamVC.delegate = self;
    
    
    [self presentViewController:liveStreamVC animated:YES completion:nil];
}
```

### 3.dismiss控制器VLiveStreamViewController

```Objective-C
//YSSJLivingViewController.m
//dismiss之后会响应这个函数  发出一个通知
-(void)viewWillAppear:(BOOL)animated{
        [[NSNotificationCenter defaultCenter] postNotificationName:CloseLiveNotification object:nil];
}

//这是通知的函数  回到原始界面
-(void)closeLive{
    if (_lastSelectedIndex == 2) {
        _lastSelectedIndex = _frontSelectedIndex;
        [self setSelectedIndex:_frontSelectedIndex];
    }else{
        _lastSelectedIndex = _frontSelectedIndex;
    }
}
```
### 4.问题出现
此时在原始界面，随便push一个controller，然后pop,发现TabBar返回已经和根控制器不同步了

# 解决方式

还记得上面发送通知的地方吗，之前在viewwillappear中，现在在viewdidappear中

```Objective-C
-(void)viewDidAppear:(BOOL)animated{
        [[NSNotificationCenter defaultCenter] postNotificationName:CloseLiveNotification object:nil];
}
```
Then all work nice!!!

解决方式很简单，但是我应该从这个问题里看到更多的问题：

当一些动画不是在同一个控制器，甚至是不是同一个view时，当这些动画显示会有冲突时，你必须让这些动画的时间错开，否则，将会真的产生冲突，从而产生各种奇葩的结果。







