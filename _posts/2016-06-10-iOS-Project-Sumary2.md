---
layout:		post
title:		"iOS项目总结(二)"
description: "iOS本地持久化总结2"
date:		2016-06-10
author:		"PfCStyle"
header-img:	"img/post/2016-06-10/head.jpg"
categories: "iOS"
keywords:
    - iOS
    - OC
    - 本地持久化
    - plist
    - Preference
    - NSKeyedArchiver
    - sqllite
    - Core Data
---

> 不积跬步，无以至千里；不积小流，无以成江海；

# iOS开发的本地存储主要有五种形式

- **XML属性列表（plist)归档**
- **Preference(偏好设置NSUserDefaults)**
- **NSKeyedArchiver归档(NSCoding)**
- **SqlLite（本地数据库）**
- **Core Data（苹果官方封装的SqlLite数据库操作接口）**

上述五种本地的存储方式在我们日常编码中都非常常用，前三种已经在上篇博文[iOS项目总结(一)-iOS本地持久化总结1](http://pfcstyle.me/2016/06/09/iOS-Project-Sumary1/)中介绍了，今天开始说后两种SqlLite和Core Data.

# SqlLite（本地数据库）

SQLite作为一中小型数据库，主要应用在移动端中，跟前三种保存方式相比，使用相对比较复杂一些，但如果你有数据库基础，这个其实也是小菜一碟了。下面看一下如何使用

1.添加libsqlite3.tbd依赖，并在对应的操作文件中导入#import "sqlite3.h"

2.打开数据库

{% raw %}

```Objective-C
- (void)openDB{
    sqlite3 *db; //声明一个sqlite3数据库 这个其实要全局的  就这样写吧，大家知道就好
    ///文件是否存在
    NSFileManager* fileManager = [NSFileManager defaultManager];
    //这里filePath是自己定义的路径，一般在沙箱的Documents里面操作，不明白沙箱的请移步我的上篇博文
    NSString *dbpath=[self filePath];
    //文件是否存在
    BOOL success = [fileManager fileExistsAtPath:dbpath];
    if (!success) {//不存在就复制过来一个
        NSString *resourcePath=[[NSBundle mainBundle]resourcePath];
        //复制
        NSString *sourceDBPath=[resourcePath stringByAppendingPathComponent:@"app.bundle/datas.sqlite"];

        NSError *error;
        success = [fileManager copyItemAtPath:sourceDBPath toPath:dbpath error:&error];
        if(!success)
        NSAssert1(0,@"数据库附加失败！'%@'.", [error localizedDescription]);
        else
        NSLog(@"数据库附加成功:%@",dbpath);
    }
    //打开数据库
    if (sqlite3_open([[self filePath] UTF8String], &db) != SQLITE_OK) {
        sqlite3_close(db);
        NSAssert(0, @"数据库打开失败。");
    }
}
```
{% endraw %}

3.执行sql语句

{% raw %}

```Objective-C
sqlite3 *db; //声明一个sqlite3数据库 这个是全局的 跟上同
//这里是模拟写了一个查找所有类型的操作
NSString *sql = @"SELECT * FROM km_types";
sqlite3_stmt *statement;
//执行sql语句  返回成功或者失败
if (sqlite3_prepare_v2(db, [sql UTF8String], -1, &statement, nil) == SQLITE_OK) {
    //指针一行行下行
    while (sqlite3_step(statement) == SQLITE_ROW) {
    KMTypes* k= [[KMTypes alloc]init];
    //按照类型取出对应字段
    int type_id = (int)sqlite3_column_int(statement,0);
    int parent_id = (int)sqlite3_column_int(statement,1);
    char *type_title = (char *)sqlite3_column_text(statement, 2);
    int type_order = (int)sqlite3_column_int(statement,3);
    int topic_count=(int)sqlite3_column_int(statement,4);
    //这里注意转换字符串编码
    NSString *type_titleStr = [[NSString alloc] initWithUTF8String:type_title];
    k.type_title = type_titleStr;
    k.type_id=type_id;
    k.parent_id = parent_id;
    k.type_order = type_order;
    k.topic_count=topic_count;

    [array addObject:k];

}
//完成操作
sqlite3_finalize(statement);
}
//最后一定要关闭数据库
sqlite3_close(db)
```
{% endraw %}

这就是SQLite的基础操作了，其实挺简单的，但是可能数据存取，表的创建等还是有点繁琐，我们看下Core Data。

# Core Data使用

Core Data实际上是对SQLite的操作封装，让我们更加易用，它完全不需要sql语句，因此，哪怕你没有数据库基础，用起来也是毫不费力。

1.添加实体和模型

首先你在创建项目的时候应该选在使用Core Data，这个就不再贴图了。如果你想在现成的项目中添加Core Data,你需要添加一下CoreData.framework的依赖，在pch文件中添加#import<CoreData/CoreData.h>，然后添加appdelegata的相关内容，你可以自己重新创建一个带有Core Data的工程，直接复制过来就行。最后再手动添加一个Core Data Model文件

![](/img/post/2016-06-10/CoreData.png)

创建Data Model文件时需要注意，文件名称要与AppDelegate.m中managedObjectModel方法中提到的文件名称相匹配，一般是你的工程名.momd

有了Data Model文件后，就可以在里面添加实体和关系，实际上就是向数据库中添加表格和建立表格之间的关联。添加实体如图所示：

![](/img/post/2016-06-10/datamodel1.png)

每个学生有一个所在的班级，每个班级中有多个学生，因此，学生和班级之间可以建立关系。建立关系如图所示：

![](/img/post/2016-06-10/datamodel2.png)

建立关系之后，可以切换显示的样式，以图表的方式查看实体之间的关系，如图所示：

![](/img/post/2016-06-10/datamodel3.png)

这样就创建好了表格以及字段了，省却了sql语句。接下来自动创建数据模型类。

2.生成数据模型类

创建好实体后，可以通过添加NSManagedObject subclass文件，系统可以自动添加实体对应的数据模型类，如图所示：

![](/img/post/2016-06-10/datamanager1.png)

![](/img/post/2016-06-10/datamanager2.png)

![](/img/post/2016-06-10/datamanager3.png)

![](/img/post/2016-06-10/datamanager4.png)

3.CoreData的代码操作

### 介绍一下appdeldgate中自动生成的代码

{% raw %}

```Objective-C
- (void)applicationWillTerminate:(UIApplication *)application
{
    //保存数据到持久层
    [self saveContext];
}
```
{% endraw %}

{% raw %}

```Objective-C
- (void)saveContext
{
    NSError *error = nil;
    NSManagedObjectContext *managedObjectContext = self.managedObjectContext;
    if (managedObjectContext != nil) {
        if ([managedObjectContext hasChanges] && ![managedObjectContext save:&error]) {
            NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
            abort();
        } 
    }
}
```
{% endraw %}

{% raw %}

```Objective-C
- (NSURL *)applicationDocumentsDirectory
{
//获取Documents的目录路径
    return [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
}
```
{% endraw %}

{% raw %}

```Objective-C
/**
*被管理的数据上下文
*初始化的后，必须设置持久化存储助理
*/
- (NSManagedObjectContext *)managedObjectContext
{
    if (__managedObjectContext != nil) {
        return __managedObjectContext;
    }
    NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
    if (coordinator != nil) {
        __managedObjectContext = [[NSManagedObjectContext alloc] init];
        [__managedObjectContext setPersistentStoreCoordinator:coordinator];
    }
    return __managedObjectContext;
}
```
{% endraw %}

{% raw %}

```Objective-C
/**
被管理的数据模型
初始化必须依赖.momd文件路径，而.momd文件由.xcdatamodeld文件编译而来
*/
- (NSManagedObjectModel *)managedObjectModel
{
    if (__managedObjectModel != nil) {
        return __managedObjectModel;
    }
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"TestApp" withExtension:@"momd"];
    __managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    return __managedObjectModel;
}
```
{% endraw %}

{% raw %}

```Objective-C
/**
持久化存储助理

初始化必须依赖NSManagedObjectModel，之后要指定持久化存储的数据类型，默认的是NSSQLiteStoreType，即SQLite数据库；并指定存储路径为Documents目录下，以及数据库名称
*/
- (NSPersistentStoreCoordinator *)persistentStoreCoordinator
{
    if (__persistentStoreCoordinator != nil) {
        return __persistentStoreCoordinator;
    }
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"TestApp.sqlite"];

    NSError *error = nil;
    __persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];

    if (![__persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {
        NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
        abort();
    }    
    return __persistentStoreCoordinator;
}
```
{% endraw %}

## 没有生成数据模型类的时候可以使用KVC来插入和查找

### 插入数据

{% raw %}
```Objective-C
- (void)insertCoreData
{
    NSManagedObjectContext *context = [self managedObjectContext];

    NSManagedObject *student = [NSEntityDescription insertNewObjectForEntityForName:@"Student" inManagedObjectContext:context];
    [student setValue:@"1" forKey:@"id"];
    [student setValue:@"name A" forKey:@"name"];

    NSManagedObject *Class = [NSEntityDescription insertNewObjectForEntityForName:@"Class" inManagedObjectContext:context];
    [Class setValue:@"1" forKey:@"id"];
    [Class setValue:@"name B" forKey:@"name"];
    
    [Class setValue:student forKey:@"student"];

    NSError *error;
    if(![context save:&error])
    {
        NSLog(@"不能保存：%@",[error localizedDescription]);
    }
}
```
{% endraw %}

### 查找数据
{% raw %}

```Objective-C
- (void)dataFetchRequest
{
    NSManagedObjectContext *context = [self managedObjectContext];
    NSFetchRequest *fetchRequest = [[NSFetchRequest alloc] init];
    NSEntityDescription *entity = [NSEntityDescription entityForName:@"ContactInfo" inManagedObjectContext:context];
    [fetchRequest setEntity:entity];
    NSError *error;
    NSArray *fetchedObjects = [context executeFetchRequest:fetchRequest error:&error];
    for (NSManagedObject *info in fetchedObjects) {
        NSLog(@"name:%@", [info valueForKey:@"name"]);
    }
}
```
{% endraw %}

## 生成数据模型类之后

### 插入数据

{% raw %}
```Objective-C
- (void)insertCoreData
{
    NSManagedObjectContext *context = [self managedObjectContext];

    Student *student = [NSEntityDescription insertNewObjectForEntityForName:@"Student" inManagedObjectContext:context];
    student.id = 1;
    student.name = @"name A";

    Class *class = [NSEntityDescription insertNewObjectForEntityForName:@"Class" inManagedObjectContext:context];
    class.id = 1;
    class.name = @"name B";

    class.student = student;

    NSError *error;
    if(![context save:&error])
    {
    NSLog(@"不能保存：%@",[error localizedDescription]);
    }
}
```

{% endraw %}

查找数据也是类似，这里就不再赘述了。

参考博文：

[IOS 数据存储之 Core Data详解](http://www.th7.cn/Program/IOS/201506/490903.shtml)

[iphone数据存储之－－ Core Data的使用（一）](http://www.cnblogs.com/xiaodao/archive/2012/10/08/2715477.html)














