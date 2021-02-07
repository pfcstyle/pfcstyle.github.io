---
layout:		post
title:		"Android Studio for Beginer(三) 之初识Gradle"
description: "The Best Android Tool"
date:		2016-04-09
author:		"PfCStyle"
header-img:	"img/post/2016-04-09/head.jpg"
categories: "Android"
keywords:
    - Android
    - Tool
    - Android studio
    - Gradle
---

> 工欲善其事，必先利其器

今天大致的介绍一下Gradle

# 为什么要用Gradle

- 一个像Ant一样灵活且通用的构建工具。
- 一种可切换的，像Maven一样的基于约定的构建框架，却又从不约束你（约定优于配置）。
- 对多项目构建的强力支持。
- 对依赖管理的强力支持（基于Apache Ivy）。
- 对已有的Maven和Ivy仓库有着全面的支持。
- 支持可传递性的依赖管理，而不需要远程仓库或者pom.xml和ivy.xml配置文件。
- Gradle能够很好地支持Ant任务和构建 。（有更好的翻译欢迎提议）
- 支持用Groovy语言编写Gradle的脚本。
- 拥有丰富的领域模型来构建你的脚本。

Gradle的核心是一个丰富的可扩展的基于Groovy的领域特定语言(DSL)。Gradle通过提供说明性语言元素将说明性构建推到下一层，您可以组装。这些元素也提供build-by-convention支持Java、Groovy、OSGi、Web和Scala项目。说了这么多，下面我们来一个快速入门。

# Gradle快速入门

### Gradle安装

Gradle需要安装1.6及以上版本的Java JDK或JRE（使用java -version来查看当前版本）。Gradle拥有自己的Groovy库，因此不需要另行安装Groovy。任何已安装的Groovy都会被Gradle给忽略。Gradle使用环境变量中设置的JDK。

	

### 代码生成设置

恰当的使用代码生成功能，这一特色将为你节约大量的时间，代码生成是生成各种方法的的强大功能，包括了构造，getters, setters, equals()，hashCode(), toString()方法等等。在你使用代码生成之前，确认Android Studio 是配置好了，可以忽略成员名称的前缀，如m和s（因为我们一般遵循成员变量前加'm'，静态变量前加's'的规则），点击File ➤ Settings ➤ Code Style ➤ Java ➤ Code Generation将得到设置对话框，将会出现代码生成的标签页，如果域和静态域文本框不包含m和s，则键入他们，并点击”应用“和”确定“，如图

![](/img/post/2016-03-30/generator.png)

### 在模板中保存自己常用的代码

Android Studio有很多模板,允许您将预定义的代码直接插入到你的源文件中。在许多ide,生成的代码只是从模板中粘贴，而从来不考虑作用域;但是Android Studion的模板是对作用域敏感的,也可以集成变量数据。在你开始使用Android Studio的动态模板之前,让我们探索动态模板和自定义模板。导航到File ➤ Settings ➤ Live Templates。选择普通模板组。现在在右上角单击绿色加号按钮并选择住模板。如图，填充好缩写、描述和模板文本字段。在这个模板可以应用之前,您必须单击Define按钮,这看起来像一个蓝色的超文本链接，位于窗口的底部。现在选择Java和选择所有范围(语句,表达式,声明等等)。单击Apply

![](/img/post/2016-03-30/live_tem.png)

图中的$selection$意思是你选择的内容，你先选择一段文字，然后按Ctrl + Alt + J，选择cb模板，就会自动出现上图中定义的内容了。

### 定义你自己的代码风格

代码风格规范在不断发展。没有固定的规则，你应该在你的方法之后放置空格的数量，还是左括号应该出现在同一行作为方法签名或略低于它。组织倾向于定义自己的代码风格,但每个程序员的代码风格也各不相同,你也可能有你习惯的代码风格。幸运的是,Android Studio很简单就能样式化和组织你的代码。在开始样式化代码之前，让我们检查一下代码风格的设置。选择File ➤ Settings ➤ Code Style弹出设置对话框,如图所示。Java和XML是我们在Android中最感兴趣的语言。在左窗格中切换打开代码风格选项,选择Java,并检查在设置窗口的每个选项卡

![](/img/post/2016-03-30/code_style.png)

代码风格的定义选项非常多，建议大家自己点击多试试，在右侧的代码框中会即时响应你的修改。

