---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2019-10-16T15:31:21+08:00'
description: linux core dump 从内存中读取shell脚本
lastmod: '2019-10-16T15:31:21+08:00'
showToc: true
tags: [Shell, Linux]
title: 使用 Core dump 解密加密的sh脚本
---

# 使用 Core dump 解密加密的脚本

之前遇到网上的集成的shell脚本有点问题想手动修改下，发现脚本是加密的，网上找了好久发现有`gzexe`、`shc`加密方法都尝试了一遍，可惜解密都不成功，最后用了一个粗暴的办法就是任何程序总要加载到内存运行的吧，那就直接中断coredump查看内存里的内容，以下是具体方法。

root用户执行如下命令

```shell
ulimit -c unlimited
echo "/core_dump/%e-%p-%t.core" > /proc/sys/kernel/core_pattern
mkdir /core_dump
```

以上第一句是设置内核coredump大小，这里设置不限制。第二句是设置coredump存储位置和格式，`%e`代表可执行程序名，`%p`代表pid， `%t`代表生成时间。然后去执行脚本如`xxx.sh`

```shell
./xxx.sh 6 start & (sleep 0.01 && kill -SIGSEGV $!)
```

之后会输出类似`[1]+  Segmentation fault (core dumped)...`的提示，然后查看`/core_dump`文件夹下，就会有dump出来的文件了，直接vim打开查看会有一些乱码手动处理一下就可以了。

如果在core_dump文件夹下没有dump出来的文件，可使用如下命令测试然后查看是否有文件生成。

```shell
sleep 15 &
killall -SIGSEGV sleep
```

正常情况下core_dump文件夹下会有以sleep开头的文件。
