---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2022-05-29T13:53:54+08:00"
description: golang 中的类型和它们的比较规则
lastmod: "2022-06-06T14:03:08+08:00"
showToc: true
tags: [Go]
title: Go 中的类型和比较
---

# Go 中的类型和比较

go 是一个强类型的语言，map 中要求键(key)必须是可比较的(comparable)，什么是可比较呢？就是能用操作符 `==` 的类型， 我们知道必须**两个类型一致才能比较**，否则编译器会报 `invalid operation: a == c (mismatched types...)` 的错误，准确的说基本类型（`int8`,`float32`,`string`）符合上面的原则，但 golang 中又有复合类型就不一样，先来看 go 中的类型

**1. 基本类型 (Basic Types)**

- 数字类型：

  `int8`, `uint8` (`byte`), `int16`, `uint16`, `int32` (`rune`), `uint32`, `int64`, `uint64`, `int`, `uint`, `uintptr`.

  `float32`, `float64`.

  `complex64`, `complex128`.

- 布尔类型：

  `bool`

- 字符串类型：

  `string`

  **2. 复合类型 (Basic Types)**

- 结构体(`struct`)类型
- 函数：go 中函数是一等公民，也是一种类型
- 数组(`array`)：包括长度和类型，不同长度的相同类型不属于同一类型
- 切片(`slice`)：切片有动态的长度和容量是一种引用类型
- 字典(`map`)：底层是哈希表也是一种引用类型
- 指针类型(`pointer`)
- 管道(`channel`)
- 接口类型(`interface`)

## 类型重定义(Type Definitions)和类型别名(Type Alias Declarations)

讲完了类型再来看看用户可以创建自己的类型（类型重定义）和创建别名，先看类型定义

```go
// Define a solo new type.
// type NewTypeName SourceType
type MyInt int
type Num int
```

上面定义了 `MyInt` , `Num` 两个类型，虽然他们的源类型都是 `int` 但他们是不同的类型，**所以他们是不可以比较的**，但可以通过转换成相同类型的再比较如

```go
var a MyInt = 1
var b Num = 1

// a == b can not compare
println(a == MyInt(b))  // true
```

有了 `type` 关键字使用者可以定义任何与业务相关的类型

新手在类型别名和类型重定义很容易搞混，类型别名只多了一个 `=`

```go
type Age = int
```

我们定义了 `Age` 只是 `int` 类型的一个别名，`Age` 类型是可以和 `int` 做比较的，**也就是创建了类型别名之后和原来的源类型是可以比较的**

```go
var age1 Age = 1
var age2 int = 1
println(age1 == age2)  // true
```

官方 `rune` 和 `int32` ，`byte` 和 `uint8` 就是类型别名

## 基本类型的比较

遵守如下规则

- 两个变量必须属于同一种基本类型
- 类型别名可以比较
- 类型重定义不能比较，可以通过转换成同一类型再比较

## 复合类型的比较

复合类型有多种情况，我们需要分开讨论

### 切片 (slice)

- 切片不可比较，只能与 `nil` 做比较

```go
	a := []int{}
	b := []int{}
	println(a == b)   // invalid operation: a == b (slice can only be compared to nil)
```

### 字典 (map)

- map 类型无法比较，只能与 `nil` 比较

```go
	a := map[string]int{}
	b := map[string]int{}
	println(a == b) // invalid operation: a == b (map can only be compared to nil)
```

### 函数 (function)

- 函数无法比较，只能与 `nil` 比较

```go
	a := func() {}
	b := func() {}
	println(a == b) // invalid operation: a == b (func can only be compared to nil)
```

### 结构体 (struct)

- 同一个类型的 struct 逐个字段比较
- 当 struct 中有字段是不可比较的成员类型时(slice, map, function)无法比较

```go
type Person struct {
	Name string
	Age  int
}

type Company struct {
	Count   int
	Persons []Person
}

func main() {
	a := Person{"foo", 1}
	b := Person{"bar", 1}
	println(a == b)   // false
	c := Company{2, []Person{a, b}}
	d := Company{2, []Person{a, b}}
	// print(c == d)  // invalid operation: c == d (struct containing []Person cannot be compared)
}
```

`Company` 中包含 `[]Person` 切片类型所以是不可比较的，尽管 `Person` 是可以比较的

### 数组 (array)

