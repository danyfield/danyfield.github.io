---
title: springcloud笔记
date: 2023-02-03 11:44:43
tags: "springcloud"
categories: "Java"
---

#### 微服务

##### 什么是微服务

提倡将单一的应用程序划分成一组小的服务，每个服务运行在自己独立的进程内；服务间互相协调，互相配置，采用轻量级的通信机制（HTTP）互相沟通

微服务化的核心即将传统的一站式应用拆分成一个一个服务，一个服务做一件事且能够自行单独启动或销毁，拥有自己独立的数据库，实现解耦

##### 微服务的优缺点

###### 优点

- 是松耦合的，无论在开发阶段或部署阶段都是独立的
- 能够使用不同语言开发
- 易于和第三方集成
- 微服务易于被一个开发人员理解，修改和维护
- 允许利用和融合最新技术
- **微服务只是业务逻辑的代码，不会和HTML，CSS，或其他的界面混合**
- **每个微服务都有存储能力，可以有自己的数据库，也可以有统一的数据库**

###### 缺点

- 要处理分布式系统的复杂性
- 随着服务的增加，运维的压力也在增大
- 系统部署依赖问题
- 服务间通信成本问题
- 数据一致性问题
- 系统集成测试问题
- 性能和监控问题

##### 微服务相关技术栈

| 微服务技术条目                         | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | SpringBoot、Spring、SpringMVC等                              |
| 服务配置与管理                         | Netfix公司的Archaius、阿里的Diamond等                        |
| 服务注册与发现                         | Eureka、Consul、Zookeeper等                                  |
| 服务调用                               | Rest、PRC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Fegin等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由(API网关)                      | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Specatator等                        |
| 全链路追踪                             | Zipkin、Brave、Dapper等                                      |
| 数据流操作开发包                       | SpringCloud Stream(封装与Redis，Rabbit，Kafka等发送接收消息) |
| 时间消息总栈                           | SpringCloud Bus                                              |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |

#### SpringCloud Rest学习环境搭建：服务提供者

##### 创建父工程

父工程为springcloud，其下有多个子mudule

