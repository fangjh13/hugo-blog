---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2020-10-27T10:04:12+08:00'
lastmod: '2020-10-27T10:04:12+08:00'
showToc: true
tags: [regex]
title: 正则表达式中的预查
---

# 正则表达式中的预查

有时候使用正则会用到非获取匹配，就是不进行存储供以后使用，也就是正则中的预查，预查分为正向预查(lookahead)和反向预查(lookbehind)。

- 正向预查

  `(?=pattern)`正向肯定预查(Positive lookahead)。如`Python(?=3)`匹配Python后跟3的语句，如输入`Python3`但其中最后的`3`不算进结果，返回`Python`
  
  `(?!pattern)`正向否定预查(Negative lookahead)。和上面的类似只是否定的，`Python(?!3)`匹配后面不带`3`的句子，输入`Python2`，也是返回`Python`

- 反向预查
  
  `(?<=pattern)`反向肯定预查(Positive lookbehind)。如`(?<=2)Python`其实和上面也差不多反向就是向左匹配就是匹配Python前面是2的语句，如输入`2Python`，返回`Python`

  `(?<!pattern)`反向否定预查(Negative lookbehind)。如`(?<!3)Python`匹配Python前面不是3的输入，如输入`2Python`，返回`Python`

还有一个长的挺像的这里也记录下

`(?:pattern)` 匹配pattern但不获取匹配结果。 `(?:t|b)oy`只匹配boy或者toy，和`toy|boy`一样但更简洁，当然如果使用`(t|b)oy`就会多一个group
