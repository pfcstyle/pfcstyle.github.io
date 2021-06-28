---
layout:		post
title:		"Android Reverse - The APK exported from AndroidKiller can't be installed"
description: "Targeting R+ (version 30 and above) requires the resources.arsc of installed APKs to be stored uncompressed and aligned on a 4-byte boundary"
date:		2021-06-23
author:		"Yawei"
categories: "Android Reverse"
keywords:
    - Android Reverse
---

# 问题

使用AndroidKiller编译签名的apk无法安装，报错：
> Targeting R+ (version 30 and above) requires the resources.arsc of installed APKs to be stored uncompressed and aligned on a 4-byte boundary

这是因为没有4k对齐导致的问题，可以通过使用zipalign工具手动对齐

zipalign工具目录：G:\AndroidSDK\build-tools\30.0.2, 基本各个版本的Android都有，选一个即可

```
zipalign -f -p 4 input.apk output.apk
```

注意，需要先对齐，再签名。因为新版现在都要求使用v2及以上的签名版本，新版本是对所有包内内容进行的hash,对齐会导致hash变化从而导致签名验证失败。

签名命令：
```
# apksigner与zipalign在同一目录
 apksigner sign --ks welltory.jks  --ks-pass pass:123456 --v2-signing-enabled true output.apk
```

