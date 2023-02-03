---
title: gin框架笔记
date: 2023-02-03 13:22:02
tags: "gin"
categories: "go"
---

#### 自定义模版函数

> main.go

```go
package main

import (
	"github.com/gin-gonic/gin"
	"html/template"
	"net/http"
)

func main() {
	//创建一个默认的路由引擎
	router := gin.Default()

	router.SetFuncMap(template.FuncMap{
		"safe": func(str string) template.HTML {
			return template.HTML(str)
		},
	})
	router.LoadHTMLFiles("./index.tmpl")

	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", "<a href='https://liwenzhou.com'>李文周的博客</a>")
	})

	router.Run(":8080")
}
```

> index.tmpl

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>修改模板引擎的标识符</title>
</head>
<body>
<div>{{ . | safe }}</div>
</body>
</html>
```

#### 静态文件处理

当渲染的html文件中引用了静态文件时，按照以下方式在渲染页面前调用**gin.static**方法即可

```go
func main() {
	r := gin.Default()
	r.Static("/static", "./static")
	r.LoadHTMLGlob("templates/**/*")
   	// ...
	r.Run(":8080")
}
```

#### JSON、XML、YMAL、protobuf渲染

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/testdata/protoexample"
	"net/http"
)

func main() {
	r := gin.Default()
	//gin.H是map[string]interface{}的缩写
	r.GET("/someJSON", func(c *gin.Context) {
		//方式一：自己拼接
		c.JSON(http.StatusOK, gin.H{"message": "Hello world!"})
	})
	r.GET("/someXML", func(c *gin.Context) {
		//方式一：自己拼接
		c.XML(http.StatusOK, gin.H{"message": "Hello world!"})
	})
	r.GET("/moreJSON", func(c *gin.Context) {
		//方式二：使用结构体
		var msg struct {
			Name    string
			Message string
			Age     int
		}
		msg.Name = "小王子"
		msg.Message = "Hello world!"
		msg.Age = 18
		c.JSON(http.StatusOK, msg)
	})
	r.GET("/moreXML", func(c *gin.Context) {
		//方式二：使用结构体
		type message struct {
			Name    string `json:"name"`
			Message string
			Age     int
		}
		var msg message
		msg.Name = "小王子"
		msg.Message = "Hello world!"
		msg.Age = 18
		c.XML(http.StatusOK, msg)
	})
	r.GET("/someYAML", func(c *gin.Context) {
		c.YAML(http.StatusOK, gin.H{"message": "ok", "status": http.StatusOK})
	})
	r.GET("/someProtoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		label := "test"
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		c.ProtoBuf(http.StatusOK, data)
	})
	r.Run(":8080")
}
```

#### 获取参数

##### 获取querystring参数

querystring指的是URL中？后面携带的参数；获取请求的querystring参数方法：

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.GET("/user/search", func(c *gin.Context) {
		username := c.DefaultQuery("username", "小王子")
		address := c.Query("address")
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  address,
		})
	})
	r.Run()
}
```

##### 获取form参数

使用c.PostForm获取请求数据

```
username := c.PostForm("username")
address := c.PostForm("address")
```

##### 获取Map参数（字典参数）

```go
r.POST("/post",func(c *gin.context){
    ids := c.QueryMap("ids")
    names := c.PostFormMap("names")
    
    c.JSON(http.StatusOK,gin.H{
        "ids": ids,
        "names": names,
    })
})
```

```
$ curl -g "http://localhost:9999/post?ids[Jack]=001&ids[Tom]=002" -X POST -d 'names[a]=Sam&names[b]=David'

{"ids":{"Jack":"001","Tom":"002"},"names":{"a":"Sam","b":"David"}}
```

##### 获取json参数

```go
package main

import (
	"encoding/json"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()

	r.POST("/json", func(c *gin.Context) {
		b, _ := c.GetRawData() //从c.Request.Body读取请求数据
		//定义map或结构体
		var m map[string]interface{}
		//反序列化
		_ = json.Unmarshal(b, &m)
		c.JSON(http.StatusOK, m)
	})

	r.Run()
}
```

##### 获取path参数

请求的参数通过URL路径传递，例如：/user/search/小王子/沙河。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.GET("/user/search/:username/:address", func(c *gin.Context) {
		username := c.Param("username")
		address := c.Param("address")
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  address,
		})
	})
	r.Run()
}
```

##### 参数绑定

为了能够更方便地获取相关参数，可以基于请求的Content-Type识别请求数据类型并利用反射机制自动提取请求中Querystring、form表单、JSON、XML等参数到结构体中，下面代码演示了.ShouldBind()强大的功能，它能够基于请求自动提取JSON、form表单和Querystring类型的数据，并把值绑定到指定结构体对象。

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

