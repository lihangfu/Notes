## 介绍

## Ribbon

### 是什么
Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡**的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer (简称LB) 后面所有的机器，Ribbon会自动的帮助你基于某种规则(如简单轮询,随机连接等)去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

**目前进入了维护，但是在生产上大规模部署**

### 能干嘛

**一句话，负载均衡+RestTemplate调用**

#### LB负载均衡
简单来说就是将用户请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。

**ribbon与nginx负载均衡的区别**

Nginx是服务器负载均衡,客户端所有请求都会交给nginx,然后由nginx实现转发请求。即负载均衡是由服务端实现的。

Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

* 集中式LB
    即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件,如F5,也可以是软件, 如nginx),由该设施负责把访问请求通过某种策略转发至服务的提供方;

* 进程式LB
    将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用,然后自己再从这些地址中选择出一个合适的服务器。

    Ribbon就属于进程内LB， 它只是一个类库,**集成于消费方**进程，消费方通过它来获取到服务提供方的地址。

**之前我们在eureka的项目中使用的就是负载轮询**

### 怎么用

**由于是作用于客户端的，所以我们用订单服务作为案例**

#### pom
由于我们使用的新版eureka坐标依赖了关于，ribbon的相关jar包。无需在导入相关jar包。

* consul，zookeeper与cloud整合jar包也有相关依赖

#### config
首先我们需要写一个配置类把RestTemplate注入进spring容器内。

RestTemplate用于服务调用

注入RestTemplate的@Bean注解下再加入@LoadBalanced用于使用负载轮询。

在Controller获取RestTemplate对象进行服务调用。

#### RestTemplate

* getForObject(url,返回数据类型字节码); ： 返回对象为响应体中数据转换成的对象
* getForEntity(url,返回数据类型字节码); :  返回对象为ResponseEntity，封装了响应中的一些重要信息，比如响应头，响应状态码，响应体等

### 负载规则

**默认负载轮询**

#### IRule
IRule作为负载规则的接口，最终实现类有多种，默认为RoundRobinRule负载轮询。

|实现类|负载规则|
|:--|:--|
|RoundRobinRule|轮询|
|RandomRule|随机|
|RetryRule|先轮询，如果失败则在指定时间内重试|
|WeightedResponseTimeRule|对轮询的扩展，响应速度越快的实例选择权重越大，越容易被选择|
|BestAvailableRule|会先过滤由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务|
|AvailabilityFilteringRule|会过滤故障实例，在选择并发较小的实例|
|ZoneAvoidanceRole|默认规则，复合判断server所在区域的性能和server的可用性选择服务器|

#### 替换默认负载规则

##### 修改订单服务

##### 新建负载规则类
**注意，该类不能放在springboot的主启动类的子包下，也就是不能被@ComponentScan扫描到**

```java
package com.lhf.rule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyRule {
    @Bean
    public IRule getIRule(){
        return new RandomRule();
    }
}
```

##### 主启动添加@RibbonClient
```java
package com.lhf.springcloud;

import com.lhf.rule.MyRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;


@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);

    }
}
```

## OpenFeign

### 是什么
Feign是一个声明式WebService客户端。使用Feign能让编写WebService客户端更加简单。
它的使用方法是**定义一个服务接口在上面添加注解**。Feign也支持可拔插的编码器和解码器。Springcloud对Feign进行了封装，使其支持了springMvc标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

### 能干嘛
Feign能干什么
Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+ RestTemplate时，利用RestTemplate对http请求的封装处理,形成了一套模版化的调用方法。但是在实际开发中,由于对服务依赖的调用可能不止一处, 往往一个接口会被多处调用,所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以, Feign在此基础上做了进一步封装， 由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

**Feign集成了Ribbon**
利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

**实际开发中，可自行选择Ribbon和Feign**

### Feign与OpenFeign
|Feign|OpenFeign|
|:--|:--|
|Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端Feign内置了Ribbon,用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是: 使用Feign的注解定义接口，调用这个接口就可以调用服务注册中心的服务|OpenFeign是Spring Cloud在Feign的基础上支持了SpringMVC的注解如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。|

### 怎么玩

#### 新建模块

* cloud-consumer-feign-order80

#### 写pom
```xml
    <dependencies>
        <dependency>
            <groupId>com.lhf.com.lhf.springcloud</groupId>
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
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--   引入eureka客户端     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--  open feign      -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

    </dependencies>
```

#### 写yml
```yml
server:
  port: 8080


eureka:
  client:
    register-with-eureka: false   #是否将自己注册到注册中心,集群必须设置为true配合ribbon
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

#### 主启动开启openfeign
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

#### service使用openfeign
```java
package com.lhf.springcloud.service;

import com.lhf.springcloud.entities.CommonResult;
import com.lhf.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient("CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}

```

#### controller调用service
```java
package com.lhf.springcloud.controller;

import com.lhf.springcloud.entities.CommonResult;
import com.lhf.springcloud.entities.Payment;
import com.lhf.springcloud.service.PaymentFeignService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class OrderFeignController {
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        CommonResult<Payment> paymentById = paymentFeignService.getPaymentById(id);
        return paymentById;
    }
}
```


#### 启动测试
**feign天生集成ribbon，自带负载均衡**

### 超时控制

#### 超时演示
由于OpenFeign默认请求等待是1秒钟，超过则报错。

##### 在8001支付服务上增加一个超时方法
```java
    @GetMapping("/payment/feign/timeout")
    public String paymentFeignTimeOut(){
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return serverPort;
    }
```

##### 在订单服务请求该方法
```java
    @GetMapping("/consumer/payment/feign/timeout")
    public String paymentFeignTimeOut(){
        String s = paymentFeignService.paymentFeignTimeOut();
        return s;
    }
```
##### 超时错误
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200314153008.png)

**默认等待为一秒**

#### 超时设置
由于默认一秒，而我们的服务端可能一秒给不出结果，这时候就需要修改超时时间。

由于feign底层整合了ribbon，这里使用ribbon控制超时时间。

在yml中配置
```yml
#设置feign客户端超时时间
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

### 日志增强
Feign提供了日志打印功能，我妈可以通过配置来调整日志级别，从而了解Feign中Http请求的细节。

**对Feign接口的调用情况进行监控输出**

#### 日志级别
* NONE：默认的，不显示任何日志
* BASIC：仅记录请求方法，url，响应状态码及执行时间
* HEADERS：除了BASIC中的信息外，还有请求和响应的头信息
* FULL：除了HEADERS中的信息外，还有请求和响应的正文及元数据

#### 操作

* 在配置类中添加定义日志等级的类
```java
package com.lhf.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}

```

* yml中开启
```yml
logging:
  level:
    com.lhf.springcloud.service.PaymentFeignService: debug
```

* 控制台查看