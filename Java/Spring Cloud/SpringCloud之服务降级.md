# 介绍
在分布式架构中，面临着一个问题，那就是当体系结构复杂的时候，应用程序间的相互依赖大，每个依赖关系在某些时候将不可避免的失败。

如果在一个客户请求中，需要同时调用A服务，B服务，C服务，如果此时c服务请求超时，就会导致整个请求失败。

## 服务雪崩
多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的”雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。
所以，
通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

# Hystrix

## 是什么
Hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控(类似熔断保险丝)，**向调用方返回一个符合预期的、可处理的备选响应(FallBack) ，而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

## 能干嘛
* 服务降级
* 服务熔断
* 接近实时的监控
* 等

### 停更进入维护

### 服务降级
**FallBack**

“断路器”开关装置

当服务出现问题后，**向调用方返回一个符合预期的、可处理的备选响应(FallBack) ，而不是长时间的等待或者抛出调用方无法处理的异常。**

比如返回一条提示：服务器正忙，请稍后再试。

也就是给出一个兜底的解决方案。

#### 哪些情况触发服务降级
* 程序运行异常
* 超时
* 服务熔断触发服务降级
* 线程池/信号量打满

### 服务熔断
**break**

当服务达到最大访问后，直接拒绝访问，然后调用服务降级的方法返回友好提示。拉闸。

相当于保险丝：降级-->熔断-->恢复

### 服务限流
**flowlimit**

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟n个，有序进行。

## 怎么玩

### 测试环境搭建

#### 新建模块

* cloud-provider-hystrix-payment8001

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
        <!--   引入eureka客户端     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--   hystrix     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

    </dependencies>
```

#### yml
```yml
server:
  port: 8001

spring:
  application:
    name: cloud-provider-hystrix-payment

eureka:
  client:
    register-with-eureka: true   #是否将自己注册到注册中心,集群必须设置为true配合ribbon
    fetch-registry: true    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka # ,http://eureka7002.com:7002/eureka
```

#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class,args);
    }
}
```

#### service
```java
package com.lhf.springcloud.service.impl;

import com.lhf.springcloud.service.PaymentService;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentServiceImpl implements PaymentService {
    @Override
    public String paymentInfo_OK(Integer id) {
        return "线程池： " + Thread.currentThread().getName() + " PaymenyInfo_OK,id: " + id + "\t" + "O(∩_∩)O哈哈~";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        int timeNumber = 3000;
        try {
            TimeUnit.MILLISECONDS.sleep(timeNumber);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池： " + Thread.currentThread().getName() + " PaymenyInfo_TimeOut,id: " + id + "\t" + "O(∩_∩)O哈哈~" + " 耗时" + timeNumber + "毫秒";
    }
}
```

#### controller
```java
package com.lhf.springcloud.controller;

import com.lhf.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {
    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_OK(id);
        log.info("****result: " + result);
        return result;
    }

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_TimeOut(id);
        log.info("*****result: " + result);
        return result;
    }
}
```

#### 访问测试

* 正常访问，正常响应
* jmeter高并发访问，出现明显卡顿

* 原因：
    tomcat线程被打满，所以出现明显卡顿，而此时我们还没有使用客户端，只是服务端的自测。

#### 新建订单客户端模块

* cloud-consumer-feign-hystrix-order80

完全参照Openfeign进行服务调用支付模块

#### 测试总结

* 正常访问，正常响应
* jmeter高并发访问，出现非常明显卡顿

**总结：**
    为了应对上诉问题引起的故障和不佳表现，才有了我们的降级/容错/限流等技术诞生

### 错误记解决方案

* 超时导致服务器变慢（转圈）：超时不再等待
* 出错（宕机或程序运行出错）：出错有兜底
* 解决
    * 调用服务超时，调用者不能一直等待，必须服务降级
    * 对方服务down机了，调用者不能一直等待，必须服务降级
    * 对方服务正常，调用者出现故障或有自我要求（等待时间小于服务调用时间），自己处理降级

### 服务降级

#### @HystrixCommand
* @HystrixCommand：使用降级
    * fallbackMethod：兜底方法
    * @HystrixProperty：条件限制，超过条件的值或者报错就会兜底方法执行，最常用就是超时，防止服务器卡死。

#### 支付服务端降级处理
设置自身调用超时时间峰值，峰值内正常运行，超过了需要有兜底的方法处理，作为服务降级fallback

