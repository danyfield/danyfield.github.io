---
title: go常用组件技巧_liwenzhou
date: 2023-02-03 13:10:48
tags: "go"
categories: "go"
---

#### json技巧

##### 忽略某个字段

如果想在json序列化/反序列化时忽略结构体的某个字段，可以按如下方式在tag中添加`-`

##### 忽略空值字段

当struct中字段没有值时，`json.Marshal()`序列化时默认输出字段的对应类型零值，若想要忽略这些字段可以添加`omitempty`tag，定义的结构体如下：

```go
// 在tag中添加omitempty忽略空值
// 注意这里 hobby,omitempty 合起来是json tag值，中间用英文逗号分隔
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
}
```

##### 忽略嵌套结构体空值字段

以下结构体嵌套示例：

```go
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
	Profile
}

type Profile struct {
	Website string `json:"site"`
	Slogan  string `json:"slogan"`
}
```

匿名嵌套`Porfile`时序列化后的json串为单层的：

```go
str:{"name":"七米","hobby":["足球","双色球"],"site":"","slogan":""}
```

若想要变成嵌套的json串，需要改为具名嵌套或定义字段tag

```go
type User struct {
	Name    string   `json:"name"`
	Email   string   `json:"email,omitempty"`
	Hobby   []string `json:"hobby,omitempty"`
	Profile `json:"profile"`
}
```

输出结果如下：

```go
str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

想要在嵌套的结构体为空值时忽略该字段，仅添加`omitempty`是不够的，还需要使用嵌套的结构体指针：

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	*Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"]}
```

##### 不修改原结构体忽略空值字段

若需要json序列化`User`，但不想把密码页序列化，又不想修改`User`结构体，这个时候我们就可以创建另外一个结构体`PublicUser`匿名嵌套原`User`，同时指定`Password`字段为匿名结构体指针类型，并添加`omitempty`tag：

```go
type User struct {
	Name     string `json:"name"`
	Password string `json:"password"`
}

type PublicUser struct {
	*User             // 匿名嵌套
	Password *struct{} `json:"password,omitempty"`
}
```

##### 处理字符串格式数字

前端传来的json数据可能会使用字符串类型的数字，此时可以在结构体tag中添加`string`解析相应字段的数据

```go
type Card struct {
	ID    int64   `json:"id,string"`    // 添加string tag
	Score float64 `json:"score,string"` // 添加string tag
}

func intAndStringDemo() {
	jsonStr1 := `{"id": "1234567","score": "88.50"}`
	var c1 Card
	if err := json.Unmarshal([]byte(jsonStr1), &c1); err != nil {
		fmt.Printf("json.Unmarsha jsonStr1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("c1:%#v\n", c1) // c1:main.Card{ID:1234567, Score:88.5}
}
```

##### 整数变浮点数

JSON协议无整型和浮点型之分，统称number。json字符串数字Go反序列化后都会变成float64类型；将JSON格式数据反序列化为`map[string]interface{}`时，都会变成科学计数法浮点数，此时需要使用decode

```go
// useNumberDemo 使用json.UseNumber
// 解决将JSON数据反序列化成map[string]interface{}时
// 数字变为科学计数法表示的浮点数问题
func useNumberDemo(){
	type student struct {
		ID int64 `json:"id"`
		Name string `json:"q1mi"`
	}
	s := student{ID: 123456789,Name: "q1mi"}
	b, _ := json.Marshal(s)
	var m map[string]interface{}
	// decode
	json.Unmarshal(b, &m)
	fmt.Printf("id:%#v\n", m["id"])  // 1.23456789e+08
	fmt.Printf("id type:%T\n", m["id"])  //float64

	// use Number decode
	decoder := json.NewDecoder(bytes.NewReader(b))
	decoder.UseNumber()
	decoder.Decode(&m)
	fmt.Printf("id:%#v\n", m["id"])  // "123456789"
	fmt.Printf("id type:%T\n", m["id"]) // json.Number
}
```

##### 使用匿名结构体添加字段

```
type UserInfo struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

u1 := UserInfo{
	ID:   123456,
	Name: "七米",
}
// 使用匿名结构体内嵌User并添加额外字段Token
b, err := json.Marshal(struct {
	*UserInfo
	Token string `json:"token"`
}{
	&u1,
	"91je3a4s72d1da96h",
})
```

