---
title: jktt-go笔记
date: 2023-02-03 14:13:19
tags: "go"
categories: "go"
---

## 极客兔兔---Go语言高性能编程

### 性能分析

#### benchmark基准测试

##### 使用示例

```go
// fib.go
package main

func fib(n int) int {
	if n == 0 || n == 1 {
		return n
	}
	return fib(n-2) + fib(n-1)
}
```

```go
// fib_test.go
package main

import "testing"

func BenchmarkFib(b *testing.B) {
	for n := 0; n < b.N; n++ {
		fib(30) // run fib(30) b.N times
	}
}
```

`go test`命令默认不运行benchmark用例，若需要运行则加上`-bench`参数如：

```
$ go test -bench .
```

`-bench`支持传入一个正则表达式，如只运行`Fib`结尾的benchmark用例：

```
$ go test -bench='Fib$'
```

##### benchmark的工作机制

benchmark用例参数`b *testing.B`有个属性`b.N`表示该用例需要运行的次数，若用例能在1s内完成则`b.N`会以 1, 2, 3, 5, 10, 20, 30, 50, 100 这样的序列递增

##### benchmark常用参数

```
-cpu=2,4	修改CPU核数，对结果几乎没有影响，因为Fib调用是串行的
-benchtime=5s		修改测试时间，默认时间为1s
实际执行时间更长，因为用例编译、执行、销毁等是需要时间的
-benchtime=30x		修改执行次数
-count=3	修改执行的轮数
```

##### StopTimer和StartTimer

每次函数调用前后需要一些准备和清理工作，可以使用`StopTimer`暂时停止计时，使用`StartTime`开始计时

#### pprof性能分析

benchmark 可以度量某个函数或方法的性能，若知道性能的瓶颈点在哪里，benchmark 是一个非常好的方式。但是面对一个未知的程序则需要使用pprof

pprof包含两部分：

- 编译到程序中的`runtime/pprof`包
- 性能剖析工具`go tool pprof`

若想度量某个应用程序的CPU性能数据，只需要添加：

```go
pprof.StartCPUProfile(os.Stdout)
defer pprof.StopCPUProfile()
```

该例将数据输出到标准输出`os.Stdout`中，若要定向到文件`cpu.pprof`中：

```
$ go run main.go > cpu.pprof 
```

也可以通过如下方式将结果记录到一个文件中：

```go
f,_ = os.Openfile("cpu.pprof",os.O_CREATE|os.O_RDWR,0644)
defer f.Close()
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()
```

此时只需运行`go run main.go`即可

##### 分析数据

可以用`go tool pprof`分析数据

```
$ go tool pprof -http=:9999 cpu.pprof
```

若提示 Graphviz 没有安装，则通过 `brew install graphviz`(MAC) 或 `apt install graphviz`(Ubuntu) 即可。访问 `localhost:9999`，可以看到：