* 使用
```java
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public String paymentInfo_TimeOut(Integer id) {
        int timeNumber = 5000;
        try {
            TimeUnit.MILLISECONDS.sleep(timeNumber);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池： " + Thread.currentThread().getName() + " PaymenyInfo_TimeOut,id: " + id + "\t" + "O(∩_∩)O哈哈~" + " 耗时" + timeNumber + "毫秒";
    }
    public String paymentInfo_TimeOutHandler(Integer id){
        return "线程池： " + Thread.currentThread().getName() + " 8001系统繁忙系统报错,请稍后再试id: " + id + "\t" + "( Ĭ ^ Ĭ )";
    }
```

* 激活
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker //激活
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class,args);
    }
}
```

#### 订单服务端降级处理

* 照葫芦画瓢完成，由于服务端和客户端都可以使用服务降级，一般用于客户端

* yml开启feign对hystrix的支持
```yml
feign:
  hystrix:
    enabled: true
```

* 主启动开启
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}

```

* controller
```java
    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        return paymentFeignService.paymentInfo_TimeOut(id);
    }
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id){
        return "我是消费者80,对方支付系统繁忙请十秒后重试";
    }
```

#### 全局服务降级
每个服务都要配置兜底的方法，方法的耦合的太高，代码质量降低。业务逻辑跟兜底方法混在一起。

* global fallback

配置一个全局的兜底方法，没有单独配置兜底方法的全部使用该方法。

@DefaultProperties(defaultFallback="")
放于类名上方，指定全局处理方法
* 1:1   每个方法配置一个服务降级方法，不明智
* 1:N   出个个别核心业务有专属，其他普通的统一处理

**上面只是解决了代码质量问题，还需要解决代码耦合的问题**

利用feign接口解决耦合问题

该方法不但实现了解耦，还可以防止服务端宕机带来的影响。

**步骤**

* 本案例在客户端80实现，跟服务端无关，只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦
```java
package com.lhf.springcloud.service;

import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentHystrixService {
    @Override
    public String paymentInfo_OK(Integer id) {
        return "-------PaymentFallbackService fall back-paymentInfo_OK,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "-------PaymentFallbackService fall back-paymentInfo_TimeOut,o(╥﹏╥)o";
    }
}
```
* 可以解决常见的三大问题：超时，运行报错，宕机

* 在Feign客户端接口指定
```java
package com.lhf.springcloud.service;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

* 主启动添加@EnableHystrix开启

* yml开启
```yml
feign:
  hystrix:
    enabled: true
```

### 服务熔断
* 类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示。

* 就是保险丝：服务降级->进而熔断->恢复调用链路

#### 是什么
**熔断机制概述**
熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时,会进行服务的降级,进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路。**

在Spring Cloud框架里,熔断机制通过Hystrix实现。 Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值,缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是@HystrixCommand.

#### 怎么玩
服务熔断位于服务侧，我们为支付服务测添加一个方法测试服务熔断。

* service
```java
// 服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties ={
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), //是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"), //请求次数，默认20
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "1000"), //时间窗口期，默认1000
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),//失败率达到多少后跳闸，默认50
            //上述参数的意思，开启断路器，10秒内满足十次请求错误率达到60熔断，之后默认5秒断路器半开尝试访问，访问成功断路器关闭，失败继续熔断
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("******id 不能为负数");
        }
        String serialNumber = IdUtil.simpleUUID();
        return Thread.currentThread().getName()+"\t"+ "调用成功，流水号：" + serialNumber;
    }

    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，o(╥﹏╥)o id：" + id;
    }
```

* 配置controller访问该接口

* 测试

#### 总结
这里只是一个简单的入门，更多配置详细深入，请访问[官网](https://github.com/Netflix/Hystrix)

### dashboard图形化监控

#### 新建模块
* cloud-consumer-hystrix-dashboard9001

#### pom
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
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
    </dependencies>
```

#### yml
```yml
server:
  port: 9001
```

#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardMain9001.class,args);
    }
}
```

#### 被监视方主启动小修改
```java
/**
     * 此配置是为了服务监控而配置，与服务器容错本身无关，springcloud升级后的坑
     * ServletRegistrationBean因为springboot的默认路径不是/hystrix.stream
     * 只要在自己的项目里配置上下文的servlet就可以了
     */
    @Bean
    public ServletRegistrationBean getservlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```

#### 测试
浏览器：http://localhost:9001/hystrix

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200316220136.png)

