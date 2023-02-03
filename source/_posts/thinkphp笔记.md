---
title: thinkphp笔记
date: 2023-02-03 13:29:44
tags: "thinkphp"
categories: "php"
---

### 架构

#### URL设计

ThinkPHP`5.0`在没有启用路由的情况下典型的URL访问规则是：

```
http://serverName/index.php（或者其它应用入口文件）/模块/控制器/操作/[参数名/参数值...]
```

命令行模式下面的访问规则是：

```
php.exe index.php(或者其它应用入口文件） 模块/控制器/操作/[参数名/参数值...]
```

注意：`5.0`取消了URL模式的概念，并且**普通模式的URL访问不再支持**，但参数可以支持普通方式传值，例如：

```
php.exe index.php(或者其它应用入口文件） 模块/控制器/操作?参数名=参数值&...
```

如果不支持PATHINFO的服务器可以使用兼容模式访问如下：

```
http://serverName/index.php（或者其它应用入口文件）?s=/模块/控制器/操作/[参数名/参数值...]
```

**默认情况下，`URL`不区分大小写**

#### 目录结构

标准应用和模块目录结构如下：

```
├─application           应用目录（可设置）
│  ├─common             公共模块目录（可选）
│  ├─common.php         公共函数文件
│  ├─route.php          路由配置文件
│  ├─database.php       数据库配置文件
│  ├─config.php         应用配置文件
│  ├─module1            模块1目录
│  │  ├─config.php      模块配置文件
│  │  ├─common.php      模块函数文件
│  │  ├─controller      控制器目录
│  │  ├─model           模型目录（可选）
│  │  ├─view            视图目录（可选）
│  │  └─ ...            更多类库目录
│  │ 
│  ├─module2            模块2目录
│  │  ├─config.php      模块配置文件
│  │  ├─common.php      模块函数文件
│  │  ├─controller      控制器目录
│  │  ├─model           模型目录（可选）
│  │  ├─view            视图目录（可选）
│  │  └─ ...            更多类库目录
```

`common`模块是一个特殊的模块，默认是禁止直接访问的，一般用于放置一些公共的类库用于其他模块的继承。

#### 模块类库

一个模块下面的类库文件的命名空间统一以`app\模块名`开头，例如：

```
// index模块的Index控制器类
app\index\controller\Index
// index模块的User模型类
app\index\model\User
```

其中`app`可以通过定义的方式更改，例如我们在应用配置文件中修改：

```php
'app_namespace' => 'application',
```

那么，index模块的类库命名空间则变成：

```
// index模块的Index控制器类
application\index\controller\Index
// index模块的User模型类
application\index\model\User
```

#### 模块和控制器隐藏

默认采用多模块的支持，多个模块的情况下必须在URL地址中标识当前模块，如果只有一个模块的话，可以进行模块绑定，在应用的入口文件中添加如下代码：

```php
// 绑定当前访问到index模块
define('BIND_MODULE','index');
```

绑定后，我们的URL访问地址则变成：

```
http://serverName/index.php/控制器/操作/[参数名/参数值...]
```

若模块和控制器都只有一个，则可以在应用公共文件中绑定模块和控制器：

```php
// 绑定当前访问到index模块的index控制器
define('BIND_MODULE','index/index');
```

设置后，我们的URL访问地址则变成：

```
http://serverName/index.php/操作/[参数名/参数值...]
```

访问的模块是`index`模块，控制器是`Index`控制器。

若应用只有单一模块，则先在应用配置文件中定义：

```php
// 关闭多模块设计
'app_multi_module'  =>  false,
```

然后，调整应用目录的结构为如下：

```
├─application        应用目录（可设置）
│  ├─controller      控制器目录
│  ├─model           模型目录
│  ├─view            视图目录
│  ├─ ...            更多类库目录
│  ├─common.php      函数文件
│  ├─route.php       路由配置文件
│  ├─database.php    数据库配置文件
│  └─config.php      配置文件
```

同时，单一模块设计下的应用类库的命名空间也有所调整:

```
app\index\controller\Index		==>		app\controller\Index
app\index\model\User			==>		app\model\User
```

#### 根命名空间（类库包）

以`\think\cache\driver\File`类为例，`think`即根命名空间，对应初始命名空间目thinkphp/library/think`，可以理解为一个根命名空间对应了一个类库包。系统内置的几个根命名空间（类库包）如下：

| 名称   | 描述          | 类库目录                |
| :----- | :------------ | :---------------------- |
| think  | 系统核心类库  | thinkphp/library/think  |
| traits | 系统Trait类库 | thinkphp/library/traits |
| app    | 应用类库      | application             |

#### 数据输出

输出采用`Response`类统一处理，而不是直接在控制器中进行输出，通过设置`default_return_type`或者动态设置不同类型的`Response`输出就可以自动进行数据转换处理：

```php
'default_return_type' => 'json'
```

还可以在控制器中明确指定输出类型的方式：

```php
namespace app\index\controller;