![](https://geektutu.com/post/hpg-pprof/bubble-sort-pprof.jpg)



### 常用数据结构

#### 字符串拼接性能及原理

go中string是不可变的，拼接的原理即创建一个新的字符串对象，若存在大量字符串拼接则对性能会产生严重影响

##### 拼接方式示例

为避免编译器优化，首先实现一个生成长度为n的随机字符串的函数

```go
const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

func randomString(n int) string {
    b := make([]byte,n)
    for i := range b {
        b[i] = letterBytes[rand.Intn(len(letterBytes))]
    }
    return string(b)
}
```

然后利用这个函数生成字符串 `str`，然后将 `str` 拼接 N 次。在 Go 语言中，常见的字符串拼接方式有如下 5 种：

- 使用`+`

```go
func plusConcat(n int,str string) string {
    s := ""
    for i := 0; i < n; i++ {
		s += str
	}
	return s
}
```

- 使用`fmt.Sprintf`

```go
func sprintfConcat(n int,str string) string {
    s := ""
    for i := 0; i < n; i++ {
        s = fmt.Sprintf("%s%s",s,str)
    }
    return s
}
```

- 使用`strings.Builder`

```go
func builderConcat(n int, str string) string {
    var builder strings.Builder
    for i := 0; i < n; i++ {
        builder.WriteString(str)
    }
    return builder.String()
}
```

- 使用`bytes.Builder`

```go
func bufferConcat(n int, s string) string {
    buf := new(bytes.Buffer)
    for i := 0; i < n; i++ {
        buf.WriteString(s)
    }
    return buf.String()
}
```

- 使用`[]byte`

```go
func byteConcat(n int, str string) string {
    buf := make([]byte,0)
    for i := 0; i < n; i++ {
        buf = append(buf,str...)
    }
    return string(buf)
}

//当长度可预知时可以预先分配切片的容量(cap)
func preByteConcat(n int, str string) string {
	buf := make([]byte, 0, n*len(str))
	for i := 0; i < n; i++ {
		buf = append(buf, str...)
	}
	return string(buf)
}
```

##### 使用benchmark进行性能比拼

每个benchmark用例中，生成一个长度为10的字符串，并拼接1w次

```go
func benchmark(b *testing.B,f func(int,string) string){
    var str = randomString(10)
    for i := 0; i < b.N; i++ {
        f(10000,str)
    }
}

func BenchmarkPlusConcat(b *testing.B) {benchmark(b,plusConcat)}
func BenchmarkSprintfConcat(b *testing.B) { benchmark(b, sprintfConcat) }
func BenchmarkBuilderConcat(b *testing.B) { benchmark(b, builderConcat) }
func BenchmarkBufferConcat(b *testing.B)  { benchmark(b, bufferConcat) }
func BenchmarkByteConcat(b *testing.B)    { benchmark(b, byteConcat) }
func BenchmarkPreByteConcat(b *testing.B) { benchmark(b, preByteConcat) }
```

运行该测试用例：

```
$ go test -bench="Concat$" -benchmem .
goos: darwin
goarch: amd64
pkg: example
BenchmarkPlusConcat-8         19      56 ms/op   530 MB/op   10026 allocs/op
BenchmarkSprintfConcat-8      10     112 ms/op   835 MB/op   37435 allocs/op
BenchmarkBuilderConcat-8    8901    0.13 ms/op   0.5 MB/op      23 allocs/op
BenchmarkBufferConcat-8     8130    0.14 ms/op   0.4 MB/op      13 allocs/op
BenchmarkByteConcat-8       8984    0.12 ms/op   0.6 MB/op      24 allocs/op
BenchmarkPreByteConcat-8   17379    0.07 ms/op   0.2 MB/op       2 allocs/op
PASS
ok      example 8.627s
```

因此`+`和`fmt.Sprintf`的效率最低，性能相差1000倍，且消耗了超过1000倍的内存，`fmt.Sprintf`通常用于格式化字符串，一般不用来拼接字符串。

`strings.Builder`、`bytes.Buffer` 和 `[]byte` 的性能差距不大，而且消耗的内存也十分接近，性能最好且消耗内存最小的是 `preByteConcat`，这种方式预分配了内存，在字符串拼接的过程中，不需要进行字符串的拷贝，也不需要分配新的内存，因此性能最好，且内存消耗最小。

综合易用性和性能，推荐用 `strings.Builder` 拼接，`string.Builder` 也提供了预分配内存的方式 `Grow`：

```go
func builderConcat(n int, str string) string {
	var builder strings.Builder
	builder.Grow(n * len(str))
	for i := 0; i < n; i++ {
		builder.WriteString(str)
	}
	return builder.String()
}
```

使用了 Grow 优化后的版本的 benchmark 结果如下：

```
BenchmarkBuilderConcat-8   16855    0.07 ns/op   0.1 MB/op       1 allocs/op
BenchmarkPreByteConcat-8   17379    0.07 ms/op   0.2 MB/op       2 allocs/op
```

与预分配内存的 `[]byte` 相比，省去了 `[]byte` 和字符串之间的转换，内存分配减少了 1 次，内存消耗减半。

##### 比较strings.Builder和`+`

`+`拼接2个字符串时，生成一个新的字符串，需要开辟一段新的空间，新空间大小是字符串大小之和；`strings.Builder`等的内存是以16、32等倍数形式申请的，在2048之后申请策略会有调整。

##### 比较`strings.Builder`和`bytes.Buffer`

 `strings.Builder` 性能比 `bytes.Buffer` 略快约 10% ，因为`bytes.Buffer` 转化为字符串时申请了一块空间，存放生成的字符串，而 `strings.Builder` 直接将底层的 `[]byte` 转换成了字符串类型返回了回来。

#### 切片

Go的切片是在数组之上的抽象数据类型，数组类型定义了长度和元素类型，长度不同的两个数组不能互相赋值，因为属于不同的类型；切片本质是一个数组片段的描述，包括了数组的指针，这个片段的长度和容量，切片不复制指向的元素，创建一个新的切片复用原来切片的底层数组

![](https://geektutu.com/post/hpg-slice/slice.jpg)

##### 切片性能陷阱

当原切片由大量元素构成，但在其上的切片只使用了一小段，底层数组仍占据了大量空间，此时推荐用`copy`代替`re-slice`

```go
func lastNumsBySlice(origin []int) []int {
    return origin[len(origin)-2:]
}

func lastNumsByCopy(origin []int) []int {
    result := make([]int,2)
    copy(result,origin[len(origin)-2:])
    return result
}
```

#### for和range的比较

`range`对每个迭代值都创建了一个拷贝，因此若每次迭代值内存占用很小的情况下，`for`和`range`性能几乎没有差异，但若每个迭代值内存占用很大则`for`的性能比`range`更好

```go
persons := []struct{ no int }{{no: 1},{no: 2},{no: 3}}
for _,s := range persons {
    s.no += 10
}
for i := 0; i < len(person); i++ {
    person[i].no += 100
} 
fmt.Println(persons)	//	[{101} {102} {103}]
```

上述使用`range`迭代时修改无效，因其返回的是拷贝，使用`for`迭代时修改有效

#### Go反射

标准库`reflect`为Go提供了运行时动态获取对象的类型和值以及动态创建对象的能力，用于序列化和反序列化的`json`、`gorm/xorm`都用到了反射

##### 反射如何简化代码

假设有一个配置类Config，每个字段是一个配置项

```go
type Config struct {
    Name	string	`json:"server-name"`  //CONFIG_SERVER_NAME
    Ip		string	`json:"server-ip"`  // CONFIG_SERVER_IP
    URL		string	`json:"server-url"`  //CONFIG_SERVER_URL
    Timeout	string	`json:"timeout"`	//CONFIG_TIMEOUT
}
```

实现一个功能：配置默认从json文件读取，若环境变量配置了某个配置项，则以环境变量中的配置为准。配置项与环境变量对应的规则：将json字段的字母转为大写，将`-`转为下划线，并添加`CONFIG_`前缀。

实现该功能可以使用`switch case`或`if else`可以实现，但是当`Config`结构发生变化时，该部分也需要发生改变，且容易出错、不好测试

此时，可以使用reflect：

```go
func readConfig() *Config {
    config := Config{}
    typ	:= reflect.Typeof(config)
    value := reflect.Indirect(reflect.ValueOf(&config))
    for i := 0; i < typ.NumField(); i++ {
        f := typ.Field(i)
        if v,ok := f.Tag.Lookup("json"); ok {
            key := fmt.Sprintf("CONFIG_%s",strings.ReplaceAll(strings.ToUpper(v),"-","_"))
          	if env, exist := os.LookupEnv(key); exist {
				value.FieldByName(f.Name).Set(reflect.ValueOf(env))
			}  
        }
    }
    return &config
}

func main() {
    os.Setenv("CONFIG_SERVER_NAME", "global_server")
	os.Setenv("CONFIG_SERVER_IP", "10.0.0.1")
	os.Setenv("CONFIG_SERVER_URL", "geektutu.com")
	c := readConfig()
	fmt.Printf("%+v", c)
}
```

在运行时利用反射获取到`Config`的每个字段的`Tag`属性，拼接出对应的环境变量的名称，查看该环境变量是否存在，若存在，则将环境变量的值赋值给该字段

环境变量中设置的配置项已经生效。之后无论结构体 `Config` 内部的字段发生任何改变，这部分代码无需任何修改即可完美的适配，出错概率也极大地降低。

##### 反射的性能

反社会增加额外的代码指令，测试对性能会产生多大的影响：

```go
func BenchmarkNew(b *testing.B) {
	var config *Config
	for i := 0; i < b.N; i++ {
		config = new(Config)
	}
	_ = config
}

func BenchmarkReflectNew(b *testing.B) {
	var config *Config
	typ := reflect.TypeOf(Config{})
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		config, _ = reflect.New(typ).Interface().(*Config)
	}
	_ = config
}
```

测试结果如下：

```
$ go test -bench .          
goos: darwin
goarch: amd64
pkg: example/hpg-reflect
BenchmarkNew-8                  26478909                40.9 ns/op
BenchmarkReflectNew-8           18983700                62.1 ns/op
PASS
ok      example/hpg-reflect     2.382s
```

通过反射创建对象的耗时约为 `new` 的 1.5 倍，相差不是特别大。

#### 空结构体struct{}的使用

##### 空结构体是否占用空间

使用`unsafe.Sizeof`计算出一个数据类型实例需要占用的字节数

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	fmt.Println(unsafe.Sizeof(struct{}{}))
}
```

```
$ go run main.go
0
```

所以空结构体struct{}实例不占据任何内存空间

##### 空结构体的作用

被广泛作为占位符使用。一是节省资源，二是空结构体本身具有很强的语义，不需要任何值，仅作为占位符

###### 实现集合(Set)

Go标准库中没有提供Set的实现，通常用map代替，对于集合来说只需要map的键，而不需要值。即使将值设置为bool也会多占据1个字节，因此可以将值类型定义为空结构体使用

```go
type Set map[string]struct{}

func (s Set) Has(key string) bool {
    _,ok := s[key]
    return ok
}

func (s Set) Add(key string) {
    s[key] = struct{}{}
}

func (s Set) Delete(key string) {
    delete(s,key)
}

func main() {
	s := make(Set)
	s.Add("Tom")
	s.Add("Sam")
	fmt.Println(s.Has("Tom"))
	fmt.Println(s.Has("Jack"))
}
```

###### 不发送数据的信道(channel)

```go
func worker(ch chan struct{}) {
    <-ch
    fmt.println("do something")
    close(ch)
}

func main() {
    ch := make(chan struct{})
    go worker(ch)
    ch <- struct{}{}
}
```

###### 仅包含方法的结构体

```go
type Door struct{}

func (d Door) Open() {
    fmt.Println("Open the door")
}

func (d Door) Close() {
    fmt.Println("Close the door")
}
```

在部分场景中，结构体只包含方法，不包含任何字段，在此情况下可以用任何数据结构替代，例如：

```
type Door int
type Door bool
```

无论`int`还是`bool`都会浪费额外的内存，因此声明为空结构体最合适

#### Go struct内存对齐

##### 为什么需要内存对齐

CPU访问内存时不是逐个字节访问，而是以字长为单位，如32位CPU字长为4字节，则CPU访问内存单位为4字节；此设计目的是为了减少CPU访问内存的次数，所以如果不进行内存对齐很可能增加CPU访问内存的次数

![](https://geektutu.com/post/hpg-struct-alignment/memory_alignment.png)

如上内存对齐时a和b占据4字节空间，读取b时只需1次内存访问；若不进行内存对齐则需要2次内存访问，第一次访问b的第1个字节，第二次访问b的后两个字节

简言之：合理的内存对齐可以提高内存读写的性能，便于变量操作的原子性。

##### unsafe.Alignof

`unsafe`标准库提供了`Alignof`方法，可以返回一个类型的对齐值，也可以叫做对齐系数或者对齐倍数，例如：

```go
type Args struct {
    num1 int
    num2 int
}

type Flag struct {
    num1 int16
    num2 int32
}

unsafe.Alignof(Args{}) // 8
unsafe.Alignof(Flag{}) // 4

```

- `Args{}`的对齐倍数是8，两个字段占据16个字节，是8的倍数，无需占据额外的空间对齐
- `Flag{}`的对齐倍数是4，因此`Flag{}`占据的空间必须是4的倍数，因此内存对齐之后是8字节

##### 对齐保证(align guarantee)

对于任意类型的变量 x ，`unsafe.Alignof(x)` 至少为 1

对于 struct 结构体类型的变量 x，计算 x 每一个字段 f ， 等于其中的最大值

对于 数组类型的变量 x，等于构成数组的元素类型的对齐倍数

没有任何字段的空 struct{} 和没有任何元素的 array 占据的内存空间大小为 0，不同的大小为 0 的变量可能指向同一块地址

##### struct 内存对齐的技巧

###### 合理布局减少内存占用

```go
type demo1 struct {
	a int8
	b int16
	c int32
}

type demo2 struct {
	a int8
	c int32
	b int16
}

func main() {
	fmt.Println(unsafe.Sizeof(demo1{})) // 8
	fmt.Println(unsafe.Sizeof(demo2{})) // 12
}
```

如上所示，结构体中字段的顺序会对struct的大小产生影响，字段按照自身对齐倍数确定内存中偏移量，字段排列顺序不同，上一字段因偏移浪费的大小也不同

对于demo1：

- a是第一个字段，默认已对齐，从第0个位置开始占据1个字节
- b是第二个字段，对齐倍数为2，必须空出1个位置，从第2个位置占据2字节
- c是第三个字段，对齐倍数为4，此时内存是对齐的，从第4个位置占据4字节

因此demo1的内存占用为8字节

对于demo2：

- a 是第一个字段，默认是已经对齐的，从第 0 个位置开始占据 1 字节
- c 是第二个字段，对齐倍数为 4，因此，必须空出 3 个字节，偏移量才是 4 的倍数，从第 4 个位置开始占据 4 字节
- b 是第三个字段，对齐倍数为 2，从第 8 个位置开始占据 2 字节

demo2 的对齐倍数由 c 的对齐倍数决定，因此，demo2 的内存占用为 12 字节

![](https://geektutu.com/post/hpg-struct-alignment/memory_alignment_order.png)

在对内存特别敏感的结构体设计上，可以通过调整字段顺序，减少内存的占用

###### 空 struct{} 的对齐

空 `struct{}` 大小为 0，作为其他 struct 的字段时，一般不需要内存对齐。但是有一种情况除外：即当 `struct{}` 作为结构体最后一个字段时，需要内存对齐。因为如果有指针指向该字段, 返回的地址将在结构体之外，如果此指针一直存活不释放对应的内存，就会有内存泄露的问题（该内存不因结构体释放而释放）

当 `struct{}` 作为其他 struct 最后一个字段时，需要填充额外的内存保证安全

```go
type demo3 struct {
	c int32
	a struct{}
}

type demo4 struct {
	a struct{}
	c int32
}

func main() {
	fmt.Println(unsafe.Sizeof(demo3{})) // 8
	fmt.Println(unsafe.Sizeof(demo4{})) // 4
}

```

可以看到demo3额外填充了字段C的大小，即额外填充了4字节空间

### 并发编程

#### 读写锁和互斥锁

##### 读写锁和互斥锁的区别

###### 互斥锁

互斥即不可同时运行，两个代码片段互相排斥，只有其中一个代码片段执行完后另一个才能执行

Go 标准库中提供了 sync.Mutex 互斥锁类型及其两个方法：

- Lock加锁
- Unlock释放锁

可以在代码前调用Lock方法，在代码后调用Unlock方法保证代码的互斥执行，也可以用defer语句保证互斥锁一定会被解锁。在一个 Go 协程调用 Lock 方法获得锁后，其他请求锁的协程都会阻塞在 Lock 方法，直到锁被释放。

###### 读写锁

多用于银行存取钱操作，大部分情况下读取余额的操作会更频繁，如果能保证读取余额的操作并发执行，程序的效率会得到很大提高

保证读操作的安全，那只要保证并发读时没有写操作在进行就行。在这种场景下我们需要一种特殊类型的锁，其允许多个只读操作并行执行，但写操作会互斥。

这种锁称之为 `多读单写锁` (multiple readers, single writer lock)，简称读写锁，读写锁分为读锁和写锁，读锁是允许同时执行的，但写锁是互斥的。一般来说，有如下几种情况：

- 读锁之间不互斥，没有写锁的情况下，读锁是无阻塞的，多个协程可以同时获得读锁。
- 写锁之间是互斥的，存在写锁，其他写锁阻塞。
- 写锁与读锁是互斥的，若存在读锁，写锁阻塞，若存在写锁，读锁阻塞。

Go 标准库中提供了 sync.RWMutex 互斥锁类型及其四个方法：

- Lock 加写锁
- Unlock 释放写锁
- RLock 加读锁
- RUnlock 释放读锁

读写锁的存在是为了解决读多写少时的性能问题，读场景较多时，读写锁可有效地减少锁阻塞的时间

##### 读写锁和互斥锁性能比较

接下来，我们测试三种情景下，互斥锁和读写锁的性能差异：

- 读多写少(读占 90%)
- 读少写多(读占 10%)
- 读写一致(各占 50%)

###### 测试用例

接下来我们实现 2 个结构体 `Lock` 和 `RWLock`，并且都继承 `RW` 接口。`RW` 接口中定义了 2 个操作，读(Read)和写(Write)，为了降低其他指令对测试的影响，假定每个读写操作耗时 1 微秒(百万分之一秒)。

`Lock`

```go
type RW interface {
	Write()
	Read()
}

const cost = time.Microsecond

type Lock struct {
	count int
	mu    sync.Mutex
}

func (l *Lock) Write() {
	l.mu.Lock()
	l.count++
	time.Sleep(cost)
	l.mu.Unlock()
}

func (l *Lock) Read() {
	l.mu.Lock()
	time.Sleep(cost)
	_ = l.count
	l.mu.Unlock()
}
```

`RWLock`

```go
type RWLock struct {
	count int
	mu    sync.RWMutex
}

func (l *RWLock) Write() {
	l.mu.Lock()
	l.count++
	time.Sleep(cost)
	l.mu.Unlock()
}

func (l *RWLock) Read() {
	l.mu.RLock()
	_ = l.count
	time.Sleep(cost)
	l.mu.RUnlock()
}
```

###### 基准测试

```go
func benchmark(b *testing.B, rw RW, read, write int) {
	for i := 0; i < b.N; i++ {
		var wg sync.WaitGroup
		for k := 0; k < read*100; k++ {
			wg.Add(1)
			go func() {
				rw.Read()
				wg.Done()
			}()
		}
		for k := 0; k < write*100; k++ {
			wg.Add(1)
			go func() {
				rw.Write()
				wg.Done()
			}()
		}
		wg.Wait()
	}
}

func BenchmarkReadMore(b *testing.B)    { benchmark(b, &Lock{}, 9, 1) }
func BenchmarkReadMoreRW(b *testing.B)  { benchmark(b, &RWLock{}, 9, 1) }
func BenchmarkWriteMore(b *testing.B)   { benchmark(b, &Lock{}, 1, 9) }
func BenchmarkWriteMoreRW(b *testing.B) { benchmark(b, &RWLock{}, 1, 9) }
func BenchmarkEqual(b *testing.B)       { benchmark(b, &Lock{}, 5, 5) }
func BenchmarkEqualRW(b *testing.B)     { benchmark(b, &RWLock{}, 5, 5) }
```

运行结果如下：

```
$ go test -bench .
goos: darwin
goarch: amd64
pkg: example/hpg-mutex
BenchmarkReadMore-8                   86          13202572 ns/op
BenchmarkReadMoreRW-8                661           1748724 ns/op
BenchmarkWriteMore-8                  87          13109525 ns/op
BenchmarkWriteMoreRW-8                94          12090900 ns/op
BenchmarkEqual-8                      85          13150321 ns/op
BenchmarkEqualRW-8                   176           6770092 ns/op
PASS
ok      example/hpg-mutex       7.816s
```

##### 互斥锁如何实现公平

```
互斥锁有两种状态：正常状态和饥饿状态。

在正常状态下，所有等待锁的 goroutine 按照FIFO顺序等待。唤醒的 goroutine 不会直接拥有锁，而是会和新请求锁的 goroutine 竞争锁的拥有。新请求锁的 goroutine 具有优势：它正在 CPU 上执行，而且可能有好几个，所以刚刚唤醒的 goroutine 有很大可能在锁竞争中失败。在这种情况下，这个被唤醒的 goroutine 会加入到等待队列的前面。 如果一个等待的 goroutine 超过 1ms 没有获取锁，那么它将会把锁转变为饥饿模式。

在饥饿模式下，锁的所有权将从 unlock 的 goroutine 直接交给交给等待队列中的第一个。新来的 goroutine 将不会尝试去获得锁，即使锁看起来是 unlock 状态, 也不会去尝试自旋操作，而是放在等待队列的尾部。

如果一个等待的 goroutine 获取了锁，并且满足一以下其中的任何一个条件：(1)它是队列中的最后一个；(2)它等待的时候小于1ms。它会将锁的状态转换为正常状态。

正常状态有很好的性能表现，饥饿模式也是非常重要的，因它能阻止尾部延迟的现象。
```

#### 如何退出协程

##### 超时返回时的陷阱

###### time.After实现超时控制

```go
func doBadthing(done chan bool) {
	time.Sleep(time.Second)
	done <- true
}

func timeout(f func(chan bool)) error {
	done := make(chan bool)
	go f(done)
	select {
	case <-done:
		fmt.Println("done")
		return nil
	case <-time.After(time.Millisecond):
		return fmt.Errorf("timeout")
	}
}

// timeout(doBadthing)
```

上述代码执行思路：

- 利用`time.After`启动了一个异步的定时器，返回一个channel，当超过指定时间后，该channel将会接收到信号
- 启动了子协程执行函数f，函数执行结束后，向channel`done`发送结束信号
- select等待`done`或`time.After`的信息，若超时返回错误，若未超时返回nil

###### 测试超时时协程是否退出

在该例中超时时间为1ms，但`doBadthing`需要1s才能结束运行，因此`timeout(doBadthing)`一定会触发超时

```go
func test(t *testing.T, f func(chan bool)) {
	t.Helper()
	for i := 0; i < 1000; i++ {
		timeout(f)
	}
	time.Sleep(time.Second * 2)
	t.Log(runtime.NumGoroutine())
}

func TestBadTimeout(t *testing.T)  { test(t, doBadthing) }
```

- `timeout(doBadthing)` 调用了 1000 次，理论上会启动 1000 个子协程。
- 利用 `runtime.NumGoroutine()` 打印当前程序的协程个数。
- 因为 `doBadthing` 执行时间为 1s，因此打印协程个数前，等待 2s，确保函数执行完毕。

```
$ go test -run ^TestBadTimeout$ . -v
=== RUN   TestBadTimeout
--- PASS: TestBadTimeout (3.43s)
    timeout_test.go:49: 1002
```

最终程序中存在着 1002 个子协程，说明即使是函数执行完成，协程也没有正常退出,问题在于`done`是一个无缓冲区的channel，若未超时，`doBadthing`向done发送信号，`select`接收done的信号，故`doBadthing`能够正常退出，子协程也能正常退出；

但是，当超时发生时，select接收到`time.After`的超市信号就返回了，`done`没有了接收方，而 `doBadthing` 在执行 1s 后向 `done` 发送信号，由于没有接收者且无缓存区，发送者(sender)会一直阻塞，导致协程不能退出。

##### 如何避免

###### 创建有缓冲区的channel

即创建channel`done`时，缓冲区设置为1，即使没有接收方，发送方也不会阻塞

```go
func timeoutWithBuffer(f func(chan bool)) error {
    done := make(chan bool, 1)
    go f(done)
    select {
        case <- done: 
        fmt.Println("done")
        return nil
        case <- time.After(time.Millisecond):
        return fmt.Errorf("timeout")
    }
}

func TestBufferTimeout(t *testing.T) {
	for i := 0; i < 1000; i++ {
		timeoutWithBuffer(doBadthing)
	}
	time.Sleep(time.Second * 2)
	t.Log(runtime.NumGoroutine())
}
```

```
$ go test -run ^TestBufferTimeout$ . -v
=== RUN   TestBufferTimeout
--- PASS: TestBufferTimeout (3.36s)
    timeout_test.go:65: 2
```

协程数量下降为 2，创建的 1000 个子协程成功退出

###### 使用select尝试发送

```go
func doGoodthing(done chan bool) {
	time.Sleep(time.Second)
	select {
	case done <- true:
	default:
		return
	}
}

func TestGoodTimeout(t *testing.T) { test(t, doGoodthing) }
```

测试结果如下：

```
$ go test -run ^TestGoodTimeout$ . -v
=== RUN   TestGoodTimeout
--- PASS: TestGoodTimeout (3.40s)
    timeout_test.go:58: 2
```

使用 select 尝试向信道 done 发送信号，如果发送失败，则说明缺少接收者(receiver)，即超时了，那么直接退出即可

###### 分开检测

将任务拆分为多段，只检测第一段是否超时，若没有超时，后续任务继续执行，超时则终止

```go
func do2phases(phase1, done chan bool) {
	time.Sleep(time.Second) // 第 1 段
	select {
	case phase1 <- true:
	default:
		return
	}
	time.Sleep(time.Second) // 第 2 段
	done <- true
}

func timeoutFirstPhase() error {
	phase1 := make(chan bool)
	done := make(chan bool)
	go do2phases(phase1, done)
	select {
	case <-phase1:
		<-done
		fmt.Println("done")
		return nil
	case <-time.After(time.Millisecond):
		return fmt.Errorf("timeout")
	}
}

func Test2phasesTimeout(t *testing.T) {
	for i := 0; i < 1000; i++ {
		timeoutFirstPhase()
	}
	time.Sleep(time.Second * 3)
	t.Log(runtime.NumGoroutine())
}
```

该场景的实际应用有：将服务端接收请求后的任务拆分为2段，一段是执行任务，一段是发送结果：

- 任务正常执行，向客户端返回执行结果。
- 任务超时执行，向客户端返回超时。

这种情况下，就只能够使用 select，而不能能够设置缓冲区的方式了。因为如果给信道 phase1 设置了缓冲区，`phase1 <- true` 总能执行成功，那么无论是否超时，都会执行到第二阶段，而没有即时返回，这是我们不愿意看到的。对应到上面的业务，就可能发生一种异常情况，向客户端发送了 2 次响应：

- 任务超时执行，向客户端返回超时，一段时间后，向客户端返回执行结果。

缓冲区不能够区分是否超时了，但是 select 可以（没有接收方，信道发送信号失败，则说明超时了）

##### 能否强制kill goroutine

不能，goroutine 只能自己退出，而不能被其他 goroutine 强制关闭或杀死

```
goroutine 被设计为不可以从外部无条件地结束掉，只能通过 channel 来与它通信。也就是说，每一个 goroutine 都需要承担自己退出的责任
```

##### channel忘记关闭的陷阱

在实际场景中，也有可能因为协程使用不当，导致无法退出的情况，随着时间的积累，造成内存耗尽、程序崩溃

```go
func do(taskCh chan int) {
    for {
        select {
            case t := <-taskCh:
            	time.Sleep(time.Millisecond)
				fmt.Printf("task %d is done\n", t)
        }
    }
}

func sendTasks() {
    taskCh := make(chan int,10)
    go do(taskCh)
    for i := 0; i < 1000; i++ {
        taskCh <- i
    }
}

func TestDo(t *testing.T) {
    t.Log(runtime.NumGoroutine())
    sendTasks()
	time.Sleep(time.Second)
	t.Log(runtime.NumGoroutine())
}
```

- `do` 实现非常简单，for + select 的模式，等待 taskCh 传递任务，并执行
- `sendTasks` 模拟向信道中发送任务

```
$ go test . -v
--- PASS: TestDo (2.34s)
    exit_test.go:29: 2
    exit_test.go:32: 3
```

子协程多了一个，也就是说，有一个协程一直没有得到释放，`sendTasks`中启动了一个子协程 `go do(taskCh)`，因为这个协程一直处于阻塞状态，等待接收任务，因此直到程序结束，协程也没有释放

###### 如何解决

```go
func doCheckClose(taskCh chan int) {
	for {
		select {
		case t, beforeClosed := <-taskCh:
			if !beforeClosed {
				fmt.Println("taskCh has been closed")
				return
			}
			time.Sleep(time.Millisecond)
			fmt.Printf("task %d is done\n", t)
		}
	}
}

func sendTasksCheckClose() {
	taskCh := make(chan int, 10)
	go doCheckClose(taskCh)
	for i := 0; i < 1000; i++ {
		taskCh <- i
	}
	close(taskCh)
}

func TestDoCheckClose(t *testing.T) {
	t.Log(runtime.NumGoroutine())
	sendTasksCheckClose()
	time.Sleep(time.Second)
	runtime.GC()
	t.Log(runtime.NumGoroutine())
}
```

两个地方修改下即可：

- `t, beforeClosed := <-taskCh` 判断 channel 是否已经关闭，beforeClosed 为 false 表示信道已被关闭。若关闭，则不再阻塞等待，直接返回，对应的协程随之退出。
- `sendTasks` 函数中，任务发送结束之后，使用 `close(taskCh)` 将 channel taskCh 关闭。

```
$ go test -run=TestDoCheckClose -v
task 999 is done
taskCh has been closed
--- PASS: TestDoCheckClose (2.34s)
    exit_test.go:59: 2
    exit_test.go:63: 2
```

```
关于通道和协程的垃圾回收

注意，一个通道被其发送数据协程队列和接收数据协程队列中的所有协程引用着。因此，如果一个通道的这两个队列只要有一个不为空，则此通道肯定不会被垃圾回收。另一方面，若一个协程处于一个通道的某个协程队列之中，则此协程也肯定不会被垃圾回收，即使此通道仅被此协程所引用。事实上，一个协程只有在退出后才能被垃圾回收。
```

```
通道关闭原则

只应当让一个通道唯一的发送者关闭此通道
```

###### 关闭channel的方法

- 粗鲁的方式（非常不推荐）

  ```go
  func SafeClose(ch chan T) (justClosed bool) {
      defer func() {
          if recover() != nil {
              // 一个函数的返回结果可以在defer调用中修改
              justClosed = false
          }
      }()
      
      //假设ch != nil
      close(ch)	//若ch已关闭，将panic
      return true	//<=> justClosed = true; return
  }
  ```

- 礼貌的方式

  使用sync.Once或互斥锁确保channel只被关闭一次

  ```go
  type MyChannel struct {
      C		chan T
      once	sync.Once
  }
  
  func NewMyChannel() *MyChannel {
      return &MyChannel{C:make(chan T)}
  }
  
  func (mc *MyChannel) SafeClose() {
      mc.once.Do(func(){
          close(mc.C)
      })
  }
  ```

#### 控制协程的并发数量

不同的应用程序，消耗的资源是不一样的。比较推荐的方式的是：应用程序来主动限制并发的协程数量

##### 利用channel的缓存区

```go
func main() {
    var wg sync.WaitGroup
    ch := make(chan struct{},3)
    for i := 0; i < 10; i++ {
        ch <- struct{}()
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            time.Sleep(time.Second)
			<-ch
        }(i)
    }
    wg.Wait()
}
```

- `make(chan struct{}, 3)` 创建缓冲区大小为 3 的 channel，在没有被接收的情况下，至多发送 3 个消息则被阻塞
- 开启协程前，调用 `ch <- struct{}{}`，若缓存区满，则阻塞
- 协程任务结束，调用 `<-ch` 释放缓冲区
- `sync.WaitGroup` 并不是必须的，例如 http 服务，每个请求天然是并发的，此时使用 channel 控制并发处理的任务数量，就不需要 `sync.WaitGroup`

##### 利用第三方库

`tunny`

```go
package main

