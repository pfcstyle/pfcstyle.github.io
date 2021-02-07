---
layout:		post
title:		"Android Studio for Beginer(二)"
description: "The Best Android Tool"
date:		2016-03-30
author:		"PfCStyle"
header-img:	"img/post/2016-03-30/head.jpg"
categories: "Android"
keywords:
    - Android
    - Tool
    - Android studio
---

> 工欲善其事，必先利其器

今天说一下代码方面常用的设置，以及快捷键。

# 常用快捷键记录

- Ctrl + N 使用输入类名对话框快速打开类文件
- Ctrl + Shift + N 搜索所有的文件名
- Ctrl + Shift + A 动作或选项搜索框，如搜索show line numbers就会自动列出显示行数的开关，支持模糊查询
- Ctrl + Shift + F/R 全局搜索/替换，可以搜索/替换文件内容，还可以设置过滤
- Ctrl + G 以行和列导航单个文件，如果只输入一个数字，就是调到行
- Ctrl + Alt + Home 列出与当前文件相关联的文件，如：xml
- Ctrl + Shift + Backspace 移动到最后编辑位置
- Ctrl + '+'(数字键盘的) 展开代码块
- Ctrl + '-'(数字键盘的) 收缩代码块
- Ctrl + Space 基本代码补全功能，附带javadoc展示
- Ctrl + Shift + Space 智能代码补全，比基本代码补全范围更广
- Ctrl + '/' 行注释
- Ctrl + Shift + '/' 块注释
- Ctrl + z 撤销
- Ctrl + Shift + z 恢复撤销
- Ctrl + J 调用动态模板，这是打出了缩略词之后调用
- Ctrl + Alt + J 显示出动态模板列表
- Ctrl + Shift + Down 向下移动代码块
- Ctrl + Shift + Up 向上移动代码块
- Ctrl + Alt + L 自动格式化代码
- Ctrl + Alt + I 自动缩进代码
- Code ➤ Rearrange 自动整理代码
- Ctrl + Alt + T 环绕代码 如try/catch,if/else等等
- Ctrl + Shift +Delete 删除环绕代码
- Ctrl + E 查看最近打开过的文件 默认最多记录50个
- Ctrl + Alt + 左箭头 遍历导航操作，上一个导航
- Ctrl + Alt + 右箭头 遍历导航操作，下一个导航
- Alt + '/' 循环扩展
- Alt + 'F1' 打开导航列表
- Alt + Insert 生成代码，包括构造器，getter,setter等等
- Shift + Tab 取消缩进

# 常用设置

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