#### 结构体转map[string]interface{}

json包在序列化空接口存放的数字类型（整型、浮点型等）都会序列化成float64

##### 使用第三方库structs

```
https://github.com/fatih/structs
```

##### 使用反射

```go
// ToMap 结构体转为Map[string]interface{}
func ToMap(in interface{}, tagName string) (map[string]interface{}, error){
    out := make(map[string]interface{})
    
    v := reflect.ValueOf(in)
    if v.Kind() == reflect.Ptr {
        v = v.Elem()
    }
    if v.Kind() != reflect.Struct {  // 非结构体返回错误提示
		return nil, fmt.Errorf("ToMap only accepts struct or struct pointer; got %T", v)
	}
    
    t := v.Type()
    //遍历结构体字段
    //指定tagName值为map中key；字段值为map中value
    for i := 0; i < v.NumField(); i++ {
        fi := t.Field(i)
        if tagValue := fi.Tag.Get(tagName); tagValue != "" {
            out[tagValue] = v.Field(i).Interface()
        }
    }
    return out,nil
}
```

#### Go语言配置管理器--Viper

##### 安装

```
go get github.com/spf13/viper
```

##### 什么是Viper

被用于处理所有类型的配置需求和格式：

- 设置默认值
- 从`JSON`、`TOML`、`YAML`、`HCL`、`envfile`和`Java properties`格式的配置文件读取配置信息
- 实时监控和重新读取配置文件（可选）
- 从环境变量中读取
- 从远程配置系统（etcd或Consul）读取并监控配置变化
- 从命令行参数读取配置
- 从buffer读取配置
- 显式配置值

Viper会按照下面的优先级，每个项目的优先级都高于它下面的项目：

- 显示调用`set`设置值
- 命令行参数（flag）
- 环境变量
- 配置文件
- key/value存储
- 默认值

**目前Viper配置的键是大小写不敏感的**

下面是一个如何使用Viper搜索和读取配置文件的示例：

```go
viper.SetConfigFile("./config.yaml") // 指定配置文件路径
viper.SetConfigName("config") // 配置文件名称(无扩展名)
viper.SetConfigType("yaml") // 如果配置文件的名称中没有扩展名，则需要配置此项
viper.AddConfigPath("/etc/appname/")   // 查找配置文件所在的路径
viper.AddConfigPath("$HOME/.appname")  // 多次调用以添加多个搜索路径
viper.AddConfigPath(".")               // 还可以在工作目录中查找配置
err := viper.ReadInConfig() // 查找并读取配置文件
if err != nil { // 处理读取配置文件的错误
	panic(fmt.Errorf("Fatal error config file: %s \n", err))
}
```

在加载配置文件出错时可以像下面这样处理找不到配置文件的特定情况：

```go
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // 配置文件未找到错误；如果需要可以忽略
    } else {
        // 配置文件被找到，但产生了另外的错误
    }
}

// 配置文件找到并成功解析
```

##### 写入配置文件

- WriteConfig - 将当前的`viper`配置写入预定义的路径并覆盖（如果存在的话）。如果没有预定义的路径，则报错。
- SafeWriteConfig - 将当前的`viper`配置写入预定义的路径。如果没有预定义的路径，则报错。如果存在，将不会覆盖当前的配置文件。
- WriteConfigAs - 将当前的`viper`配置写入给定的文件路径。将覆盖给定的文件(如果它存在的话)。
- SafeWriteConfigAs - 将当前的`viper`配置写入给定的文件路径。不会覆盖给定的文件(如果它存在的话)。

标记为`safe`的所有方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。

```go
viper.WriteConfig() // 将当前配置写入“viper.AddConfigPath()”和“viper.SetConfigName”设置的预定义路径
viper.SafeWriteConfig()
viper.WriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.config") // 因为该配置文件写入过，所以会报错
viper.SafeWriteConfigAs("/path/to/my/.other_config")
```

##### 监控并重新读取配置文件

<font color=red>首先确保在调用`WatchConfig()`前添加了所有配置路径</font>

```go
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
    // 配置文件发生变更之后会调用的回调函数
	fmt.Println("Config file changed:", e.Name)
})
```

##### 从io.Reader读取配置

Viper预先定义了许多配置源，如文件、环境变量、标志和远程K/V存储，但不受其约束。可以实现自己所需的配置源并将其提供给viper