type Login struct {
	User     string `form:"user" json:"user" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}

func main() {
	router := gin.Default()
	router.POST("/loginJSON", func(c *gin.Context) {
		var login Login
		if err := c.ShouldBind(&login); err == nil {
			fmt.Printf("login info:%#v\n", login)
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})
	//绑定form表单示例
	router.POST("/loginForm", func(c *gin.Context) {
		var login Login
		//ShouldBind()会根据请求的Content-type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		}
	})
	//绑定QueryString示例
	router.GET("/loginQuery", func(c *gin.Context) {
		var login Login
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})
	router.Run()
}
```

ShouldBind会按照下面的顺序解析请求中数据完成绑定：

1. 若是GET请求，只使用Form绑定引擎（query）
2. 若是POST请求，首先检查content-type是否为JSON或XML，然后再使用Form（form-data）

#### 在gin框架中使用JWT

JWT全称JSON Web Token是一种跨域认证解决方案，属于一个开放的标准，规定了一种Token实现方式，多用于前后端分离项目和OAuth2.0业务场景下

**为什么要使用JWT：**用户可能使用浏览器也可能使用app访问服务，有时候还需要支持第三方登录，因此Cookie-Session模式有些力不从心了；JWT即基于Token的轻量级认证模式，认证通过后会生成一个JSON对象，经过签名后得到一个token（令牌）再发回给用户，用户后续请求只需带上该token，服务端解密后就能获取用户相关信息

##### JWT的使用

若直接使用JWT中默认的字段，没有其他定制化的需求则可以直接使用这个包中的和方法快速生成和解析token

```go
// 用于签名的字符串
var mySigningKey = []byte("liwenzhou.com")

// GenRegisteredClaims 使用默认声明创建jwt
func GenRegisteredClaims() (string, error) {
	// 创建 Claims
	claims := &jwt.RegisteredClaims{
		ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour * 24)), // 过期时间
		Issuer:    "qimi",                                             // 签发人
	}
	// 生成token对象
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	// 生成签名字符串
	return token.SignedString(mySigningKey)
}

// ParseRegisteredClaims 解析jwt
func ValidateRegisteredClaims(tokenString string) bool {
	// 解析token
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		return mySigningKey, nil
	})
	if err != nil { // 解析token失败
		return false
	}
	return token.Valid
}
```

#### 常用HTTP服务压测工具

##### 压测相关术语

- 响应时间（RT）：指系统对请求作出响应的时间
- 吞吐量（Throughput）：指系统在单位时间内处理请求的数量
- QPS每秒查询率（Query Per Second）：一台服务器每秒能够响应的查询次数，是对一个特定查询服务器在规定时间内处理流量多少的衡量标准
- TPS（TransactionPerSecond）：每秒钟系统能够处理的交易或事务的数量
- 并发连接数：某个时刻服务器所接受的请求总数

##### 压力测试工具

###### ab

**ab**：Apache Bench，指定同时连接数、请求数及URL，即可测试网站或网站程序的性能，通过ab发送请求模拟多个访问者同时对某一URL地址进行访问，可以得到每秒传送字节数、每秒处理请求数、每请求处理时间等数据

命令格式：

```
ab [options] [http://]hostname[:port]/path
```

常用参数如下：

```
-n requests 总请求数
-c concurrency 一次产生的请求数，可以理解为并发数
-t timelimit 测试所进行的最大秒数, 可以当做请求的超时时间
-p postfile 包含了需要POST的数据的文件
-T content-type POST数据所使用的Content-type头信息
```

例如测试某个POST请求接口：

```
ab -n 10000 -c 100 -t 10 -p post.json -T "application/json" "http://127.0.0.1:8080/api/v1/post"
```

###### wrk

**wrk**：比ab功能更加强大，可以通过编写lua脚本来支持更复杂的测试场景

常用参数如下：

```
-c --conections：保持的连接数
-d --duration：压测持续时间(s)
-t --threads：使用的线程总数
-s --script：加载lua脚本
-H --header：在请求头部添加一些参数
--latency 打印详细的延迟统计信息
--timeout 请求的最大超时时间(s)
```

使用示例：

```
wrk -t8 -c100 -d30s --latency http://127.0.0.1:8080/api/v1/posts?size=10
```

输出结果：

```
Running 30s test @ http://127.0.0.1:8080/api/v1/posts?size=10
  8 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.55ms    2.02ms  31.59ms   76.70%
    Req/Sec   828.16     85.69     0.97k    60.46%
  Latency Distribution
     50%   14.44ms
     75%   15.76ms
     90%   16.63ms
     99%   21.07ms
  198091 requests in 30.05s, 29.66MB read
Requests/sec:   6592.29
Transfer/sec:      0.99MB
```

###### go-wrk

go-wrk：go语言版本的wrk

安装方式：

```
go get github.com/adeven/go-wrk
```

使用方法同`wrk`类似，基本格式如下：

```
go-wrk [flags] url
```

常用参数如下：

```
-H="User-Agent: go-wrk 0.1 bechmark\nContent-Type: text/html;": 由'\n'分隔的请求头
-c=100: 使用的最大连接数
-k=true: 是否禁用keep-alives
-i=false: if TLS security checks are disabled
-m="GET": HTTP请求方法
-n=1000: 请求总数
-t=1: 使用的线程数
-b="" HTTP请求体
-s="" 如果指定，它将计算响应中包含搜索到的字符串s的频率
```

执行测试：

```
go-wrk -t=8 -c=100 -n=10000 "http://127.0.0.1:8080/api/v1/posts?size=10"
```

输出结果：

```
==========================BENCHMARK==========================
URL:                            http://127.0.0.1:8080/api/v1/posts?size=10

Used Connections:               100
Used Threads:                   8
Total number of calls:          10000

===========================TIMINGS===========================
Total time passed:              2.74s
Avg time per request:           27.11ms
Requests per second:            3644.53
Median time per request:        26.88ms
99th percentile time:           39.16ms
Slowest time for request:       45.00ms

=============================DATA=============================
Total response body sizes:              340000
Avg response body per request:          34.00 Byte
Transfer rate per second:               123914.11 Byte/s (0.12 MByte/s)
==========================RESPONSES==========================
20X Responses:          10000   (100.00%)
30X Responses:          0       (0.00%)
40X Responses:          0       (0.00%)
50X Responses:          0       (0.00%)
Errors:                 0       (0.00%)
```

#### 常用限流策略---漏桶和令牌桶

##### 漏桶

将请求比作水，以限定的速度处理请求；当请求来的过猛时会导致直接溢出，即拒绝服务

###### 开源库

```
https://github.com/uber-go/ratelimit
```

```go
import (
	"fmt"
	"time"

	"go.uber.org/ratelimit"
)

func main() {
    rl := ratelimit.New(100) // per second

    prev := time.Now()
    for i := 0; i < 10; i++ {
        now := rl.Take()
        fmt.Println(i, now.Sub(prev))
        prev = now
    }
}
```

`Take()`方法会返回漏桶下一次滴水时间

##### 令牌桶

以恒定的速度往桶里放入令牌，若请求需要被处理，则需要先从桶里获取一个令牌，没有令牌可取时则拒绝服务

###### 开源库

```
https://github.com/juju/ratelimit
```

###### 创建令牌桶的方法

```go
//创建指定填充速率和容量大小的令牌桶
func NewBucket(fillInterval time.Duration,capacity int64) *Bucket
//创建指定填充速率、容量大小和每次填充的令牌数的令牌桶
func NewBucketWithQuantum(fillInterval time.Duration,capacity,quantum int64) *Bucket
//创建填充速度为指定速率和容量大小的令牌桶
// NewBucketWithRate(0.1, 200) 表示每秒填充20个令牌
func NewBucketWithRate(rate float64，capacity int64) *Bucket
```

取出令牌的方法如下：

```go
//取token(非阻塞)
func (tb *Bucket) Take(count int64) time.Duration
func (tb *Bucket) TakeAvailable(count int64) int64

//最多等maxWait时间取token
func (tb *Bucket) TakeMaxDuration(count int64,maxWait time.Duration) (time.Duration, bool)

//取token(阻塞)
func (tb *Bucket) Wait(count int64)
func (tb *Bucket) WaitMaxDuration(count int64,maxWait time.Duration) bool
```

没有必要生成令牌放入桶中，只需要每次取令牌的时候计算一下当前是否有足够的令牌即可，具体计算方式如下：

```
当前令牌数 = 上一次剩余的令牌数 + (本次取令牌的时刻-上一次取令牌的时刻)/放置令牌的时间间隔 * 每次放置的令牌数
```

##### 漏桶和令牌桶区别

漏桶可以限制数据的传输速率，令牌桶能够限制数据的平均传输速率的同时还允许某种程度的突发传输；通常需要将漏桶和令牌桶结合起来使用

##### gin框架中使用限流中间件

```go
func RateLimitMiddleware(fillInterval time.Duration, cap int64) func(c *gin.Context) {
	bucket := ratelimit.NewBucket(fillInterval, cap)
	return func(c *gin.Context) {
		// 如果取不到令牌就中断本次请求返回 rate limit...
		if bucket.TakeAvailable(1) < 1 {
			c.String(http.StatusOK, "rate limit...")
			c.Abort()
			return
		}
		c.Next()
	}
}
```

