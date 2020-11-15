## 介绍
单个模块之间的远程调用，客户端调用微服务时会存在很多的微服务部署在集群上，当集群的规模很大时，服务之间的调用以及服务的管理就非常麻烦，这时候就需要服务注册管理中心。
## Eureka

### 介绍
Eureka采用CS的设计模式，Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中其它的微服务，使用Eureka的客户端连接到Eureka Server并维持心跳。
系统维护人员通过Eureka Server来监控系统中各个微服务是否正常运行。

### 流程分析
在服务注册与发现中，有一个注册中心，当服务器启动的时侯，会把当前自己服务器的信息，比如服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系（服务治理概念）。在任何RPC远程框架中，都会有一个注册中心（存放服务地址相关信息（接口地址））

### 构建Eureka Server单机版

#### 创建maven模块

* cloud-eureka-server7001

#### 改pom
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <!--    引入自定义的api通用包，可以使用Payment支付Entity    -->
        <dependency>
            <groupId>com.sc2020</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- boot web actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--  一般通用配置  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--  热部署      -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!--   单元测试     -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>
```

#### 写yml
```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost  #eureka服务端的实例名称
  client:
    register-with-eureka: false   #false表示不向注册中心注册自己
    fetch-registry: false   #false表示自己端就是注册中心
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   #单机
```

#### 启动类
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
//声明自身是EurekaServer
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308214746.png)

### 支付模块入驻Eureka Server

#### 修改支付模块的pom
增加坐标
```xml
<!--           引入eureka客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

#### 修改yml
```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql:///db2019?serverTimezone=UTC
    username: root
    password: 286728

eureka:
  client:
    #是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须为true才能配合ribbon使用负载均衡
    fetch-registry: true
    #EurekaServer地址
    service-url:
      defaultZone: http://localhost:7001/eureka

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.lhf.springcloud.entities
```

#### 修改主启动添加注解
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308220653.png)

#### 微服务注册名称配置说明
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308220740.png)

### 订单模块入驻Eureka Server

#### 改pom，跟支付模块相同

#### 改yml
```yml
server:
  port: 8080

spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须为true才能配合ribbon使用负载均衡
    fetch-registry: true
    #EurekaServer地址
    service-url:
      defaultZone: http://localhost:7001/eureka
```

#### 启动类同支付模块

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308221653.png)

### 构建Eureka Server集群版

#### 为了防止Eureka Server宕机，会布置两到三台，应该不会倒霉到三台同时宕机

#### 原理
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308222849.png)

**集群可以高可用，实现负载均衡**

**互相注册，相互守望，对外暴露出一个整体**

#### 仿照Eureka Server单机版创建新模块

* cloud-eureka-server7002

#### 只修改yml
```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com  #eureka服务端的实例名称
  client:
    register-with-eureka: false   #false表示不向注册中心注册自己
    fetch-registry: false   #false表示自己端就是注册中心
    service-url:
      #   defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/   #单机
      defaultZone: http://eureka7001.com:7002/eureka/  #集群,向对方注册自己
```

#### 将服务注册进集群

**只修改yml文件,其他服务类似**
```yml
server:
  port: 8080

spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须为true才能配合ribbon使用负载均衡
    fetch-registry: true
    #EurekaServer地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308232948.png)

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200308233018.png)

### 支付微服务集群配置

#### 新建支付微服务模块

* cloud-provider-payment8002

全部复制cloud-provider-payment8001的内容，只修改端口号为8002

#### 修改controller

**因为服务名一样，只是端口不同，所以修改controller获取服务端口号**

```java
import com.lhf.springcloud.entities.CommonResult;
import com.lhf.springcloud.entities.Payment;
import com.lhf.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {
    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String serverPort;

    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("插入结果:" + result);
        if (result > 0){
            return new CommonResult(200,"插入数据库成功,serverPort:"+serverPort,result);
        }else {
            return new CommonResult(444,"插入数据库失败",null);
        }
    }
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult create(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("查询结果:" + payment);
        if (payment != null){
            return new CommonResult(200,"查询成功,serverPort:"+serverPort,payment);
        }else {
            return new CommonResult(444,"查询失败",null);
        }
    }
}
```

#### 修改订单服务的controller

**由于之前订单服务调用支付服务的url是写死的，这使得支付服务的集群毫无意识，所以修改url为服务名，让订单服务从Eureka Server获取具体的url**
```java
package com.lhf.springcloud.controller;

import com.lhf.springcloud.entities.CommonResult;
import com.lhf.springcloud.entities.Payment;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
public class OrderController {

    public static final String PYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment){
        return restTemplate.postForObject(PYMENT_URL+"/payment/create",payment,CommonResult.class);
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PYMENT_URL+"/payment/get/"+id,CommonResult.class);
    }
}
```

**由于微服务名是无法直接访问的，还需要开启负载均衡**
```java
package com.lhf.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200309213643.png)

### 总结完善

**到这里一个小型的微服务架构已经搭建完成了**
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200309214611.png)

1. 查找服务
2. 注册服务
3. 返回服务的url及端口号
4. 访问服务

#### 完善

##### 主机名的修改

**需要下面依赖支持**
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

**在eurekaserver中可以发现，服务是以主机名:服务名:端口号的形式存在的，为了方便管理维护，需要修改主机名**
```yml
eureka:
  instance:
    instance-id: payment8002
```

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200309215946.png)

**显示ip**
```yml
eureka:
  instance:
    instance-id: payment8002
    prefer-ip-address: true
```
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200309220156.png)

### 服务发现Discovery

**对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息**

**类似于一个自我介绍，有服务对外开发，可以获取服务信息**