```go
viper.SetConfigType("yaml")	//或viper.SetConfigType("YAML")

//任何需要将此配置添加到程序中的方法
var yamlExample = []byte(`
    Hacker: true
    name: steve
    hobbies:
    - skateboarding
    - snowboarding
    - go
    clothing:
      jacket: leather
      trousers: denim
    age: 35
    eyes : brown
    beard: true
`)

viper.ReadConfig(bytes.NewBuffer(yamlExample))
viper.Get("name")	//得到name的设置值
```

##### 覆盖设置

```go
viper.Set("Verbose",true)
Viper.Set("LogFile",LogFile)
```

##### 注册和使用别名

允许多个键引用单个值

```go
viper.RegisterAlias("loud","Verbose")	//注册别名

viper.Set("Verbose",true)
viper.Set("loud",true)

viper.GetBool("loud")	//true
viper.GetBool("Verbose")	//true
```

##### 远程Key/Value存储支持

在Viper中启用远程支持，需要在代码中匿名导入`viper/remote`包

`import _ "github.com/spf13/viper/remote"`

Viper读取从Key/Value存储（例如etcd或Consul）中的路径检索到的配置字符串（如`JSON`、`TOML`、`YAML`、`HCL`、`envfile`和`Java properties`格式）（译注：也就是说Viper加载配置值的优先级为：磁盘上的配置文件>命令行标志位>环境变量>远程Key/Value存储>默认值。）

##### 远程Key/Value存储示例-未加密

`etcd`

```go
viper.AddRemoteProvide("etcd","http://127.0.0.1:4001","/config/hugo.json")
viper.SetConfigType("json")	//因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```

`Consul`

需要在Consul Key/Value存储中设置一个Key保存包含所需配置的JSON值。例如，创建一个key`MY_CONSUL_KEY`将下面的值存入Consul key/value 存储：

```json
{
    "port": 8080,
    "hostname": "liwenzhou.com"
}
```

```go
viper.AddRemoteProvider("consul","localhost:8500","MY_CONSUL_KEY")
viper.SetConfigType("json")	//需要显示设置成json
err := viper.ReadRemoteConfig()

fmt.Println(viper.Get("port")) // 8080
fmt.Println(viper.Get("hostname")) // liwenzhou.com
```

`Firestore`

```go
viper.AddRemoteProvider("firestore", "google-cloud-project-id", "collection/document")
viper.SetConfigType("json")
err := viper.ReadRemoteConfig()
```

##### 远程Key/Value存储示例-加密

```go
viper.AddSecureRemoteProvider("etcd","http://127.0.0.1:4001","/config/hugo.json","/etc/secrets/mykeyring.gpg")
viper.SetConfigType("json") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```

##### 监控etcd中的更改-未加密

```go
var runtime_viper = viper.New()

runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")
runtime_viper.SetConfigType("yaml")

//第一次从远程读取配置
err := runtime_viper.ReadRemoteConfig()

//反序列化
runtime_viper.Unmarshal(&runtime_conf)

//开启一个单独的goroutine一直监控远端的变更
go func(){
    for {
        time.Sleep(time.Second * 5)		//每次请求后延迟一下
        //目前只测试了etcd支持
        err := runtime_viper.WatchRemoteConfig()
        if err != nil {
            log.Errorf("unable to read remote config: %v", err)
	        continue
        }
        //将新配置反序列化到运行时的配置结构体中。可以借助channel实现一个通知系统更改的信号
        runtime_viper.Unmarshal(&runtime_conf)
    }
}
```

##### 从Viper中获取值的方法

```go
Get(key string) : interface{}
GetBool(key string) : bool
GetFloat64(key string) : float64
GetInt(key string) : int
GetIntSlice(key string) : []int
GetString(key string) : string
GetStringMap(key string) : map[string]interface{}
GetStringMapString(key string) : map[string]string
GetStringSlice(key string) : []string
GetTime(key string) : time.Time
GetDuration(key string) : time.Duration
IsSet(key string) : bool
AllSettings() : map[string]interface{}
```

每一个Get方法找不到值的时候都会返回零值，为了检查特定的键是否存在，应使用`IsSet()`方法

```go
viper.GetString("logfile")	//不区分大小写的设置和获取，读取环境变量时区分大小写
if viper.GetBool("verbose") {
    fmt.Println("verbose enabled")
}
```

