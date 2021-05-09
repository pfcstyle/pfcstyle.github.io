---
layout:		post
title:		"Remove profile in xcode"
description: "替换同名的profile时，经常发现不成功，需要删除原来的才行"
date:		2021-05-09
author:		"Yawei"
categories: ["iOS"]
keywords:
    - iOS
    - profile
---

https://stackoverflow.com/questions/26732251/how-to-remove-provisioning-profiles-from-xcode

```
rm ~/Library/MobileDevice/Provisioning\ Profiles/*.mobileprovision  
```