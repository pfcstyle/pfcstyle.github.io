---
layout:		post
title:		"shell命令-chmod, chown, chgrp"
description: "linux权限控制"
date:		2021-03-30
author:		"Yawei"
categories: ["shell", "linux"]
keywords:
    - shell
    - linux
    - chmod
    - chown
    - chgrp
---

# chmod 
此命令用来更改某个文件对于`文件所有者`, `群组`和`其他人`三种角色的`读`, `写`和`执行`权限。权限表示：

```
r(Read，读取，权限值为4)：对文件而言，具有读取文件内容的权限；对目录来说，具有浏览目 录的权限。
w(Write,写入，权限值为2)：对文件而言，具有新增、修改文件内容的权限；对目录来说，具有删除、移动目录内文件的权限。
x(eXecute，执行，权限值为1)：对文件而言，具有执行文件的权限；对目录了来说该用户具有进入目录的权限。
```
对于每一个角色，都包含rwx三种权限，如下表：
rwx rw- r–	764
rw- r– r–	644
rw- rw- r–	664

```bash
# 对单文件
chmod 777 test.out

# 对文件夹
chmod -R 764 ~/test/
```

# chown
此命令用来修改文件所属组或用户

```
# 将test.out的owner改为yusi
chown yusi test.out
# 将test目录及其子目录（包括文件）的owner改为user, 把所属组改为users
chown -R user.users ~/test/
```

# chgrp
此命令专门更改所属组

```
# 将 ~/test 和 /usr/local/bin两个文件目录的属组更改为users
chgrp -R users ~/test /usr/local/bin
```