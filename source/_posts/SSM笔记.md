---
title: SSM笔记
date: 2023-01-29 15:20:08
tags: SSM
---

### Mybatis

#### CRUD

```
id: 		对应的userMapper中的方法名，即该sql语句需要调用的接口的某个方法名
resultType: 	sql语句执行的返回值类型
parameterType： userMapper中相应的方法传递的参数类型
Select 		需要写返回值类型，update、insert、delete则不需要写
```

`编写接口`

```java
//根据ID查询用户
User getUserById(int Id);

//添加用户
int addUser(User user);

//修改用户
int updateUser(User user);

//删除一个用户
int deleteUser(int userId);
```

`编写对应的UserMapper.xml中的sql语句`

```java
<select id="getUserById" parameterType="int" resultType="com.kuang.pojo.User">
        select * from mybatis.user where id = #{id}
</select>

<!--    对象中的属性，可以直接取出来-->
<insert id="addUser" parameterType="com.kuang.pojo.User">
        insert into mybatis.user (id, name, pwd) value (#{id},#{name},#{pwd})
</insert>

<update id="updateUser" parameterType="com.kuang.pojo.User">
        update mybatis.user set name = #{name}, pwd = #{pwd} where id = #{id};
</update>

<delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id = #{id};
</delete>
```

`测试`

```java
  	@Test
    public void getUserById(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById(1);
        System.out.println(user);

        sqlSession.close();
    }

    @Test
    public void addUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int res = mapper.addUser(new User(4,"哈哈","123456"));
        //增删改必须要提交事务
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void updateUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.updateUser(new User(4,"呵呵","123123"));
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void deleteUser(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(4);
        sqlSession.commit();
        sqlSession.close();
    }
```

##### 使用Map传递参数

若实体类或数据库中的表字段或参数过多，可以考虑使用map传递参数

`UserMapper.java`

```java
//使用map传递参数，查询用户
User getUserById(Map<string,Object> map);
//使用map传递参数，添加用户
int addUser(Map<string,Object> map);
```

`UserMapper.xml`

```xml
<select id="getUserById" parameterType="map" resultType="com.kuang.pojo.User">
        select * from mybatis.user where id = #{userId} and name =#{userName};
</select>
    
<!--    传递的map的key-->
<insert id="addUser" parameterType="map">
        insert into mybatis.user (id, name) value (#{userId},#{userName})
</insert>
```

`UserMapperTest.java`

```java
    //使用map传递参数
    @Test
    public void getUserById2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<>();
        map.put("userId",6);
        map.put("userName","侯帅鑫");
        User user = mapper.getUserById(map);
        System.out.println(user);
        sqlSession.close();
    }
    
    @Test
    public void addUser2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<>();
        map.put("userId",6);
        map.put("userName","侯帅鑫");
        mapper.addUser(map);
        //增删改必须要提交事务
        sqlSession.commit();
        sqlSession.close();
    }
```

### Spring

Spring是一个轻量级的控制反转（IOC）和面向切面编程（AOP）的框架

#### AOP（Aspect Oriented Programming）

即面向切片编程的简称，基于IOC，是对OOP的有益补充；

AOP利用“横切”的技术剖解开封装的对象内部，将影响了多个类的公共行为封装到一个可重用模块并将其名为`Aspect（方面）`，此举有利于减少系统的重复代码，降低模块间的耦合度，和对未来的可操作性和可维护性；

实现AOP的技术分为：

- 采用动态代理技术：用截取消息的方式对消息装饰，取代原对象行为的执行
- 采用静态织入的方式：引入特定的语法创建`方面`使得编译器可以在编译期间织入有关“方面的代码”

Spring实现AOP：

- JDK动态代理：代理对象必须是某个接口的实现，通过在运行期间创建一个接口的实现类完成对目标对象的代理，核心是InvocationHandler和Proxy
- CGLIB代理：原理类似JDK代理，在运行期间生成的代理对象是针对目标类扩展的子类，底层依靠ASM（开源的Java字节编辑类库）操作字节码实现，需要引入包asm.jar和cglib.jar，使用AspectJ注入式切面和@AspectJ注解驱动的切面实际上底层也是通过动态代理实现

##### AOP的作用

1. 弥补了OOP的不足
2. 对业务逻辑的各个部分进行隔离，降低业务逻辑的耦合性，提高程序的可重用性和开发效率
3. 主要用于同一对象层次的公用行为建模

##### 使用AOP的几种方式

1. 经典的基于代理的AOP
2. @AspectJ注解驱动的切面
3. 纯POJO切面
4. 注入式AspectJ切面

##### AOP使用场景

- Authentication 权限检查
- Caching 缓存
- Context passing 内容传递
- Error handling 错误处理
- Lazy loading　延迟加载
- Debugging　　调试
- logging, tracing, profiling and monitoring　日志记录，跟踪，优化，校准
- Performance optimization　性能优化，效率检查
- Persistence　　持久化
- Resource pooling　资源池
- Synchronization　同步
- Transactions 事务管理

##### 常见的通知

- 前置通知 方法执行前调用 对应注解 @Before
- 后置通知 方法执行后调用 对应注解 @After
- 返回通知 方法返回后调用 对应注解 @AfterReturning
- 异常通知 方法出现异常调用 对应注解 @AfterThrowing
- 环绕通知 动态代理、手动推荐方法运行 对应注解 @Around