import (
	"log"
	"time"

	"github.com/Jeffail/tunny"
)

func main() {
    pool := tunny.NewFunc(3,func(i interface{}) interface{} {
        log.Println(i)
		time.Sleep(time.Second)
		return nil
    })
    defer pool.Close()
    
    for i := 0; i < 10; i++ {
		go pool.Process(i)
	}
    time.Sleep(time.Second * 4)
}
```

- `tunny.NewFunc(3, f)` 第一个参数是协程池的大小(poolSize)，第二个参数是协程运行的函数(worker)。
- `pool.Process(i)` 将参数 i 传递给协程池定义好的 worker 处理。
- `pool.Close()` 关闭协程池。

#### sync.Pool复用对象

##### sync.Pool的使用场景

保存和复用临时对象，减少内存分配，降低GC压力；sync.Pool用于存储被分配了但是没有被使用，而未来可能会使用的值，这样就可以不用再次经过内存分配

sync.Pool 的大小是可伸缩的，高负载时会动态扩容，存放在池中的对象如果不活跃了会被自动清理

##### 使用方式

###### 声明对象池

```go
var studentPool = sync.Pool {
    New: func() interface{} {
        return new(Student)
    }
}
```

###### Get & Put

```go
stu := studentPool.Get().(*Student)
json.Unmarshal(buf,stu)
studentPool.Put(stu)
```

- `Get()`从对象池中获取对象，因返回值是`interface{}`，因此需要类型转换
- `Put()`在对象使用完毕后，返回对象池

#### sync.Once如何提升性能

使函数只执行一次的实现，常应用于单例模式，如：初始化配置、保持数据库连接等，作用与`init`函数类似

`sync.Once` 常被用于控制变量的初始化，这个变量的读写满足如下三个条件：

- 当且仅当第一次访问某个变量时，进行初始化（写）；
- 变量初始化过程中，所有读都被阻塞，直到初始化完成；
- 变量仅初始化一次，初始化完成后驻留在内存里。

`sync.Once` 仅提供了一个方法 `Do`，参数 f 是对象初始化函数

```
func (o *Once) Do(f func())
```

##### 简单demo

使用场景：ReadConfig需要读取环境变量并转换为对应的配置。环境变量在程序执行前已经确定，执行过程中不会发生改变。ReadConfig 可能会被多个协程并发调用，为了提升性能，使用 `sync.Once` 是一个比较好的方式

```go
type Config struct {
	Server string
	Port   int64
}

