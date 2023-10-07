---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-10-08T07:26:02+08:00'
lastmod: '2017-11-29T02:48:29+08:00'
showToc: true
tags: [Python]
title: Python中@propery 使用
---

##  Python中@propery 装饰器的使用

python是面向对象的语言，当我们想在类中封装一个变量，并提供设置和获取值的时候，往往会使用如下方法。

    class Student:
        def __init__(self, score):
    	    self.__score = score

        def get_score(self):
    	    return self.__score

        def set_score(self, score):
    	    self.__score = score

然后如下输出

    >>> s = Student(99)
    >>> s.get_score()
    99
    >>> s.set_score(100)
    >>> s.get_score()
    100

这是最简单的封装，但有没有像`s.score`这样属性直接调用`s.score = 100`直接赋值的呢，有！而且很简单。

    class Student:
    	def __init__(self, score):
    		self.score = score

没错就是`__init__`方法直接设置。

    >>> s = Student(99)
    >>> s.score
    99
    >>> s.score = 100
    >>> s.score
    100

以上的方法没有封装，而且如果我想要判断score的值范围（0~100）也无法做到，使用第一种`set_score`倒是可以做到。

    class Student:
    	def __init__(self, score):
    		self.set_score(score)

    	def get_score(self):
    		return self.__score

    	def set_score(self, score):
    		if score < 0:
    			self.__score = 0
    		elif score > 100:
    			self.__score = 100
    		else:
    			self.__score = score

效果

    >>> s = Student(1000)
    >>> s.get_score()
    100
    >>> s.set_score(80)
    >>> s.get_score()
    80

看起来已经满足我们的需求但不够Pythonic，所以Python提供了`@propery`装饰器

    class Student:
    	def __init__(self, score):
    		self.score = score

    	@property
    	def score(self):
    		return self.__score

    	@score.setter
    	def score(self, score):
    		if score < 0:
    			self.__score = 0
    		elif score > 100:
    			self.__score = 100
    		else:
    			self.__score = score

运用了Python提供的property装饰器，结果和上面一样。但更优雅可读

    >>> s = Student(101)
    >>> s.score
    100
    >>> s.score = -99
    >>> s.score
    0


使用`propery`需要注意以下几点
> * 使用property的两个方法都是相同的名字(score)只是参数不同，设置(setter)方法上的装饰器要用`score.setter`装饰
> * 在`__init__`方法中`self.score = score`是初始化`self.__score`的值
> * 在另外两个名称相同的方法中使用`self.__score`私有变量储存值

还有一种写法参考[官方文档](https://docs.python.org/3/library/functions.html#property)个人感觉比较繁琐。

    class Student:
    	def __init__(self, score):
    		self.score = score

    	def __get_score(self):
    		return self.__score

    	def __set_score(self, score):
    		if score < 0:
    			self.__score = 0
    		elif score > 100:
    			self.__score = 100
    		else:
    			self.__score = score

    	score = property(__get_score, __set_score)

注意使用了`__get_score`和`__set_score`成为私有方法(前面加`__`)，使用户不能访问。

**参考：**

[https://www.python-course.eu/](https://www.python-course.eu/python3_properties.php)

[https://docs.python.org/3/](https://docs.python.org/3/library/functions.html#property)

