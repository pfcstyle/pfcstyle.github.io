---
layout:		post
title:		"Android Studio for Beginer(四)之Gradle示例 "
description: "Gradle实现多渠道打包"
date:		2016-05-26
author:		"PfCStyle"
header-img:	"img/post/2016-05-26/head.jpg"
categories: "Android"
keywords:
    - Android
    - Gradle
    - 多渠道打包
---

> 如果你不够懒，那么，你就做不好一个程序员。

不说废话，先直接说怎么做吧。

# 友盟多渠道打包

1.首先是在AndroidManifest.xml里面添加最下面一段：

{% raw %}

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="me.pfcstyle.helloword">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
	<!--这里 -->
    <meta-data
        android:name="UMENG_CHANNEL"
        android:value="{UMENG_CHANNEL_VALUE}" />
</manifest>
```

{% endraw %}

里面的{UMENG_CHANNEL_VALUE}就是渠道指示，我们配置为PlaceHolder,这样可以在build.gradle里设置productFlavors,从而让其在编译时自动变化。

2.在build.gradle设置productFlavors

{% raw %}

```Gradle
productFlavors {
        xiaomi {}
        _360 {}
        baidu {}
        wandoujia {}
    }

    productFlavors.all {
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
    }
```

{% endraw %}

或者这样写也是一样的：

{% raw %}

```Gradle
productFlavors {
        xiaomi {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
        }
        _360 {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "_360"]
        }
        baidu {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
        }
        wandoujia {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
        }
    }  
```

{% endraw %}

如果你想确认一下你的productFlavors是否添加成功了，你可以通过下图验证：

![](/img/post/2016-05-26/flavors.png)

然后命令行定位到你的项目的根目录，执行./gradlew assembleRelease，然后就可以静静的等待各渠道打包完成了。

### 一些可能的问题

如果你在执行./gradlew 的时候，提示你要下载类似gradle-2.10-all(这个版本可能不同)，这个下载比较慢，而且你会发现每一个工程都要安装，这本就很不合理，看解决方法！

去[官网](https://services.gradle.org/distributions/)下载对应版本的gradle-v-all.zip，然后到工程根路径->gradle->wrapper下找到gradle-wrapper.properties，编辑最后一行的

{% raw %}

```property
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
//更改为
distributionUrl=gradle-2.10-all.zip
```

{% endraw %}

然后将你下载好的zip包放到工程根路径->gradle->wrapper下，如图：

![](/img/post/2016-05-26/gradlezip.png)

好了，再去执行gradlew -v ok啦，显示如下图：

![](/img/post/2016-05-26/zip.png)

除此之外 assemble 还能和 Product Flavor 结合创建新的任务，其实 assemble 是和 Build Variants 一起结合使用的，而 Build Variants = Build Type + Product Flavor ， 举个例子大家就明白了：

如果我们想打包wandoujia渠道的release版本，执行如下命令就好了：

./gradlew assembleWandoujiaRelease
如果我们只打wandoujia渠道版本，则：

./gradlew assembleWandoujia
此命令会生成wandoujia渠道的Release和Debug版本

同理我想打全部Release版本：

./gradlew assembleRelease
这条命令会把Product Flavor下的所有渠道的Release版本都打出来。

# Gradle管理依赖

gradle最常用的还是管理依赖吧，看看有多简单：


{% raw %}

```Gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.android.support.constraint:constraint-layout:1.0.0-alpha1'
    testCompile 'junit:junit:4.12'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
    androidTestCompile 'com.android.support.test:runner:0.5'
    androidTestCompile 'com.android.support:support-annotations:23.4.0'
}
```

{% endraw %}

直接执行上述的编译，gradle就会自动帮你下载添加到依赖，根本不用管了。这在我们使用第三方的时候就会非常方便，一句话，所有需要的jar包什么的都有了。

# Gradle依赖的统一管理

那么，嘿嘿，更简单的来了，下面说一下依赖的统一管理方式。

### 统一一个依赖管理文件

你要先自己创建一个config.gradle文件来统一管理你的依赖和其他系统版本这些参数，注意，这个文件不是属于某一个工程的，而是属于你个人或者公司的维护的文件，在你的工程中只是引用它。

{% raw %}

```Gradle
ext {

    android = [compileSdkVersion: 23,
               buildToolsVersion: "23.0.2",
               applicationId    : "me.storm.ninegag",
               minSdkVersion    : 14,
               targetSdkVersion : 22,
               versionCode      : 2,
               versionName      : "1.1.0"]

    dependencies = ["support-v4"               : 'com.android.support:support-v4:23.1.1',
                    "appcompat-v7"             : 'com.android.support:appcompat-v7:23.1.1',
                    "design"                   : 'com.android.support:design:23.1.1',
                    "cardview-v7"              : 'com.android.support:cardview-v7:23.1.1',
                    "recyclerview-v7"          : 'com.android.support:recyclerview-v7:23.1.1',
                    "multidex"                 : "com.android.support:multidex:1.0.+",
                    "butterknife"              : 'com.jakewharton:butterknife:7.0.1',
                    "volley"                   : 'com.mcxiaoke.volley:library:1.0.19',
                    "okhttp"                   : 'com.squareup.okhttp:okhttp:2.7.0',
                    "okhttp-urlconnection"     : 'com.squareup.okhttp:okhttp-urlconnection:2.7.0',
                    "leakcanary"               : 'com.squareup.leakcanary:leakcanary-android:1.3.1',
                    "glide"                    : 'com.github.bumptech.glide:glide:3.6.1',
                    "glide-okhttp-integration" : 'com.github.bumptech.glide:okhttp-integration:1.3.1',
                    "foldable-layout"          : 'com.alexvasilkov:foldable-layout:1.0.1',
                    "etsy-grid"                : 'com.etsy.android.grid:library:1.0.5']
}
```

{% endraw %}

上面是我的config.gradle文件，你们放你们需要的依赖以及配置其他的参数。

### 如何引用？

如下图：

![](/img/post/2016-05-26/config.png)

只需在最顶部加上上面一行代码，意思就是所有的子项目或者所有的modules都可以从这个配置文件里读取内容。

最后在到app目录下的build.gradle文件里看下具体如何读取的呢？

android节点下的读取：

![](/img/post/2016-05-26/config_use1.png)

denpendencies节点下的读取：

![](/img/post/2016-05-26/config_use2.png)

参考博客：[Gradle依赖的统一管理](http://stormzhang.com/android/2016/03/13/gradle-config/)



