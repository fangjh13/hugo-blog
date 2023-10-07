---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2021-04-08T03:17:59+08:00"
lastmod: "2022-03-31T08:14:14+08:00"
showToc: true
tags: [Go]
title: " Go 中的 Arrays 和 Slices"
---

# Golang 中的 Arrays 和 Slices

在 go 语言中，我们经常使用`Slices`类型因为它的方便和灵活，它和另一个`Arrays`类型有着密切的关系，Slices 是建立在 Arrays 的基础上的，搞明白它们的原理能使我们更加的轻松的使用它们

## Arrays

Arrays 和别的语言(C、Java)的类型一样，**有固定的长度**，在内存里是一块连续的空间，用以存储相同类型的 types。用如下方式申明

```go
var array [5]int
```

1. 像`[size]T`在 go 中申明`array`，size 是 type 的一部分 如上面的`[5]int`代表 5 个 int 元素的 Arrays，和另一个如`[10]int`是不同的类型，Arrays 有确定的长度。并且申明之后带默认值(各类型的零值)。也可以使用`[...]`符号省略 size 申明，编译器自动计算 如`array := [...]int{1, 2, 3, 4, 5}`
2. 变量`array`引用的是整个 Array 而不是 Array 的第一个元素，如果将一个数组另外赋值是将这个数组拷贝了一份，数组作为函数参数也是将整个数组拷贝一份，非引用数组的指针

## Slices

就是因为 Arrays 比较难用，go 在此基础上建立了 Slices，它是可以**动态调整长度(dynamically-sized)的描述 Arrays 一部分的 types**，Slices 可以使用切片数组的方式得到

```go
array := [5]int{1, 2, 3, 4, 5}  // Arrays
var slice = array[1:4]    // same as `var slice []int = array[1:4]`
fmt.Println(slice)        // [2 3 4]
fmt.Println(len(slice))   // 3
fmt.Println(cap(slice))   // 4
array[2] = 9
fmt.Println(slice)        // [2 9 4]
slice = slice[:4]
fmt.Println(slice)        // [2 9 4 5]
fmt.Println(array)        // [1 2 9 4 5]
```

1. Slices 的底层指向的是 Arrays，**它描述底层一部分的 Arrays**，如果被引用的 array 变化了，引用它的所有 slice 都会随之变化
2. Slices 有长度(l*ength*)和容量(_capacity_)，分别通过`len`和`cap`获取，长度就是切片的长度，**容量是从 slice 的第一个元素到底层引用的 Arrays 的末尾元素的个数**，也就是这个 slices 最大能达到的长度，例如上面的 slice 从第二个元素`2`到引用底层 array 末尾的元素`5`所以 cap 等于 4，所以 slices 可以动态调整但不能大于它的容量

将整个 Arrays 转化成 Slices 可以忽略前后索引`slice := array[:]`。Slices 以`[]T`的形式申明

```go
var slice1 []int
slice2 := []int{1, 2, 3}
slice3 := make([]int, 3)
```

第一种这看起来和申明 Arrays 差不多，就是少了 size。第二种通过`:=`并初始化了三个值。第三种是通过`make`可以指定`len`和`cap`，格式为`make([]T, len, cap)`。更加详细的`make`用法下面再说

### slice header

说了很多知道了 Slices 不是 Arrays，那`var slice = array[1:4]`中`slice`引用的是什么呢，我们可以想象成**`slice`变量是一个存储指向 Slices 头(slice header)的指针加一个长度(length)和容量(capacity)的数据结构**

```go
array := [5]int{1, 2, 3, 4, 5}

type sliceHeader struct {
  Length int
  Capacity   int
  ZerothElement *int
}

slice := sliceHeader{
  Length: 3,
  Capacity: 5,
  ZerothElement: &array[1],
}
```

PS: 我们可以使用`unsafe.Pointer`和`reflect.SliceHeader`查看 slice header

