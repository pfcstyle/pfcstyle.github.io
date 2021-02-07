---
layout:		post
title:		"spatialite-android-sdk的编译和使用"
description: "填坑"
date:		2016-12-29
author:		"PfCStyle"
header-img:	"img/post/2016-12-29/head.jpg"
categories: "Android"
keywords:
    - Android
    - spatialite-android
---

> 好记性不如烂笔头

项目用到spatialite,记录下使用方法

# 准备工作

* 环境

有android studio就行了

* 源码下载

如果想使用最新版本，去google[下载](https://code.google.com/archive/p/spatialite-android/)源码吧。

我这里提供博客日期为止最新的[源码](https://github.com/pfcstyle/spatialite-android)

这里还有已经把所有的坑填上了[完整工程](https://github.com/pfcstyle/spatialite-android-gradle)

# 开始编译

## 使用android studio打开项目

File=>New=>import project

![](/img/post/2016-12-29/open-spatialite.png)

### 会报一些错误，直接贴图

缺少compile-sdk 版本

![](/img/post/2016-12-29/open-spatialite.png)

最小skd版本设置错误

![](/img/post/2016-12-29/sp-vmin-bug.png)

package-name重复
![](/img/post/2016-12-29/package-name.png)

缺少ndk编译模块

这个首先你应该已经安装了ndk,这个我就不多说了。

![](img/post/2016-12-29/sourceSets.png)

![](img/post/2016-12-29/property.png)

这时候sync gradle应该就没有问题了

### 配置工程

我没有重新去打包so文件，在[这里](https://code.google.com/archive/p/spatialite-android/)有具体的打包流程。但是我有尝试过，确实是很麻烦的一件事，工程中没有完整的make文件，依赖都需要自己去管理，我直接去网上找到了合适的[so文件](http://pan.baidu.com/s/1c25jTfi)。**如果你是自己找的so文件，那么，一定要确定是ndk-r8以上版本编译的，否则会有text alloc...类似的错误**

![](img/post/2016-12-29/sp-jnilibs.png)

删除掉demo app目录下的Google Map相关的类，因为需要配置api_key,而且现在google map好像都已经完全更新了，不知道接口变了没有，要验证spatialite 是否成功并不需要展示到map上，所以删除掉`MappingActivity.java`和`MapSelectionOverlay.java`两个类文件,记得修改清单文件`AndroidManifest.xml`

![](img/post/2016-12-29/delete-java.png)

现在已经可以编译成功并运行了。但是现在只能在32bit的安卓机上运行，因为so只有32bit的，64bit的机子会找错文件夹，需要再设置一下：

![](img/post/2016-12-29/abifilter.png)

然后再运行就没有任何问题了。

# spatialite-android-sdk 的使用

* 首先在build/outputs中找到aar文件，具体aar怎么用可以参考我的[上篇](http://pfcstyle.me/2016/12/28/mapbox-compile/)博客。
* 下载[spatialite-gui](http://www.gaia-gis.it/gaia-sins/windows-bin-amd64/)
* 普及一下spatialite的基础知识

```
spatialite> select spatialite_version();
4.0.0
spatialite> select proj4_version();
Rel. 4.8.0, 6 March 2012
spatialite> select geos_version();
3.3.6-CAPI-1.7.6
spatialite> select lwgeom_version();
2.0.2
spatialite> select HasIconv();
1
spatialite> select HasMathSQL();
1
spatialite> select HasGeoCallbacks();
1
spatialite> select HasProj();
1
spatialite> select HasGeos();
1
spatialite> select HasGeosAdvanced();
1
spatialite> select HasGeosTrunk();
0
spatialite> select HasLwGeom();
1
spatialite> select HasEpsg();
1
spatialite> select HasFreeXL();
1

輸入由 WGS84 為基準的經緯度（4326 是 WGS84 2D 的 EPSG CRS SRID 編號）：
spatialite> select AsText(MakePoint(114.1689,22.4518,4326));
POINT(114.1689 22.4518)

將它轉換為 WGS84 UTM 50N 格網座標（32650 是 WGS84 / UTM zone 50N 的 EPSG CRS SRID 編號）：
spatialite> select AsText(ST_Transform(MakePoint(114.1689,22.4518,4326),32650));
POINT(208621.605201 2485587.067636)

將它轉換為 HK1980 格網座標（2326 是 Hong Kong 1980 Grid System 的 EPSG CRS SRID 編號）：
spatialite> select AsText(ST_Transform(MakePoint(114.1689,22.4518,4326),2326));
POINT(835447.180293 834705.40192)

利用格網座標計算大埔中心至九龍坑山的距離：
spatialite> select ST_Distance(MakePoint(208838.738969, 2488246.942923), MakePoint(208621.605201, 2485587.067636));
2668.72321824495

利用經緯度計算大埔中心至九龍坑山的距離：
spatialite> select ST_Length(MakeLine(MakePoint(114.17052, 22.475837,4326), MakePoint(114.1689,22.4518,4326)), 1);
2666.99235712016


計算大埔中心至九龍坑山的方位角：
spatialite> select Degrees(ST_Azimuth(MakePoint(114.1689,22.4518,4326), MakePointZ(114.17052, 22.475837, 437.639187, 4326)));
3.85568120514838

輸出九龍坑山三角網測站的 KML：
spatialite> select AsKml("Cloudy Hill", "description", MakePointZ( 835614.056, 837367.172, 440.8, 2326));
<Placemark><name>Cloudy Hill</name><description>description</description><Point><coordinates>114.170519962613,22.47583709798935,437.6391865937039</coordinates></Point></Placemark>

點的 Union：
spatialite> select astext(ST_Union(MakePoint(114.17052, 22.475837,4326), MakePoint(114.1689,22.4518,4326)));
MULTIPOINT(114.17052 22.475837, 114.1689 22.4518)

創建有地理數據的表，先是一般格式的欄：
spatialite> CREATE TABLE TestTable(
id INTEGER PRIMARY KEY AUTOINCREMENT,
Name TEXT NOT NULL);

創建該地理數據欄，儲存以 WGS84 為基準的點：
spatialite> SELECT AddGeometryColumn('TestTable', 'Geometry', 4326, 'POINT', 'XY');
1

加入 R* index，以加快檢索：(添加索引)
spatialite> SELECT CreateSpatialIndex('TestTable', 'Geometry');
1

加入點數據：
spatialite> insert into TestTable (Name, Geometry) VALUES ("a", MakePoint(114.1689,22.4518,4326));
spatialite> insert into TestTable (Name, Geometry) VALUES ("b", MakePoint(114.17052,22.475837,4326));

列出數據（Geometry 未能直接顯示）：
spatialite> select * from TestTable;
1|a|
2|b|

列出數據：
spatialite> select id, Name, AsText(Geometry) from TestTable;
1|a|POINT(114.1689 22.4518)
2|b|POINT(114.17052 22.475837)

利用 R* index 查出在 22.4518N 114.1689E, 22.4520N 114.1690E 內的點：
spatialite> SELECT  id, Name, AsText(Geometry) FROM TestTable WHERE ROWID IN (SELECT pkid FROM idx_TestTable_Geometry WHERE pkid MATCH RTreeIntersects(114.1689,22.4518,114.1690,22.4520));
1|a|POINT(114.1689 22.4518)

將點數據轉成 GeoHash（可以文字方式儲存於一般資料庫，以方便查詢某一點附近的其他點）：
spatialite> SELECT Name, GeoHash(Geometry) FROM TestTable;
a|wecptzpr0ny5c1eeemw3
b|wecpy587jztypffhy099
```

* sdk的简单使用，我直接按照demo理一下。


```
//上面这些获取数据库路径的代码就省略了
...

// 1.
//Open the database 打开数据库
jsqlite.Database db = new jsqlite.Database();
db.open(dbFile.toString(), jsqlite.Constants.SQLITE_OPEN_READONLY);

// 2.
// Test prepare statements  测试下准备好了没有？我的理解就是查询一下数据库的基本状态和信息，可能真正使用时也是用不到的。
String query = "SELECT name, peoples, AsText(Geometry) from Towns where peoples > 350000";
Stmt st = db.prepare(query);
st.step();
st.close();

// 3.测试各种查询  你们也可以把我上面简单介绍的其他命令拿来测试下
// Test various queries
db.exec("select Distance(PointFromText('point(-77.35368 39.04106)', 4326), PointFromText('point(-77.35581 39.01725)', 4326));",
		cb);
db.exec("SELECT name, peoples, AsText(Geometry), GeometryType(Geometry), NumPoints(Geometry), SRID(Geometry), IsValid(Geometry) from Towns where peoples > 350000;",
		cb);
db.exec("SELECT Distance( Transform(MakePoint(4.430174797, 51.01047063, 4326), 32631), Transform(MakePoint(4.43001276, 51.01041585, 4326),32631));",
		cb);

		
// 4.
// Close the database 搞完了就关闭
db.close();


```
最后，这里还有一个[官方的tutorial](https://www.gaia-gis.it/fossil/libspatialite/wiki?name=spatialite-android-tutorial)，虽然很老了，但是api基本没变化，大家可以找到更多关于使用spatialite的使用方法。感兴趣的可以看下。

好了，就到这里了。

