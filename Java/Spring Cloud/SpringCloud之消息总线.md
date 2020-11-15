# 消息总线

## Bus
什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题, 并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息

基本原理

ConfigClient实例都监听MQ中同-个topic(默认是springCloudBus)。 当-个服务刷新数据的时候,它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置。

## 是什么

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架,它整合了Java的事件处理机制和消息中间件的功能。

Spring Clud Bus目前支持RabbitMQ和Kafka.

## 能干嘛

Spring Cloud Bus能管理和传播分布式系统间的消息， 就像一个分布式执行器，可用于广播状态更改、事件推送等, 也可以当作微服务间的通信通道。

## RabbitMQ

### 环境搭建
* 安装Erlang

* 安装RibbitMQ

* 进入RibbitMQ安装目录下得sbin目录

* 输入命令，并在菜单栏找到RibbitMQ服务并启动
    rabbitmq-plugins enable rabbitmq_ management
* 访问地址查看是否成功
    http://127.0.0.1:15672/
* 输入账号密码：guest

## 动态刷新
**必须成功安装使用RibbitMQ**

### 为演示广播，再以3355为模板新建一个3366

### 设计思想
* 利用消息总线触发一个客户端/bus/refresh，而刷新所有客户端的配置
* 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

* 第二种更加符合，第一种不符合的原因如下：
    * 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责。
    * 破坏了微服务各节点的对等性。
    * 有一定局限性。例如微服务在迁移时，它的网络地址常常发生变化，此时如果想要做到自动刷新，那就会增加更多的修改。

### 怎么玩

#### 3344服务端添加总线支持
* pom
```xml
 <!--    添加消息总线RabbitMQ支持    -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

* yml
```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/exclusiver/springcloud-config.git
          search-paths:
            - springcoud-config
      label: master
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```

#### 3355,3366集群配置
* pom
```xml
        <!--    添加消息总线RabbitMQ支持    -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

* yml
```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

**主要controller上的@RefreshScope注解**

#### 刷新

curl -X POST "http://localhost:3344/actuator/bus-refresh"

此时，所有节点都会刷新。

### 定点通知

动态刷新默认是通知所有集群刷新配置，此时我们想通知某一个刷新，而不是全体，例如我们现在只想通知3355刷新。

**公式:http://localhost:配置中心的端口号/actuator/bus-refresh/微服务名:端口号**

# 消息驱动

由于MQ（消息中间件）有多种，一个系统可能存在两种MQ，会导致切换维护开发等问题。

有没有一种新的技术诞生，让我们不再关注具体MQ的细节，我么只需要用一种适配绑定的方式，自动的给我们在各种MQ内切换。

## Cloud Stream

### 是什么
屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型。

官方定义Spring Cloud Stream是一个构建消息驱动微服务的框架。

应用程序通过inputs或者outputs来与Spring Cloud Stream中binder对象交互。
通过我们配置来binding(绑定) ,而Spring Cloud Stream的binder对象负责与消息中间件交互。
所以,我们只需要搞清楚如何与Spring Cloud Stream交互就可以方便使用消息驱动的方式。
通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。
Spring Cloud Stream为- -些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。

目前仅支持RabbitMQ、Kafka。

### 设计思想

在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候,由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性通过定义绑定器作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。

过向应用程序暴露统一的Channel通道， 使得应用程序不需要再考虑各种不同的消息中间件实现。

通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。

通信方式遵循了发布-订阅模式

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200318221402.png)

* Binder：很方便的中间件，屏蔽差异
* Channel：通道，是队列的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置。
* Soure和Sink：消息的生产者和消费者。

### 注解解释
* @Input：注解标识输入通道,通过该输入通道接收到的消息进入应用程序
* @Output：注解标识输出通道，发布的消息将通过该通道离开应用程序
* @StreamL istener|：监听队列，用于消费者的队列的消息接收
* @EnableBinding：指信道channel和exchange绑定在一-起


### 消息驱动之生产者

#### 新建模块

* cloud-stream-rabbitmq-provider8801

#### pom
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
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
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

#### yml
```yml
server:
  port: 8801
spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders:  #需要绑定的rabbitmq的服务信息
        defaultRabbit: #定义的名称，用于binding整合
          type: rabbit #消息组件类型
          environment: #环境配置
            spring:
              rabbitmq:
                host: 127.0.0.1
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json
          binder: defaultRabbit #设置要绑定的消息服务的具体设置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5
    instance-id: send-8801.com
    prefer-ip-address: true
```

#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class, args);
    }
}

```

#### service
```java
package com.lhf.springcloud.service.impl;

import cn.hutool.core.lang.UUID;
import com.lhf.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;

import javax.annotation.Resource;

@EnableBinding(Source.class)
public class MessageProviderImpl implements IMessageProvider {

    @Resource
    private MessageChannel output;

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("*****serial: " +serial);
        return null;
    }
}

```

#### controller
```java
package com.lhf.springcloud.controller;

import com.lhf.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class SendMessageController {

    @Resource
    private IMessageProvider iMessageProvider;

    @GetMapping("/sendMessage")
    public String sendMessage(){
        return iMessageProvider.send();
    }
}

```

#### 登录ribbitMQ测试

### 消息驱动之消费者

#### 新建模块

* cloud-stream-rabbitmq-consumer8802

#### pom
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
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
            <groupId>com.lhf.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

#### yml
```yml
server:
  port: 8802
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders:  #需要绑定的rabbitmq的服务信息
        defaultRabbit: #定义的名称，用于binding整合
          type: rabbit #消息组件类型
          environment: #环境配置
            spring:
              rabbitmq:
                host: 127.0.0.1
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json
          binder: defaultRabbit #设置要绑定的消息服务的具体设置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5
    instance-id: receive-8802.com
    prefer-ip-address: true
```

#### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8802.class, args);
    }
}

```

#### controller
```java
package com.lhf.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController {

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("消费者1号，------>接收到的消息："+message.getPayload()+"\t port" +serverPort);
    }
}

```

#### 测试

查看控制台输出

### 重复消息

当我们消费者变成集群之后，生产者发出的消息，会被所有消费者接受，不符合我们实际场景的使用。

如果商城系统发出一个订单消息，被两个订单服务接受处理，会造成数据错乱的情况。

我们必须保证，每个消息有且只能有一个消费者消费。

**解决方法：group**

处于同一个group中的多个消费者是竞争关系，就能保证消息只会被其中一个应用消费一次。

不同组是可以全面消费的（重复消费）
同一组内会发生竞争关系，只有一个可以消费。

只需要在yml配置为同一组：
```yml
server:
  port: 8803
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders:  #需要绑定的rabbitmq的服务信息
        defaultRabbit: #定义的名称，用于binding整合
          type: rabbit #消息组件类型
          environment: #环境配置
            spring:
              rabbitmq:
                host: 59.110.137.127
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json
          binder: defaultRabbit #设置要绑定的消息服务的具体设置
          group: atguiguA   #分组避免重复消费
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2
    lease-expiration-duration-in-seconds: 5
    instance-id: receive-8802.com
    prefer-ip-address: true
```

### 持久化

分组之后的消费者自动启动持久化。

如果消费者掉线宕机，生产者发出消息，分组之后的消费者重新启动后依然可以获得消息，而没有分组的则不会。

# 链路追踪

在微服务框架中，一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

针对链路复杂，大型的微服务

## Sleuth

### 环境准备
* 下载并执行ziplin-server的jar包
* 访问127.0.0.1:9411 确认环境安装成功

### 修改支付服务8001

* pom
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

* yml
```yml
spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411

  sleuth:
    sampler:
      probability: 1 #采样率 1为全部采集
```

### 修改订单服务80

* pom
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

* yml
```yml
spring:
  zipkin:
    base-url: http://localhost:9411

  sleuth:
    sampler:
      probability: 1 #采样率 1为全部采集
```

### 测试
* http://localhost:9411/

