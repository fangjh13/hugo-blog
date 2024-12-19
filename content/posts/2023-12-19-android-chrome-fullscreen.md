---
title: "Android 安卓 Chrome 全屏方法"
date: "2023-12-19T21:22:11+08:00"
lastmod: "2023-12-19T21:22:11+08:00"
description: "Android Chrome Fullscreen"
tags: [Android, Chrome]
categories: [Android, Chrome]
draft: false
TocOpen: false
UseHugoToc: true
showToc: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
---

最近公司的项目，遇到一台安卓系统的开发板，接了块 16:9 的大屏幕，我们开发了个web页应用想要全屏显式，下面是我捣鼓出来的小技巧

**前提**

- 电脑和安卓设备都要安装上chrome 应用
- 打开了开发者模式（一般都是 _Build number_ 多点几下）
  - https://developer.android.com/studio/debug/dev-options.html
- 电脑上要有 `adb` 工具
  - https://developer.android.com/tools/adb

1.  电脑有线或者无线 adb 连上设备，确保 `adb devices` 有设备在，我用的是无线连的
    ![](../images/image_20241219210427.png)
2.  电脑上打开chrome，地址栏输 `chrome://inspect#devices` 进去找到 _inspect_ 进去调试
    ![](../images/image_20241219211144.png)
3.  在控制台(Console)中输，完成
    ```
    document.documentElement.webkitRequestFullScreen();
    ```

##### Reference

- https://wiki.appstudio.dev/How_to_run_fullscreen_in_an_Android_Chrome_app#JavaScript-1