##### 常见名词

- 连接点（Joinpoint）	指程序能够应用通知的“时机”
- 切入点（Pointcut）     通知定义了切面要发生的“故事”，连接点定义了“故事”发生的时机，那么切入点就定义了“故事”发生的地点，例如某个类或方法的名称，Spring中允许我们方便的用正则表达式来指定
- 切面（Aspect）           通知、连接点、切入点共同组成了切面：时间、地点和要发生的“故事”
- 引入（Introduction） 引入允许我们向现有的类添加新的方法和属性(Spring提供了一个方法注入的功能）
- 目标（Target）            被通知的对象，若无AOP则通知逻辑写在目标对象中
- 织入（Weaving）        把切面应用到目标对象创建新代理对象过程，一般发生在`编译时`、`类加载时`、`运行时`      

#### IOC（Inversion of Control）

将复杂系统分解为相互合作的对象，将对象类封装以后内部实现对外部是透明的，从而降低解决问题的复杂度，且可以灵活地被重用和扩展

![](https://img-blog.csdnimg.cn/564a2e2fff9347a6a72cbfafd622a90d.png)

引入中间位置的“第三方”即IOC容器，全部对象的控制权上缴给IOC容器使其起到“粘合剂”的作用，若其消失则对象间失去联系

##### 控制反转名字的由来

A获得B的过程由主动行为变为了被动行为，控制权颠倒，这就是名称的由来

##### IOC也称为依赖注入（DI）

控制被反转后，获得依赖对象的过程由自身管理变为了IOC容器主动注入，因此“控制反转”获得了一个更合适的名字：`依赖注入（Dependency Injection）`

#### OOP（Object Oriented Programming）

##### 面向对象三大特性

- 封装：隐藏对象属性和实现细节，仅对外提供公共访问方式
- 继承：提高代码复用性；继承是多态的前提
- 多态：父类或接口定义的引用变量可以指向子类或具体实现类的实例对象

##### 五大基本原则

1. 单一职责原则SRP（**Single Responsibility Principle**）

   类的功能要单一

2. 开放封闭原则OCP（**Open－Close Principle**）

   一个模块对拓展开放，对修改封闭

3. 里氏替换原则LSP（**the Liskov Substitution Principle**）

   子类可以替换父类，出现在父类能够出现的任何地方

4. 依赖倒置原则DIP（**the Dependency Inversion Principle**）

   高层次模块不应依赖于低层次模块，都应该依赖于抽象；抽象不应该依赖于具体实现，具体实现要依赖于抽象

5. 接口分离原则ISP（**the Interface Segregation Principle**）

   采用多个与特定客户类有关接口比采用一个通用接口更好，将不同功能拆分为不同接口比放在一个接口里更好

#### IOC理论推导

`UserDao接口`

```java
public interface UserDao {
    void getUser();
}
```

`UserDaoImpI实现类`

```java
public class UserDaoImpl implements UserDao {
    public void getUser() {
        System.out.println("默认获取用户数据");
    }
}

```

`UserService业务接口`

```java
public interface UserService {
    void getUser();
}

```

`UserServiceImpl业务实现类`

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao = new UserDaoImpl();

    public void getUser() {
        userDao.getUser();
    }
}

```

`测试`

```java
public class MyTest {
    public static void main(String[] args) {
        //用户实际调用的是业务层，dao层他们不需要接触！
        UserService userService = new UserServiceImpl();
        userService.getUser();
    }
}

```

#### HelloSpring

`新建一个maven项目，编写实体类`

```java
public class Hello {
    private String str;
    
    public String getStr() {
        return str;
    }
    
    public void setStr(String str) {
        this.str = str;
    }
    
    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }
}

```

`编写xml配置文件`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--使用Spring来创建对象，在Spring这些都称为bean
    类型 变量名 = new 类型();
    Hello hello = new Hello();

    id = 变量名
    class = new的对象
    property 相当于给对象中的属性设置一个值！
        -->
    <bean id="hello" class="com.kuang.pojo.Hello">
        <property name="str" value="Spring"/>
    </bean>
</beans>

```

`测试`

```java
public class MyTest {
    public static void main(String[] args) {
        //获取Spring的上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        //对象都在Spring中管理
        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.toString());
    }
}

```

#### IOC创建对象的方式

1. 默认使用无参构造创建对象

2. 若要使用有参构造创建对象：

   - 下标赋值

     ```xml
     <!--第一种方式：下标赋值    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg index="0" value="狂神说Java"/>
     </bean>
     
     ```

   - 类型

     ```xml
     <!--第二种方式：通过类型的创建，不建议使用    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg type="java.lang.String" value="lifa"/>
     </bean>
     
     ```

   - 参数名

     ```xml
     <!--第三种方式：直接通过参数名来设置    -->
     <bean id="user" class="com.kuang.pojo.User">
         <constructor-arg name="name" value="李发"/>
     </bean>
     
     ```

#### Bean

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean，其是由Spring IoC容器实例化、组装和管理的对象

