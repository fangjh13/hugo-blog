---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2019-01-25T08:59:14+08:00'
lastmod: '2020-05-25T08:59:14+08:00'
showToc: true
tags: [Mac, macOS]
title: Mac不安装第三方应用读写NTFS格式硬盘
---

# Mac不安装第三方应用读写NTFS格式硬盘

首先插入硬盘或者U盘，现在的盘只能读取，我们先`umount`以`/dev/disk3s2`分区为例

```shell
❯ mount                           # 列出挂载分区
/dev/disk1s5 on / (apfs, local, read-only, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk1s1 on /System/Volumes/Data (apfs, local, journaled, nobrowse)
/dev/disk1s4 on /private/var/vm (apfs, local, journaled, nobrowse)
/dev/disk3s2 on /Volumes/TOSHIBA EXT (ntfs, local, nodev, nosuid, read-only, noowners)
❯ sudo umount /Volumes/TOSHIBA\ EXT
```

创建挂在路径，然后手动挂载

```shell
sudo mkdir /Volumes/mount
sudo mount -t ntfs -o rw,auto,nobrowse /dev/disk3s2 /Volumes/mount
cd /Volumes/mount
```

就这样移动硬盘可读写了，也可以打开Finder试试