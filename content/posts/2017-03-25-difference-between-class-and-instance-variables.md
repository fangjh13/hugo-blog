---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-03-25T14:38:27+08:00'
lastmod: '2020-04-11T14:47:42+08:00'
showToc: true
tags: [Python]
title: Python中的类变量(class variables)和实例变量(instance variables)
---

# Python中的类变量(class variables)和实例变量(instance variables)

## 类变量(Class Variables)

类变量是所有实例共享的变量，类和实例都能访问。也就是说类创建了之后类变量就已经初始化了之后所有的实例都共享这个变量不会单独创建。类变量需要定义在类的里面方法的外面

```python hl_lines="2"
class Foo:
    cls_var = "this is class method"

    def __init__(self):
        pass


f0 = Foo()
f1 = Foo()
print(f0.cls_var, id(f0.cls_var))
print(f1.cls_var, id(f1.cls_var))
print(Foo.cls_var, id(Foo.cls_var))
Foo.cls_var = "changed"
print(f0.cls_var, id(f0.cls_var))
print(f1.cls_var, id(f1.cls_var))
print(Foo.cls_var, id(Foo.cls_var))
```

输出

```shell
this is class method 4328472272
this is class method 4328472272
this is class method 4328472272
changed 4328412136
changed 4328412136
changed 4328412136
```

类`Foo`定义了`cls_var`类变量没有绑定到任何实例，类`Foo`和实例`f0`,`f1`它们引用的变量`cls_var`都是同一个地址，所以共享一个类变量，只要改变了所有引用的都会影响。以下是统计一共创建了多少实例的例子

```python
class Person:
    count = 0              # class variable

    def __init__(self, name):
        self.name = name
        Person.count += 1  # 递增

s0 = Person(name="s0")
s1 = Person(name="s1")
print(Person.count)         # 2
```

`__init__`构造方法引用`Person.count`类变量每次实例对象都会自增

## 实例变量(Instance Variables)

实例变量是每个实例拥有的单独变量在实例创建后才能访问，各个实例变量相互独立。与类变量不同的是实例变量一般定义在方法内(`__init__`方法)

```python hl_lines="3"
class Person:
    def __init__(self, name):
        self.name = name         # 实例变量

s0 = Person("student0")
s1 = Person("student1")
print(s0.name, id(s0.name))
print(s1.name, id(s1.name))
s1.name = "teacher"
print(s0.name, id(s0.name))
print(s1.name, id(s1.name))
```

输出

```shell
student0 4304628528
student1 4304628464
student0 4304628528
teacher 4303557272
```

上面每个实例初始化后都会单独绑定一个`name`变量，通过`实例.变量名`访问，每个实例互不影响，也不能通过类`Person`访问，如果访问`Person.name`会抛出`AttributeError`

## 其他

下面看个特殊的例子

```python
class Person:
    name = "person"

    def __init__(self):
        pass

s0 = Person()
s1 = Person()
print(s0.name)            # person
print(s1.name)            # person
print(Person.name)        # person
s0.name = "changed"
print(s0.name)            # changed
print(s1.name)            # person
print(Person.name)        # person
```

输出

```shell
person
person
person
changed
person
person
```

按照刚才类变量共享的原则不是通过`s0`改变了`name`所有引用的都会收到影响么，怎么就变了`s0`的。其实这是实例`s0`本来引用的是类变量，在重新赋值之后实例新建了`name`属于自己本身的实例属性存在`__dict__`属性中，可以通过`s0.__dict__`查看

```python
>>> s0.__dict__
{'name': 'changed'}
>>> s1.__dict__
{}
>>> s0.__class__.name    # 访问类变量
'person'
```

## Conclusions

- 类变量是所有实例共享的，实例变量绑定到单独的实例每个都不同
- 类变量定义在类内方法外实例和类都能访问，实例变量定义在方法内只能实例访问
- 如果实例新赋值一个和类变量相同的变量名，不会覆盖掉类变量，会自己新建实例变量