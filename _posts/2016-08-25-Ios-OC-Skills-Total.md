---
layout:		post
title:		"iOS一些小技巧及小知识点总结(持续更新)"
description	"小技巧，大功能！"
date:		2016-08-25
author:		"PfCStyle"
header-img:	"img/post/2016-08-25/head.jpg"
categories: "iOS"
keywords
    - Ios
    - OC
    - Skills
    - custome view
    - 日期/字符串
    - Debug
    - Status bar
---

> 不积跬步，无以至千里；不积小流，无以成江海；

其实早就想开一个这样的blog,这样每次写代码有感的时候就可以来记录一下。由于拖延等等问题，今天才补上这么个开始，那么，就开始吧，万事，总得有个开始才好继续。

# 没有不能自定义的控件

苹果官方提供了许多的精美控件，但是出于某些原因，一些控件给出的自定义接口不足，致使我们无法定制出符合自己需求的控件，从而不得不每次自己去构建控件，这不免让人感到遗憾与无力。接下来，我就总结几种修改官方控件的方式，有了这几种方式，在我看来，几乎没有无法修改的控件了。

### 最优雅的自定义方式（重写）

最优雅的自定义方式，无非就是继承父控件进行重写了。这种方式对于大部分官方控件都是有效的，举个栗子：你现在要重写tablecell,满足表格距屏幕左右边界都是20pt.

{% raw %}

```Object-C
//继承tableviewcell
@interface BaseCell : UITableViewCell

@end
//重写setframe
@implemention BaseCell
-(void)setFrame:(CGRect)frame{
    frame.origin.x += 20;
    frame.size.width -= 2 * 20;
    [super setFrame:frame];
}

@end
```

{% endraw  %}

如何，是不是非常简单。当然，我相信大家对这种方式其实是非常熟悉的，因为这算是我们最为常用的了。这种方式对于那些特殊控件，比如UISwitch是没有作用的，无论你怎么修改frame，它都不会改变。但是，这并不是说没有办法了。

### 有点暴力的自定义方式（transform)

UISwitch控件，宽高都是固定不变的，而且，个人感觉，有点大啊，不是那么好看。那么，如何更改呢？

{% raw %}

```Object-C
UISwitch *switch = [[UISwitch alloc] init];
switch.transform = CGAffineTransformMakeScale(0.75, 0.75);
```

{% endraw%}

