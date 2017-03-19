---
layout: post
title: 修改/etc/profile文件后, 命令不可用的解决方法
categories: Linux
description: 修改/etc/profile文件后, 命令不可用的解决方法
keywords: Linux, /etc/profile
---

修改/etc/profile文件后, 命令不可用了, 怎么办呢. 

# 修改/etc/profile文件后, 命令不可用的解决方法

**前因:**
想要添加一个环境变量. 
然后运行

```
#source /etc/profile
```

结果就悲剧了, 

然后, 我自己的想法就是, 进入`/bin`或者`/sbin`下进行命令操作, 修改/etc/profile后, 怎么运行`source /etc/profile`呢.

这个我不知道. 

要么重启, 重启后能重新载入`/etc/profile`文件
要么执行`export PATH="/usr/sbin:/usr/bin:/usr/local/bin:/usr/local/sbin:/bin:/sbin" `, 设置临时的环境变量. 






