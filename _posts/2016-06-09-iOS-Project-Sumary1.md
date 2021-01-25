---
layout:		post
title:		"iOS项目总结(一)"
description	"iOS本地持久化总结1"
date:		2016-06-09
author:		"PfCStyle"
header-img:	"img/post/2016-06-09/head.jpg"
categories: "iOS"
keywords
    - iOS
    - OC
    - 本地持久化
    - plist
    - Preference
    - NSKeyedArchiver
    - sqllite
    - CoreData
---

> 不积跬步，无以至千里；不积小流，无以成江海；

之前就知道闷头写项目，在项目中解决了什么问题也不知道记录，现在很是后悔，如今来总结一下，希望能够回忆起一些。

# iOS开发的本地存储主要有五种形式

- **XML属性列表（plist)归档**
- **Preference(偏好设置NSUserDefaults)**
- **NSKeyedArchiver归档(NSCoding)**
- **SqlLite（本地数据库）**
- **CoreData（苹果官方封装的SqlLite数据库操作接口）**

上述五种本地的存储方式在我们日常编码中都非常常用，本篇文章先介绍前三种，后两种请参考我的后续博文[iOS项目总结(二)-iOS本地持久化总结2](http://pfcstyle.me/2016/06/10/iOS-Project-Sumary2/)

# 应用沙盒

要想真正了解本地数据存储，你需要先了解什么是应用沙盒。我们都知道，iOS的各个应用的文件夹是对其他应用封闭的，也就是说它的文件系统是隔离的，而这每一个应用的数据文件夹就是应用沙盒。那么，如何获取应用沙盒的路径呢？可以通过打印NSHomeDirectory()来获取应用沙盒路径。

{% raw %}

```console
test[15254:733802] 沙盒：/Users/developer/Library/Developer/CoreSimulator/Devices/0360A858-A0E7-45A7-AE71-09D7988C089F/data/Containers/Data/Application/2FB5B4BB-A097-411D-A8BA-6043155C171E
```

{% endraw %}

这里需要提醒大家注意的是，如果你是在调试，那么你每次从XCode运行应用，沙盒路径都会发生改变。我之前做项目有一个涉及到管理草稿的，当时在这个坑里跳了一整天才跳出来。。。当然，如果你的应用发布了，那么，沙盒路径就是固定不变的。

接下来我们看一下沙盒的结构，Finder的快捷键shift+com+g可以前往任意路径

![](/img/post/2016-06-09/sandtable.png)

- **Documents:** 保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏存档保存在该目录
- **Library/Caches:** 保存应用运行时生成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大、不需要备份的非重要数据
- **Library/Preference:** 保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes同步设备时会备份该目录
- **tmp:** 保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录

### Document文件夹获取

{% raw %}

```Objective-C
// NSDocumentDirectory 要查找的文件 枚举
// NSUserDomainMask 代表从用户文件夹下找 枚举
// 最后的Yes代表返回完整路径，No是这样的形式:~/Documents
// 在iOS中，只有一个目录跟传入的参数匹配，所以这个集合里面只有一个元素
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
NSString *filePath = [path stringByAppendingPathComponent:@"xxx.plist"];
```
{% endraw %}

其他的文件夹路径的获取方式我就不一一介绍了。接下来正式说持久化方式

# XML属性列表(plist)归档

plist文件只能是数组、字典、数值、字符串、Bool值这几种类型，而根类型必须是数组或者字典。

### plist文件的归档

{% raw %}

```Objective-C
NSString *filePath = [path stringByAppendingPathComponent:@"xxx.plist"];
// 解档
NSArray *arr = [NSArray arrayWithContentsOfFile:filePath];
NSLog(@"%@", arr);
```

{% endraw %}

### plist文件的解档

{% raw %}

```Objective-C
NSArray *arr = [[NSArray alloc] initWithObjects:@"1", @"2", nil];
// NSDocumentDirectory 要查找的文件
// NSUserDomainMask 代表从用户文件夹下找
// 在iOS中，只有一个目录跟传入的参数匹配，所以这个集合里面只有一个元素
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
NSString *filePath = [path stringByAppendingPathComponent:@"xxx.plist"];
[arr writeToFile:filePath atomically:YES];
```

{% endraw %}

# Preference(偏好设置NSUserDefaults)

OC中有一个NSUserDefaults的单例，它可以用来存储用户的偏好设置，例如：用户名，字体的大小，用户的一些设置等。

### 保存用户偏好设置

{% raw %}

```Objective-C
// 获取用户偏好设置对象
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
// 保存用户偏好设置
[defaults setBool:self.one.isOn forKey:@"one"];
[defaults setBool:self.two.isOn forKey:@"two"];
// 注意：UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。这应该是iOS7之前的问题
// 出现以上问题，可以通过调用synchornize方法强制写入
// 现在这个版本不用写也会马上写入 不过之前的版本不会
[defaults synchronize];
```

{% endraw %}

### 读取用户偏好设置

{% raw %}

```Objective-C
// 读取用户偏好设置
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults]; 
self.one.on = [defaults boolForKey:@"one"];
self.two.on = [defaults boolForKey:@"two"];
```

{% endraw %}

但是要注意的是，userDefault支持的类型有限，无法直接归档自定义类型，只能是NSData、NSString、NSNumber、NSDate、NSArray、NSDictionary，所以如果你要归档自定义类型，要先转换成前面的类型才行。

# NSKeyedArchiver归档(NSCoding)

只有遵守了NSCoding协议的类才可以用NSKeyedArchiver归档和NSKeyedUnarchiver解档，但如果如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型就不需要了。
下面举的是归档解档一个Account模型

### 实现encodeWithCoder和initWithCoder方法

{% raw %}

```Objective-C
@implementation Account
- (void)encodeWithCoder:(NSCoder *)encoder
{
    [encoder encodeObject:_accessToken forKey:@"accessToken"];
    [encoder encodeObject:[NSString stringWithFormat:@"%d", _userID] forKey:@"user_id"];
}

- (id)initWithCoder:(NSCoder *)decoder
{
    if (self = [super init]) {
        self.accessToken = [decoder decodeObjectForKey:@"accessToken"];
        self.userID = [[decoder decodeObjectForKey:@"user_id"] intValue];
    }
    return self;
}
@end
```

{% endraw %}

### 归档

{% raw %}

```Objective-C
_account = [[Account alloc] init];
_account.accessToken = @"123456789";
_account.userID = 1;
[NSKeyedArchiver archiveRootObject:account toFile:kFilePath];
```

{% endraw %}

### 解档

{% raw %}

```Objective-C
_account = [NSKeyedUnarchiver unarchiveObjectWithFile:kFilePath];
```

{% endraw %}

# UserDefault和KeyArchive结合

### 归档

{% raw %}

```Objective-C
- (void)saveAccount:(Account *)account
{
_account = account;
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:account];

NSUserDefaults *user = [NSUserDefaults standardUserDefaults];
[user setObject:data forKey:@"kAccount"];

}
```

{% endraw %}

### 解档

{% raw %}

```Objective-C
NSUserDefaults *user = [NSUserDefaults standardUserDefaults];

NSData *data = [user objectForKey:@"kAccount"];

_account = [NSKeyedUnarchiver unarchiveObjectWithData:data];
```

{% endraw %}

嗯，是的，好吧 没有任何优点，反正就是能用userdefault存储了。。。。

参考博文：[iOS开发中本地数据存储的总结](http://www.jianshu.com/p/cd475693e2f8)








