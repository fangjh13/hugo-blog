---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2018-02-09T15:17:02+08:00'
lastmod: '2020-04-17T03:25:30+08:00'
showToc: true
tags: [Python]
title: collections模块中的namedtuple和defaultdict
---

## collections模块中的namedtuple和defaultdict

`collections`模块是Python中对内置类型(dict, list, tuple)的拓展。就是说它们本身具有普通内置类型的所有特性，并添加了新的功能。

### namedtuple()

`namedtuple(typename, field_names)`是`tuple`的子类，定义的元组可以通过属性访问，也易于理解，self-document。`field_names`可以是列表或者是通过空格或者逗号分隔的字符串。

```python
>>> from collections import namedtuple
>>> Point = namedtuple('point', 'x, y')
>>> p = Point(3, y=4)
>>> p
point(x=3, y=4)
>>> p[0]
3
>>> p.x
3
>>> x, y = p
>>> print(x, y)
3 4
```

`namedtuple`有几个比较有用的方法和属性

- `SomeNamedTuple._make(iterable)`类方法，从可迭代对象中取值，长度必须和传入的field_names一致，返回一个新的对象
- `somenamedtuple._asdict()`方法返回一个有序字典(OrderedDict)
- `somenamedtuple._replace(**kwargs)`方法替换值，返回一个新的对象，原来的不变
- `somenamedtuple._fields`属性返回键名(field name)，可用于创建新的named tuple

再来看两个[官方文档](https://docs.python.org/3/library/collections.html#collections.namedtuple)上的例子读取csv文件或者从sqlite读取

```python
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')

import csv
for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
    print(emp.name, emp.title)

import sqlite3
conn = sqlite3.connect('/companydata')
cursor = conn.cursor()
cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
for emp in map(EmployeeRecord._make, cursor.fetchall()):
    print(emp.name, emp.title)
```

### defaultdict

`defaultdict([default_factory[,...]])`顾名思义继承自`dict`它接收一个函数(default_factory)作为参数但它除了`dict`所有的属性和方法外，另外加了`__missing__`方法和`default_factory`属性。这两个的作用就是访问不存在的`key`时它会新增一个key，value为调用`default_factory`的值。

```python
>>> from collections import defaultdict
>>> d = defaultdict(list) # 默认函数为list
>>> print(d.items())  # 创建空字典
dict_items([])
>>> d.default_factory
<class 'list'>
>>> d['key']  # 调用会设置该key的值为空list并返回
[]
>>> print(d.items())
dict_items([('key', [])])
```

我们可以利用这个特性来做很多事，比如说给一个包含元组的的列表分类整理。

```python
>>> s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
>>> d = defaultdict(list)
>>> for k, v in s:
...     d[k].append(v)
>>> sorted(d.items())
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```

上面的代码其实可以用`dict`的`setdefault`实现，但效率和可读性差点，如下

```python
>>> d = {}
>>> for k, v in s:
...     d.setdefault(k, []).append(v)
>>> sorted(d.items())
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```

最后看一下自定义`default_factory`参数

```python
>>> def constant_factory(value):
...     return lambda: value
>>> d = defaultdict(constant_factory('<missing>'))
>>> d.update(name='John', action='ran')
>>> '%(name)s %(action)s to %(object)s' % d
'John ran to <missing>'
```

一个是比较讨巧的例子，利用了递归。

```python
>>> tree = lambda: defaultdict(tree)
>>> d = tree()
>>> d['colours']['favourite'] = 'yellow'
>>> d['animals']['pets']['name'] = ['cat', 'dog']
>>> import json
>>> print(json.dumps(d))
{"colours": {"favourite": "yellow"}, "animals": {"pets": {"name": ["cat", "dog"]}}}
```

#### References

1. [https://docs.python.org/](https://docs.python.org/3/library/collections.html)
2. [https://github.com/](https://github.com/yasoob/intermediatePython/blob/master/collections.rst)