![](https://s1.ax1x.com/2023/02/03/pSsiikt.png)

springcloud-consumer-dept-80访问springcloud-provider-dept-8001下的controller使用REST方式，如**DeptConsumerController.java**

```java
@RestController
public class DeptConsumerController {
    //消费者不应有service层，直接调用RestTemplate ... 注册到Spring中
    //使用RestTemplete先需要放入Spring容器中
    @Autowired
    private RestTemplate restTemplate;
    
    //服务提供方地址前缀
    private static final String REST_URL_PREFIX = "http://localhost:8001";
    
    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        // postForObject(服务提供方地址(接口),参数实体,返回类型.class)
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dep, Boolean.class);
    }
    
    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        // getForObject(服务提供方地址(接口),返回类型.class)
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }
    
    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
    }
}
```

`ConfigBean.java`

```java
@Configuration
public class ConfigBean {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

springcloud-provider-dept-8001的dao接口调用springcloud-api模块下的pojo，可使用在springcloud-provider-dept-8001的pom文件导入springcloud-api模块依赖的方式：

```xml
 <!--我们需要拿到实体类，所以要配置api module-->
        <dependency>
            <groupId>com.haust</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

#### Eureka服务注册中心

Eureka采用了C-S的架构设计，系统中的其他微服务，使用Eureka的客户端连接到EurekaServer并维持心跳连接。这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行

![](https://s1.ax1x.com/2023/02/03/pSsi90A.png)

- 和Dubbo架构对比

![](https://s1.ax1x.com/2023/02/03/pSsikff.png)

##### 构建步骤

`eureka-server`

1. springcloud-eureka-7001 模块建立

2. pom.xml 配置

   ```xml
   <!--导包~-->
   <dependencies>
       <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
       <!--导入Eureka Server依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka-server</artifactId>
           <version>1.4.6.RELEASE</version>
       </dependency>
       <!--热部署工具-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
       </dependency>
   </dependencies>
   ```

3. application.yml

   ```yml
   server:
   	port: 7001
   
   #Eureka配置
   eureka:
   	instance:
   		# Eureka服务端的实例名字
   		hostname: 127.0.0.1
       client:
       	# 表示是否向 Eureka 注册中心注册自己(这个模块本身是服务器,所以不需要)
       	register-with-eureka: false
       	# fetch-registry如果为false,则表示自己为注册中心,客户端的化为 ture
       	fetch-registry: false
       	# Eureka监控页面~
       	service-url:
         		defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 主启动类

   ```java
   @SpringBootApplication
   // @EnableEurekaServer 服务端的启动类，可以接受别人注册进来~
   @EnableEurekaServer
   public class EurekaServer_7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer_7001.class,args);
       }
   }
   ```

5. 启动成功后访问 http://localhost:7001/ 得到以下页面

   ![](https://s1.ax1x.com/2023/02/03/pSsiFtP.png)

`euraka-client`

**调整之前创建的springlouc-provider-dept-8001**

1. 导入Eureca依赖

   ```xml
   <!--Eureka依赖-->
   <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
       <version>1.4.6.RELEASE</version>
   </dependency>
   ```

2. application中新增Eureca配置

   ```yaml
   # Eureka配置：配置服务注册中心地址
   eureka:
   	client:
   		service-url:
   			defaultZone: http://localhost:7001/eureka/
   ```

3. 为主启动类添加@EnableEurekaClient注解

   ```java
   @SpringBootApplication
   // @EnableEurekaClient 开启Eureka客户端注解，在服务启动后自动向注册中心注册服务
   @EnableEurekaClient
   public class DeptProvider_8001 {
       public static void main(String[] args) {
           SpringApplication.run(DeptProvider_8001.class,args);
       }
   }
   
   ```

4. 先启动7001服务端后启动8001客户端，访问监控页http://localhost:7001/ 

   ![](https://s1.ax1x.com/2023/02/03/pSsipmd.png)

   若停掉springcloud-provider-dept-8001 等**30s**后 监控会开启保护机制：

   ![](https://s1.ax1x.com/2023/02/03/pSsiEp8.png)

##### Eureka自我保护机制

**某时刻某微服务不可用，eureka不立即清理，依旧会对该微服务信息进行保存**

- 默认情况下，当eureka server在一定时间内没有收到实例的心跳，便会把该实例从注册表中删除（**默认是90秒**），但是，如果短时间内丢失大量的实例心跳，便会触发eureka server的自我保护机制，比如在开发测试时，需要频繁地重启微服务实例，但是我们很少会把eureka server一起重启（因为在开发过程中不会修改eureka注册中心）；
- 该保护机制的目的是避免网络连接故障，在发生网络故障时，微服务和注册中心之间无法正常通信，但服务本身是健康的，不应该注销该服务
- 但是我们在开发测试阶段，需要频繁地重启发布，若触发了保护机制，则旧的服务实例未被删除，这时请求有可能跑到旧的实例中，而该实例已经关闭了，这就导致请求错误，影响开发测试。所以，在开发测试阶段，我们可以把自我保护模式关闭，只需在eureka server配置文件中加上如下配置即可：eureka.server.enable-self-preservation=false【不推荐关闭自我保护机制】

##### 集群环境配置

![](https://s1.ax1x.com/2023/02/03/pSsiV1S.png)

1. 初始化

   新建springcloud-eureka-7002、springcloud-eureka-7003 模块

   1. 为pom.xml添加依赖 (与springcloud-eureka-7001相同)

      ```xml
      <!--导包~-->
      <dependencies>
          <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
          <!--导入Eureka Server依赖-->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-eureka-server</artifactId>
              <version>1.4.6.RELEASE</version>
          </dependency>
          <!--热部署工具-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-devtools</artifactId>
          </dependency>
      </dependencies>
      
      ```

   2. application.yml配置(与springcloud-eureka-7001相同)

      ```yaml
      server:
        port: 7003
      
      # Eureka配置
      eureka:
        instance:
          hostname: localhost # Eureka服务端的实例名字
        client:
          register-with-eureka: false # 表示是否向 Eureka 注册中心注册自己(这个模块本身是服务器,所以不需要)
          fetch-registry: false # fetch-registry如果为false,则表示自己为注册中心
          service-url: # 监控页面~
            defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      
      ```

   3. 主启动类(与springcloud-eureka-7001相同)

      ```java
      @SpringBootApplication
      // @EnableEurekaServer 服务端的启动类，可以接受别人注册进来~
      public class EurekaServer_7003 {
          public static void main(String[] args) {
              SpringApplication.run(EurekaServer_7003.class,args);
          }
      }
      
      ```

2. 集群成员相互关联

   配置一些自定义本机名字，找到本机hosts文件并打开

   ![](https://s1.ax1x.com/2023/02/03/pSsieXQ.png)

   修改application.yml的配置，如图为springcloud-eureka-7001配置，springcloud-eureka-7002/springcloud-eureka-7003同样分别修改为其对应的名称即可

   ![](https://s1.ax1x.com/2023/02/03/pSsiZ6g.png)

   在集群中使springcloud-eureka-7001关联springcloud-eureka-7002、springcloud-eureka-7003

   完整的springcloud-eureka-7001下的application.yml如下：

   ```yaml
   server:
     port: 7001
   
   #Eureka配置
   eureka:
     instance:
       hostname: eureka7001.com #Eureka服务端的实例名字
     client:
       register-with-eureka: false #表示是否向 Eureka 注册中心注册自己(这个模块本身是服务器,所以不需要)
       fetch-registry: false #fetch-registry如果为false,则表示自己为注册中心
       service-url: #监控页面~
         #重写Eureka的默认端口以及访问路径 --->http://localhost:7001/eureka/
         # 单机： defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
         # 集群（关联）：7001关联7002、7003
         defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
   
   ```

   springcloud-eureka-7002和springcloud-eureka-7003的配置方式类似

   通过springcloud-provider-dept-8001下的yml配置文件，修改**Eureka配置：配置服务注册中心地址**

   ```yaml
   # Eureka配置：配置服务注册中心地址
   eureka:
     client:
       service-url:
         # 注册中心地址7001-7003
         defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
     instance:
       instance-id: springcloud-provider-dept-8001 #修改Eureka上的默认描述信息
   
   ```

   这样模拟集群就搭建好了，可以把一个项目挂载到三个服务器上了

   ![](https://s1.ax1x.com/2023/02/03/pSsinmj.png)

##### 对比和Zookeeper的区别

###### 回顾CAP和ACID原则

RDBMS (MySQL\Oracle\sqlServer) ===> ACID

NoSQL (Redis\MongoDB) ===> CAP

- **ACID是什么？**
  - A (Atomicity) 原子性
  - C (Consistency) 一致性
  - I (Isolation) 隔离性
  - D (Durability) 持久性
- **CAP是什么?**
  - C (Consistency) 强一致性
  - A (Availability) 可用性
  - P (Partition tolerance) 分区容错性
- **CAP理论的核心**
  - 一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性
  - 根据CAP原理，将NoSQL数据库分成了满足CA原则，满足CP原则和满足AP原则三大类
    - CA：单点集群，满足一致性，可用性的系统，通常可扩展性较差
    - CP：满足一致性，分区容错的系统，通常性能不是特别高
    - AP：满足可用性，分区容错的系统，通常可能对一致性要求低一些
- Zookeeper 保证的是 CP —> 满足一致性；当向注册中心查询服务列表时，可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。但当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30-120s，且选举期间整个zookeeper集群是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因为网络问题使得zookeeper集群失去master节点是较大概率发生的事件，虽然服务最终能够恢复，但是，漫长的选举时间导致注册长期不可用，是不可容忍的。
- Eureka 保证的是 AP —> 满足可用性；Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有之中自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：
  - Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
  - Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上 (即保证当前节点依然可用)
  - 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

#### Ribbon：负载均衡（基于客户端）

##### 负载均衡简单分类

###### 集中式LB

在服务的提供方和消费方之间使用独立的LB设施，如Nginx，由该设施负责将访问请求通过某种策略转发至提供方

###### 进程式LB

- 将LB逻辑集成到消费方，消费方从服务注册中心获知哪些地址可用，然后自己再从这些地址中选出一个合适的服务器
- **Ribbon属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址

##### 集成Ribbon

springcloud-consumer-dept-80向pom.xml中添加Ribbon和Eureka依赖

```xml
<!--Ribbon-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
<!--Eureka: Ribbon需要从Eureka服务中心获取要拿什么-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>