var (
	once   sync.Once
	config *Config
)

func ReadConfig() *Config {
    once.Do(func() {
        var err error
        config = &Config{Server: os.Getenv("TT_SERVER_URL")}
        config.Port,err = strconv.ParseInt(os.Getenv("TT_PORT"), 10, 0)
        if err != nil {
			config.Port = 8080 // default port
        }
        log.Println("init config")
    })
    return config
}

func main() {
	for i := 0; i < 10; i++ {
		go func() {
			_ = ReadConfig()
		}()
	}
	time.Sleep(time.Second)
}
```

##### sync.Once的原理

首先：保证变量仅被初始化一次，需要有个标志判断变量是否已初始化过，若没有则需要初始化

第二：线程安全，支持并发，无疑需要互斥锁实现

##### sync.Once的源码实现

```go
package sync

import (
	"sync/atomic"
)

//使用done标记是否已经初始化，使用锁m实现线程安全
type Once struct {
    done 	unit32
    m 		Mutex
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 0 {
        o.doSlow(f)
    }
}

func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done,1)
        f()
    }
}
```

###### done为什么是第一个字段

done在热路径中，done放在第一个字段能够减少CPU指令，这样做能提升性能

- 热路径是程序频繁执行的一系列指令，sync.Once 大部分场景会访问 `o.done`
- 结构体第一个字段的地址和结构体的指针是相同的，若是第一个字段，直接对结构体指针解引用即可，若是其它字段还需计算与第一个值的偏移。在机器码中，偏移量是随指令传递的附加值，CPU需要做一次偏移值与指针的加法运算才能获取访问值的地址，访问第一个字段的机器码更紧凑，速度更快

#### sync.Cond条件变量

##### sync.Cond的使用场景

用于协调想要访问共享资源的goroutine，当共享资源状态发生变化时用来通知被互斥锁阻塞的goroutine；`sync.Cond`常用在多goroutine等待，一个goroutine通知（事件发生）的场景，如果是一个通知，一个等待，使用互斥锁或channel就能搞定。

模拟一个场景：

有一个协程在异步地接收数据，剩下的多个协程必须等待该协程接收完数据才能读取到正确的数据，此时使用chan或互斥锁则只能有一个协程可以等待并读取数据；需要有一个全局变量标记第一个协程数据是否接收完毕，剩下的协程反复检查该变量的值，直到满足要求，或者创建多个channel，每个协程阻塞在一个channel上，由接收数据的协程在数据接收完毕后逐个通知。

Go 语言在标准库 sync 中内置一个 `sync.Cond` 用来解决这类问题

##### sync.Cond 的四个方法

sync.Cond 的定义如下：

```go
type Cond struct {
    noCopy 	noCopy
    L 		Locker
    notify	notifyList
    checker	copyChecker
}
```

###### NewCond创建实例

```
func NewCond(l Locker) *Cond
```

NewCond创建Cond实例时需要关联一个锁

###### Broadcast广播唤醒所有

```
func (c *Cond) Broadcast()
```

Broadcast唤醒所有等待条件变量c的goroutine，无需锁保护

###### Signal唤醒一个协程

```
func (c *Cond) Signal()
```

Signal只唤醒任意1个等待条件变量c的goroutine，无需锁保护

###### Wait等待

```
func (c *Cond) wait()
```

调用Wait会自动释放锁c.L，并挂起调用者所在的goroutine，因此当前协程会阻塞在Wait方法调用的地方，如果其它协程调用Signal或Broadcast唤醒该协程，那么Wait方法在结束阻塞时会重新给c.L加锁继续执行Wait后面的代码

对条件的检查，使用了 `for !condition()` 而非 `if`，是因为当前协程被唤醒时，条件不一定符合要求，需要再次 Wait 等待下次被唤醒。为了保险起见，使用 `for` 能够确保条件符合要求后，再执行后续的代码

```go
c.L.Lock()
for !condition() {
	c.Wait()
}
... make use of condition ...
c.L.Unlock()
```

##### 使用示例

下面场景中三个协程调用`Wait()`等待，另一个协程调用`Broadcast()`唤醒所有等待的协程

```go
var done = false