```go
slice := array[1:4]
fmt.Printf("%+v", *(*reflect.SliceHeader)(unsafe.Pointer(&slice))) // {Data:824634355504 Len:3 Cap:4}
```

在 Slices 传参给函数当作参数时也是传的 slice 头(slice header)的一个副本给函数，如果有修改 slices 其实是修改了底层的 Arrays，引用这个 Arrays 的所有 Slices 都会随之改变。看下面官方博客上的例子

```go
package main

import "fmt"

func AddOneToEachElement(slice []byte) {
	for i := range slice {
		slice[i]++
	}
}

func SubtractOneFromLength(slice []byte) []byte {
	slice = slice[0: len(slice) - 1]
	return slice
}


var buffer [265]byte

func main() {
	slice := buffer[10:20]
	for i := 0; i < len(slice); i++ {
		slice[i] = byte(i)
	}
	fmt.Println("before", slice)
	AddOneToEachElement(slice)
	fmt.Println("after", slice)

	fmt.Println("Before: len(slice) =", len(slice))
	newSlice := SubtractOneFromLength(slice)
	fmt.Println("After:  len(slice) =", len(slice))
	fmt.Println("After:  len(newSlice) =", len(newSlice))
}
```

输出结果

```go
before [0 1 2 3 4 5 6 7 8 9]
after [1 2 3 4 5 6 7 8 9 10]
Before: len(slice) = 10
After:  len(slice) = 10
After:  len(newSlice) = 9
```

调用`AddOneToEachElement`的 slice 底层还是指向 Arrays，所以改变了之后所有 slice 引用的 Array 都会改变(都增加了 1)

传给`SubtractOneFromLength`的是 slice header 的一个拷贝，在函数里面改变了 slice 但在外部的 slice 是不会改变的如果想要更改后的 slice 只能返回值变量重新赋值(newSlice)得到，也就是 pass by value。

### make

使用内置的`make`函数可以指定创建固定长度(len)和容量(cap)的 slices

```go
slice := make([]int, 5, 10)
```

如上创建了一个 slices 长度为 5，容量为 10，且已经初始化默认值为 type 的零值这里就是 0。如果忽略第三个容量参数时，cap 默认和 len 的值相同。下面创建一个容量两倍新的 slices，填入原来 slice 的值，也就是扩容了一倍

```go
slice := make([]int, 5, 10)
fmt.Printf("len: %d, cap: %d\n", len(slice), cap(slice))   // len: 5, cap: 10
newSlice := make([]int, len(slice), 2*cap(slice))
for i := range slice {
	newSlice[i] = slice[i]
}
fmt.Printf("len: %d, cap: %d\n", len(newSlice), cap(newSlice))   // len: 5, cap: 20
```

### copy

上面手动使用 for 循环复制原来的`slice`，golang 中有自带的`copy`函数能实现同样的功能

```go
newSlice := make([]int, len(slice), 2*cap(slice))
copy(newSlice, slice)
```

`copy`是很聪明的，他自动判断复制两个 slices 中的最小长度，返回复制个数

### append

`make`创建的 slices 都是固定容量的如果新增元素容量超了怎么办呢，有个`append`函数能很好的处理其中的问题

```go
func main() {
	slice := make([]int, 0, 2)
	for i := 0; i < 5; i++ {
		slice = append(slice, i)
		fmt.Println(&slice[0], slice)
	}
}
```

输出

```go
❯ go run code.go
0xc0000140d0 [0]
0xc0000140d0 [0 1]
0xc0000220a0 [0 1 2]
0xc0000220a0 [0 1 2 3]
0xc00001a100 [0 1 2 3 4]
```

注意上面 slice header 它是不同的，当 slices 的容量不够无法 append 时`apped`函数会新建一个 slices，增大容量返回新的 slices，所以每次使用`append`函数时必须把返回结果赋值到 slice 变量

## Reference

1. [https://blog.golang.org/slices-intro](https://blog.golang.org/slices-intro)
2. [https://blog.golang.org/slices](https://blog.golang.org/slices)