##### 访问嵌套的键

```json
{
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}
```

```go
GetString("datastore.metric.host") // (返回 “127.0.0.1”)
```

这遵守上面建立的优先规则；搜索路径将遍历其余配置注册表，直到找到为止。(译注：因为Viper支持从多种配置来源，例如磁盘上的配置文件>命令行标志位>环境变量>远程Key/Value存储>默认值，我们在查找一个配置的时候如果在当前配置源中没找到，就会继续从后续的配置源查找，直到找到为止。)

例如，在给定此配置文件的情况下，`datastore.metric.host`和`datastore.metric.port`均已定义（并且可以被覆盖）。如果另外在默认值中定义了`datastore.metric.protocol`，Viper也会找到它。

然而，如果`datastore.metric`被直接赋值覆盖（被flag，环境变量，`set()`方法等等…），那么`datastore.metric`的所有子键都将变为未定义状态，它们被高优先级配置级别“遮蔽”（shadowed）了。

最后，如果存在与分隔的键路径匹配的键，则返回其值。例如：

```go
{
    "datastore.metric.host": "0.0.0.0",
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}

GetString("datastore.metric.host") // 返回 "0.0.0.0"
```

##### 提取子树

从Viper中提取子树。

例如，`viper`实例现在代表了以下配置：

```yaml
app:
  cache1:
    max-items: 100
    item-size: 64
  cache2:
    max-items: 200
    item-size: 80
```

执行后：

```go
subv := viper.Sub("app.cache1")
```

`subv`现在就代表：

```yaml
max-items: 100
item-size: 64
```

##### 反序列化

可以选择将所有或特定的值解析到结构体、map等

有两种方法可以做到这一点：

- `Unmarshal(rawVal interface{}) : error`
- `UnmarshalKey(key string, rawVal interface{}) : error`

举个例子：

```go
type config struct {
    Port int
    Name string
    PathMap string `mapstructure:"path_map"`
}

var C config

err := viper.Unmarshal(&C)
if err != nil {
	t.Fatalf("unable to decode into struct, %v", err)
}
```

如果想要解析的键本身就包含`.`(默认的键分隔符）的配置，则需要修改分隔符：

```go
v := viper.NewWithOptions(viper.KeyDelimiter("::"))
v.SetDefault("chart::values",map[string]interface{}{
        "ingress": map[string]interface{}{
        "annotations": map[string]interface{}{
            "traefik.frontend.rule.type":                 "PathPrefix",
            "traefik.ingress.kubernetes.io/ssl-redirect": "true",
        },
    },
})

type config struct {
    Chart struct {
        Values map[string]interface{}
    }
}

var C config

v.Unmarshal(&C)
```

##### 序列化成字符串

可能需要将viper中保存的所有设置序列化到一个字符串而不是写入到一个文件中，可以将序列化器与`AllSettings()`返回的配置一起使用

```go
import (
	yaml "gopkg.in/yaml.v2"
)

func yamlStringSettings() string {
    c := viper.AllSettings()
    bs,err := yaml.Marshal(c)
    if err != nil {
        log.Fatalf("unable to marshal config to YAML: %v", err)
    }
    return string(bs)
}
```

##### viper示例

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/spf13/viper"
)

func main() {
    viper.SetConfigFile("./conf/config.yaml")	//指定配置文件路径
    err := viper.ReadInConfig()			//读取配置信息
    if err != nil {                    // 读取配置信息失败
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}
    //监控配置文件变化
    viper.WatchConfig()
    
    r := gin.Default
    // 访问/version的返回值会随配置文件的变化而变化
    r.GET("/version",func(c *gin.Context){
        c.String(http.StatusOK,viper.GetString("version"))
    })
    if err := r.Run(
		fmt.Sprintf(":%d", viper.GetInt("port"))); err != nil {
		panic(err)
	}
}
```

##### 使用结构体变量保存配置信息

`viper`加载完配置信息后使用结构体变量保存配置信息

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/fsnotify/fsnotify"

	"github.com/gin-gonic/gin"
	"github.com/spf13/viper"
)

type Config struct {
	Port    int    `mapstructure:"port"`
	Version string `mapstructure:"version"`
}

var Conf = new(Config)

func main() {
    viper.SetConfigFile("./conf/config.yaml")	//指定配置文件路径
    err := viper.ReadInConfig()					//读取配置信息
    if err != nil {                           // 读取配置信息失败
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}
    //将读取的配置信息保存至全局变量Conf
    if err := viper.Unmarshal(Conf); err != nil {
		panic(fmt.Errorf("unmarshal conf failed, err:%s \n", err))
	}
    //监控配置文件变化
    viper.WatchConfig()
    //配置文件变化后同步到全局变量
    viper.OnConfigChange(func(in fsnotify.Event){
        if err := viper.Unmarshal(Conf); err != nil {
			panic(fmt.Errorf("unmarshal conf failed, err:%s \n", err))
		}
    })
    
    r := gin.Default()
    // 访问/version的返回值会随配置文件的变化而变化
    r.GET("/version", func(c *gin.Context) {
		c.String(http.StatusOK, Conf.Version)
	})

	if err := r.Run(fmt.Sprintf(":%d", Conf.Port)); err != nil{
		panic(err)
	}
}
```

