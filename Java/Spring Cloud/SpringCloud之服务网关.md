# 介绍
所有微服务的入口。
起到代理，权限，限流，熔断，日志监控等功能。

# Getewat

## 是什么
Geteway是在spring生态系统之上构建的API网关服务，基于Spring5，SpringBoot2和ProjectReactor等技术。

Getewat旨在提供一种简单有效的方式对API进行路由，以及提供一些强大的过滤器功能，例如：熔断，限流，重试等。

SpringCloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Zuul,在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

Spring Cloud Gateway的目标提供统一的路由方式且基于Filter链的方式提供了网关基本的功能，例如:安全，监控/指标，和限流。

* zuul1：基于servlet之上的一个阻塞式处理模型

在servlet3.1之后有了异步非阻塞的支持。而WebFlux是一个典型非阻塞异步的框架，其核心是基于Reactor的相关api实现的。相对与传统的web框架，他可以运行在诸如netty，undertow及支持serblet3.1的容器上。非阻模式+函数式编程。

Spring WebFlux是Spring 5.0引入的新的响应式框架,区别于Spring MVC,它不需要依赖Servlet API, 它是完全异步非阻塞的,并且基于Reactor来实现响应式流规范。

Spring Cloud Gateway底层基于WebFlux框架，能够更好的应对高并发，提高系统性能。

## 三大核心

### Route（路由）

* 路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

### Predicate（断言）

* 参考的是Java8的java.util.function.Predicate
* 开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），**如果请求与断言相匹配则进行路由**

### Filter（过滤）
* 指的是Spring框架中GetewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

### 总结
请求到达网关后，进行断言，断言为true进入一系列过滤器，然后根据目标uri，进行路由。
* 转到哪
* 能不能转

**匹配方式就叫断言，实现这个匹配方式就叫filter，对外表现出来就是路由的功能。同一件事的三个不同维度的描述。**

## 执行流程
客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由,将其发送到Gateway Web Handler。Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前( "pre" )或之后( "post" )执行业务逻辑。

Filter在"pre" 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、 协议转换等，

在"post" 类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

**路由转发+执行过滤器链**

## 怎么玩

### 新建网关模块
* cloud-gateway-gateway9527

### pom
**网关不需要web模块**
```xml
<dependencies>
        <!--   gateway     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
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

### yml
```yml
server:
  port: 9527
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class,args);
    }
}
```

### yml配置路由
```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh
          uri: http://localhost:8001
          predicates:
            - Path=/payment/get/**
        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**
```
**uri跟path拼接就是对方路径**

### 配置类配置路由（可选不推荐）
```java
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GateWayConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){

        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_route"
                , r->r.path("/guonei").uri("http://news.baidu.com/guonei"))
                .build();
        return routes.build();
    }
}
```

### 动态路由
**默认情况下Gateway会根据注册中心注册的微服务名为路径创建动态路由转发，从而实现动态路由功能，lb前缀负载均衡**
```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启注册中心路由功能
#          lower-case-service-id: true
      routes:
        - id: payment_routh
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service #此处如果有问题，请注意依赖spring-cloud-starter-netflix-eureka-client依赖不能错
          predicates:
            - Path=/payment/get/**
        - id: payment_routh2
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**
```

### 断言分类
**[predicates](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/#gateway-request-predicates-factories)**

下面列举一些常用的，更多的查看官方文档

|断言|实例|备注|
|:--|:--|:--|
|After|2020-02-21T15:51:37.485+08:00[Asia/Shanghai]|什么时间之后才可以访问|
|Before|2020-02-21T15:51:37.485+08:00[Asia/Shanghai]|什么时间之前可以访问|
|Between|时间一,时间二|什么时间之间可以访问|
|Coolie|cookie name,正则表达式|携带指定cookie键值对|
|Header|请求头,正则表达式|携带指定的请求头|
|Host|\**.somehost.org,**.anotherhost.org|:--|
|Method|GET,POST|只有对应的请求才能访问|
|Path|/test|指定路径才能访问|
|Query|参数名,正则表达式|指定参数才能访问|
|RemoteAddr|192.168.1.1/24|指定网络地址才能访问|

### Filter

#### 介绍
路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应,路由过滤器只能指定路由进行使用。

Spring Cloud Gateway内置了多种路由过滤器，他们都由GatewayFilter的工厂 类来产生

#### 分类

* 声明周期：pre（前置），post（后置）
* 种类：GatewayFilter（单一），GlobalFilter（全局）

单一全局有几十种，详细使用查看官网。

#### 自定义全局过滤器

* 实现GlobalFilter,Ordered两个接口

* 功能：全局日志纪录，统一网关鉴权等

```java
package com.lhf.springcloud.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;


@Slf4j
@Component
public class MyGatewayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        Date date = new Date();
        log.info("*********"+date);
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname==null){
            log.info("*******非法用户名");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 1;
    }
}
```