class Index 
{
    public function index()
    {
        $data = ['name'=>'thinkphp','url'=>'thinkphp.cn'];
        // 指定json数据输出
        return json(['data'=>$data,'code'=>1,'message'=>'操作完成']);
    }
}
```

核心支持的数据类型包括`view`、`xml`、`json`和`jsonp`，其他类型需要扩展。

### 配置

#### 配置目录

系统默认的配置文件目录就是应用目录，也就是默认的`application`下面，并分为应用配置（整个应用有效）和模块配置（仅针对该模块有效）。

```
├─application         应用目录
│  ├─config.php       应用配置文件
│  ├─database.php     数据库配置文件
│  ├─route.php        路由配置文件
│  ├─index            index模块配置文件目录
│  │  ├─config.php    index模块配置文件
│  │  └─database.php  index模块数据库配置文件
```

若不希望配置文件放到应用目录下面，可以在入口文件中定义独立的配置目录，添加`CONF_PATH`常量定义即可：

```php
// 定义配置文件目录和应用目录同级
define('CONF_PATH', __DIR__.'/../config/');
```

```
├─application         应用目录
├─config              配置目录
│  ├─config.php       应用配置文件
│  ├─database.php     数据库配置文件
│  ├─route.php        路由配置文件
│  ├─index            index模块配置文件目录
│  │  ├─config.php    index模块配置文件
│  │  └─database.php  index模块数据库配置文件
```

#### 配置格式

默认方式为php数组方式定义配置文件，可以在入口文件定义`CONF_EXT`常量更改为其它的配置类型：

```php
// 更改配置格式为ini格式
define('CONF_EXT', '.ini');
```

#### 配置加载

在thinkphp中应用配置文件的加载顺序为：

```
惯例配置->应用配置->扩展配置->场景配置->模块配置->动态配置
```

因后面的配置会覆盖之前的同名配置（在没有生效的前提下），故配置的优先顺序为从右到左

##### 惯例配置

框架内置一个惯例配置文件（`thinkphp/convention.php`）

##### 应用配置

应用初始化时首先加载的公共配置文件，默认在`application/config.php`

##### 扩展配置

扩展配置文件由`extra_config_list`配置参数定义的额外的配置文件，默认加载`database`和`validate`两个扩展配置文件

<font color=red>`V5.0.1`开始，取消了该配置参数，扩展配置文件直接放入`application/extra`目录会自动加载。</font>

##### 场景配置

举个例子，在公司和家里分别设置不同的数据库测试环境。那么可以这样处理，在公司环境中，在应用配置文件中配置：

```php
'app_status'=>'office'
```

则自动加载该状态对应的配置文件（默认位于`application/office.php`）

<font color=red>场景配置文件和应用配置文件`config.php`是一样的定义</font>

当回家后修改定义为：

```php
'app_status'=>'home'
```

则自动加载该状态对应的配置文件（位于`application/home.php`）

##### 模块配置

每个模块会自动加载自己的配置文件（位于`application/当前模块名/config.php`）

模块还可以支持独立的状态配置文件，命名规范为：`application/当前模块名/应用状态.php`

如果应用配置文件比较大，想分成几个单独的配置文件或需要加载额外的配置文件的话，可以考虑采用扩展配置或动态配置

<font size=5>加载配置文件</font>

```php
Config::load('配置文件名');
```

配置文件一般位于`APP_PATH`目录下面，若需要加载其它位置的配置文件，需要使用完整路径，例如：

```php
Config::load(APP_PATH.'config/config.php');
```

`parse`方法除了支持读取配置文件外，也支持直接传入配置内容，如：

```php
$config = 'var1=val
var2=val';
Config::parse($config,'ini');
```

#### 读取配置参数

设置完配置参数后就可以使用get方法或助手函数`config`读取配置：

```php
echo Config::get('配置参数');	或	echo config('配置参数');
```

读取所有配置参数：

```php
dump(Config::get());	或	dump(config());
```

判断是否存在某个设置参数：

```php
Config::has('配置参数');	或	config('?配置参数');
```

读取二级配置：

```php
echo Config::get('配置参数.二级参数');
echo config('配置参数.二级参数');
```

#### 动态配置参数

使用`set`方法动态设置参数，如：

```php
Config::set('配置参数','配置值'); 或	config('配置参数','配置值');
```

```php
Config::set([
    '配置参数1'=>'配置值',
    '配置参数2'=>'配置值'
]);
// 或者使用助手函数
config([
    '配置参数1'=>'配置值',
    '配置参数2'=>'配置值'
]);
```

#### 独立配置文件

将扩展配置文件放入`application/extra`目录，即可自动读取扩展配置文件

<font color=#5bc0de>自动读取的配置文件都是二级配置参数，一级配置名称就是扩展配置的文件名</font>

### 路由

#### 路由定义

##### 注册路由规则

路由定义采用`\think\Route`类的`rule`方法注册，通常在应用的路由配置文件`application/route.php`进行注册，格式是：<font color=#5bc0de>Router::rule('路由表达式','路由地址','请求类型','路由参数(数组)','变量规则(数组)');</font>

```php
use think\Router;
//注册路由到index模块的News控制器的read操作
Router::rule('new/:id','index/News/read');//可以在第三个参数指定请求类型
```

当我们访问：

```
http://serverName/new/5
```

会自动路由到：

```
http://serverName/index/news/read/id/5
```

系统提供了为不同的请求类型定义路由规则的简化方法，例如：

```
Route::get('new/:id','News/read'); // 定义GET请求路由规则
Route::post('new/:id','News/update'); // 定义POST请求路由规则
Route::put('new/:id','News/update'); // 定义PUT请求路由规则
Route::delete('new/:id','News/delete'); // 定义DELETE请求路由规则
Route::any('new/:id','News/read'); // 所有请求都支持的路由规则
```

如果要定义get和post请求支持的路由规则，也可以用：

```
Route::rule('new/:id','News/read','GET|POST');
```

也可以批量注册路由规则，例如：

```
Route::rule(['new/:id'=>'News/read','blog/:name'=>'Blog/detail']);
Route::get(['new/:id'=>'News/read','blog/:name'=>'Blog/detail']);
Route::post(['new/:id'=>'News/update','blog/:name'=>'Blog/detail']);
```

若希望所有的路由定义都是完全匹配的话，可以直接配置

```php
// 开启路由定义的全局完全匹配
'route_complete_match'  =>  true,
```

若个别路由不需要使用完整匹配，可以添加路由参数覆盖定义：

```php
Route::rule('new/:id','News/read','GET|POST',['complete_match' => false]);
```

默认情况下，只会加载一个路由配置文件`route.php`，若需要定义多个路由文件，可以修改`route_config_file`配置参数，如：

```php
// 定义路由配置文件（数组）
'route_config_file' =>  ['route', 'route1', 'route2'],
```

#### 路由参数

设置路由匹配的条件参数，主要用于验证当前的路由规则是否有效，主要包括：

| 参数             | 说明                              |
| :--------------- | :-------------------------------- |
| method           | 请求类型检测，支持多个请求类型    |
| ext              | URL后缀检测，支持匹配多个后缀     |
| deny_ext         | URL禁止后缀检测，支持匹配多个后缀 |
| https            | 检测是否https请求                 |
| domain           | 域名检测                          |
| before_behavior  | 前置行为（检测）                  |
| after_behavior   | 后置行为（执行）                  |
| callback         | 自定义检测方法                    |
| merge_extra_vars | 合并额外参数                      |
| bind_model       | 绑定模型（`V5.0.1+`）             |
| cache            | 请求缓存（`V5.0.1+`）             |
| param_depr       | 路由参数分隔符（`V5.0.2+`）       |
| ajax             | Ajax检测（`V5.0.2+`）             |
| pjax             | Pjax检测（`V5.0.2+`）             |

#### 资源路由

设置`RESTFUL`请求资源方式如下：

```php
Router::resource('blog','index/blog');
```

或者在路由配置文件中使用`__rest__`添加资源路由定义：

```php
return [
    //定义资源路由
    '__rest__'=>[
        //指向index模块的blog控制器
        'blog'=>'index/blog',
    ],
    //定义普通路由
    'hello/:id'=>'index/hello',
]
```

设置后会自动注册7个路由规则，如下：

| 标识   | 请求类型 | 生成路由规则    | 对应操作方法（默认） |
| :----- | :------- | :-------------- | :------------------- |
| index  | GET      | `blog`          | index                |
| create | GET      | `blog/create`   | create               |
| save   | POST     | `blog`          | save                 |
| read   | GET      | `blog/:id`      | read                 |
| edit   | GET      | `blog/:id/edit` | edit                 |
| update | PUT      | `blog/:id`      | update               |
| delete | DELETE   | `blog/:id`      | delete               |

上面的设置，会对应index模块的blog控制器，对应方法如下：

```php
namespace app\index\controller;
class Blog {
    public function index(){
    }
    
