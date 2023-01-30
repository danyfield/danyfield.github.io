---
title: springboot笔记
date: 2023-01-29 15:32:37
tags: "springboot"
categories: "Java"
---

#### 第一个SpringBoot程序

##### 创建基础项目说明

###### 方式一

使用Spring Initializr 的 Web页面创建项目

①打开 https://start.spring.io/

②填写项目信息

③点击”Generate Project“按钮生成项目；下载此项目

④解压项目包，并用IDEA以Maven项目导入，一路下一步即可，直到项目导入完毕

⑤如果是第一次使用，可能速度会比较慢，包比较多、需要耐心等待一切就绪

###### 方式二

使用 IDEA 直接创建项目

①创建一个新项目

②选择spring initalizr ， 可以看到默认就是去官网的快速构建工具那里实现

③填写项目信息

④选择初始化的组件（初学勾选 Web 即可）

⑤填写项目路径

⑥等待项目构建成功

###### 项目结构分析

通过上面步骤完成了基础项目的创建。就会自动生成以下文件

1、程序的主启动类（程序的主入口）

2、一个 application.properties 配置文件（SpringBoot的核心配置文件）

3、一个 测试类

4、一个 pom.xml

编写一个http接口

①在主程序的同级目录下新建一个controller包
②在包中新建一个HelloController类

```java
@RestController
public class HelloController {

   @RequestMapping("/hello")
   public String hello() {
       return "Hello World";
   } 
}
```

③编写完毕后，从主程序启动项目，浏览器发起请求，看页面返回；控制台输出了 Tomcat 访问的端口号！

##### 容器功能

###### 常用组件及注解