func read(name string,c *sync.Cond) {
    c.L.Lock()
    for !done {
        c.Wait()
    }
    log.Println(name, "starts reading")
    c.L.Unlock()
}

func write(name string,c *sync.Cond) {
    log.Println(name,"starts writing")
    time.Sleep(time.Second)
    c.L.Lock()
    done = true
    c.L.Unlock
    log.Println(name, "wakes all")
	c.Broadcast()
}

func main() {
	cond := sync.NewCond(&sync.Mutex{})

	go read("reader1", cond)
	go read("reader2", cond)
	go read("reader3", cond)
	write("writer", cond)

	time.Sleep(time.Second * 3)
}
```

- `done`即互斥锁需要保护的条件变量
- `read()`调用`wait()`等待通知，直到done为true
- `write()`接收数据，接收完成后将done置为true，调用`Broadcast()`通知所有等待的协程
- `write()`中的暂停了1s，一方面是模拟耗时，另一方面确保前面的3个read协程都执行到`Wait()`，处于等待状态。main函数最后暂停了3s，确保所有操作执行完毕

### 编译优化

#### 减小编译体积

##### 默认编译

```
$ go build -o server main.go
$ ls -lh server
-rwxr-xr-x  1 dj  staff   9.8M Dec  7 23:57 server
```

##### 编译选项

Go编译器默认编译的程序会带有符号表和调试信息，一般来说release版本可以去除调试信息以减少二进制体积

```
$ go build -ldflags="-s -w" -o server main.go
$ ls -lh server
-rwxr-xr-x  1 dj  staff   7.8M Dec  8 00:29 server
```

- -s：忽略符号表和调试信息
- -w：忽略DWARFv3调试信息，使用该选项后将无法使用gdb进行调试

##### 使用upx

upx 有很多参数，最重要的是压缩率，`1-9`，`1` 代表最低压缩率，`9` 代表最高

```go
$ go build -o server main.go && upx -9 server
        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
  10253684 ->   5210128   50.81%   macho/amd64   server 