    public function read($id){
    }    
    
    public function edit($id){
    }    
}
```

可以改变默认的id参数名，例如：

```php
Route::resource('blog','index/blog',['var'=>['blog'=>'blog_id']]);
```

因此blog控制器对应方法调整为：

```php
namespace app\index\controller;
class Blog {
    public function index(){
    }
    
    public function read($blog_id){
    }    
    
    public function edit($blog_id){
    }    
}
```

可以在定义资源路由的时候限定执行的方法（标识），例如：

```php
// 只允许index操作
Router::resource('blog','index/blog',['only'=>['index']]);
//排除index操作
Router::resource('blog','index/blog',['except'=>['index']]);
```

资源路由的标识不可更改，但生成的路由规则和对应操作方法可以修改：

```php
Route::rest('create',['GET', '/add','add']);
```

设置之后，URL访问变为：

```
http://serverName/blog/create
变成
http://serverName/blog/add
```

创建blog页面的对应的操作方法也变成了add。

支持批量更改，如下：

```php
Route::rest([
    'save'   => ['POST', '', 'store'],
    'update' => ['PUT', '/:id', 'save'],
    'delete' => ['DELETE', '/:id', 'destory'],
]);
```

#### 路由分组

将相同路由合并分组，提高路由匹配的效率，不必每次都遍历完整的路由规则。

```php
'blog/:id' => ['Blog/read',['method'=>'get'],['id'=>'\d+']],
'blog/:name' => ['Blog/read', ['method'=>'post']],
```

```php
'[blog]' => [
    ':id' => ['Blog/read',['method'=>'get'],['id'=>'\d+']],
    ':name' => ['Blog/read', ['method'=>'post']],
],
```

可以使用`Route`类的`group`方法进行注册，如下：

```php
Route::group('blog',[
    ':id' => ['Blog/read',['method'=>'get'],['id'=>'\d+']],
    ':name' => ['Blog/read',['method'=>'post']],
]);
```

还可以给分组路由定义一些公用的路由设置参数，例如：

```php
Route::group('blog',[
    ':id' => ['Blog/read',[],['id =>'\d+']],
    ':name' => ['Blog/read', []],
],['method'=>'get','ext'=>'html']);
```

支持用闭包的方式注册路由分组：

```php
Route::group('blog',function(){
    Route::any(':id','blog/read',[],['id'=>'\d+']);
    Route::any(':name','blog/read',[],['name'=>'\w+']);
},['method'=>'get','ext'=>'html']);
```

#### MISS路由

若希望在没有匹配到所有路由规则后执行一条设定的路由，可以使用`MISS`，只需要在路由配置文件中定义：

```php
return [
    'new/:id' => 'News/read',
    'blog/:id' => ['Blog/update',['method' => 'post|put'], ['id' => '\d+']],
    '__miss__'  => 'public/miss',
];
```

或者使用`miss`方法注册路由

```php
Route::miss('public/miss')
```

分组支持独立的`MISS`路由，例如如下定义：

```php
return [
    '[blog]' =>  [
        'edit/:id'  => ['Blog/edit',['method' => 'get'], ['id' => '\d+']],
        ':id'       => ['Blog/read',['method' => 'get'], ['id' => '\d+']],
        '__miss__'  => 'blog/miss',
    ],
    'new/:id'   => 'News/read',
    '__miss__'  => 'public/miss',
];
```

#### 闭包支持

可以使用闭包的方式定义一些特殊需求的路由，不去执行控制器的操作方法：

```php
Route::get('hello/:name',function($name){
   return 'Hello,'.$name 
});
```

当访问URL地址为：

```
http://serverName/hello/thinkphp
```

则浏览器输出的结果是：

```
Hello,thinkphp
```

#### 域名路由

定义域名部署规则支持两种方式：动态注册和配置定义

##### 动态注册

可以在应用的公共文件或配置文件中动态注册域名部署规则，如：

```php
// blog子域名绑定到blog模块
Route::domain('blog','blog');
// 完整域名绑定到admin模块
Route::domain('admin.thinkphp.cn','admin');
// IP绑定到admin模块
Route::domain('114.23.4.5','admin');
```

blog子域名绑定后，URL访问规则变成：

```
// 原来的URL访问
http://www.thinkphp.cn/blog/article/read/id/5
// 绑定到blog子域名访问
http://blog.thinkphp.cn/article/read/id/5
```

### 控制器