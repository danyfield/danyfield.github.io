---
title: TypeScript笔记
date: 2023-02-23 14:18:03
tags: "typescript"
categories: "前端"
---

### 基础语法

#### 面向对象实例

```typescript
class Site {
    name():void {
        console.log("Runoob")
    }
}
var obj = new Site();
obj.name();
```

上述定义了一个含有name()方法的类Site，编译后生成的JavaScript代码如下：

```js
var Site = /** @class */ (function() {
    function Site(){}
    Site.prototype.name = function () {
        console.log("Runoob");
    };
    return Site;
}());
var obj = new Site();
obj.name();
```

#### 注意事项

- 空白和换行：忽略程序中出现的空格、制表符和换行符；空格和制表符通常用于缩进代码
- 区分大小写
- 分号可选：每行指令都是一段语句，建议使用分号；若语句在同行一定要使用分号
- 注释：单行注释`//`；多行注释`/**/`
- 编译：使用`tsc 文件名`编译

### 基础类型

| 数据类型   | 关键字    | 描述                                                         |
| ---------- | --------- | ------------------------------------------------------------ |
| 任意类型   | any       | 可以赋予任意类型的值；且会动态改变                           |
| 数字类型   | number    | 双精度64位浮点值，用于表示整数和分数                         |
| 字符串类型 | string    | 可使用`'`或`"`表示，`定义多行文本和内嵌表达式                |
| 布尔类型   | boolean   | 逻辑值：true和false                                          |
| 数组类型   | 无        |                                                              |
| 元组       | 无        | 用于表示已知元素数量和类型的数组，各元素类型不必相同，对应位置的类型需相同 |
| 枚举       | enum      | 用于定义数值集合                                             |
| void       | void      | 用于标识方法返回值类型，表示方法无返回值                     |
| null       | null      | 表示对象缺失                                                 |
| undefined  | undefined | 初始化变量为一个未定义的值                                   |
| never      | never     | 其它类型的子类型，代表从不会出现的值                         |

TypeScript 和 JavaScript 没有整数类型

```typescript
let decLiteral: number = 6;
let words: string = `您好，今年是 ${ name } 发布 ${ years + 1} 周年`;
let flag: boolean = true;
let arr: number[] = [1, 2];		//或let arr: Array<number> = [1, 2];
let x: [string,number];		x = ['Runoob',1];
enum Color {Red,Green,Blue};	let c: Color = Color.Blue;
function hello():void{alert("Hello Runoob");}
```

### 函数

#### 可选参数和默认参数

##### 可选参数

可选参数使用问号标识，可选参数必须在必须参数后面：

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
 
let result1 = buildName("Bob");  // 正确
let result2 = buildName("Bob", "Adams", "Sr.");  // 错误，参数太多了
let result3 = buildName("Bob", "Adams");  // 正确
```

##### 默认参数

可以设置参数的默认值，参数不能同时设置为：

```typescript
function function_name(param1[:type],param2[:type]=default_value){}
```

```typescript
function calculate_discount(price:number,rate:number=0.50){
    var discount = price * rate;
    console.log("计算结果：",discount);
}
calculate_discount(1000)
calculate_discount(1000,0.30)
```

编译以上代码，可以得到以下JavaScript代码：

```javascript
function calculate_discount(price,rate){
    if (rate === void 0) { rate = 0.50; }
    var discount = price * rate;
    console.log("计算结果：",discount);
}
calculate_discount(1000);
calculate_discount(1000, 0.30);
```

### Number

#### 方法

Number对象 支持以下方法：

