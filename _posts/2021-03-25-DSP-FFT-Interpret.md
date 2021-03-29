---
layout:		post
title:		"DSP-FFT（快速傅里叶变换）使用解释"
description: "将信号在时域和频域之间进行转换"
date:		2021-03-24
author:		"Yawei"
categories: ["DSP", "iOS"]
keywords:
    - DSP
    - FFT
    - iOS
---

> 各种算法真是太多了，我只求怎么用，真的一点也不想知道其内部原理。很多文件把内部原理和理解说的很通透，但最后我却仍然不知道怎么用！很简单，告诉我这个函数的各个参数是什么，返回值都是啥意思就ok了，怎么就这么难？

# 构造FFT

我们以iOS vDSP中[vDSP_fft_zrip](https://developer.apple.com/documentation/kernel/1579997-vdsp_fft_zrip?language=occ)为例，其第一个参数需要使用[vDSP_create_fftsetup](https://developer.apple.com/documentation/kernel/1580009-vdsp_create_fftsetup?language=occ)来构造设置选项。

```swift
void vDSP_fft_zrip(FFTSetup __Setup, const DSPSplitComplex *__C, vDSP_Stride __IC, vDSP_Length __Log2N, FFTDirection __Direction);

FFTSetup vDSP_create_fftsetup(vDSP_Length __Log2n, FFTRadix __Radix);
```

> 注意，并不是所有DSP库都会提供fft的构建选项，很多都是内置了默认Radix为2，所以使用时，直接传入需要变换的信号即可，如python的scipy.fft(signal)。

`vDSP_create_fftsetup`使用解释：
* 第一个参数：信号的最大长度，要求必须是2的n次方或2的n次方的倍数，比如radix3就是(3*2**Log2n)。实际信号的长度可以小于这个值, 但必须是2的n次方
* 第二个参数：fft的基数，基数越大，速度越快，但对硬件要求越高
* 返回值：这是一个用于在fft计算过程中使用的数据结构，它的构建效率比较低，因此建议初始化一次，多个fft共用，取最长的信号长度即可。

`vDSP_fft_zrip`使用解释：
* 第一个参数：setup，参见上文
* 第二个参数：信号的复数形式表示，如何转换为复数，见下图，参照：https://developer.apple.com/documentation/accelerate/data_packing_for_fourier_transforms

![图1](/img/post/2021-03-25/complex.png)

* 第三个参数：步长，就是从第二个参数(信号)每隔几个读取一次，交叉存储的数据结构，性能较低。所以一般都是顺序存储，步长为1。
* 第四个参数：信号长度
* 第五个参数：方向。正向(时域转频域)或反向(频域转时域)
* 输出：第二个参数为inout类型，因此输出也是存储在第二个参数中。输出的是频域的复数表示。
  
* fft输出解释：(这里的n为信号长度)
1. 复数索引0，实部为DC(时域值的和)，虚部为0
2. 复数索引从1到n/2, 包含频域值
3. 复数索引n/2+1为Nyquist部分，实部为余弦分量系数，虚部为0
4. 剩余的部分为第二部分的共轭（镜像）

* vDSP_fft_zrip输出解释：(注意这里的n为信号长度的一半)
1. 复数索引0，DC的虚部填上了Nyquist的实部
2. 复数索引从1到n, 包含频域值
![图2](/img/post/2021-03-25/output.png)
* 输出的使用：
  1. 频率=sqrt(real * real + imag * imag)
  2. 频谱宽度=(频率 * 采样率 / 2) / (n / 2)) = 频率 * 采样率 / n