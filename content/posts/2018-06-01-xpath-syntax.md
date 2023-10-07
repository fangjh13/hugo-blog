---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2018-06-01T03:23:12+08:00'
lastmod: '2020-05-06T03:24:31+08:00'
showToc: true
tags: [W3C]
title: XPath语法
---

# XPath语法

XPath是在XML中检索信息的语言，也就是遍历XML文档，找出有用的信息。这里记录下XPath的语法

## 选取节点

XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。


|  表达式   |             描述             |
| -------- | --------------------------- |
| nodename | 选取此节点的所有子节点          |
| /        | 从根节点选取                  |
| //       | 从整个文档选取，不考虑它们的位置 |
| .        | 选取当前节点                  |
| ..       | 选取当前节点的父节点           |
| @        | 选取属性                      |

### 实例

```xml
 <?xml version="1.0" encoding="UTF-8"?>
<bookstore>

<book category="cooking">
  <title lang="en">Everyday Italian</title>
  <author>Giada De Laurentiis</author>
  <year>2005</year>
  <price>30.00</price>
</book>

<book category="children">
  <title lang="zh">哈利波特</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

<book category="web">
  <title lang="en">XQuery Kick Start</title>
  <author>James McGovern</author>
  <author>Per Bothner</author>
  <author>Kurt Cagle</author>
  <author>James Linn</author>
  <author>Vaidyanathan Nagarajan</author>
  <year>2003</year>
  <price>49.99</price>
</book>

<book category="web" cover="paperback">
  <title lang="en">Learning XML</title>
  <author>Erik T. Ray</author>
  <year>2003</year>
  <price>39.95</price>
</book>

</bookstore>
```

|    路径表达式	     |                                       结果                                        |
| ------------------ | -------------------------------------------------------------------------------- |
| bookstore	         | 选取 bookstore 元素的所有子节点                                                     |
| /bookstore	     | 选取根元素                                                                         |
| bookstore/book     | 选取属于 bookstore 的子元素的所有 book 元素                                         |
| //book	         | 选取所有 book 子元素，而不管它们在文档中的位置                                         |
| bookstore//book	 | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| //@lang	          | 选取名为 lang 的所有属性                                                            |

使用上面的xml文档作为测试，node.js可以使用`xpath`，Python可以使用`lxml`库

PS: 如果在chrome调试可以直接使用`$x("some xpath")`表达式

## 谓语（Predicates）

谓语用来查找某个特定的节点或者包含某个指定的值的节点，用来筛选。

谓语被嵌在方括号中。

### 实例

|              路径表达式              |                                        结果                                         |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| /bookstore/book[1]                 | 选取属于 bookstore 子元素的第一个 book 元素。注意索引从1开始。                           |
| /bookstore/book[last()]            | 选取属于 bookstore 子元素的最后一个 book 元素。                                        |
| /bookstore/book[last()-1]          | 选取属于 bookstore 子元素的倒数第二个 book 元素。                                       |
| /bookstore/book[position()<3]      | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。                                |
| //title[@lang]                      | 选取所有拥有名为 lang 的属性的 title 元素。                                            |
| //title[@lang='en']                | 选取所有 title 元素，且这些元素拥有值为 en 的 lang 属性。                                |
| /bookstore/book[price>35.00]       | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。               |
| /bookstore/book[price>35.00]/title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |


## 选取未知节点

XPath 通配符可用来选取未知的 XML 元素。

| 通配符  |         描述          |
| ------ | --------------------- |
| *      | 匹配任何元素节点。      |
| @*     | 匹配任何属性节点。      |
| node() | 匹配任何类型的节点。    |
| text() | 匹配任何文本类型的节点。 |

### 实例