|                         方法 & 描述                          |                             实例                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|         `toExponential()`把对象的值转换为指数计数法          | `var num1 = 1225.30  var val = num1.toExponential();  console.log(val) // 输出： 1.2253e+3` |
|       `toFixed()`数字转换为字符串，并对小数点指定位数        | `var num3 = 177.234  console.log("num3.toFixed() 为 "+num3.toFixed())    // 输出：177 console.log("num3.toFixed(2) 为 "+num3.toFixed(2))  // 输出：177.23 console.log("num3.toFixed(6) 为 "+num3.toFixed(6))  // 输出：177.234000` |
|   `toLocaleString()`数字转换为字符串，使用本地数字格式顺序   | `var num = new Number(177.1234);  console.log( num.toLocaleString());  // 输出：177.1234` |
|           toPrecision()把数字格式化为指定的长度。            | `var num = new Number(7.123456);  console.log(num.toPrecision());  // 输出：7.123456  console.log(num.toPrecision(1)); // 输出：7 console.log(num.toPrecision(2)); // 输出：7.1` |
| toString()把数字转换为字符串，使用指定的基数。数字的基数是 2 ~ 36 之间的整数。若省略该参数，则使用基数 10。 | `var num = new Number(10);  console.log(num.toString());  // 输出10进制：10 console.log(num.toString(2)); // 输出2进制：1010 console.log(num.toString(8)); // 输出8进制：12` |
|         valueOf()返回一个 Number 对象的原始数字值。          | `var num = new Number(10);  console.log(num.valueOf()); // 输出：10` |

### String

#### string对象属性

|              属性 & 描述              |                             实例                             |
| :-----------------------------------: | :----------------------------------------------------------: |
| `constructor`对创建该对象的函数的引用 | `var str = new String( "This is string" );  console.log("str.constructor is:" + str.constructor)`输出结果：`str.constructor is:function String() { [native code] }` |
|       `length`返回字符串的长度        | `var uname = new String("Hello World")  console.log("Length "+uname.length)  // 输出 11` |
|  `prototype`允许向对象添加属性和方法  | `function employee(id:number,name:string) {      this.id = id      this.name = name   }   var emp = new employee(123,"admin")   employee.prototype.email="admin@runoob.com" // 添加属性 email  console.log("员工号: "+emp.id)   console.log("员工姓名: "+emp.name)   console.log("员工邮箱: "+emp.email)` |

### String 方法

下表列出了 String 对象支持的方法：

|                         方法 & 描述                          |                             实例                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                `charAt()`返回在指定位置的字符                | `var str = new String("RUNOOB");  console.log("str.charAt(0) 为:" + str.charAt(0)); // R console.log("str.charAt(1) 为:" + str.charAt(1)); // U  console.log("str.charAt(2) 为:" + str.charAt(2)); // N  console.log("str.charAt(3) 为:" + str.charAt(3)); // O  console.log("str.charAt(4) 为:" + str.charAt(4)); // O  console.log("str.charAt(5) 为:" + str.charAt(5)); // B` |
|       `concat()`连接两个或更多字符串，并返回新的字符串       | `var str1 = new String( "RUNOOB" );  var str2 = new String( "GOOGLE" );  var str3 = str1.concat( str2 );  console.log("str1 + str2 : "+str3) // RUNOOBGOOGLE` |
|        `indexOf()`返回某个指定字符串值首次出现的位置         | `var str1 = new String( "RUNOOB" );   var index = str1.indexOf( "OO" );  console.log("查找的字符串位置 :" + index );  // 3` |
| `lastIndexOf()`从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置 | `var str1 = new String( "This is string one and again string" );  var index = str1.lastIndexOf( "string" ); console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 29      index = str1.lastIndexOf( "one" );  console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 15` |
|         `match()`查找找到一个或多个正则表达式的匹配          | `var str="The rain in SPAIN stays mainly in the plain";  var n=str.match(/ain/g);  // ain,ain,ain` |
|            `replace()`替换与正则表达式匹配的子串             | `var re = /(\w+)\s(\w+)/;  var str = "zara ali";  var newstr = str.replace(re, "$2, $1");  console.log(newstr); // ali, zara` |
|             `search()`检索与正则表达式相匹配的值             | `var re = /apples/gi;  var str = "Apples are round, and apples are juicy."; if (str.search(re) == -1 ) {     console.log("Does not contain Apples" );  } else {     console.log("Contains Apples" );  } ` |
|  slice()提取字符串的片断，并在新的字符串中返回被提取的部分   |                                                              |
|             `split()`把字符串分割为子字符串数组              | `var str = "Apples are round, and apples are juicy.";  var splitted = str.split(" ", 3);  console.log(splitted)  // [ 'Apples', 'are', 'round,' ]` |
|       substr()从起始索引号提取字符串中指定数目的字符。       |                                                              |
|     `substring()`提取字符串中两个指定的索引号之间的字符      | `var str = "RUNOOB GOOGLE TAOBAO FACEBOOK";  console.log("(1,2): "    + str.substring(1,2));   // U console.log("(0,10): "   + str.substring(0, 10)); // RUNOOB GOO console.log("(5): "      + str.substring(5));     // B GOOGLE TAOBAO FACEBOOK` |
|              `toLowerCase()`把字符串转换为小写               | `var str = "Runoob Google";  console.log(str.toLowerCase( ));  // runoob google` |
|                    `toString()`返回字符串                    | `var str = "Runoob";  console.log(str.toString( )); // Runoob` |
|              `toUpperCase()`把字符串转换为大写               | `var str = "Runoob Google";  console.log(str.toUpperCase( ));  // RUNOOB GOOGLE` |
|            `valueOf()`返回指定字符串对象的原始值             | `var str = new String("Runoob");  console.log(str.valueOf( ));  // Runoob` |

