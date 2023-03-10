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

可以将 `s2` 向后移动一位 `s2 = s2[1:]`，但是`s2 = s2[-1:]` 会导致编译错误，切片不能被重新分片获取数组的前一个元素

##### 创建切片

当相关数组未定义时，可以用 `make()` 创建切片，同时创建好相关数组：`var slice1 []type = make([]type, len)`；可以简写为`slice1 := make([]type,len)`，`len`是数组的长度且是`slice`的初始长度

故定义`s2 := make([]int,10)`，则`cap(s2)==len(s2)==10`

若想创建一个 `slice1`不占用整个数组，只占用以 `len` 为个数个项，那么只要：`slice1 := make([]type, len, cap)`

`make()` 的使用方式是：`func make([]T, len, cap)`，其中 `cap` 是可选参数

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

##### 切片的扩容

###### 扩容函数

```go
func growslice(et *_type,old slice,cap int) slice {
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    }else{
        if old.cap < 1024 {
            newcap = doublecap
        }else{
            for 0 < newcap && newcap < cap {
                newcap += newcap/4
            }
            if newcap <= 0 {
                newcap = cap
            }
        }
    }
}
```

###### 扩容原理

- 当前所需容量（cap）大于原容量两倍（doublecap），则最终申请容量（newcap）为当前所需容量（cap）
- 若不大于原容量两倍时
  1. 原切片长度小于1024则申请原容量的两倍
  2. 否则，最终申请容量（newcap，初始值等于 old.cap）每次增加newcap/4，直到大于所需容量（cap）为止，然后判断最终容量是否溢出，若溢出，最终申请容量等于所需容量

###### go切片扩容为什么是2倍

1. 确定切片的大致容量

2. 根据元素所占字节大小，最终确定容量

   当元素所占字节大小为1、8或2的倍数时，会执行内存对齐操作

##### 多维切片

 Go 的多维切片可以任意切分，而且，内层的切片必须单独分配

##### bytes包

`bytes` 包和字符串包十分类似，而且还包含一个十分有用的类型 `Buffer`

长度可变的 `bytes` 的 buffer，提供 `Read()` 和 `Write()` 方法，读写长度未知的 `bytes` 最好使用 `buffer`

###### buffer定义

- `var buffer bytes.Buffer`

- `var r *bytes.Buffer = new(bytes.Buffer)`
- `func NewBuffer(buf []byte) *Buffer`；`NewBuffer` 最好用在从 `buf` 读取的时候

###### 通过buffer串联字符串

```go
var buffer bytes.Buffer
for {
    if s,ok := getNextString();ok{
        buffer.WriteString(s)
    }else{
        break
    }
}
fmt.Print(buffer.String(), "\n")
```

该方式比使用`+=`更节省内存和CPU，尤其是串联的字符串数目较多时

#### new()和make()的区别

两者都在堆上分配内存，但它们的行为不同，适用于不同类型

- `new(T)` 为每个新类型 `T` 分配一片内存，初始化为 `0` 并且返回类型为 `*T` 的内存地址：这种方法 **返回一个指向类型为 `T`，值为 `0` 的地址的指针**，它适用于值类型如数组和结构体，相当于 `&T{}`
- `make(T)` **返回一个类型为 T 的初始值**，它只适用于 3 种内建的引用类型：切片、`map` 和 `channel`

即`new()`分配内存，`make()`初始化，如下图所示：

<img src="https://s1.ax1x.com/2023/03/09/ppm4rY6.png" style="zoom:50%"/>

#### for-range结构

可以应用于数组和切片

```go
for ix, value := range slice1 {
	...
}
```

第一个返回值 `ix` 是数组或者切片的索引，第二个是在该索引位置的值

`value` 只是 `slice1` 某个索引位置的值的一个拷贝，不能用来修改 `slice1` 该索引位置的值

#### 切片重组

在切片达到容量上限后扩容改变切片长度的过程