#### 增加以下controller内容
```java
    //获取服务信息封装类
    @Resource
    private DiscoveryClient discoveryClient;
    @GetMapping(value = "/payment/discover")
    public Object discover(){
        //获取所有服务名称
        List<String> services = discoveryClient.getServices();
        for (String service : services) {
            log.info("****"+service+"****");
        }
        //根据服务名获取服务实例信息
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        return this.discoveryClient;
    }
```

#### 主启动类开启服务发现
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200310000951.png)

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200310001050.png)

### eurekaserver的自我保护

#### 现象
某时刻某一个微服务不可用，eureka不会立刻清理，依旧会对该微服务的信息进行保存。

#### 原因
为了防止eurekaclient可以正常运行，但是eurekaserver网络不通畅，eurekaserver不会立刻将eurekaclient服务剔除。

#### 什么是自我保护模式？
默认情况下，如果eurekaserver在一定时间内没有收到某个服务实例的心跳，eurekaserver将会注销该实例（默认90秒）。但是当网络分区故障发生（延时，卡顿，拥挤）时，微服务与eurekaserver之间无法正常通信，以上行为可能变得非常危险————因为微服务本身是健康的，此时根本不应该注销这个微服务。eurekaserver通过自我保护模式来解决这个问题————当eurekaserver节点在短时间内丢失过多客户端时（可能发送了网络分区故障），那么这个节点就会进入自我保护模式。

**宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例**

#### 禁用自我保护
在eurekaserver的yml中配置：
```yml
eureka:
  server:
    enable-self-preservation: false #关闭自我保护，默认为true
    eniction-interval-timer-in-ms: 2000 #间隔时间，超过两秒接收不到就剔除，默认90秒
```

为了方便测试，在服务端做如下改变：
```yml
eureka:
  instance:
    lease-renewal-interval-in-seconds: 1
    #发送心跳间隔时间,默认30秒，s
    lease-expiration-duration-in-seconds: 2
    #服务端在收到最后一次心跳后等待时间上限，默认90秒，超时剔除服务，s
```
### 官方已停止更新

## Zookeeper

### 服务器搭建

* 环境：centos7
* 准备：静态ip，关闭防火墙
* 安装：jdk1.8配置环境变量，zookeeper3.5.7
* 成功启动zookeeper，默认端口2181

### 支付服务入驻zookeeper

#### 新建支付模块
* cloud-provider-payment8003

#### 导入pom,zookeeper有jar冲突，需要排除
```xml
    <dependencies>
        <dependency>
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!--   引入zookeeper客户端     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.7</version>
        </dependency>
    </dependencies>
```

#### 写yml
```yml
server:
  port: 8003

# 服务别名---zookeeper注册中心名称
spring:
  application:
    name: cloud-provider-payment
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name:
    url: jdbc:mysql:///db2019?serverTimezone=UTC
    username: root
    password: 286728

  cloud:
    zookeeper:
      connect-string: 192.168.83.50:2181
      max-retries: 10
```

#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
//服务注册发现
@EnableDiscoveryClient
public class PaymentMain8003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8003.class,args);
    }
}
```

#### 写controller以方便测试
```java
package com.lhf.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentzk(){
        return "springcloud with zookeeper: " + serverPort + "\t" + UUID.randomUUID().toString();
    }
}
```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311144911.png)

此时的zookeeper服务器
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311145146.png)

### 临时节点与持久节点

#### 思考
由于zookeeper的节点分为临时和永久，那么服务节点是临时还是永久那。

#### 结论
临时节点，zookeeper在一定时间内收不到心跳，就会干掉该注册信息，重新注册是重新生成。渣男

### 订单服务入驻zookeeper

#### 新建模块
* cloud-consumerzk-order80

#### 改pom
```xml
    <dependencies>
        <dependency>
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!--   引入zookeeper客户端     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.5.7</version>
        </dependency>
    </dependencies>
```

#### 写yml
```yml
server:
  port: 8080

# 服务别名---zookeeper注册中心名称
spring:
  application:
    name: cloud-order-service
  cloud:
    zookeeper:
      connect-string: 192.168.83.50:2181
```


#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

#### config
```java
package com.lhf.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContext {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

#### controller
```java
package com.lhf.springcloud.controller;
import com.lhf.springcloud.entities.CommonResult;
import com.lhf.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {
    public static final String PYMENT_URL = "http://cloud-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/zk")
    public String create(Payment payment){
        return restTemplate.getForObject(PYMENT_URL+"/payment/zk", String.class);
    }
}

```

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311155636.png)

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311155700.png)

## consul

### 安装

之间下载下来是一个exe文件，打开cmd执行：consul agent -dev

访问：127.0.0.1:8500 查看

### 服务注册

关于服务注册，于zookeeper相同，只是将zookeeper改成了consul。

#### pom
```xml
    <dependencies>
        <dependency>
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```

#### yml
```yml
server:
  port: 8080

spring:
  application:
    name: cloud-order-service
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

#### 其他

其他的内容防照zookeeper写即可。理念相同。

#### 测试
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311223656.png)

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200311223728.png)

## 比较

|组件名|语言|CAP|服务健康检查|对外暴露接口|SpringCloud集成|
|:--|:--|:--|:--|:--|:--|
|Eureka|Java|AP|可配支持|HTTP|以集成|
|Consul|Go|CP|支持|HTTP/DNS|以集成|
|Zookeeper|Java|CP|支持|客户端|以集成|

**CAP**
* C:Consistency（强一致性）
* A:Availability（可用性）
* P:Partition tolerance（分区容错性）
* CAP理论关注粒度是数据，而不是整体系统设计

**最多只能同时较好的满足两个**
CAP理论的核心是: 一个分布式系统不可能同时很好的满足一致性,可用性和分区容错性这三个需求，因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类:
* CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
* CP：满足一致性分区容忍必的系统，通常性能不是特别高。
* AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低些