![](https://img-blog.csdnimg.cn/20200728173028735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NhcnBlX2RpZW0w,size_16,color_FFFFFF,t_70)

- Bean容器，或称Spring IoC容器，主要用来管理对象和依赖，及依赖的注入
- bean是一个Java对象，根据bean规范编写的类并由Bean容器生成的对象
- bean规范：
  - 所有属性为private
  - 提供默认构造方法
  - 提供getter和setter
  - 实现serializable接口

#### Bean的配置

```xml
   <!--
    id：bean的唯一标识符，也就是相当于我们学的对象名
    class：bean对象所对应的全限定名：包名+类名
    name：也是别名，而且name可以同时取多个别名
        -->
    <bean id="userT" class="com.kuang.pojo.UserT" name="user2 u2,u3;u4">
        <property name="name" value="黑心白莲"/>
    </bean>

```

#### 测试实例

```java
public class Address {
    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}

```

```java
public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbies;
    private Map<String,String> card;
    private Set<String> games;
    private String wife;
    private Properties info;
}

```

```xml
    <bean id="address" class="com.kuang.pojo.Address">
        <property name="address" value="西安"/>
    </bean>

    <bean id="student" class="com.kuang.pojo.Student">
        <!--第一种：普通值注入，value        -->
        <property name="name" value="黑心白莲"/>

        <!--第二种        -->
        <property name="address" ref="address"/>

        <!--数组        -->
        <property name="books">
            <array>
                <value>红楼梦</value>
                <value>西游记</value>
                <value>水浒传</value>
                <value>三国演义</value>
            </array>
        </property>

        <!--List        -->
        <property name="hobbies">
            <list>
                <value>打篮球</value>
                <value>看电影</value>
                <value>敲代码</value>
            </list>
        </property>

        <!--Map        -->
        <property name="card">
            <map>
                <entry key="身份证" value="123456789987456321"/>
                <entry key="银行卡" value="359419496419481649"/>
            </map>
        </property>

        <!--Set        -->
        <property name="games">
            <set>
                <value>LOL</value>
                <value>COC</value>
                <value>BOB</value>
            </set>
        </property>

        <!--NULL        -->
        <property name="wife">
            <null/>
        </property>

        <!--Properties        -->
        <property name="info">
            <props>
                <prop key="driver">20191029</prop>
                <prop key="url">102.0913.524.4585</prop>
                <prop key="user">黑心白莲</prop>
                <prop key="password">123456</prop>
            </props>
        </property>
    </bean>

```

#### Bean的自动装配

Spring三种装配属性方式：1、在xml中显示配置；2、在java中显示配置；3、隐式自动装配bean

`创建项目，一个人有两个宠物`

```xml
<bean id="cat" class="com.kuang.pojo.Cat" />
<bean id="dog" class="com.kuang.pojo.Dog" />
<bean id="people" class="com.kuang.pojo.People">
    <property name="name" value="小白莲" />
    <property name="cat" ref="cat" />
    <property name="dog" ref="dog" />
</bean>

```

`ByName自动装配`

```xml
<!--byName:自动在容器上下文查找，和对象set方法后面的值对应的bean id-->
<bean id="people" class="com.kuang.pojo.People" autowire="byName">
    <property name="name" value="小白莲" />
</bean>

```

`ByType自动装配`

```xml
<!--byType:自动在容器上下文查找和自己对象属性类型相同的bean-->
<bean id="people" class="com.kuang.pojo.People" autowire="byType">
    <property name="name" value="小白莲"/>
</bean>

```

- ByName的时候，需要保证所有bean的id唯一，且该bean需要和自动注入的属性的set方法的值一致
- ByType的时候，需要保证所有bean的class唯一，且该bean需要和自动注入的属性的类型一致

#### 使用注解实现自动装配

```xml
<!--开启注解的支持    -->
<context:annotation-config/>

```

直接在属性上使用`@Autowired`即可，也可以在set方法上使用；使用Autowired就可以不用编写set方法，前提是自动配置属性在IOC存在，且符合名字ByName

```java
public class People {
    //若显示定义了Autowired的required属性为false，则该对象可以为null，否则不允许为空
    @Autowired(required = false)
    private Cat cat;
    @Autowired
    private Dog dog;
    private String name;
}

```

#### 代理模式

##### 静态代理

`接口`

```java
//租房
public interface Rent {
    public void rent();
}

```

`真实角色`

```java
//房东
public class Host implements Rent {
    public void rent() {
        System.out.println("房东出租房子！");
    }
}

```

`代理角色`

```java
public class Proxy implements Rent {
    private Host host;
    
    public Proxy() {
    }
    
    public Proxy(Host host) {
        this.host = host;
    }
    
    public void rent() {
        host.rent();
        seeHouse();
        sign();
        fee();
    }
    
    //看房
    public void seeHouse(){
        System.out.println("中介带着看房子！");
    }
    
    //签合同
    public void sign(){
        System.out.println("和中介签署租赁合同！");
    }
    
    //收费用
    public void fee(){
        System.out.println("中介收取费用！");
    }
}

```

`客户端访问代理角色`

```java
public class Client {
    public static void main(String[] args) {
        //房东要出租房子
        Host host = new Host();

        //代理，中介帮房东出租房子，并且代理角色一般会有一些附属操作！
        Proxy proxy = new Proxy(host);

        //不用面对房东，直接找中介租房即可！
        proxy.rent();
    }
}

```

<font color=red>代理模式</font>可以使真实角色的操作更纯粹，不用关注公共的业务，公共业务交给代理角色，公共业务发生扩展的时候，方便集中管理；但此时一个真实角色就会产生一个代理角色，代码量会翻倍，开发效率会变低

##### 动态代理

- 动态代理和静态代理的角色一样
- 动态代理的代理类是动态生成的，不是直接写好的
- 动态代理分为：基于接口的动态代理、基于类的动态代理
  - 基于接口：JDK动态代理
  - 基于类：cglib
  - java字节码实现：javassist

```java
public interface Rent {
    public void rent();
}

```

```java
public class Host implements Rent{
    public void rent() {
        System.out.println("房东要出租房子！");
    }
}

```

```java
//我们会用这个类，自动生成代理类！
public class ProxyInvocationHandler implements InvocationHandler {
    //被代理的接口
    private Rent rent;

    public void setRent(Rent rent) {
        this.rent = rent;
    }

    //生成得到代理类
    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(),
                rent.getClass().getInterfaces(),this);
    }

    //处理代理实例，并返回结果
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //动态代理的本质，就是使用反射机制实现！
        Object result = method.invoke(rent, args);
        seeHose();
        fee();
        return result;
    }

    public void seeHose(){
        System.out.println("中介带着看房子！");
    }

    public void fee(){
        System.out.println("中介收取费用！");
    }
}

```

```java
public class Client {
    public static void main(String[] args) {
        //真实角色
        Host host = new Host();

        //代理角色：现在没有
        ProxyInvocationHandler pih = new ProxyInvocationHandler();

        //通过调用程序处理角色来处理我们要调用的接口对象！
        pih.setRent(host);
        Rent proxy = (Rent) pih.getProxy(); //这里的proxy就是动态生成的，我们并没有写
        proxy.rent();
    }
}

```

- 一个动态代理类代理的是一个接口，一般就是对应的一类业务
- 一个动态代理类可以代理多个类，只要是实现了同一个接口即可

### SpringMVC

#### SpringMVC执行原理

![](https://img-blog.csdnimg.cn/img_convert/8aeda5056f0de84109fe41ed18963e3f.png)

##### 执行流程

DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求；假设请求的url为 : http://localhost:8080/SpringMVC/hello

将上述url拆分为三部分：

http://localhost:8080：8080服务器域名

SpringMVC：部署在服务器上的web站点

hello：控制器

故该url表示为：请求位于服务器localhost:8080上SpringMVC站点的hello控制器

1. HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler
2. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，如上url被查找控制器为：hello
3. HandlerExecution将解析后信息传给DispatcherServlet,如解析控制器映射等
4. HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler
5. Handler让具体的Controller执行
6. Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView
7. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet
8. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名
9. 视图解析器将解析的逻辑视图名传给DispatcherServlet
10. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图
11. 最终视图呈现给用户

#### HelloSpring示例

##### 配置版

1. 新建一个Moudle ， springmvc-02-hello ， 添加web的支持！

2. 确定导入了SpringMVC 的依赖！

3. 配置web.xml ， 注册DispatcherServlet

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
       <!--1.注册DispatcherServlet-->
       <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:springmvc-servlet.xml</param-value>
           </init-param>
           <!--启动级别-1-->
           <load-on-startup>1</load-on-startup>
       </servlet>
       <!--/ 匹配所有的请求；（不包括.jsp）-->
       <!--/* 匹配所有的请求；（包括.jsp）-->
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   </web-app>
   
   ```

4. 编写SpringMVC 的配置文件！名称：**springmvc-servlet.xml** : [servletname]-servlet.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
   </beans>
   
   ```

5. 添加处理映射器

   ```xml
   <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
   
   ```

6. 添加处理器适配器

   ```xml
   <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
   
   ```

7. 添加视图解析器

   ```xml
   <!--视图解析器:DispatcherServlet给他的ModelAndView-->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
       <!--前缀-->
       <property name="prefix" value="/WEB-INF/jsp/"/>
       <!--后缀-->
       <property name="suffix" value=".jsp"/>
   </bean>
   
   ```

8. 编写要操作业务Controller ，要么实现Controller接口，要么增加注解；需要返回一个ModelAndView，装数据，封视图

   ```java
   package com.kuang.controller;
   import org.springframework.web.servlet.ModelAndView;
   import org.springframework.web.servlet.mvc.Controller;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   //注意：这里我们先导入Controller接口
   public class HelloController implements Controller {
       public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
           //ModelAndView 模型和视图
           ModelAndView mv = new ModelAndView();
           //封装对象，放在ModelAndView中。Model
           mv.addObject("msg","HelloSpringMVC!");
           //封装要跳转的视图，放在ModelAndView中
           mv.setViewName("hello"); //: /WEB-INF/jsp/hello.jsp
           return mv;
       }
   }
   
   ```

9. 将自己的类交给SpringIOC容器，注册bean

   ```xml
   <!--Handler-->
   <bean id="/hello" class="com.kuang.controller.HelloController"/>
   
   ```

10. 写要跳转的jsp页面，显示ModelandView存放的数据以及正常页面

    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <html>
            <head>
                <title>Kuangshen</title>
            </head>
            <body>
                ${msg}
            </body>
        </html>
    
    ```

11. 配置Tomcat 启动测试

##### 注解版

1. **新建一个Moudle，springmvc-03-hello-annotation 。添加web支持！**

   建立包结构 com.kuang.controller

2. 由于Maven可能存在资源过滤的问题，将配置完善

   ```xml
   <build>
       <resources>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>false</filtering>
           </resource>
           <resource>
               <directory>src/main/resources</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>false</filtering>
           </resource>
       </resources>
   </build>
   
   ```

3. 在pom.xml文件引入相关的依赖：主要有Spring框架核心库、Spring MVC、servlet , JSTL等

4. 配置web.xml

   - 注册DispatcherServlet

   - 关联SpringMVC的配置文件

   - 启动级别为1

   - 映射路径为 / 【不要用/*，会404】

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
              version="4.0">
         <!--1.注册servlet-->
         <servlet>
             <servlet-name>SpringMVC</servlet-name>
             <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
             <!--通过初始化参数指定SpringMVC配置文件的位置，进行关联-->
             <init-param>
                 <param-name>contextConfigLocation</param-name>
                 <param-value>classpath:springmvc-servlet.xml</param-value>
             </init-param>
             <!-- 启动顺序，数字越小，启动越早 -->
             <load-on-startup>1</load-on-startup>
         </servlet>
         <!--所有请求都会被springmvc拦截 -->
         <servlet-mapping>
             <servlet-name>SpringMVC</servlet-name>
             <url-pattern>/</url-pattern>
         </servlet-mapping>
     </web-app>
     
     ```

5. 添加Spring MVC配置文件

   在resource目录下添加springmvc-servlet.xml配置文件，配置的形式与Spring容器配置基本类似，为了支持基于注解的IOC，设置了自动扫描包的功能，具体配置信息如下：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              https://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/mvc
                              https://www.springframework.org/schema/mvc/spring-mvc.xsd">
       <!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
       <context:component-scan base-package="com.kuang.controller"/>
       <!-- 让Spring MVC不处理静态资源 -->
       <mvc:default-servlet-handler />
       <!--
       支持mvc注解驱动
           在spring中一般采用@RequestMapping注解来完成映射关系
           要想使@RequestMapping注解生效
           必须向上下文中注册DefaultAnnotationHandlerMapping
           和一个AnnotationMethodHandlerAdapter实例
           这两个实例分别在类级别和方法级别处理。
           而annotation-driven配置帮助我们自动完成上述两个实例的注入。
        -->
       <mvc:annotation-driven />
       <!-- 视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
             id="internalResourceViewResolver">
           <!-- 前缀 -->
           <property name="prefix" value="/WEB-INF/jsp/" />-
           <!-- 后缀 -->
           <property name="suffix" value=".jsp" />
       </bean>
   </beans>
   
   ```

   在视图解析器中把所有视图存放在/WEB-INF/目录下，保证视图安全，这个目录下的文件，客户端不能直接访问

6. 创建Controller

   编写一个Java控制类： com.kuang.controller.HelloController

   ```java
   package com.kuang.controller;
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.RequestMapping;
   @Controller
   @RequestMapping("/HelloController")
   public class HelloController {
       //真实访问地址 : 项目名/HelloController/hello
       @RequestMapping("/hello")
       public String sayHello(Model model){
           //向模型中添加属性msg与值，可以在JSP页面中取出并渲染
           model.addAttribute("msg","hello,SpringMVC");
           //web-inf/jsp/hello.jsp
           return "hello";
       }
   }
   
   ```

7. 创建视图层

   WEB-INF/ jsp目录中创建hello.jsp ，视图可直接取出并展示从Controller带回的信息；通过EL取Model中存放的值或对象；

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>SpringMVC</title>
   </head>
   <body>
       ${msg}
   </body>
   </html>
   
   ```

8. 配置Tomcat运行

#### 数据显示到前端

##### 通过ModelAndView

```java
public class ControllerTest1 implements Controller {
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        //返回一个模型视图对象
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","ControllerTest1");
        mv.setViewName("test");
        return mv;
    }
}

```

##### 通过ModelMap

```java
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name, ModelMap model){
    //封装要显示到视图中的数据
    //相当于req.setAttribute("name",name);
    model.addAttribute("name",name);
    System.out.println(name);
    return "hello";
}

```

##### 通过Model

```java
@RequestMapping("/ct2/hello")
public String hello(@RequestParam("username") String name, Model model){
    //封装要显示到视图中的数据
    //相当于req.setAttribute("name",name);
    model.addAttribute("msg",name);
    System.out.println(name);
    return "test";
}

```

##### 对比

```
Model		只有几个方法只适合于储存数据，简化了对于Model对象的操作和理解
ModelMap	除实现自身的方法，也继承LinkedMap的方法和特性		
ModelAndView	可在储存数据的同时设置返回的逻辑视图，控制展示层的跳转

```

### 整合SSM

#### 数据库环境

创建一个存放书籍数据的数据库表

```sql
CREATE DATABASE `ssmbuild`;
USE `ssmbuild`;
DROP TABLE IF EXISTS `books`;
CREATE TABLE `books` (
    `bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
    `bookName` VARCHAR(100) NOT NULL COMMENT '书名',
    `bookCounts` INT(11) NOT NULL COMMENT '数量',
    `detail` VARCHAR(200) NOT NULL COMMENT '描述',
    KEY `bookID` (`bookID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
INSERT  INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`) VALUES 
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');

```

#### 基本环境搭建

1. 新建Maven项目ssmbuild，添加web的支持

2. 导入相关pom依赖

   ```xml
   <dependencies>
       <!--Junit-->
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
       </dependency>
       <!--数据库驱动-->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.47</version>
       </dependency>
       <!-- 数据库连接池 -->
       <dependency>
           <groupId>com.mchange</groupId>
           <artifactId>c3p0</artifactId>
           <version>0.9.5.2</version>
       </dependency>
       <!--Servlet - JSP -->
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>servlet-api</artifactId>
           <version>2.5</version>
       </dependency>
       <dependency>
           <groupId>javax.servlet.jsp</groupId>
           <artifactId>jsp-api</artifactId>
           <version>2.2</version>
       </dependency>
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>jstl</artifactId>
           <version>1.2</version>
       </dependency>
       <!--Mybatis-->
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.5.2</version>
       </dependency>
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis-spring</artifactId>
           <version>2.0.2</version>
       </dependency>
       <!--Spring-->
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>5.1.9.RELEASE</version>
       </dependency>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-jdbc</artifactId>
           <version>5.1.9.RELEASE</version>
       </dependency>
       <dependency>
           <groupId>org.aspectj</groupId>
           <artifactId>aspectjweaver</artifactId>
           <version>1.8.13</version>
       </dependency>
       <!--Spring-->
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.18.22</version>
       </dependency>
   </dependencies>
   
   ```

3. Maven资源过滤设置

   ```xml
   <build>
       <resources>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>false</filtering>
           </resource>
           <resource>
               <directory>src/main/resources</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>false</filtering>
           </resource>
       </resources>
   </build>
   
   ```

4. 搭建基本结构和配置框架

   - com.kuang.pojo

   - com.kuang.dao

   - com.kuang.service

   - com.kuang.controller

   - mybatis-config.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
     </configuration>
     
     ```

   - applicationContext.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd">
     </beans>
     
     ```

#### Mybatis层编写

1. 数据库配置文件**database.properties**

   ```
   jdbc.driver=com.mysql.jdbc.Driver
   jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8
   jdbc.username=root
   jdbc.password=123456
   
   ```

2. IDEA关联数据库

3. 编写Mybatis的核心配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <typeAliases>
           <package name="com.kuang.pojo"/>
       </typeAliases>
       <mappers>
           <mapper resource="com/kuang/dao/BookMapper.xml"/>
       </mappers>
   </configuration>
   
   ```

4. 编写数据库对应的实体类 com.kuang.pojo.Books

   ```java
   package com.kuang.pojo;
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Books {
       private int bookID;
       private String bookName;
       private int bookCounts;
       private String detail;
   }
   
   ```

5. 编写Dao层的 Mapper接口

   ```java
   package com.kuang.dao;
   import com.kuang.pojo.Books;
   import java.util.List;
   public interface BookMapper {
       //增加一个Book
       int addBook(Books book);
       //根据id删除一个Book
       int deleteBookById(int id);
       //更新Book
       int updateBook(Books books);
       //根据id查询,返回一个Book
       Books queryBookById(int id);
       //查询全部Book,返回list集合
       List<Books> queryAllBook();
   }
   
   ```

6. 编写接口对应的 Mapper.xml 文件。需要导入MyBatis的包

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.kuang.dao.BookMapper">
       <!--增加一个Book-->
       <insert id="addBook" parameterType="Books">
           insert into ssmbuild.books(bookName,bookCounts,detail)
           values (#{bookName}, #{bookCounts}, #{detail})
       </insert>
       
       <!--根据id删除一个Book-->
       <delete id="deleteBookById" parameterType="int">
           delete from ssmbuild.books where bookID=#{bookID}
       </delete>
       
       <!--更新Book-->
       <update id="updateBook" parameterType="Books">
           update ssmbuild.books
           set bookName = #{bookName},bookCounts = #{bookCounts},detail = #{detail}
           where bookID = #{bookID}
       </update>
       
       <!--根据id查询,返回一个Book-->
       <select id="queryBookById" resultType="Books">
           select * from ssmbuild.books
           where bookID = #{bookID}
       </select>
       
       <!--查询全部Book-->
       <select id="queryAllBook" resultType="Books">
           SELECT * from ssmbuild.books
       </select>
   </mapper>
   
   ```

7. 编写Service层的接口和实现类

   ```java
   package com.kuang.service;
   import com.kuang.pojo.Books;
   import java.util.List;
   //BookService:底下需要去实现,调用dao层
   public interface BookService {
       //增加一个Book
       int addBook(Books book);
       //根据id删除一个Book
       int deleteBookById(int id);
       //更新Book
       int updateBook(Books books);
       //根据id查询,返回一个Book
       Books queryBookById(int id);
       //查询全部Book,返回list集合
       List<Books> queryAllBook();
   }
   
   ```

   ```java
   package com.kuang.service;
   import com.kuang.dao.BookMapper;
   import com.kuang.pojo.Books;
   import java.util.List;
   public class BookServiceImpl implements BookService {
       //调用dao层的操作，设置一个set接口，方便Spring管理
       private BookMapper bookMapper;
       public void setBookMapper(BookMapper bookMapper) {
           this.bookMapper = bookMapper;
       }
       public int addBook(Books book) {
           return bookMapper.addBook(book);
       }
       public int deleteBookById(int id) {
           return bookMapper.deleteBookById(id);
       }
       public int updateBook(Books books) {
           return bookMapper.updateBook(books);
       }
       public Books queryBookById(int id) {
           return bookMapper.queryBookById(id);
       }
       public List<Books> queryAllBook() {
           return bookMapper.queryAllBook();
       }
   }
   
   ```

#### Spring层

1. 配置**Spring整合MyBatis**，我们这里数据源使用c3p0连接池

2. 编写Spring整合Mybatis的相关的配置文件； spring-dao.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              https://www.springframework.org/schema/context/spring-context.xsd">
       <!-- 配置整合mybatis -->
       <!-- 1.关联数据库文件 -->
       <context:property-placeholder location="classpath:database.properties"/>
       <!-- 2.数据库连接池 -->
       <!--数据库连接池
           dbcp  半自动化操作  不能自动连接
           c3p0  自动化操作（自动的加载配置文件 并且设置到对象里面）
       -->
       <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <!-- 配置连接池属性 -->
           <property name="driverClass" value="${jdbc.driver}"/>
           <property name="jdbcUrl" value="${jdbc.url}"/>
           <property name="user" value="${jdbc.username}"/>
           <property name="password" value="${jdbc.password}"/>
           <!-- c3p0连接池的私有属性 -->
           <property name="maxPoolSize" value="30"/>
           <property name="minPoolSize" value="10"/>
           <!-- 关闭连接后不自动commit -->
           <property name="autoCommitOnClose" value="false"/>
           <!-- 获取连接超时时间 -->
           <property name="checkoutTimeout" value="10000"/>
           <!-- 当获取连接失败重试次数 -->
           <property name="acquireRetryAttempts" value="2"/>
       </bean>
       <!-- 3.配置SqlSessionFactory对象 -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <!-- 注入数据库连接池 -->
           <property name="dataSource" ref="dataSource"/>
           <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
           <property name="configLocation" value="classpath:mybatis-config.xml"/>
       </bean>
       <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 -->
       <!--解释 ： https://www.cnblogs.com/jpfss/p/7799806.html-->
       <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
           <!-- 注入sqlSessionFactory -->
           <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
           <!-- 给出需要扫描Dao接口包 -->
           <property name="basePackage" value="com.kuang.dao"/>
       </bean>
   </beans>
   
   ```

3. **Spring整合service层**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           https://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!--扫描service下面的包-->
       <context:component-scan base-package="com.kuang.service"/>
   
       <!--2.将我们所有的业务类，注入到spring，可以通过配置，或者注解实现-->
       <bean id="bookServiceImpl" class="com.kuang.service.BookServiceImpl">
           <property name="bookMapper" ref="bookMapper"/>
       </bean>
   
       <!--3.声明式事务-->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <!--注入数据库连接池-->
           <property name="dataSource" ref="dataSource"/>
       </bean>
   
       <!--4.aop事务支持-->
       <!--配置事务通知-->
       <tx:advice id="txAdvice" transaction-manager="transactionManager">
           <tx:attributes>
               <!--配置哪些方法使用什么样的事务,配置事务的传播特性-->
               <!--<tx:method name="add" propagation="REQUIRED"/>
               <tx:method name="delete" propagation="REQUIRED"/>
               <tx:method name="update" propagation="REQUIRED"/>
               <tx:method name="search*" propagation="REQUIRED"/>
               <tx:method name="get" read-only="true"/>-->
               <tx:method name="*" propagation="REQUIRED"/>
           </tx:attributes>
       </tx:advice>
       <!--配置aop织入事务-->
       <aop:config>
           <aop:pointcut id="txPointcut" expression="execution(* com.kuang.dao.*.*(..))"/>
           <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
       </aop:config>
   </beans>
   
   ```

   Spring就是一个大杂烩，一个容器！

#### SpringMVC层

1. web.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
       <!--DispatcherServlet-->
       <servlet>
           <servlet-name>DispatcherServlet</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <!--一定要注意:我们这里加载的是总的配置文件，之前被这里坑了！-->   
               <param-value>classpath:applicationContext.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>DispatcherServlet</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
       <!--encodingFilter-->
       <filter>
           <filter-name>encodingFilter</filter-name>
           <filter-class>
               org.springframework.web.filter.CharacterEncodingFilter
           </filter-class>
           <init-param>
               <param-name>encoding</param-name>
               <param-value>utf-8</param-value>
           </init-param>
       </filter>
       <filter-mapping>
           <filter-name>encodingFilter</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
       <!--Session过期时间-->
       <session-config>
           <session-timeout>15</session-timeout>
       </session-config>
   </web-app>
   
   ```

2. **spring-mvc.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">
       <!-- 配置SpringMVC -->
       <!-- 1.开启SpringMVC注解驱动 -->
       <mvc:annotation-driven />
       <!-- 2.静态资源默认servlet配置-->
       <mvc:default-servlet-handler/>
       <!-- 3.配置jsp 显示ViewResolver视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
           <property name="prefix" value="/WEB-INF/jsp/" />
           <property name="suffix" value=".jsp" />
       </bean>
       <!-- 4.扫描web相关的bean -->
       <context:component-scan base-package="com.kuang.controller" />
   </beans>
   
   ```

3. **Spring配置整合文件，applicationContext.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
       <import resource="spring-dao.xml"/>
       <import resource="spring-service.xml"/>
       <import resource="spring-mvc.xml"/>
   </beans>
   
   ```

**配置文件，暂时结束！Controller 和 视图层编写**

1. BookController 类编写 ， 方法一：查询全部书籍

   ```java
   @Controller
   @RequestMapping("/book")
   public class BookController {
       @Autowired
       @Qualifier("BookServiceImpl")
       private BookService bookservice;
       @RequestMapping("/allBook")
       public String list(Model model) {
           List<Books> list = bookService.queryAllBook();
           model.addAttribute("list",list);
           return "allBook";
       }
   }
   
   ```

2. 编写首页 **index.jsp**

   ```jsp
   <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
     <!DOCTYPE HTML>
       <html>
           <head>
               <title>首页</title>
               <style type="text/css">
                   a {
                       text-decoration: none;
                       color: black;
                       font-size: 18px;
                   }
                   h3 {
                       width: 180px;
                       height: 38px;
                       margin: 100px auto;
                       text-align: center;
                       line-height: 38px;
                       background: deepskyblue;
                       border-radius: 4px;
                   }
               </style>
           </head>
           <body>
               <h3>
                   <a href="${pageContext.request.contextPath}/book/allBook">点击进入列表页</a>
               </h3>
           </body>
       </html>
   
   ```

3. 书籍列表页面 **allbook.jsp**

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
       <head>
           <title>书籍列表</title>
           <meta name="viewport" content="width=device-width, initial-scale=1.0">
           <!-- 引入 Bootstrap -->
           <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
       </head>
       <body>
           <div class="container">
               <div class="row clearfix">
                   <div class="col-md-12 column">
                       <div class="page-header">
                           <h1>
                               <small>书籍列表 —— 显示所有书籍</small>
                           </h1>
                       </div>
                   </div>
               </div>
               <div class="row">
                   <div class="col-md-4 column">
                       <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增</a>
                   </div>
               </div>
               <div class="row clearfix">
                   <div class="col-md-12 column">
                       <table class="table table-hover table-striped">
                           <thead>
                               <tr>
                                   <th>书籍编号</th>
                                   <th>书籍名字</th>
                                   <th>书籍数量</th>
                                   <th>书籍详情</th>
                                   <th>操作</th>
                               </tr>
                           </thead>
                           <tbody>
                               <c:forEach var="book" items="${requestScope.get('list')}">
                                   <tr>
                                       <td>${book.getBookID()}</td>
                                       <td>${book.getBookName()}</td>
                                       <td>${book.getBookCounts()}</td>
                                       <td>${book.getDetail()}</td>
                                       <td>
                                           <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.getBookID()}">更改</a> |
                                           <a href="${pageContext.request.contextPath}/book/del/${book.getBookID()}">删除</a>
                                       </td>
                                   </tr>
                               </c:forEach>
                           </tbody>
                       </table>
                   </div>
               </div>
           </div>
   
   ```

4. BookController 类编写 ， 方法二：添加书籍

   ```java
   @RequestMapping("/toAddBook")
   public String toAddPaper() {
       return "addBook";
   }
   @RequestMapping("/addBook")
   public String addPaper(Books books) {
       System.out.println(books);
       bookService.addBook(books);
       return "redirect:/book/allBook";
   }
   
   ```

5. 添加书籍页面：**addBook.jsp**

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>新增书籍</title>
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <!-- 引入 Bootstrap -->
       <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
   </head>
   <body>
   <div class="container">
       <div class="row clearfix">
           <div class="col-md-12 column">
               <div class="page-header">
                   <h1>
                       <small>新增书籍</small>
                   </h1>
               </div>
           </div>
       </div>
       <form action="${pageContext.request.contextPath}/book/addBook" method="post">
           书籍名称：<input type="text" name="bookName"><br><br><br>
           书籍数量：<input type="text" name="bookCounts"><br><br><br>
           书籍详情：<input type="text" name="detail"><br><br><br>
           <input type="submit" value="添加">
       </form>
   </div>
   
   ```

6. BookController 类编写 ， 方法三：修改书籍

   ```java
   @RequestMapping("/toUpdateBook")
   public String toUpdateBook(Model model, int id) {
       Books books = bookService.queryBookById(id);
       System.out.println(books);
       model.addAttribute("book",books );
       return "updateBook";
   }
   @RequestMapping("/updateBook")
   public String updateBook(Model model, Books book) {
       System.out.println(book);
       bookService.updateBook(book);
       Books books = bookService.queryBookById(book.getBookID());
       model.addAttribute("books", books);
       return "redirect:/book/allBook";
   }
   
   ```

7. 修改书籍页面 **updateBook.jsp**

   ```xml
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>修改信息</title>
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <!-- 引入 Bootstrap -->
       <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
   </head>
   <body>
   <div class="container">
       <div class="row clearfix">
           <div class="col-md-12 column">
               <div class="page-header">
                   <h1>
                       <small>修改信息</small>
                   </h1>
               </div>
           </div>
       </div>
       <form action="${pageContext.request.contextPath}/book/updateBook" method="post">
           <input type="hidden" name="bookID" value="${book.getBookID()}"/>
           书籍名称：<input type="text" name="bookName" value="${book.getBookName()}"/>
           书籍数量：<input type="text" name="bookCounts" value="${book.getBookCounts()}"/>
           书籍详情：<input type="text" name="detail" value="${book.getDetail() }"/>
           <input type="submit" value="提交"/>
       </form>
   </div>
   
   ```

8. BookController 类编写 ， 方法四：删除书籍

   ```java
   @RequestMapping("/del/{bookId}")
   public String deleteBook(@PathVariable("bookId") int id) {
       bookService.deleteBookById(id);
       return "redirect:/book/allBook";
   }
   
   ```

#### 项目结构

![](https://s1.ax1x.com/2023/01/29/pSa4cW9.png)