|    路径表达式    |              结果               |
| -------------- | ------------------------------ |
| /bookstore/*   | 选取 bookstore 元素的所有子元素。 |
| //*            | 选取文档中的所有元素。            |
| //title[@*]    | 选取所有带有属性的 title 元素。   |
| //title/text() | 选取所有title元素下的文本         |

## 选取若干路径

通过在路径表达式中使用“|”运算符，您可以选取若干个路径。

### 实例

|           路径表达式            |                  结果                  |
| ------------------------------ | -------------------------------------- |
| `//book/title | //book/price` | 选取 book 元素的所有 title 和 price 元素 |
| `//title | //price`           | 选取文档中的所有 title 和 price 元素     |

## 轴（axes）

轴可定义相对于当前节点的节点集。

|       轴名称        |                       结果                        |
| ------------------ | ------------------------------------------------ |
| ancestor           | 选取当前节点的所有先辈（父、祖父等）。                |
| ancestor-or-self   | 选取当前节点的所有先辈（父、祖父等）以及当前节点本身。   |
| attribute          | 选取当前节点的所有属性。                            |
| child              | 选取当前节点的所有子元素。                           |
| descendant         | 选取当前节点的所有后代元素（子、孙等）。               |
| descendant-or-self | 选取当前节点的所有后代元素（子、孙等）以及当前节点本身。 |
| following          | 选取文档中当前节点的结束标签之后的所有节点。           |
| namespace          | 选取当前节点的所有命名空间节点。                      |
| parent             | 选取当前节点的父节点。                              |
| preceding          | 选取文档中当前节点的开始标签之前的所有节点。           |
| preceding-sibling  | 选取当前节点之前的所有同级节点。                      |
| self               | 选取当前节点。                                     |

### 实例

轴的使用语法如下

`轴名称::节点测试[谓语]`

|          例子           |                            结果                             |
| ----------------------- | ----------------------------------------------------------- |
| child::book            | 选取所有属于当前节点的子元素的 book 节点。                       |
| attribute::lang        | 选取当前节点的 lang 属性。                                    |
| child::*               | 选取当前节点的所有子元素。                                     |
| attribute::*           | 选取当前节点的所有属性。                                       |
| child::text()          | 选取当前节点的所有文本子节点。                                  |
| child::node()          | 选取当前节点的所有子节点。                                     |
| descendant::book       | 选取当前节点的所有 book 后代。                                 |
| ancestor::book         | 选择当前节点的所有 book 先辈。                                 |
| ancestor-or-self::book | 选取当前节点的所有 book 先辈以及当前节点（如果此节点是 book 节点） |
| child::*/child::price  | 选取当前节点的所有 price 孙节点。                              |

## XPath 运算符

XPath支持`>`，`<`，`>=`，`<=`，`or`，`and`运算符

### 实例

|               例子               |             结果              |
| -------------------------------- | ---------------------------- |
| /bookstore/book[price>35]/title | 选取 price 大于35的 title 节点 |

## 常用函数

XPaht内置了很多函数，上面我们已经接触国一些如`last()`、`position()`以下列出常用的函数，完整可以查看[W3C文档](https://www.w3.org/TR/xpath-functions-31/)

|          路径表达式          |                      结果                      |
| --------------------------- | --------------------------------------------- |
| count(node-set)            | 返回节点的数量                                  |
| contains(string1, string2) | 返回 boolean 值，string1 是否包含 string2 的内容 |
| last()                     | 返回最后一个节点的索引                            |
| not(boolean)               | 取反                                           |
| string-length(string)      | 返回字符长度                                         |

### 实例

|                  例子                  |                   结果                    |
| -------------------------------------- | ----------------------------------------- |
| count(//book[price>35])               | 统计 price 大于 35 的 book 节点一共有几个    |
| //book[contains(author, 'Ray')]       | 选取 author 节点包含 'Ray' 的 book 节点     |
| //title[contains(@lang, 'zh')]        | 选取 lang 属性包含 'zh' 的 title 节点       |
| //title[contains(text(), '波特')]      | 选取 title 节点内容包含 '波特' 的 title 节点 |
| //title[not(@lang='en')]              | 选取 lang 属性不等于 'en' 的 title 节点     |
| string-length(//book[2]/title/text()) | 第二个 book 节点中 title 节点下的文本字符长度 |


## Reference

1. [www.w3school.com.cn](https://www.w3school.com.cn/xpath/xpath_syntax.asp)
2. [docs.oracle.com](https://docs.oracle.com/cd/E35413_01/doc.722/e35419/dev_xpath_functions.htm)
