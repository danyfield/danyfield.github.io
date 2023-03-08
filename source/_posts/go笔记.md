---
title: go笔记
date: 2023-03-07 17:55:49
tags: "go"
categories: "go"
---

## 函数

Go 里面有三种类型的函数，函数参数、返回值以及它们的类型被统称为函数签名：  

- 普通的带有名字的函数
- 匿名函数或者lambda函数
- 方法（Methods）

这样是不正确的 Go 代码：

```go
func g()
{
}
```

它必须是这样的：

```go
func g() {
}
```

函数重载 (function overloading) 指的是可以编写多个同名函数，只要它们拥有不同的形参/或者不同的返回值，在 Go 里面函数重载是不被允许的

函数值之间可以比较：若引用的是相同函数或都是 `nil` ，则认为是相同的函数

### 函数的参数和返回值

任何一个有返回值（单个或多个）的函数都必须以 `return` 或 `panic`结尾

在函数调用时，切片 (slice)、字典 (map)、接口 (interface)、通道 (channel) 这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）

#### 命名返回值

命名返回值作为结果形参被初始化为相应类型的零值，当需要返回的时候，我们只需要一条简单的不带参数的 `return` 语句，即使只有一个命名返回值也需用括号括起来；当有多个非命名返回值时需用括号括起来，如`(int,int)`，任何一个非命名返回值在`return`中都要指出返回值变量或是一个可计算的值

#### 传递变长参数

若变长参数的类型并不都相同时的传递方法：

1. 定义一个结构类型，假设它叫 `Options`，用以存储所有可能的参数：

   ```go
   type Options struct {
   	par1 type1,
   	par2 type2,
   	...
   }
   ```

   函数 `F1()` 可以使用正常的参数 `a` 和 `b`，以及一个没有任何初始化的 `Options` 结构： `F1(a, b, Options {})`。如果需要对选项进行初始化，则可以使用 `F1(a, b, Options {par1:val1, par2:val2})`

2. 使用空接口：

   若一个变长参数的类型没有被指定，则可以使用默认的空接口`interface{}`，该方案不仅可以用于长度未知的参数，还可以用于任何不确定类型的参数：

   ```go
   func typecheck(..,..,values … interface{}) {
   	for _, value := range values {
   		switch v := value.(type) {
   			case int: …
   			case float: …
   			case string: …
   			case bool: …
   			default: …
   		}
   	}
   }
   ```

### defer和追踪

关键字 `defer` 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数

当有多个 `defer` 行为被注册时，它们会以逆序执行（类似栈，即后进先出）

关键字 `defer` 允许我们进行一些函数执行完成后的收尾工作

#### 使用`defer`实现代码追踪

```go
package main

import "fmt"

func trace(s string) string {
	fmt.Println("entering:", s)
	return s
}

func un(s string) {
	fmt.Println("leaving:", s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

#### 使用 `defer` 语句记录函数的参数与返回值

```go
package main

import (
	"io"
	"log"
)

func func1(s string) (n int, err error) {
	defer func() {
		log.Printf("func1(%q) = %d, %v", s, n, err)
	}()
	return 7, io.EOF
}

func main() {
	func1("Go")
}
```

```
Output: 2011/10/04 10:46:11 func1("Go") = 7, EOF
```

### 内置函数

| 名称                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `close()`                        | 用于管道通信                                                 |
| `len()`、`cap()`                 | `len()` 用于返回某类型的长度或数量（字符串、数组、切片、`map` 和管道）；`cap()` 用于返回某个类型的最大容量（只能用于数组、切片和管道，不能用于 `map`） |
| `new()`、`make()`                | 分配内存，`new()` 值类型和自定义类型，`make` 内置引用类型（切片、`map` 和管道） |
| `copy()`、`append()`             | 用于复制和连接切片                                           |
| `panic()`、`recover()`           | 均用于错误处理机制                                           |
| `print()`、`println()`           | 底层打印函数，在部署环境中建议使用`fmt`包                    |
| `complex()`、`real ()`、`imag()` | 用于创建和操作复数                                           |

### 闭包

可以将匿名函数赋值给变量，即保存函数的地址到变量中，然后通过变量名对函数进行调用，也可以直接对匿名函数进行调用，匿名函数也被称为闭包：

`func(x, y int) int { return x + y } (3, 4)`

#### 应用闭包：将参数作为返回值

```go
package main

import "fmt"

func main() {
    var f = Adder()
    fmt.Print(f(1),"-")
    fmt.Print(f(20),"-")
    fmt.Print(f(300))
}

func Adder() func(int) int {
    var x int
    return func(delta int) int {
        x += delta
        return x
    }
}

//output
//1 - 21 - 321
```

闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量

##### 工厂函数

一个返回值为另一个函数的函数；在创建一系列相似函数时非常有用

```go
func MakeAddSuffix(suffix string) func(string) string {
    return func(name string) string {
        if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
    }
}

addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")
addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```

#### 使用闭包调试

在分析和调试复杂的程序时，无数个函数在不同的代码文件中相互调用，如果这时候能够准确地知道哪个文件中的具体哪个函数正在执行，对于调试是十分有帮助的；包 `runtime` 中的函数 `Caller()` 提供了相应的信息，因此可以在需要的时候实现一个 `where()` 闭包函数来打印函数执行的位置：

```go
where := func() {
    _,file,line,_ := runtime.Caller(1)
    log.Printf("%s:%d",file,line)
}
where()
```

#### 通过内存缓存提升性能

在大量计算时，提升性能最直接有效的方式即避免重复计算，在缓存重复利用相同计算的结果称为内存缓存

如斐波那契数列，要计算数列中第 n 个数字，需要先得到之前两个数的值，但很明显绝大多数情况下前两个数的值都是已经计算过的，此时将第 n 个数的值存在数组中索引为 n 的位置

## 数组和切片

### 声明和初始化

#### 数组

数组是具有相同 **唯一类型** 的一组已编号且长度固定的数据项序列，声明格式为：

`var arr1 [5]int`

go数组是值类型，可通过`new()`创建：`var arr1 = new([5]int)`，故在函数中作为参数传入时不修改原始数组，若想修改，则需使用`&`操作符以引用方式传入

该种方式和 `var arr2 [5]int` 的区别是：`arr1` 的类型是 `*[5]int`，而 `arr2` 的类型是 `[5]int`；这样的结果就是当把一个数组赋值给另一个时，需要再做一次数组内存的拷贝操作：

```go
arr2 := *arr1
arr2[2] = 100
//这样两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效
```

#### 切片

切片提供了计算容量的函数 `cap()` 可以测量切片最长可以达到多少：切片的长度 + 数组除切片之外的长度。若`s` 是一个切片，`cap(s)` 就是从 `s[0]` 到数组末尾的数组长度：`0 <= len(s) <= cap(s)`

多个切片如果表示同一个数组的片段，它们可以共享数据；相反，不同的数组总是代表不同的存储

切片的初始化格式是：`var slice1 []type = arr1[start:end]`，这表示 `slice1` 是由数组 `arr1` 从 `start` 索引到 `end-1` 索引之间的元素构成的子集

一个由数字 1、2、3 组成的切片可以这么生成：`s := [3]int{1,2,3}[:]`（注：应先用 `s := [3]int{1, 2, 3}` 生成数组, 再使用 `s[:]` 转成切片）甚至更简单的 `s := []int{1,2,3}`