---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2022-08-06T10:47:16+08:00"
lastmod: "2022-08-06T10:47:16+08:00"
showToc: true
tags: [openwrt]
title: openwrt 自动重拨
---

有的 openwrt 固件不会断网自动重播，检查 `/etc/ppp/options` 文件是否存在如下参数，没有的话自己加上一般就好了

```bash
maxfail 0
persist
```

或者使用 crotab 重启接口，每天 3 点重新拨号

```bash
0 3 * * * ifdown wan && sleep 3 && ifup wan
```

#### Reference:

- https://www.right.com.cn/FORUM/thread-4107089-1-1.html
