Introduction

Go's slice type provides a convenient(方便的) and efficient(有效的) means of working with sequences(序列) of typed data. Slices are analogous(类似于) to arrays in other languages, but have some unusual properties. This article(文章) will look at what slices are and how they are used.

介绍
Go的切片类型提供了一种方便有效的处理类型数据序列的方法,
切片类似于其他语言中的数组,但是有一些不寻常的属性,
本文将介绍哪些切片以及它们的使用方式。


Arrays
The slice type is an abstraction(抽象) built on top of Go's array type, and so to understand slices we must first understand arrays.

An array type definition specifies a length and an element type. For example, the type [4]int represents an array of four integers. An array's size is fixed; its length is part of its type ([4]int and [5]int are distinct, incompatible types). Arrays can be indexed in the usual way, so the expression s[n] accesses the nth element, starting from zero.

数组

切片类型是构建在Go的数组类型之上的抽象，因此为了理解切片，我们必须首先理解数组。
数组类型定义指定长度和元素类型。例如，type [4] int表示一个包含四个整数的数组。数组的大小是固定的;它的长度是它的类型的一部分（[4] int和[5]int是不同的，不兼容的类型）。数组可以通常的方式进行索引，因此表达式 s[n] 从零开始访问第n个元素

```
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```

Arrays do not need to be initialized(初始化) explicitly(明确地); the zero value of an array is a ready-to-use array whose elements are themselves zeroed:


数组不需要显式初始化;数组的零值是一个现成的数组，其元素本身为零：


The in-memory representation(表示) of [4]int is just four integer values laid out(布局) sequentially(顺序):


[4] int的内存中表示只是顺序排列的四个整数值：

![image](https://blog.golang.org/go-slices-usage-and-internals_slice-array.png)



Go's arrays are values. An array variable denotes the entire array; it is not a pointer to the first array element (as would be the case in C). This means that when you assign or pass around an array value you will make a copy of its contents. (To avoid the copy you could pass a pointer to the array, but then that's a pointer to an array, not an array.) One way to think about arrays is as a sort of struct but with indexed rather than named fields: a fixed-size composite value.

An array literal can be specified(规定) like so:


Go的数组是值。数组变量表示整个数组;它不是指向第一个数组元素的指针（如C中的情况)。这意味着当您分配或传递数组值时，您将复制其内容。(为了避免复制，你可以传递一个指向数组的指针，但那是一个指向数组的指针，而不是一个数组。）一种思考数组的方法是作为一种结构，但有索引而不是命名字段：一个固定的大小的复合值


可以像这样指定数组文字：




```
b := [2]string{"Penn", "Teller"}
```
Or, you can have the compiler count the array elements for you:

或者，你可以让编译器为您计算数组元素:

```
b := [...]string{"Penn", "Teller"}
```
In both cases, the type of b is [2]string.
在这两种情况,b的类型都是[2]string



Slices
Arrays have their place, but they're a bit inflexible(死板、刻板), so you don't see them too often in Go code. Slices, though, are everywhere. They build on arrays to provide great power and convenience.

The type specification(规格) for a slice is []T, where T is the type of the elements of the slice. Unlike an array type, a slice type has no specified length.

A slice literal(字面意思) is declared just like an array literal, except you leave out the element count:

```
letters := []string{"a", "b", "c", "d"}
```
切片
数组有它们的位置，但是它们有点不灵活，所以你不会在Go代码中经常看到它们。然而，切片无处不在。它们以阵列为基础，提供强大的功能和便利性

切片的类型规范是[]T，其中T是切片元素的类型。与数组类型不同，切片类型没有指定的长度。切片文字声明就像数组文字一样,除了你省略元素数量


A slice can be created with the built-in function called make, which has the signature,

```
func make([]T, len, cap) []T
```

where T stands for the element type of the slice to be created. The make function takes a type, a length, and an optional capacity. When called, make allocates an array and returns a slice that refers to that array.


其中T代表要创建的切片的元素类型,make函数采用类型/长度/和可选容量。调用时，make分配一个数组并返回一个引用该数组的切片。
```
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```
When the capacity argument is omitted, it defaults to the specified length. Here's a more succinct version of the same code:
```
s := make([]byte, 5)
```
The length and capacity of a slice can be inspected using the built-in len and cap functions.
```
len(s) == 5
cap(s) == 5
```
The next two sections discuss the relationship between length and capacity.

The zero value of a slice is nil. The len and cap functions will both return 0 for a nil slice.

切片的零值为零,对于零切片，len和cap函数都将返回0。

A slice can also be formed by "slicing" an existing slice or array. Slicing is done by specifying a half-open range with two indices separated by a colon. For example, the expression b[1:4] creates a slice including elements 1 through 3 of b (the indices of the resulting slice will be 0 through 2).
```
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
```

The start and end indices of a slice expression are optional; they default to zero and the slice's length respectively:
```
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```
This is also the syntax to create a slice given an array:
```
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```