$ ls -lh server
-rwxr-xr-x  1 dj  staff   5.0M Dec  8 00:45 server
```

##### upx和编译选项组合

```go
$ go build -ldflags="-s -w" -o server main.go && upx -9 server
        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
   8213876 ->   3170320   38.60%   macho/amd64   server 
$ ls -lh server
-rwxr-xr-x  1 dj  staff   3.0M Dec  8 00:47 server
```

#### 逃逸分析对性能的影响

Go 程序会在 2 个地方为变量分配内存，一个是全局的堆(heap)空间用来动态分配内存，另一个是每个 goroutine 的栈(stack)空间；Go 语言的内存管理是自动的，通常开发者并不需要关心内存分配在栈上，还是堆上。但是从性能的角度出发，在栈上分配内存和在堆上分配内存，性能差异是非常大的

在函数中申请一个对象，如果分配在栈中，函数执行结束时自动回收，如果分配在堆中，则在函数结束后某个时间点进行垃圾回收

在栈上分配和回收内存的开销很低，只需要 2 个 CPU 指令：PUSH 和 POP，一个是将数据 push 到栈空间以完成分配，pop 则是释放空间，也就是说在栈上分配内存，消耗的仅是将数据拷贝到内存的时间，而内存的 I/O 通常能够达到 30GB/s，因此在栈上分配内存效率是非常高的

##### 逃逸分析

编译器决定内存分配位置的方式就称为逃逸分析，逃逸分析作用于编译阶段

###### 指针逃逸

函数中创建了一个对象，返回了该对象的指针，此时函数虽然退出了，但是因为指针的存在，对象的内存不能随着函数结束而回收，因此只能分配在堆上

```go
// main_pointer.go
package main