```

在application.yml文件中配置Eureka

```yaml
# Eureka配置
eureka:
	client:
		register-with-eureka: false	# 不向Eureka注册自己
		service-url:
			defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/

```

主启动类加上@EnableEurekaClient注解，开启Eureka

```java
//Ribbon和Eureka整合后，客户端可以直接调用，不用关心IP地址和端口号
@SpringBootApplication
@EnableEurekaClient	//开启Eureka客户端
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class, args);
    }
}


```

自定义Spring配置类：ConfigBean.java配置负载均衡实现RestTemplate

```java
@Configuration
public class ConfigBean {
    @LoadBalanced	//配置负载均衡实现RestTemplate
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}


```

修改conroller：DeptConsumerController.java

```java
//Ribbon:我们这里的地址，应该是一个变量，通过服务名来访问
//private static final String REST_URL_PREFIX = "http://localhost:8001";
private static final String REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";


```

##### 使用Ribbon实现负载均衡

![](https://s1.ax1x.com/2023/02/03/pSsir9K.png)

1. 新建两个服务提供者Moudle：springcloud-provider-dept-8003、springcloud-provider-dept-8002
2. 参照springcloud-provider-dept-8001 依次为另外两个Moudle添加pom.xml依赖 、resourece下的mybatis和application.yml配置，Java代码
3. 启动所有服务测试(根据自身电脑配置决定启动服务的个数)，访问http://eureka7001.com:7002/查看结果

![](https://s1.ax1x.com/2023/02/03/pSsiBh6.png)

多次测试访问http://localhost/consumer/dept/list发现随机某个服务提供者提供服务，这种情况叫做轮询，轮询算法在SpringCloud中可以自定义。

#### Feign：负载均衡（基于服务端）

Feign默认集成了Ribbon，与Ribbon不同的是，通过Feign只需要定义服务绑定接口且以声明式的方法

Feign和Ribbon二者对比，前者显现出面向接口编程特点，代码看起来更清爽，而且Feign调用方式更符合SSM或者SprngBoot项目时，Controller层调用Service层的编程习惯

#### Hystrix：服务熔断

