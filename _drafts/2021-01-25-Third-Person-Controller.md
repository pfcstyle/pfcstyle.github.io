---
layout:		post
title:		"Unity插件-UltimateCharacterController"
description:	"这个插件适合用于FPS和ARPG游戏，主要封装了人物视角，武器切换，坐骑，物体交互等常用功能"
date:		2021-01-25
author:		"Yawei"
categories: "Unity3d"
keywords:
	- ThirdPersonController
	- UltimateCharacterController
    - Unity3d
    - Plugin
---

# 初始化设置

打开Tools=>Opsive=>UltimateCharacterController=>Main Manager

## Setup

1. Click `Add Managers`
   会自动生成一个Game的管理对象
2. Camera Setup, 选择游戏需要的视角等，Click`Setup Camera`
   在Main Camera上会自动添加一个Controller脚本
3. Add UI(自动生成一个FPS常用的UI界面)
4. Add Virtual Controls（生成虚拟摇杆， For Mobile）


## Character

1. 拖动角色模型到Hierarchy, 生成一个人物实例
2. 将实例拖到面板的Character上
3. Click Build Character