import "fmt"

type Demo struct {
	name string
}

func createDemo(name string) *Demo {
	d := new(Demo) // 局部变量 d 逃逸到堆
	d.name = name
	return d
}

func main() {
	demo := createDemo("demo")
	fmt.Println(demo)
}
```

编译时可以借助选项 `-gcflags=-m`，查看变量逃逸的情况：

```
$ go build -gcflags=-m main_pointer.go 
./main_pointer.go:10:6: can inline createDemo
./main_pointer.go:17:20: inlining call to createDemo
./main_pointer.go:18:13: inlining call to fmt.Println
./main_pointer.go:10:17: leaking param: name
./main_pointer.go:11:10: new(Demo) escapes to heap
./main_pointer.go:17:20: new(Demo) escapes to heap
./main_pointer.go:18:13: demo escapes to heap
./main_pointer.go:18:13: main []interface {} literal does not escape
./main_pointer.go:18:13: io.Writer(os.Stdout) escapes to heap
<autogenerated>:1: (*File).close .this does not escape
```

`new(Demo) escapes to heap` 即表示 `new(Demo)` 逃逸到堆上了

###### interface{}动态类型逃逸

若函数参数为 `interface{}`，编译期间很难确定参数的具体类型，也会发生逃逸

例如上面例子中的局部变量 `demo`：

```
func main() {
	demo := createDemo("demo")
	fmt.Println(demo)
}
```

```
./main_pointer.go:18:13: demo escapes to heap
```

`demo` 是 main 函数的一个局部变量，该变量作为实参传递给 `fmt.Println()`，但是因为 `fmt.Println()` 的参数类型定义为 `interface{}`，因此也发生了逃逸

`fmt` 包中的 `Println` 函数的定义如下：

```
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```

###### 栈空间不足

因为栈空间通常比较小，因此递归函数实现不当时，容易导致栈溢出；操作系统对内核线程使用的栈空间有大小限制，64 位系统上通常是 8 MB；对 Go 编译器而言，超过一定大小的局部变量将逃逸到堆上，不同 Go 版本大小限制可能不一样。

当占用内存超过一定大小或无法确定当前切片长度时，对象占用内存将在堆上分配

###### 闭包

```go
func Increase() func() int {
	n := 0
	return func() int {
		n++
		return n
	}
}