#### 获取系统性能数据gopsutil库

Go语言部署简单、性能好的特点非常适合做一些诸如采集系统信息和监控的服务，[gopsutil](https://github.com/shirou/gopsutil)库是知名Python库：[psutil](https://github.com/giampaolo/psutil)的一个Go语言版本的实现。

`安装`

```
go get github.com/shirou/gopsutil
```

##### 采集CPU相关信息

```go
import "github.com/shirou/gopsutil/cpu"

func getCpuInfo() {
    cpuInfos,err := cpu.Info()
    if err != nil {
		fmt.Printf("get cpu info failed, err:%v", err)
	}
    for _,ci := range cpuInfos {
        fmt.Println(ci)
    }
    //cpu使用率
    for {
        percent,_ := cpu.Percent(time.second,false)
        fmt.Printf("cpu percent:%v\n", percent)
    }
}
```

##### 采集CPU负载信息

```go
import "github.com/shirou/gopsutil/load"

func getCpuLoad() {
	info, _ := load.Avg()
	fmt.Printf("%v\n", info)
}
```

##### Memory

```go
import "github.com/shirou/gopsutil/mem"

// mem info
func getMemInfo() {
	memInfo, _ := mem.VirtualMemory()
	fmt.Printf("mem info:%v\n", memInfo)
}
```

##### Host

```go
import "github.com/shirou/gopsutil/host"

// host info
func getHostInfo() {
	hInfo, _ := host.Info()
	fmt.Printf("host info:%v uptime:%v boottime:%v\n", hInfo, hInfo.Uptime, hInfo.BootTime)
}
```

##### Disk

```go
import "github.com/shirou/gopsutil/disk"

func getDiskInfo(){
    parts,err := disk.Partitions(true)
    if err != nil {
		fmt.Printf("get Partitions failed, err:%v\n", err)
		return
	}
    for _,part := range parts {
        fmt.Printf("part:%v\n",part.String())
        diskInfo,_ := disk.Usage(part.Mountpoint)
        fmt.Printf("disk info:used:%v free:%v\n",diskInfo.UsedPercent,diskInfo.Free)
    }
    ioStat,_ := disk.IOCounters()
    for k,v := range ioStat {
        fmt.Printf("%v:%v\n", k, v)
    }
}
```

##### net IO

```go
import "github.com/shirou/gopsutil/net"

func getNetInfo() {
	info, _ := net.IOCounters(true)
	for index, v := range info {
		fmt.Printf("%v:%v send:%v recv:%v\n", index, v, v.BytesSent, v.BytesRecv)
	}
}
```

##### 获取本机IP的两种方式

```go
func GetLocalIP() (ip string,err error) {
    addr,err := net.InterfaceAddrs()
    if err != nil {
		return
	}
    for _,addr := range addrs {
        ipAddr,ok := addr.(*net.IPNet)
        if !ok {
			continue
		}
        if ipAddr.IP.IsLoopback() {
			continue
		}
		if !ipAddr.IP.IsGlobalUnicast() {
			continue
		}
        return ipAddr.IP.string(),nil
    }
    return
}
```

```go
func GetOutboundIP() string {
    conn,err := net.Dial("udp","8.8.8.8:80")
    if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
    
    localAddr := conn.LocalAddr().(*net.UDPAddr)
    fmt.Println(localAddr.String())
	return localAddr.IP.String()
}
```