1. @Configuration	添加此注解等同向bean注入bean.xml文件

   - 对比原生添加组件方式的区别

     `原生组件添加到容器的方式`

     1. 需要在resources目录下创建一个xml配置文件

     2. 创建bean标签

        ![](https://img-blog.csdnimg.cn/2021071505525410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MzgzNzY5,size_16,color_FFFFFF,t_70)

     `注解添加`

     1. 创建一个类
     2. 使用注解@Configuration，告诉SpringBoot这是一个配置类

     使用@bean注解构建user和pet对象

     ![](https://img-blog.csdnimg.cn/2021071505531170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1MzgzNzY5,size_16,color_FFFFFF,t_70)

   - 使用实例

   ```java
   /*
   	1、配置类中使用@Bean标注在方法上给容器注册组件，默认单实例
   	2、配置类本身也是组件
   	3、proxyBeanMethod：代理bean的方法
   	Full(proxyBeanMethods = true)【保证每个@Bean方法被调用多少次返回的组件都是单实例的】
   	Lite(proxyBeanMethod = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】
   	组件依赖必须使用Full模式默认，proxyBeanMethods = true
   */
   
   @Configuration(proxyBeanMethods = true)	//告诉SpringBoot这是一个配置类 == 配置文件
   public class MyConfig {
       @Bean	//给容器添加组件，以方法名作为组件id，返回类型就是组件类型，返回值就是组件在容器中的实例
       public User user01(){
           User zhangsan = new User("zhangsan",18);
           zhangsan.setPet(tomcatPet());	//User组件依赖Pet组件
           return zhangsan;
       }
       
       @Bean("tom")
       public Pet tomcatPet(){
           return new Pet("tomcat")
       }
   }
   
   @SpringBootConfiguration
   @EnableAutoConfiguration
   @ComponentScan("com.atguigu.boot")
   public class MainApplication {
       public static void main(String[] args) {
           //1、返回IOC容器
           ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class,args);
           //2、查看容器里的组件
           String[] names = run.getBeanDefinitionNames();
           for (String name : names) {
               System.out.println(name);
           }
           //3、从容器中获取组件
           Pet tom01 = run.getBean("tom",Pet.class);
           Pet tom02 = run.getBean("tom",Pet.class);
           System.out.println("组件："+(tom01 == tom02));
           
           MyConfig bean = run.getBean(MyConfig.class);
           System.out.println(bean);
       }
   }
   ```

2. @Controller、@Service、@Repository、@Component

   ```
   @Controller：控制器层（注入服务）
   @Service：服务层（注入dao）
   @Repository：dao持久层（实现dao访问）
   @Component：标注一个类为Spring容器的Bean（把普通pojo实例化到spring容器中，相当于配置文件中的<bean id="" class=""/>）
   ```

3. @ComponentScan、@Import

   1. @ComponentScan注解的作用

      @ComponentScan注解一般和@Configuration注解一起使用，主要作用为定义包扫描的规则，然后根据定义的规则找出哪些类需要自动装配到spring的bean容器中，然后交由spring进行统一管理

   2. @ComponentScan注解属性介绍

      `value`：指定要扫描的包路径

      `excludeFilters（排除规则）`：excludeFilters=Filter[]指定包扫描的时候根据规则指定要排除的组件

      `includeFilters（包含规则）`：includeFilters=Filter[]指定包扫描的时候根据规则指定要包含的组件

   3. 使用实例

      ```java
      //includeFilters 包含Animal.class类可以被扫描到，包括其子类
      @ComponentScan(value = "com.spring"
      	includeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = {Animal.class})})
      
      //excludeFilters 排除包含@Controller注解的类
      @ComponentScan(value = "com.spring"
                    ,excludeFilters = {
                        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})})
      ```

   4. @Import注解提供了三种用法

      1. @Import一个普通类，spring会将该类加载到spring容器中
      2. @Import一个类，该类实现了ImportBeanDefinitionRegistrar接口，在重写的registerBeanDefinitions方法里面，能拿到BeanDefinitionRegistry bd的注册器，能手工往beanDefinitionMap中注册 beanDefinition
      3. @Import一个类，该类实现了ImportSelector 重写selectImports方法，返回String[]数组对象，数组里的类都会注入到spring容器当中

   5. 使用场景

      `import普通类`

      1. 自定义一个类，无任何注解

         ```java
         public class MyClass {
             public void test() {
                 System.out.println("test方法");
             }
         }
         ```

      2. 写一个importConfig类 import该myClass类

         ```java
         @Import(MyClass.class)
         public class ImportConfig {}
         ```

      3. 通过AnnotationConfigApplicationContext 初始化spring容器

         ![](https://img-blog.csdnimg.cn/aee98405add545bfac5cdda6318b510b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)

      `实现ImportBeanDefinitionRegistrar`

      1. 创建一个普通类MyClassRegistry

         ```java
         public class MyClassRegistry {
             public void test() {
                 System.out.println("MyClassRegistry test方法");
             }
         }
         ```

      2. 创建MyImportRegistry 实现ImportBeanDefinitionRegistrar接口

         ```java
         public class MyImportRegistry implements ImportBeanDefinitionRegistrar {
             @Override
             public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
                 RootBeanDefinition bd = new RootBeanDefinition();
                 bd.setBeanClass(MyClassRegistry.class);
                 registry.registerBeanDefinition("myClassRegistry",bd);
             }
         }
         ```

      3. import该类

         ```java
         @Import(MyImportRegistry.class)
         public class ImportConfig {}
         
         ```

      4. 执行main方法，查看输出

         ![](https://img-blog.csdnimg.cn/f1f654a234544847b28f25aff31a5538.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)

      `实现ImportSelector`

      1. 创建一个普通类MyClassImport 

         ```java
         public class MyClassImport {
         	public void test() {
         		System.out.println("MyClassImport test方法");
         	}
         }
         
         ```

      2. 创建MyImportSelector实现ImportSelector接口

         ```java
         public class MyImportSelector implements ImportSelector {
         	@Override
             public String[] selectImports(AnnotationMetadata importingClassMetadata) {
                 return new String[] {MyClassImport.class.getName()}
             }
         }
         
         ```

      3. import该类

         ```java
         @Import(MyImportSelector.class)
         public class ImportConfig {}
         
         ```

      4. 执行main方法，查看输出

         ![](https://img-blog.csdnimg.cn/3839e967fd2942c5bbf40fb9ce17ae45.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)

4. @Override注解作用

   1. 当作注释
   2. 表示对父类方法的重写
   3. 编译器可验证@Override下的方法名称是否为父类所有，若无就会报错

5. @Conditional（条件装配注解）

   条件装配：当满足Conditional指定的条件则进行组件注入

###### 配置绑定

读取properties文件中的内容，将其封装到JavaBean中，以供随时使用

```java
public class getProperties {
    public static void main(String[] args) throws FileNotFoundException, IOException {
        Properties pps = new Properties();
        pps.load(new FileInputStream("a.properties"));
        Enumeration enum1 = pps.propertyNames();	//得到配置文件的名字
        while(enum1.hasMoreElements()) {
            String strKey = (String) enum1.nextElement();
            String strValue = pps.getProperty(strKey);
            System.out.println(strKey + "=" + strValue);
            //封装到JavaBean
        }
    }
}

```

**@ConfigurationProperties**

要使用该注解必须声明组件注解@Component；通常是用来将properties和yml配置文件属性转化为bean对象使用；@Component+@ConfigurationProperties可以由@EnableConfigurationProperties替换

#### 自动配置原理

##### @SpringBootApplication注解

标注该类是一个SpringBoot应用；关键功能由@Import提供，其导入的AutoConfigurationImportSelector的selectImports()方法通过SpringFactoriesLoader.loadFactoryNames()扫描有META-INF/spring.factories的jar包

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan (
	excludeFilters = {@Filter(
    	type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ),@Filter(
    	type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
)
public @interface SpringBootApplication {}

```

##### @EnableAutoConfiguration注解：开启自动配置

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}

```

###### @AutoConfigurationPackage注解

```java
@Import({Registrar.class})
//利用Registrar导入一系列组件
//将指定一个包下所有组件导入
public @interface AutoConfigurationPackage {}

```

###### @Import({AutoConfigurationImportSelector.class})注解

AutoConfigurationImportSelector ：自动配置导入选择器

自动配置真正实现是从classpath中搜寻所有的META-INF/spring.factories配置文件 ，将其中对应的 org.springframework.boot.autoconfigure包下的配置项通过反射实例化为对应标注了@Configuration的JavaConfig形式IOC容器配置类，然后将这些都汇总成为一个实例并加载到IOC容器中

##### 自动配置生效

```
@ConditionalOnBean：当容器里有指定的bean的条件下。

@ConditionalOnMissingBean：当容器里不存在指定bean的条件下。

@ConditionalOnClass：当类路径下有指定类的条件下。

@ConditionalOnMissingClass：当类路径下不存在指定类的条件下。

@ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。

```

##### 实例

以ServletWebServerFactoryAutoConfiguration配置类为例，解释一下全局配置文件中的属性如何生效，比如：server.port=8081，是如何生效的（当然不配置也会有默认值，这个默认值来自于org.apache.catalina.startup.Tomcat）

```java
@Configuration(proxyBeanMethods = false)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({
    ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
    ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
    ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
    ServletWebServerFactoryConfiguration.EmbeddedUndertow.class
})
public class ServletWebServerFactoryAutoConfiguration {}

```

ServletWebServerFactoryAutoConfiguration类上，有个@EnableConfigurationProperties注解：**开启配置属性**

后面的参数是一个ServerProperties类，这就是约定大于配置的最终落地点

springboot所有自动配置在启动时扫描并加载，扫描了`spring.properties`配置文件，所有配置类都在这里，但是不一定生效，要判断条件是否成立，只要导入对应的start就有了对应的启动器，有了启动器自动装配就会生效，此时配置成功

**步骤：**

1. SpringBoot在启动时从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值
2. 将这些值作为自动配置类导入容器，自动配置类生效
3. 整个J2EE整体解决方案和自动配置都在springboot-autoconfigure的jar包中
4. 它会给容器中导入非常多的自动配置类 （xxxAutoConfiguration）@Bean给容器中导入这个场景需要的所有组件， 并配置好这些组件 @Configuration
5. 自动配置类免去手动编写配置注入功能组件等工作

#### SpringBoot配置文件

- application.properties
  - 语法结构：key=value
- application.yaml
  - 语法结构：key:空格value

**原始实体对象赋值方法**

```java
@Component	//注册bean到容器中，使得这个类可以被扫描到
public class Dog {
    @Value("阿黄")
    private String name;
    @Value("18")
    private Integer age;
}

```

```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired	//将Dog类自动注入
    Dog dog;
    @Test
    public void contextLoads(){System.out.println(dog);}
}

```

**通过yaml文件赋值**

```yaml
person:
	name: qinjiang
	age: 3
	happy: false
	birth: 2000/01/01
	maps: {k1: v1,k2: v2}
	lists: [code,girl,music]
	dog:
		name: 旺财
		age: 1

```

```java
/*
	@ConfigurationProperties作用：
	默认从全局配置文件中获取值
	将配置文件中配置的每一个属性的值映射到该组件中；
	告诉SpringBoot将本类所有属性和配置文件中相关的配置进行绑定
	参数 prefix = “person”: 将配置文件中person下面的所有属性一一对应
*/
@Component //注册bean到容器中
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}

```

**@PropertySource**：加载指定的配置文件

**使用示例**

首先在resources目录下新建一个person.properties文件

```java
@PropertySource(value = "classpath:person.properties")
@Component
public class Person {
    @Value("${name}")
    private String name;
}

```

**若yaml和properties同时配置了端口且没有激活其它环境，默认会使用properties配置文件**

#### Thymeleaf模板引擎

##### 使用示例

```java
@RequestMapping("/t2")
public String test2(Map<String,Object> map){
    //存入数据
    map.put("msg","<h1>Hello</h1>");
    map.put("users", Arrays.asList("qinjiang","kuangshen"));
    //classpath:/templates/test.html
    return "test";
}

```

```jsp
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
    	<title>狂神说</title>
    </head>
    <body>
        <h1>测试页面</h1>
        <div th:text="${msg}"></div>
        <!--不转义-->
        <div th:utext="${msg}"></div>
        <!--遍历数据-->
		<!--th:each每次遍历都会生成当前这个标签：官网#9-->
        <h4 th:each="user :${users}" th:text="${user}"></h4>
        <h4>
            <!--行内写法：官网#12-->
            <span th:each="user:${users}">[[${user}]]</span>
        </h4>
    </body>
</html>

```

##### @RequestMapping

将请求和处理请求的控制器方法关联起来，建立映射关系，即设置route

###### 各个属性

- value属性：默认只写一个参数的话就是给value赋值；value是字符串类型的数组，表示请求映射匹配多个请求地址所对应的请求

- method属性：设置get或post请求映射，是一个RequestMethod类的数组，表示请求映射能够匹配多种请求方式

  对处理指定请求方式的控制器方法，SpringMVC提供了@RequestMapping的派生注解：

  **处理get请求的映射-->@GetMapping**

  **处理post请求的映射-->@PostMapping**

  **处理put请求的映射-->@PutMapping**

  **处理delete请求的映射-->@DeleteMapping**