### 数组

#### 数组方法

|                         方法 & 描述                          |                             实例                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|          `concat()`连接两个或更多的数组，并返回结果          | `var alpha = ["a", "b", "c"];  var numeric = [1, 2, 3];  var alphaNumeric = alpha.concat(numeric);  console.log("alphaNumeric : " + alphaNumeric );    // a,b,c,1,2,3   ` |
|        `every()`检测数值元素的每个元素是否都符合条件         | `function isBigEnough(element, index, array) {          return (element >= 10);  }           var passed = [12, 5, 8, 130, 44].every(isBigEnough);  console.log("Test Value : " + passed ); // false` |
|             `filter()`检测并返回符合条件元素数组             | `function isBigEnough(element, index, array) {     return (element >= 10);  }             var passed = [12, 5, 8, 130, 44].filter(isBigEnough);  console.log("Test Value : " + passed ); // 12,130,44` |
|            `forEach()`每个元素都执行一次回调函数             | `let num = [7, 8, 9]; num.forEach(function (value) {     console.log(value); }); `编译成 JavaScript 代码：`var num = [7, 8, 9]; num.forEach(function (value) {     console.log(value);  // 7   8   9 });` |
| `indexOf()`搜索数组中的元素，并返回它所在的位置。如果搜索不到，返回值 -1 | `var index = [12, 5, 8, 130, 44].indexOf(8);  console.log("index is : " + index );  // 2` |
|             `join()`把数组所有元素放入一个字符串             | `var arr = new Array("Google","Runoob","Taobao");             var str = arr.join();  console.log("str : " + str );  // Google,Runoob,Taobao            var str = arr.join(", ");  console.log("str : " + str );  // Google, Runoob, Taobao            var str = arr.join(" + ");  console.log("str : " + str );  // Google + Runoob + Taobao` |
|     `lastIndexOf()`返回一个指定的字符串值最后出现的位置      | `var index = [12, 5, 8, 130, 44].lastIndexOf(8);  console.log("index is : " + index );  // 2` |
|  `map()`通过指定函数处理数组的每个元素，并返回处理后的数组   | `var numbers = [1, 4, 9];  var roots = numbers.map(Math.sqrt);  console.log("roots is : " + roots );  // 1,2,3` |
|       `pop()`删除数组的最后一个元素并返回删除的元素。        | `var numbers = [1, 4, 9];             var element = numbers.pop();  console.log("element is : " + element );  // 9            var element = numbers.pop();  console.log("element is : " + element );  // 4` |
|       `push()`向数组末尾添加一个或多元素，返回新的长度       | `var numbers = new Array(1, 4, 9);  var length = numbers.push(10);  console.log("new numbers is : " + numbers );  // 1,4,9,10  length = numbers.push(20);  console.log("new numbers is : " + numbers );  // 1,4,9,10,20` |
|         `reduce()`将数组元素计算为一个值（从左到右）         | `var total = [0, 1, 2, 3].reduce(function(a, b){ return a + b; });  console.log("total is : " + total );  // 6` |
|      `reduceRight()`将数组元素计算为一个值（从右到左）       | `var total = [0, 1, 2, 3].reduceRight(function(a, b){ return a + b; });  console.log("total is : " + total );  // 6` |
|                `reverse()`反转数组的元素顺序                 | `var arr = [0, 1, 2, 3].reverse();  console.log("Reversed array is : " + arr );  // 3,2,1,0` |
|              `shift()`删除并返回数组第一个元素               | `var arr = [10, 1, 2, 3].shift();  console.log("Shifted value is : " + arr );  // 10` |
|        `slice()`选取数组的的一部分，并返回一个新数组         | `var arr = ["orange", "mango", "banana", "sugar", "tea"];  console.log("arr.slice( 1, 2) : " + arr.slice( 1, 2) );  // mango console.log("arr.slice( 1, 3) : " + arr.slice( 1, 3) );  // mango,banana` |
|         `some()`检测数组元素中是否有元素符合指定条件         | `function isBigEnough(element, index, array) {     return (element >= 10);             }             var retval = [2, 5, 8, 1, 4].some(isBigEnough); console.log("Returned value is : " + retval );  // false            var retval = [12, 5, 8, 1, 4].some(isBigEnough);  console.log("Returned value is : " + retval );  // true` |
|                 `sort()`对数组的元素进行排序                 | `var arr = new Array("orange", "mango", "banana", "sugar");  var sorted = arr.sort();  console.log("Returned string is : " + sorted );  // banana,mango,orange,sugar` |
|               `splice()`从数组中添加或删除元素               | `var arr = ["orange", "mango", "banana", "sugar", "tea"];   var removed = arr.splice(2, 0, "water");   console.log("After adding 1: " + arr );    // orange,mango,water,banana,sugar,tea  console.log("removed is: " + removed);             removed = arr.splice(3, 1);   console.log("After removing 1: " + arr );  // orange,mango,water,sugar,tea  console.log("removed is: " + removed);  // banana` |
|          `toString()`把数组转换为字符串，并返回结果          | `var arr = new Array("orange", "mango", "banana", "sugar");          var str = arr.toString();  console.log("Returned string is : " + str );  // orange,mango,banana,sugar` |
|  `unshift()`向数组的开头添加一个或更多元素，并返回新的长度   | `var arr = new Array("orange", "mango", "banana", "sugar");  var length = arr.unshift("water");  console.log("Returned array is : " + arr );  // water,orange,mango,banana,sugar  console.log("Length of the array is : " + length ); ` |

