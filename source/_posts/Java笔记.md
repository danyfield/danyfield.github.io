---
title: Java笔记
date: 2023-02-03 12:56:34
tags: "Java"
categories: "Java"
---

### Java基础

#### JDK JRE JVM

- JDK：Java Development Kit (Java开发者工具，包括 JRE，JVM)
- JRE：Java Runtime Environment (Java运行时环境)
- JVM：Java Virtual Machine (Java虚拟机，跨平台核心)

![](https://img-blog.csdnimg.cn/20210427081317290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZsbG93X3dpbmQ=,size_16,color_FFFFFF,t_70)

#### 数据类型

![](https://img-blog.csdnimg.cn/20210427081748164.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZsbG93X3dpbmQ=,size_16,color_FFFFFF,t_70)

#### 权限修饰符

| 修饰符    | 类内部 | 同一个包 | 不同包的子类 | 同一个工程 |
| --------- | ------ | -------- | ------------ | ---------- |
| private   | Yes    |          |              |            |
| 缺省      | Yes    | Yes      |              |            |
| protected | Yes    | Yes      | Yes          |            |
| public    | Yes    | Yes      | Yes          | Yes        |

#### 构造器

##### 什么是构造器

构造器通常也叫构造方法、构造函数，构造器在每个项目中几乎无处不在。当你new一个对象时，就会调用构造器。构造器格式如下：[修饰符，比如public] 类名 (参数列表，可以没有参数){ //这里不能有return}

##### 构造器的注意事项

1. 构造器的名称必须和类名一致
2. 一个类中可以定义多个构造器，但是构造器的参数列表必须不同（重载）
3. 若没有定义构造器，则Java系统会提供一个默认的构造器；定义后系统会将默认的构造器收回
4. 构造器的作用：实例化对象，给对象赋初始值
5. 代码游离块优先执行
6. 构造方法中的参数可以根据需要自行定义，参数的不同构造方法构成重载
7. 子类构造器默认调用父类无参构造器，若父类没有，则必须在子类构造器的第一行通过super关键字指定调用父类哪个构造器

```java
public 构造方法名(参数){
    ...
}

/*注意：
  1.构造方法没有返回值类型
  2.构造方法名必须和该类的类名保持一致，大小写都一样
*/
```

```java
 class Fu
   {
     public Fu(){} //无参的公有构造方法
     public Fu(int i){} //参数类型为int的公有构造方法
   ......
   }


   public class Demo extends Fu
   {
     public Demo(){} //无参的公有构造方法
     public Demo(int i){} //参数类型为int的公有构造方法
     public Demo(int i,double d){} //参数类型为int和double的公有构造方法
     ...
   }
```

##### 构造器的使用

1. Demo demo = new Demo(); //这里是调用的是一个无参的构造方法，必须声明在方法中，最好声明在主方法
2. super、this关键字后的括号内填写参数

#### 泛型

##### 泛型的本质

参数化类型，即给类型指定一个参数，然后在使用时再指定此参数具体的值，因此这个类型可以在使用时决定了。这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

##### 为什么使用泛型

泛型在编译的时候检查类型安全，且所有的强制转换都是自动和隐式的，提高代码的重用率

1. 保证了类型的安全性

   在没有泛型之前，从集合中读取到的每一个对象都必须进行类型转换，如果不小心插入了错误的类型对象，在运行时的转换处理就会出错。

   `没有泛型的情况下使用集合：`

   ```java
   public static void noGeneric() {
   ArrayList names = new ArrayList();
   names.add("mikechen的互联网架构");
   names.add(123); //编译正常
   }
   ```

   `有泛型的情况下使用集合：`

   ```java
   public static void useGeneric() {
   ArrayList<String> names = new ArrayList<>();
   names.add("mikechen的互联网架构");
   names.add(123); //编译不通过
   }
   ```

2. 消除强制转换

   以下没有泛型的代码段需要强制转换：

   ```java
   List list = new ArrayList();
   list.add("hello");
   String s = (String) list.get(0);
   ```

   当重写为使用泛型时，代码不需要强制转换：

   ```java
   List<String> list = new ArrayList<String>();
   list.add("hello");
   String s = list.get(0); // no cast
   ```

3. 避免了不必要的装箱、拆箱操作，提高程序的性能

   在非泛型编程中，将简单类型作为Object传递时会引起Boxing（装箱）和Unboxing（拆箱）操作，这两个过程都是具有很大开销的；引入泛型后不必进行了，因此效率相对较高。泛型变量固定了类型，使用的时候就已经知道是值类型还是引用类型，避免了不必要的装箱、拆箱操作。

   ```java
   object a=1;//由于是object类型，会自动进行装箱操作。
    
   int b=(int)a;//强制转换，拆箱操作。这样一去一来，当次数多了以后会影响程序的运行效率。
   ```

   使用泛型后：

   ```java
   public static T GetValue<T>(T a)
    
   {
   　　return a;
   }
    
   public static void Main()
    
   {
   　　int b=GetValue<int>(1);//使用这个方法的时候已经指定了类型是int，所以不会有装箱和拆箱的操作。
   }
   ```

4. 提高了代码的重用性

##### 泛型类

`定义格式`

![](https://img-blog.csdnimg.cn/img_convert/1db3182a6d057494269abdce8295bc36.png)

**注意事项：泛型类型必须是引用类型（非基本数据类型）**

```
T：任意类型 type
E：集合中元素的类型 element
K：key-value形式 key
V： key-value形式 value

```

##### 泛型接口

![](https://img-blog.csdnimg.cn/img_convert/ed46d1a9cd3bd4a129b4cefffb49a7ef.png)

- 方法声明中定义的形参只能在该方法里使用，而接口、类声明中定义的类型形参则可以在整个接口、类中使用。当调用fun()方法时，根据传入的实际对象，编译器就会判断出类型形参T所代表的实际类型。

使用泛型时，前后定义的泛型类型必须保持一致，否则会编译异常；或者不指定类型，则new什么类型都可以。

##### 泛型方法

![](https://img-blog.csdnimg.cn/img_convert/cbd36e763feb3d7b14c3820a3457d8bc.png)

##### 泛型通配符

**Java泛型通配符是用于解决泛型之间引用传递问题的特殊语法**

![](https://img-blog.csdnimg.cn/img_convert/dd75905a8af533dd3f90a52877e8c84b.png)

```java
//表示类型参数可以是任何类型
public class Apple<?>{}

//表示类型参数必须是A或A的子类
public class Apple<T extends A>{}

//表示类型参数必须是A或者是A的超类型
public class Apple<T supers A>{}

```

#### 反射机制

**Java反射机制（`Java Reflection`）是Java语言中一种`动态（运行时）访问、检测 & 修改它本身`的能力，主要作用是`动态（运行时）获取类的完整结构信息 & 调用对象的方法`**，即Java运行时通过创建一个类的反射对象，再对类进行相关操作：

- 获取对象的成员变量&赋值
- 调用对象的方法（含构造方法，有参/无参）
- 判断对象所属的类

**一般情况下，我们使用某个类，都会知道这个类，以及要用它来做什么，可以直接通过`new`实例化创建对象，然后使用这个对象对类进行操作，这个就属于`正射`**

**而`反射`则是一开始并不知道要初始化的是什么类，无法使用`new`来实例化创建对象，主要是通过JDK提供的反射API来实现，在运行时才知道要操作的是什么类，并且可以获取到类的完整构造以及调用对应的方法**

`示例`

```java
package com.justin.java.lang;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

public class Student {
    private int id;
    public void setId(int id) {
        this.id = id;
    }
    public int getId() {
        return id;
    }
    
    public static void main(String[] args) throws Exception {
        //一、正射调用过程
        Student student = new Student();
        student.setId(1);
        System.out.println("正射调用过程Student id:" + student.getId());
        
        //二、反射调用过程
        Class clz = Class.forName("com.justin.java.lang.Student");
        Constructor studentConstructor = clz.getConstructor();
        Object studentObj = studentConstructor.newInstance();
        
        Method setIdMethod = clz.getMethod("setId",int.class);
        setIdMethod.invoke(studentObj,2);
        Method getIdMethod = clz.getMethod("getId");
        System.out.println("正射调用过程Student id:" + getIdMethod.invoke(studentObj));
    }
}

```

```
正射调用过程Student id:1
反射调用过程Student id:2

```

`获取一个类反射对象的主要过程`

- 获取类的`Class`实例对象
- 根据`Class`实例对象获取`Constructor`对象
- 根据`Constructor`对象的`newInstance`方法获取类的反射对象

**获取到类的反射对象后就可以对类进行操作了**

- 根据`Class`实例对象获取类的`Method`对象
- 根据`Method`对象的`invoke`方法调用到具体类的方法

![](https://img-blog.csdnimg.cn/54f7a810b4514da790958ce554d9c826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0p1c3RpblFpbg==,size_16,color_FFFFFF,t_70)

##### 反射机制使用场景

![](https://img-blog.csdnimg.cn/3a790d5d55bf44b1854eedb953c583d3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0p1c3RpblFpbg==,size_16,color_FFFFFF,t_70)

###### 实现简单工厂模式

```
实现简单工程模式的核心是创建一个工厂类，并且在内部定义了一个静态方法，传入不同的参数标识通过switch进行分组，通过new实例化创建不同的子类对象返回~

```

`示例`

```java
//创建抽象产品类
public interface Product {
    public abstract void show();
}

```

```java
//创建具体产品类
public class ProductA implements Product {
    @Override
    public void show() {
        System.out.println("生产了产品A");
    }
}
public class ProductB implements Product {
    @Override
    public void show() {
        System.out.println("生产了产品B");
    }
}

public class ProductC implements Product {
    @Override
    public void show() {
        System.out.println("生产了产品C");
    }
}

```

```java
//创建简单工厂类
public class SimpleFactory {
    /**
     * 实现简单工厂模式
     * @param pName 产品标识
     * @return 返回具体的产品
     */
    public static Product createProduct(String pName){
        switch (pName){
            case "A":
                return new ProductA();
            case "B":
                return new ProductB();
            case "C":
                return new ProductC();
            default:
                return null;
        }
    }
}

```

```java
//调用简单工厂类
public class SimpleFactoryTest {
    public static void main(String[] args) {
        try {
            SimpleFactory.createProduct("A").show();
        } catch (NullPointerException e) {
            System.out.println("没有A这款产品，无法生产~");
        }
        try {
            SimpleFactory.createProduct("B").show();
        } catch (NullPointerException e) {
            System.out.println("没有B这款产品，无法生产~");
        }
        try {
            SimpleFactory.createProduct("C").show();
        } catch (NullPointerException e) {
            System.out.println("没有C这款产品，无法生产~");
        }
        try {
            SimpleFactory.createProduct("D").show();
        } catch (NullPointerException e) {
            System.out.println("没有D这款产品，无法生产~");
        }
    }
}

```

###### 简单工厂模式的优化

- 弊端：

  - 操作成本高：每增加一个接口的子类，必须修改工厂类的逻辑
  - 系统复杂性提高：每增加一个接口的子类，都必须向工厂类添加逻辑

- 优化思路：通过传入`子类全局定名（包名+类名）` 动态的创建不同的`子类对象实例`，从而使得在不增加产品接口子类和修改工厂类的逻辑的情况下还能实现了工厂类对子类实例对象的统一创建

- 优化步骤：

  - 创建工厂类

    ```java
    public class Factory {
        public static Product getInstance(String className) {
            Product realProduct = null;
            try {
                Class pClass = Class.forName(className);
                realProduct = (Product) pClass.newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return realProduct;
        }
    }
    
    ```

  - 调用工厂类

    ```java
    public class FactoryTest {
        public static void main(String[] args) {
            try {
                Product productA = Factory.getInstance("com.justin.java.lang.ProductA");
                productA.show();
            } catch (NullPointerException e) {
                System.out.println("没有A这款产品，无法生产~");
            }
    
            try {
                Product productB = Factory.getInstance("com.justin.java.lang.ProductB");
                productB.show();
            } catch (NullPointerException e) {
                System.out.println("没有B这款产品，无法生产~");
            }
    
            try {
                Product productC = Factory.getInstance("com.justin.java.lang.ProductC");
                productC.show();
            } catch (NullPointerException e) {
                System.out.println("没有C这款产品，无法生产~");
            }
    
            try {
                Product productD = Factory.getInstance("com.justin.java.lang.ProductD");
                productD.show();
            } catch (Exception e) {
                System.out.println("没有D这款产品，无法生产~");
            }
        }
    }
    
    ```

#### String、StringBuffer和StringBuilder的区别

##### String

string类是一个不可变类，一旦创建包括这个对象中的字符序列是不可改变的，直至对象被销毁

```java
String a = "123";
a = "456";
// 打印出来的a为456
System.out.println(a)

```

![](https://img-blog.csdnimg.cn/20190616102116187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzeHlwcg==,size_16,color_FFFFFF,t_70)

再次给a赋值时，不是对原来堆中实例对象重新赋值，而是生成新的示例对象并指向该对象，之前的实例对象任然存在，如果没有被再次引用，则会被垃圾回收。

##### StringBuffer

StringBuffer对象代表一个字符序列可变的字符串，通过append()、insert()、reverse()、setCharAt()、setLength()等方法可以改变该字符串对象的字符序列，若生成了最终想要的字符串，可以调用toString()方法将其转换为String对象。

```java
StringBuffer b = new StringBuffer("123");
b.append("456");
// b打印结果为：123456
System.out.println(b);

```

![](https://img-blog.csdnimg.cn/20190616102134212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzeHlwcg==,size_16,color_FFFFFF,t_70)

StringBuffer对象没有重新生成一个对象，可以在原来对象中连接新的字符串。

##### StringBuilder

StringBuilder类也代表可变字符串对象，和StringBuffer基本相似，两个类的构造器方法基本相同，不同的是：**StringBuffer是线程安全的，而StringBuilder则没有实现线程安全功能，所以性能略高。**

##### StringBuffer是如何实现线程安全的呢？

StringBuffer类中实现的方法：

![](https://img-blog.csdnimg.cn/20190616102221263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzeHlwcg==,size_16,color_FFFFFF,t_70)

StringBuilder类中实现的方法：

![](https://img-blog.csdnimg.cn/2019061610223156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzeHlwcg==,size_16,color_FFFFFF,t_70)

##### Java9的改进

Java9前采用char[]数组保存字符，字符串每个字符占2字节，Java9字符串采用byte[]数组再加一个encoding-flag字段保存字符，因此字符串每个字符只占1个字节，所以更节省空间，字符串功能方法也没有受到影响。

#### 逻辑运算符

##### |=位或运算符

`|= 运算符和 += 这一类的运算符一样，拆解开就是 a = a | b`

运算规则：两数转为二进制从高位开始比较，只要有一个数为1则为1

##### &=位与运算符

`&= 运算符和 += 这一类的运算符一样，拆解开就是 a = a & b`

运算规则：两数转为二进制从高位开始比较，两个数都为1则为1

##### ^=位异或运算

`^= 运算符和 += 这一类的运算符一样，拆解开就是 a = a^b`

运算规则：两个数转为二进制，然后从高位开始比较，相同为0，不同为1

##### ~=位非运算符

运算规则：位为0则结果为1，位为1则结果为0

### Java Web

#### Servlet

 JavaWeb 中，处理请求和发送响应的过程是由 Servlet 来完成，且其是为了解决实现动态页面而衍生的。

##### Tomcat 和 Servlet 的关系

Tomcat 是Web应用服务器，是一个Servlet/JSP容器。 Tomcat 作为 Servlet 容器，负责处理客户请求,把请求传送给 Servlet，并将 Servlet 的响应传送回客户，而 Servlet 是一种运行在支持 Java 语言的服务器上的组件。

#### JSP

JSP（JavaServerPages）是一种动态网页开发技术。即在传统的HTML文件中插入Java代码和JSP标签，后缀名为.jsp。JSP与PHP、ASP等语言类似，都运行在服务端。通常返回给客户端的就是一个HTML文件，因此只要有浏览器就能查看JSP页面。JSP使用JSP标签在HTML网页中插入Java代码，是Servlet的扩展。

- Tomcat 将 Http 请求文本接收并解析，然后封装成 HttpServletRequest 类型的 request 对象，所有的 Http 头数据可以通过request 对象调用对应的方法查询到。
- Tomcat 会将要响应的信息封装为 HttpServletResponse 类型的response 对象，设置 response 属性可以控制输出到浏览器的内容，然后将 response 交给 Tomcat，Tomcat 就会将其变成响应文本的格式发送给浏览器。

##### ActionContext、ServletContext、pageContext的区别

###### ActionContext

当前Action的上下文环境，通过其可以获取到request、session、ServletContext等与Action有关的对象的引用

###### ServletContext

域对象，即当前web应用的上下文

###### pageContext

可以通过pageContext获取其他域对象的应用，其也是一个域对象，作用范围为当前页面，当前页面结束时销毁

#### JSP九大内置对象和四大作用域

##### JSP的原理

**jsp在编译后会转换为java类，在本质上就是一个Servlet**，当浏览器访问http://localhost:8080/index.jsp。服务器发现后缀为.jsp，它会根据路径找到index.jsp⽂件，会将index.jsp翻译成index_jsp.java⽂件，对这个java⽂件进⾏编译，产⽣⼀个index_jsp.class⽂件，将class⽂件加载运⾏。

##### JSP九大内置对象

`page、config、request、response、session、application、out、pageContext 、exception`

###### **page页面对象**

代表当前JSP页面，是当前JSP编译后的Servlet类的对象。相当于this

###### **config配置对象**

主要作用是取得服务器的配置信息。通过 pageConext对象的 getServletConfig() ⽅法可以获取⼀个config对象。当⼀个Servlet 初始化时，容器把某些信息通过config对象传递给这个Servlet。

###### **request请求对象**

request对象是HttpServletRequest类型的对象。该对象代表了客户端的请求消息，主要用于接收通过HTTP协议传送到服务器的数据（包括请求头消息，请求方式，请求参数等）。request对象的作用域为一次请求(一次请求可能包含一个页面，也可能包含多个页面)。

###### **response响应对象**

response是HttpServletResponse类型的对象，代表的是客户端的响应，主要将JSP容器处理过的对象传回客户端。response对象作用域只在JSP页面内有效。

###### **session会话对象**

session 对象是由服务器⾃动创建的与⽤户请求相关的对象。服务器为每个⽤户⽣成⼀个session对象，⽤于保存该⽤户的信息，跟踪⽤户的操作状态。session对象内部使⽤Map类来保存数据，因此保存数据的格式为“Key/value”。 session对象的value可以使复杂的对象类型，⽽不仅仅局限字符串类型。

###### **application全局对象**

application对象可以将信息保存在服务器中，直到服务器关闭，否则application对象中保存的信息会在整个应用中都有效。与session对象相比，application对象生命周期更长，类似系统的 "全局变量"ServletContext。

###### **out输出对象**

out 对象用于输出JSP页面的信息，且管理应⽤服务器上的输出缓冲区。在使⽤ out 对象输出数据时，可以对数据缓冲区进⾏操作，及时清除缓冲区中残余数据，为其他输出让出缓冲空间。数据输出完毕后，要及时关闭输出流。

###### **pageContext页面上下文对象**

pageContext 对象的作⽤是取得任何范围的参数，通过它可以获取 JSP⻚⾯的out、request、reponse、session、application 等对象。pageContext对象的创建和初始化都是由容器来完成的，在JSP⻚⾯可以直接使⽤pageContext对象。

###### **exception异常对象**

表示发生异常对象，类型 Throwable。作用域：page。

##### JSP四大作用域

**JSP的四大作用域：pageContext、request、session、application**。
**这四个是因为其能存储数据，所以是jsp的四大作用域**

###### pageContext

把变量放到pageContext里，则它的作用域是page，变量只能在当前页面生效。

###### request

代表变量能在一次请求中生效，一次请求可能包含一个或多个页面，不同页面的跳转中都可以使用该变量。

###### session

代表变量能在一次会话中生效，所谓当前会话，就是指从用户打开浏览器开始，到用户关闭浏览器这中间的过程。

###### application

代表变量能在不同服务器下的都能够使用。列如：你用一个账号在谷歌游览器登录的CSDN后，在360，火狐…都能够登录，不用重复输入账号密码。只有当关闭了服务器或者手工删除了，才需要重新输入账号登录。

#### Jar包和War包

`JAR（Java Archive，Java归档文件）`与平台无关的文件格式，允许将许多文件组合压缩成一个压缩文件，JavaSE程序可以打包成Jar包；JAR文件格式以流行的ZIP文件格式为基础；即Jar包是已经完成的一些类，然后将其打包，可以将这些Jar包引入项目中。直接使用这些Jar包中的类和属性；Jar包一般放在lib目录下

`WAR`是一个可以直接运行的web模块，常用于网站，打成包部署到容器中；以Tomcat为例，将War包放置在\webapps\目录下，然后启动Tomcat，该包会自动解压，相当于发布。

War包文件按照一定目录结构组织，根据其根目录下包含有html和jsp文件，或者包含有这两种文件的目录，另外还有WEB-INF目录，通常WEB-INF目录下含有一个web.xml文件和一个classes目录；web.xml是应用的配置文件，而classes目录则包含编译好的servlet类和jsp，或servlet所依赖的其他类（如JavaBean），通常这些所依赖的类也可以打包成jar包放在WEB-INF下的lib目录下，一个war包可以理解为是一个web项目，里面是项目的所有东西。

##### 两者区别

WAR文件代表了一个Web应用程序，JAR是类的归档文件

WAR文件和JAR文件的文件格式是一样的，并且都是使用jar命令来创建

在开发阶段不适合使用WAR文件，因为在开发阶段，经常需要添加或删除Web应用程序的内容，更新 Servlet类文件，而每一次改动后，重新建立WAR文件将是一件浪费时间的事情。在产品发布阶段，使用WAR文件是比较合适的，因为在这个时候，几乎不需要再做什么改动了

在开发阶段，通常将Servlet源文件放到Web应用程序目录的src子目录下，以便和Web资源文件区分。在建立WAR文件时，只需要将src目录从Web应用程序目录中移走，就可以打包了

