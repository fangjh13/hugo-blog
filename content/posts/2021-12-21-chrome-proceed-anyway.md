---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2021-12-21T03:31:55+08:00'
lastmod: '2021-12-21T03:32:21+08:00'
showToc: true
title: Chrome  继续访问证书错误的网页
---

## Chrome  继续访问证书错误的网页

chrome 有时访问 https 网页会出现警告 `NET::ERR_CERT_DATE_INVALID`

```shell
Your connection is not private
Attackers might be trying to steal your information from xxx.example.com (for example, passwords, messages, or credit cards). Learn more
NET::ERR_CERT_DATE_INVALID
```

有个彩蛋 如果要继续访问只需要点击页面然后键盘输入 **`thisisunsafe`** 就可以继续访问了


### Reference

[https://stackoverflow.com/questions/58802767/no-proceed-anyway-option-on-neterr-cert-invalid-in-chrome-on-macos](https://stackoverflow.com/questions/58802767/no-proceed-anyway-option-on-neterr-cert-invalid-in-chrome-on-macos)

