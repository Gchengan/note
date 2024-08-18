## 1.导入依赖

   <scope>import</scope>的作用：将  spring-cloud-dependencies中dependencyManagement的dependencies，全部引入到当前工程的dependencyManagement 

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-dependencies</artifactId>
   <type>pom</type>
   <scope>import</scope>
   <version>${spring-cloud.version}</version>
</dependency>
```



## 2.注册中心

### Eureka

#### 1.导入服务端依赖，配置服务端

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>     
```

```java
@SpringBootApplication
//开启Eureka的自动装配
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class,args);
    }
}
```

```yml
#服务端口
server:
  port: 10086
spring:
  application:
    name: eurekaserver  # eureka的服务名称

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka  #eureka的地址
```

#### 2.导入客户端依赖，客户端配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

@LoadBalanced表示在客户端启用负载均衡， RestTemplate可以发送http请求

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

```java
@Autowired
private RestTemplate restTemplate;

@Override
public User userList() {
    User user = getById(1);
    //order表示Eureka注册的名称
    String url="http://order/"+user.getOrderId();
    Order u = restTemplate.getForObject(url, Order.class);
    ArrayList<Order> orders = new ArrayList<>();
    orders.add(u);
    user.setOrderList(orders);
    return user;
}
```

```yml
spring:
  application:
    name: order
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka        
```

###  Nacos

### 1.定义父工程依赖版本

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.study</groupId>
    <artifactId>springcloud-nacos</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>springcloud-nacos</name>
    <description>springcloud-nacos</description>
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>17</java.version>
        <spring-cloud.version>2021.0.6</spring-cloud.version>
        <spring-boot.version>2.7.11</spring-boot.version>
        <spring-cloud-alibaba.version>2021.0.5.0</spring-cloud-alibaba.version>
        <mysql.version>8.0.31</mysql.version>
        <mybatis-plus.version>3.5.2</mybatis-plus.version>
        <druid.version>1.2.12</druid.version>
    </properties>
    <modules>
        <module>user-service</module>
        <module>order-service</module>
    </modules>
<dependencyManagement>
    <dependencies>
<!--        springboot依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <type>pom</type>
            <version>${spring-boot.version}</version>
            <scope>import</scope>
        </dependency>
<!--   spring cloud依赖     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <type>pom</type>
            <version>${spring-cloud.version}</version>
            <scope>import</scope>
        </dependency>
<!--        spring cloud-alibaba依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

<!--        mysql依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
<!--        mybatisplus依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
<!--        druid依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
<!--lombok依赖-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
    </dependencies>
</dependencyManagement>



    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



### 2.nacos简单使用

#### 1.执行 .\startup.cmd -m standalone 打开nacos服务端



#### 2.子模块导入nacos依赖即可

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```



#### 3.注意事项

2021.0.1版本的nacos-discovery移除了Ribbon依赖，并且sprngcloud2020版本就已经不再使用netflix，也就没有了ribbon，需要引入loadbalancer依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



#### 4.配置nacos服务地址

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos服务地址
```



### 3.nacos服务多级存储模型

#### 1.nacos服务分级模型

![nacos](E:\笔记\photo\springcloud\nacos.svg)



**服务调用尽可能选择本地集群的服务，跨集群调用延迟较高。本地服务不可访问时，再去访问其他集群**



#### 2.服务集群属性配置



##### 修改application.yml

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos服务地址
      discovery:
        cluster-name: HZ #配置集群名称,也就是机房位置，HZ代指杭州机房
```



### 4.Nacos负载均衡

#### 1.说明

![](E:\笔记\photo\springcloud\1.png)

![](E:\笔记\photo\springcloud\2.png)

在图二中为HZ集群，在调用时应尽量选择本地集群，但是默认的负载均衡是轮询，并不会优先同集群

#### 2.修改负载均衡规则

```yml
server:
  port: 8081

spring:
  application:
    name: user-service
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/cloud_user
      username: root
      password: gc0311
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos服务地址
      discovery:
        cluster-name: HZ #配置集群名称,也就是机房位置，HZ代指杭州机房
        
     #开启的是nacos的默认负载均衡策略
     #此策略会先筛选出相同的集群,再根据集群里的服务器的权重进行筛选
     #如果权重相同,则采用轮询策略
    loadbalancer:
      nacos:
        enabled: true

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: assign_id
      table-underline: true
```

##### 





### 5.Nacos权重

#### 1.Nacos控制台可以设置权重值，0~1之间

#### 2.同集群的多个实例，权重访问越高被访问频率越高

#### 3.权重若设置为0则完全不访问





### 6.Nacos环境隔离    -Namespace

#### 1.作用

**<span style="font-size:20px" >Namespace 通常用来做环境隔离。例如开发环境 `dev` 、测试环境 `test` 和生产环境 `pro` 之间的服务/数据相互隔离，无法相互访问。</span>**



#### 2.新建命名空间

#####     1.在nacos控制台新建命名空间



#####     2.修改配置文件

```yml
server:
  port: 8081

spring:
  application:
    name: user-service
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/cloud_user
      username: root
      password: gc0311
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos服务地址
      discovery:
        cluster-name: HZ #配置集群名称
        namespace: 01255c3c-6895-407b-8d8e-2bde96c3ad41 #填入对应的id
        #开启的是nacos的默认负载均衡策略
        #此策略会先筛选出相同的集群,再根据集群里的服务器的权重进行筛选
        #如果权重相同,则采用轮询策略
    loadbalancer:
      nacos:
        enabled: true

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: assign_id
      table-underline: true
```

​     

#### 3.注意点

**<span style="font-size:20px">不同的namespace的服务是不能访问的</span>**