func main() {
	in := Increase()
	fmt.Println(in()) // 1
	fmt.Println(in()) // 2
}
```

`Increase()` 返回值是一个闭包函数，该闭包函数访问了外部变量 n，那变量 n 将会一直存在，直到 `in` 被销毁。很显然，变量 n 占用的内存不能随着函数 `Increase()` 的退出而回收，因此将会逃逸到堆上

```
$ go build -gcflags=-m main_closure.go 
# command-line-arguments
./main_closure.go:6:2: moved to heap: n
```

##### 如何利用逃逸分析提升性能

###### 传值 VS 传指针

传值会拷贝整个对象，而传指针只会拷贝指针地址，指向的对象是同一个。传指针可以减少值的拷贝，但是会导致内存分配逃逸到堆中，增加垃圾回收(GC)的负担。在对象频繁创建和删除的场景下，传递指针导致的 GC 开销可能会严重影响性能

一般情况下，对于需要修改原对象值，或占用内存比较大的结构体，选择传指针。对于只读的占用内存较小的结构体，直接传值能够获得更好的性能

#### Go死码消除与调试模式

##### 什么是死码消除

即一种编译器优化技术，在编译阶段去掉对程序运行结果没有影响的代码；可以减小程序体积，程序运行中避免执行无用的指令，缩短运行时间

##### Go语言中的应用

###### 使用常量提升性能

在某些场景下，将变量替换为常量，性能会有很大的提升

```go
// maxvar.go
func max(num1,num2 int) int {
    if num1 > num2 {
        return num1
    }
    return num2
}

var a,b = 10,20
const a,b = 10,20	//maxconst.go	将var修改为const

func main() {
    if max(a,b) == a {
        fmt.Println(a)
    }
}
```

```
go build -o maxvar maxvar.go
go build -o maxconst maxconst.go
ls -l maxvar maxconst
-rwxr-xr-x  1 x x 1895424 Jan 10 00:01 maxconst
-rwxr-xr-x  1 x x 2120368 Jan 10 00:01 maxvar
```

我们可以看到 `maxconst` 比 `maxvar` 体积小了约 10% = 0.22 MB

我们使用 `-gcflags=-m` 参数看一下编译器做了哪些优化：

```
go build -gcflags=-m  -o maxvar maxvar.go
# command-line-arguments
./maxconst.go:7:6: can inline max
./maxconst.go:17:8: inlining call to max
```

max 函数被内联了，即被展开了，手动展开后如下：

```go
func main() {
	var result int
	if a > b {
		result = a
	} else {
		result = b
    }
	if result == a {
		fmt.Println(a)
	}
}
```

那如果 a 和 b 均为常量（const）呢？那在编译阶段就可以直接进行计算：

```go
func main() {
	var result int
	if 10 > 20 {
		result = 10
	} else {
		result = 20
    }
	if result == 10 {
		fmt.Println(a)
	}
}
```

计算之后，`10 > 20` 永远为假，那么分支消除后：

```go
func main() {
	if 20 == 10 {
		fmt.Println(a)
	}
}
```

进一步，`20 == 10` 也永远为假，再次分支消除：

```
func main() {}
```

因此，在声明全局变量时，如果能够确定为常量，尽量使用 const 而非 var，这样很多运算在编译器即可执行。死码消除后，既减小了二进制的体积，又可以提高运行时的效率，如果这部分代码是 `hot path`，那么对性能的提升会更加明显

###### 可推断的局部变量

```go
// maxvarlocal
func main() {
	var a, b = 10, 20
	if max(a, b) == a {
		fmt.Println(a)
	}
}
```

编译结果如下，大小与 `varconst` 一致，即 a、b 作为局部变量时，编译器死码消除是生效的

```
$ go build -o maxvarlocal maxvarlocal.go
$ ls -l maxvarlocal                      
-rwxr-xr-x  1 x x 1895424 Jan 10 00:05 maxvarlocal
```

###### 调试模式

在源代码中定义全局常量debug，值设置为`false`，在需要增加调试代码的地方使用条件语句`if debug`包裹：

```go
const debug = false

func main() {
	if debug {
		log.Println("debug mode is enabled")
	}
}
```

###### 条件编译

不修改源代码也能编译出debug版本的方式：结合build tags实现条件编译：

```go
// debug.go

package main

const debug = true
```

```go
// release.go

package main

const debug = false
```

### 语言陷阱

#### 数组和切片

##### 第一个陷阱

```go
func foo(a [2]int) {
	a[0] = 200
}

func main() {
	a := [2]int{1, 2}
	foo(a)
	fmt.Println(a)
}
```

正确的输出是 `[1 2]`，数组 `a` 没有发生改变

- 在Go语言中，数组是一种值类型，且不同长度的数组属于不同的类型

为避免数组的拷贝，提高性能，建议传数组的指针作为参数，或用切片代替数组

需将上述示例替换为：

```go
func foo(a *[2]int) {
	(*a)[0] = 200
}

func main() {
	a := [2]int{1, 2}
	foo(&a)
	fmt.Println(a)
}
```

或

```go
func foo(a []int) {
	a[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(a)
	fmt.Println(a)
}
```

因此，将切片作为参数时，拷贝了一个新切片，即拷贝了构成切片的三个值，包括底层数组的指针。对切片中某个元素的修改，实际上是修改了底层数组中的值，因此原切片也发生了改变

##### 第二个陷阱

```go
func foo(a []int) {
	a = append(a, 1, 2, 3, 4, 5, 6, 7, 8)
	a[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(a)
	fmt.Println(a)
}
```

输出仍是 `[1 2]`，切片 `a` 没有发生改变

传参时拷贝了新的切片，因此当新切片的长度发生改变时，原切片并不会发生改变。而且在函数 `foo` 中，新切片 `a` 增加了 8 个元素，原切片对应的底层数组不够放置这 8 个元素，因此申请了新的空间来放置扩充后的底层数组。这个时候新切片和原切片指向的底层数组就不是同一个了。因此，对新切片第 0 个元素的修改，并不会影响原切片的第 0 个元素

如果如果希望 `foo` 函数的操作能够影响原切片呢？两种方式：

- 设置返回值，将新切片返回并赋值给 `main` 函数中的变量 `a`
- 切片也使用指针方式传参

```go
func foo(a []int) []int {
	a = append(a, 1, 2, 3, 4, 5, 6, 7, 8)
	a[0] = 200
	return a
}

func main() {
	a := []int{1, 2}
	a = foo(a)
	fmt.Println(a)
}
```

或

```go
func foo(a *[]int) {
	*a = append(*a, 1, 2, 3, 4, 5, 6, 7, 8)
	(*a)[0] = 200
}

func main() {
	a := []int{1, 2}
	foo(&a)
	fmt.Println(a)
}
```