- 需要数组的长度和类型一致才能比较
- 数组的长度是类型的一部分，如果数组长度不同无法比较
- 值逐个比较
- 如果数组元素的类型是不可比较的类型(slice, map, function)，则数组也不能比较

```go
	a := [2]int{1, 2}
	b := [3]int{1, 2, 3}
	c := [3]int{1, 2, 3}
	d := [2]int{1, 1}
	// a == b       // invalid operation: a == b (mismatched types [2]int and [3]int)
	println(c == b) // true
	println(a == d) // false

	type myFunc func()
	e := [1]myFunc{}
	f := [1]myFunc{}
	// print(e == f) // invalid operation: e == f ([1]myFunc cannot be compared)

```

### 指针 (Pointer) 和 管道 (channel)

pointer 和 channel 归一起说是因为他们其实都是引用类型，指向一个内存地址，所以必须地址一致才会相等

- 指向的地址一致为相等
- 可以和 `nil` 比较判断是否为空

```go
package main

type A struct {
	x string
}

func main() {
	a1 := &A{"foo"}
	a2 := &A{"foo"}
	a3 := a1
	println(a1 == a2) // false 地址不同
	println(a1 == a3) // true

	var a4 *A
	println(a4 == nil) // true

	c1 := make(chan int)
	c2 := make(chan int)
	println(c1 == c2) // false

	var c3 chan int
	println(c3 == nil) // true
}
```

### 接口 (interface)

- 接口的比较需要接口的**动态类型 (`_type`)**和**值 (`data`)**都相等时才相同
- 实现接口的动态类型其指向一定是可比较，不能是不可比较的类型(slice, map, function)

```go
package main

type MyInterface interface {
	Echo() string
}

type A struct {
	AField string
}

func (a A) Echo() string {
	return a.AField
}

type B struct {
	BField string
}

func (b B) Echo() string {
	return b.BField
}

func compare(a, b MyInterface) bool {
	return a == b
}

func main() {
	A1 := A{
		AField: "foo",
	}
	A2 := A{
		AField: "foo",
	}
	B1 := B{
		BField: "foo",
	}
	println(compare(A1, A2)) // true
	println(compare(A1, B1)) // false 动态类型不同


	A3 := &A{
		AField: "foo",
	}
	A4 := &A{
		AField: "foo",
	}
	println(compare(A3, A4)) // false 使用了指针值不同

}
```

如果动态类型是不可比较的编译能通过，但运行时会 panic

```go
type MySlice []int

func (m MySlice) Echo() string {
	return "echo"
}

	e1 := MySlice([]int{1})
	e2 := MySlice([]int{1})
	compare(e1, e2)         // panic: runtime error: comparing uncomparable type main.MySlice
```

### 使用 reflect 比较 slice 和 map 类型

使用 `==` 操作符是无法比较 slice 和 map 的，可使用提供的 `reflect` 包中的 `reflect.DeepEqual` 是可以比较 slice 和 map 类型的

```go
m1 := map[string]int{"foo": 1, "bar": 2}
m2 := map[string]int{"bar": 2, "foo": 1}
m3 := map[string]string{"foo": "bar"}
println(reflect.DeepEqual(m1, m2))  // true
println(reflect.DeepEqual(m1, m3))  // false
var m4 map[string]int
println(reflect.DeepEqual(m4, nil)) // false
println(m4 == nil)                  // true

s1 := []int{1, 2}
s2 := []int{2, 1}
s3 := []int{1, 2}
println(reflect.DeepEqual(s1, s2))  // false
println(reflect.DeepEqual(s1, s3))  // true
var s4 []int
println(reflect.DeepEqual(s4, nil)) // false
println(s4 == nil)                  // true
```

- map 比较只要相同的键值对都相同两个 map 相等，与顺序无关
- slice 顺序不同值相同就不相等
- 声名空的 map 和 slice 和 nil 比较会返回 flase ， 如果用 `==` 操作符就返回 true

### 总结

- slice 、map、function 不可比较
- 任何 struct 和 array 含有上述不可比较的类型就不能比较
- 接口的比较要实现接口的动态类型和动态值都相同时才相等，动态类型也不能为不可比较的类型
- slice、map 的比较可以用 `reflect.DeepEqual`

### Reference

- https://go101.org/article/type-system-overview.html
- https://www.jianshu.com/p/a982807819fa
