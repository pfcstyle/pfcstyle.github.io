---
layout:		post
title:		"DSP-傅里叶变换-Z变换-拉普拉斯变换-小波变换-离散余弦变换-都干啥的"
description: ""
date:		2021-03-29
author:		"Yawei"
categories: ["DSP"]
keywords:
    - DSP
    - 傅里叶变换
    - 拉普拉斯变换
    - Z变换
    - 小波变换
---

傅里叶变换是为了将信号从时域分解为频域

Z变换是离散信号的傅里叶变换，为计算机处理信号而生

拉普拉斯变换是因为傅里叶变换无法解决某些特殊波形，比如爆炸式增长的曲线（因为傅里叶变换是以正弦波拟合为基础，都是趴着的），才出现的。拉普拉斯基于傅里叶变换乘以一个衰减因子，让波形变为了可以傅里叶可以处理的波形。

小波变换，傅里叶变换因为无法得知某一个频率是什么时候开始产生的，即在频域中没有时间信息。小波变换让傅里叶变换后的频域具备了时间信息，可以得知频率是什么时候添加进去的。

离散余弦变换是离散傅里叶变换的一种特殊形式，当输入信号是实偶函数时，离散傅里叶变换公式可以简化为离散余弦变换。