什么感觉？简单！而且这样基本不会有任何其他影响。不认识[CGAffineTransform(变换)](http://www.cocoachina.com/ios/20150104/10816.html)？ 认识一下，很强大，制作动画的利器。嗯，想优雅一点？结合继承：

{% raw %}

```Object-C
//继承UISwitch
@interface MySwitch : UISwitch

@end
//重写setframe
@implemention MySwitch
-(void)setFrame:(CGRect)frame{
    [super setFrame:frame];
	//ios7以后 uiswitch大小更改为（51,31）
	float scaleX = frame.size.width/51;
	float scaleY = frame.size.height/31;
	self.transform = CGAffineTransformMakeScale(scaleX,scaleY);
}

@end
```

{% endraw  %}

ok，这样，你就可以任意的更改Uiswitch的大小了。我就不截图了，大家试一下就知道了，没有任何问题。

### 最暴力的自定义方式（subview）

什么意思呢？之前使用父控件，这次又来折腾子view了！这次以什么为例呢？UIAlertController你怕不怕？到了ios9,uialertview已经废弃了，现在全面使用uialertcontroller。但是不管使用什么，最让开发者头疼的是，明明很好的一个控件，却大多数时候都需要自定义，因为主界面颜色完全无法自定义。。。。谁说的？

{% raw %}

```Object-C
//实例化alertcontroller
UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"提示"message:message preferredStyle:UIAlertControllerStyleAlert];
//添加按钮
[alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction*action) {NSLog(@"点击了确定按钮");}]];
//设置字体颜色（包括按钮的）
[alert.view setTintColor:[UIColor redColor]];
//获取alertview的主窗体子view
UIView *view = alert.view.subviews[0];
view = view.subviews[0];


//这是uialertview没有字时的大小  注意alert的大小跟提示信息的多少是有关系的，所以，这里104不是固定的  但是可以自己动态计算
UIImageView *imageView = [[UIImageView alloc]initWithFrame:CGRectMake(0, 0, 270, 104)];
//圆角 经测试  只要小于13就不会露出背景白色
imageView.layer.cornerRadius = 10.f;
[imageView setBackgroundColor:[UIColor greenColor]];
[view insertSubview:imageView atIndex:0];
//嗯  一不小心又想起来一种更简单的方法  使用下面两句 上面的就不用了  core animation框架
//还是很强大的  这样也不用去计算高度了 哈哈
//view.layer.backgroundColor = [UIColor grayColor].CGColor;
//view.layer.cornerRadius = 13.f;

```

{% endraw %}

ok,diy成功。

### 隐藏的自定义方式（KVC)

其实官方控件中，许多属性虽然没有开放出来，但是苹果其实提供了一种类似于java反射机制的编程方式：KVC。KVC应该不用介绍了，不懂得就直接去搜索吧，我这里只是介绍它在自定义界面上的应用。这次就说UITextField的PlaceHolder属性，你想要斜体，想换个字体，想换个颜色？看代码

{% raw %}

```Object-C
[_nameTF setValue:kTextViewPlaceHolderFont forKeyPath:@"_placeholderLabel.font"];
[_nameTF setValue:kTextViewPlaceColor forKeyPath:@"_placeholderLabel.textColor"];
```

{% endraw %}

ok了，现在去看看你的placeholder什么样吧，那个字体和颜色自己自定义哈。

# Debug

为了开发方便，我们经常会有很多调试设置，但是这些设置在release模式下又是不需要，这时候我们往往会需要这些(例如)：

```C
#ifdef DEBUG
//调试状态
#define MyLog(...) NSLog(__VA_ARGS__)
#else
//发布状态
#define MyLog(...)
```

那么，这个DEBUG是在哪里设置的呢？在 `"Target > Build Settings > Preprocessor Macros > Debug"` 里有一个"`DEBUG=1`"。如果你把`DEBUG`改成`haha`呢?试一下！

# 状态栏显示与隐藏

### app启动时隐藏状态栏

1. 在*info.plist*里面  *Status bar is initially hidden* 设置为 *YES*
2. 如果你想启动后显示，在appdelegate.m中加入`[application setStatusBarHidden:NO withAnimation:UIStatusBarAnimationFade];`

### 其他页面的状态栏的显示与隐藏


```Objective-C
//在对应的控制器实现下面的方法
- (UIStatusBarStyle)preferredStatusBarStyle
{
	//状态栏样式
    return UIStatusBarStyleLightContent;
}
- (BOOL)prefersStatusBarHidden
{
    return YES; // 返回NO表示要显示，返回YES将hiden
}
```

# UIScrollView捏合手势关闭

UIScrollView的自动缩放虽然方便，但也挡不住产品设计部的怪癖好，就是要停，要通过双击放大缩小，这就要求只关闭手势，而要保留缩放功能。通过查看`UIScrollView.h`发现有`pinchGestureRecognizer`属性，很是兴奋，于是

```
//在初始化函数中添加  但是并没有作用！！==！  我这里是继承的UIScrollView
[self removeGestureRecognizer:self.pinchGestureRecognizer];
self.pinchGestureRecognizer.enabled = NO;
```

后来，我是这样解决的：

```
//在代理设置捏合手势失效
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView {
    scrollView.pinchGestureRecognizer.enabled = NO;
        return _photoImageView;

}
```

这样就是可以的，我表示很方！！并不理解第一种方式为什么不行,猜测是在其他地方才初始化的pinch,不得而知。

### 双击放大缩小

既然说到了这个，就说一下如何通过双击实现放大缩小。


```
- (void)handleDoubleTap:(CGPoint)touchPoint {
	
	// Zoom
	if (self.zoomScale != self.minimumZoomScale && self.zoomScale != [self initialZoomScaleWithMinScale]) {
		
		// Zoom out  主要是这个函数
		[self setZoomScale:self.minimumZoomScale animated:YES];
		
	} else {
		
		// Zoom in to twice the size
        CGFloat newZoomScale = ((self.maximumZoomScale + self.minimumZoomScale) / 2);
        CGFloat xsize = self.bounds.size.width / newZoomScale;
        CGFloat ysize = self.bounds.size.height / newZoomScale;
        [self zoomToRect:CGRectMake(touchPoint.x - xsize/2, touchPoint.y - ysize/2, xsize, ysize) animated:YES];

	}

}
```

# App前后台切换的状态保存与恢复

```Objective-C
//这些通知不只是在appdelegate中使用  每一个viewcontroller都是可以的
//WillResignActive
//DidEnterBackground
//WillEnterForeground
//WillResignActive
//切换前后台时  上面四个依次执行
//注册已经激活的通知   可以这里恢复状态
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onDidBecomeActive) name:UIApplicationDidBecomeActiveNotification object:nil];
//注册将要进入后台的通知  可以这里保存状态  注意这里非激活状态不只是切换前后台才会出现，比如切换页面的时候也会响应
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onWillResignActive) name:UIApplicationWillResignActiveNotification object:nil];
//注册将要进入前台通知   也可以这里恢复状态
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onWillEnterForeground) name:UIApplicationWillEnterForegroundNotification object:nil];
//这侧已经进入后台通知    也可以这里保存状态  注意操作不要超过5s，否则会被杀死
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onDidEnterBackground) name:UIApplicationDidEnterBackgroundNotification object:nil];
```

# thread 1: sinal sigabrt

这个一般是因为你的xib或者storyboard与代码的连线有问题，或者是连多了，或者是连了之后改了名字又重连了一次，但是没有删除之前的。

# UIScrollView与UICollectionView手势冲突

之前在制作一个图集浏览的控件，使用UICollectionView作为载体，每一个Cell中加入UIImageView和UIScrollView.

UIImageView用来放图片与点击，双击，长按事件，UIScrollView主要是利用其放大缩小的功能。

UIScrollView的放大缩小本质是滑动手势的响应，这时，就偶尔会和UICollectionView的滑动事件冲突

解决方法是：

在cell上放一个UIView,比如contentView，然后把UIImageView和UIScrollView放到contentView上即可。

# 解决UIScrollview与NavigationBar的冲突

iOS7之后，scrollview都会自动留白一个navigationbar的高度，直接在你的controller里的viewdidload中假如下述代码即可。

```
if (DEVICE_OS_VERSION>7) {
        self.edgesForExtendedLayout = UIRectEdgeNone;
        self.extendedLayoutIncludesOpaqueBars = NO;
        self.modalPresentationCapturesStatusBarAppearance = NO;
        self.automaticallyAdjustsScrollViewInsets = NO;
    }
```

需要注意的是，添加这些后，view的位置就是从navigationbar开始，即屏幕的Y:64的位置现在是view的0。当然，你的view的总高度已经变成了DEVICE_SIZE.HEIGHT-64。