#### 切片的复制与追加

如果想增加切片的容量，必须创建一个更大的切片并把原分片的内容都拷贝过来

可以使用`copy()`和`append()`函数，追加的元素必须和原切片元素是同类型

##### append

如果 `s` 的容量不足以存储新增元素，`append()` 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组，`append()` 方法总是返回成功，除非系统内存耗尽

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) {
        newSlice := make([]byte,(n+1)*2)
        copy(newSlice,slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n],data)
    return slice
}
```

###### append常见操作

1. 将切片 `b` 的元素追加到切片 `a` 之后：`a = append(a, b...)`

2. 复制切片 `a` 的元素到新的切片 `b` 上：

   ```go
   b = make([]T, len(a))
   copy(b, a)
   ```

3. 删除位于索引 `i` 的元素：`a = append(a[:i], a[i+1:]...)`

4. 切除切片 `a` 中从索引 `i` 至 `j` 位置的元素：`a = append(a[:i], a[j:]...)`

5. 为切片 `a` 扩展 `j` 个元素长度：`a = append(a, make([]T, j)...)`

6. 在索引 `i` 的位置插入元素 `x`：`a = append(a[:i], append([]T{x}, a[i:]...)...)`

7. 在索引 `i` 的位置插入长度为 `j` 的新切片：`a = append(a[:i], append(make([]T, j), a[i:]...)...)`

8. 在索引 `i` 的位置插入切片 `b` 的所有元素：`a = append(a[:i], append(b, a[i:]...)...)`

9. 取出位于切片 `a` 最末尾的元素 `x`：`x, a = a[len(a)-1], a[:len(a)-1]`

10. 将元素 `x` 追加到切片 `a`：`a = append(a, x)`

##### copy

`func copy(dst, src []T) int` 方法将类型为 `T` 的切片从源地址 `src` 拷贝到目标地址 `dst`，覆盖 `dst` 的相关元素，并且返回拷贝的元素个数。源地址和目标地址可能会有重叠。拷贝个数是 `src` 和 `dst` 的长度最小值。若 `src` 是字符串则元素类型就是 `byte`。若还想继续使用 `src`，在拷贝结束后执行 `src = dst`

#### 字符串生成字节切片

可以通过`c := []byte(s)`获取一个字节的切片`c`，还可以通过 `copy()` 函数来达到相同的目的：`copy(dst []byte, src string)`

将一个字符串追加到某一个字节切片的尾部：

```go
var b []byte
var s string
b = append(b, s...)
```

#### 字符串和切片的内存结构

在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数。因为指针对用户来说是完全不可见，因此我们可以依旧把字符串看做是一个值类型，也就是一个字符数组

字符串 `string s = "hello"` 和子字符串 `t = s[2:3]` 在内存中的结构可以用下图表示：

<img src="https://s1.ax1x.com/2023/03/09/ppmHohn.png" style="zoom:50%"/>

#### 修改字符串的某个字符

Go 中字符串不可变，即 `str[index]` 这样的表达式是不可以被放在等号左侧的

因此必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式

```go
s := "hello"
c := []byte(s)
c[0] = 'c'
s2 := string(c)	//s2 == "cello"
```

#### 字节数组对比函数

```go
func Compare(a, b[]byte) int {
    for i:=0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    // 数组的长度可能不同
    switch {
    case len(a) < len(b):
        return -1
    case len(a) > len(b):
        return 1
    }
    return 0 // 数组相等
}
```

#### 搜索及排序切片和数组

标准库提供了 `sort` 包实现常见的搜索和排序操作。可以使用 `sort` 包中的函数 `func Ints(a []int)` 实现对 `int` 类型的切片排序。例如 `sort.Ints(arri)`，其中变量 `arri` 就是需要被升序排序的数组或切片。为了检查某个数组是否已经被排序，可通过函数 `IntsAreSorted(a []int) bool` 检查，若返回 `true` 则表示已经被排序

类似的，可以使用函数 `func Float64s(a []float64)` 来排序 `float64` 的元素，或使用函数 `func Strings(a []string)` 排序字符串元素

想要在数组或切片中搜索一个元素，该数组或切片必须先被排序（因为标准库的搜索算法使用的是二分法）。使用函数 `func SearchInts(a []int, n int) int` 进行搜索，并返回对应结果的索引值

当然，还可以搜索 `float64` 和字符串：

```go
func SearchFloat64s(a []float64, x float64) int
func SearchStrings(a []string, x string) int
```

#### 切片和垃圾回收

只有在没有任何切片指向的时候，底层的数组内存才会被释放，这种特性有时会导致程序占用多余的内存

**示例** 函数 `FindDigits()` 将一个文件加载到内存，然后搜索其中所有的数字并返回一个切片。

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

这段代码可以顺利运行，但返回的 `[]byte` 指向的底层是整个文件的数据。只要该返回的切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。换句话说，一点点有用的数据却占用了整个文件的内存。

想要避免这个问题，可以通过拷贝我们需要的部分到一个新的切片中：

```go
func FindDigits(filename string) []byte {
   b, _ := ioutil.ReadFile(filename)
   b = digitRegexp.Find(b)
   c := make([]byte, len(b))
   copy(c, b)
   return c
}
```

事实上，上面这段代码只能找到第一个匹配正则表达式的数字串。要想找到所有的数字，可以尝试下面这段代码：

```go
func FindFileDigits(filename string) []byte {
   fileBytes, _ := ioutil.ReadFile(filename)
   b := digitRegexp.FindAll(fileBytes, len(fileBytes))
   c := make([]byte, 0)
   for _, bytes := range b {
      c = append(c, bytes...)
   }
   return c
}
```

## Map

### 键值对元素

`val1,isPresent = map1[key1]`

`isPresent`返回一个`bool`值：若 `key1` 存在于 `map1`，`val1` 就是 `key1` 对应的 `value` 值，并且 `isPresent` 为 `true`；若`key1` 不存在，`val1` 就是一个空值，且 `isPresent` 返回 `false`

判断某个`key`是否存在的常规方法：

```go
if _, ok := map1[key1]; ok {
	// ...
}
```

在`map1`中删除`key1`：`delete(map1,key1)`；若`key1`不存在，该操作不会产生错误

 `map` 不是按照 key 的顺序排列的，也不是按照 value 的序排列的

> map 的本质是散列表，而 map 的增长扩容会导致重新进行散列，这就可能使 map 的遍历结果在扩容前后变得不可靠，Go 设计者为了让大家不依赖遍历的顺序，每次遍历的起点--即起始 bucket 的位置不一样，即不让遍历都从某个固定的 bucket0 开始，所以即使未扩容时我们遍历出来的 map 也总是无序的

### map类型的切片

获取一个 `map` 类型的切片，必须使用两次 `make()` 函数，第一次分配切片，第二次分配切片中每个 `map` 元素

```go
package main
import "fmt"

func main() {
	// Version A:
    items := make([]map[int]int,5)
    for i := range items {
        items[i] = make(map[int]int, 1)
		items[i][1] = 2
    }
    fmt.Printf("Version A: Value of items: %v\n", items)
	
    // Version B: NOT GOOD!
	items2 := make([]map[int]int, 5)
	for _, item := range items2 {
		item = make(map[int]int, 1) // item is only a copy of the slice element.
		item[1] = 2 // This 'item' will be lost on the next iteration.
	}
	fmt.Printf("Version B: Value of items: %v\n", items2)
}
```

输出结果：

```
Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
Version B: Value of items: [map[] map[] map[] map[] map[]]
```

 A 通过索引使用切片的 `map` 元素。 B 中获得的项只是 `map` 值的一个拷贝，所以真正的 `map` 元素没有得到初始化