### Map

#### 创建map

```typescript
let myMap = new Map();

let myMap = new Map([
	["key1","value1"],
	["key2","value2"]
]);
```

Map 相关的函数与属性：

- **map.clear()** – 移除 Map 对象的所有键/值对 
- **map.set()** – 设置键值对，返回该 Map 对象
- **map.get()** – 返回键对应的值，如果不存在，则返回 undefined
- **map.has()** – 返回一个布尔值，用于判断 Map 中是否包含键对应的值
- **map.delete()** – 删除 Map 中的元素，删除成功返回 true，失败返回 false
- **map.size** – 返回 Map 对象键/值对的数量
- **map.keys()** - 返回一个 Iterator 对象， 包含了 Map 对象中每个元素的键 
- **map.values()** – 返回一个新的Iterator对象，包含了Map对象中每个元素的值 

#### 迭代map

```typescript
let nameSiteMapping = new Map();
 
nameSiteMapping.set("Google", 1);
nameSiteMapping.set("Runoob", 2);
nameSiteMapping.set("Taobao", 3);
 
// 迭代 Map 中的 key
for (let key of nameSiteMapping.keys()) {
    console.log(key);                  
}
 
// 迭代 Map 中的 value
for (let value of nameSiteMapping.values()) {
    console.log(value);                 
}
 
// 迭代 Map 中的 key => value
for (let entry of nameSiteMapping.entries()) {
    console.log(entry[0], entry[1]);   
}
 
// 使用对象解析
for (let [key, value] of nameSiteMapping) {
    console.log(key, value);            
}
```



