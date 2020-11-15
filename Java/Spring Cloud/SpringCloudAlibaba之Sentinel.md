# Hystrix的一些缺点

之前我们学习过Hystrix来进行熔断降级，那么她有着一些什么问题那。

* 需要我们自己手工搭建监控平台
* 没有一套完整的web界面可以给我们进行更加细粒度化得配置流控，速率控制，服务熔断，服务降级。。。。
# Sentinel
* 单独一个组件，可以独立出来
* 直接界面化得细粒度统一配置
* 流控，速率控制，服务熔断，服务降级。。。。

* 约定>配置>编码

# 在哪下

[官方Github](https://github.com/alibaba/sentinel)

# 能干嘛

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200322095553.png)

# 怎么玩

## 环境搭建
* 下载sentinel-dashboard
* 执行java -jar 运行该jar包
    * jdk8以上
    * 8080不能被占用
* 访问本地的8080端口成功访问

## 初始化监控

### 新建模块
* cloudalibaba-sentinel-service8401

### pom
```xml
<dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--    后续做持久化用到    -->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--   sentinel     -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--   openfeign     -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!--  web组件      -->
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

### yml
```yml
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery: #Nacos注册中心地址
        server-addr: 59.110.137.127:1111
    sentinel:
      transport: #dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认端口，如果被占用则从8719依次+1扫描
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class, args);
    }
}
```

### controller
```java
package com.lhf.springcloud.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.TimeUnit;

@RestController
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA(){
        return "---------testA";
    }

    @GetMapping("/testB")
    public String testB(){
        return "----------testB";
    }
}
```

### 测试
* 必须访问controller才能被监控到
* 访问后查看web监控端

## 流控

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200322173830.png)

* 资源名：唯一名称，默认请求路径
* 针对来源：sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）
* 阈值类型/单机阈值：
    * QPS（每秒请求量）：当调用该api的QPS达到阈值的时候，进行限流
    * 线程数：当调用该api的线程数达到阈值的时候，进行限流
* 是否集群：不需要集群
* 流控模式：
    * 直接：api达到限流条件时，直接限流
    * 关联：当关联的资源到达阈值时，就限流自己
    * 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流）【api级别的针对来源】
* 流控效果：
    * 快速失败：直接失败，抛异常
    * Warm Up：根据codeFactor（冷加载因子，默认3）的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值
    * 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效

## 测试

### 使用唯一资源名
* 阈值QPS=1,其它默认直接失败
* 关联，关联资源超标，自己限流
* 。。。。

### 流控效果
* 直接抛出异常：Blocked by Sentinel（flow limiting）
* 预热Warm Up
    * 阈值为最终要承受的QPS
    * 阈值除以冷加载因子，默认3为最初承担的QPS
    * 预热时长，经过多长时间逐渐达到阈值承受能力，防止一瞬间大量访问系统
* 排队等待：匀速排队，对应漏桶算法，间接性的大量请求

## 降级
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200322210525.png)

* RT（平均响应时间）
    * 平均响应时间，超出阈值且在时间窗口期内通过的请求>=5，两个条件同时满足后触发降级
    * 窗口期过后关闭断路器
    * RT最大4900（更大的需要通过-Dcsp.sentinel.statistic.max.rt=xxxx才能生效）
* 异常比列（秒级）
    * QPS>=5且异常比例（秒级统计）超过阈值时，触发降级；时间窗口期结束后，关闭降级
* 异常数（分钟级）
    * 异常数（分种统计）超过阈值时，触发降级；时间窗口期结束后，关闭降级

## 热点key
配置一个唯一资源名，配置某个参数得访问阈值，当请求参数达到阈值后，会报错。
尽量使用@RentinelResource来设置兜底方法，进行友好提示

### 参数例外项
我们希望当参数为某个特殊值时，它的限流值和平时不一样。
这时候可以使用参数例外项，进行设置。

## 系统配置
* 不建议使用，比较危险
* 针对系统全局设置

## @RentinelResource

* value：资源唯一名称，可作为配置的资源名
* blockHandlerClass：指定兜底方法所在的类
* blockHandler：指定配置规则兜底方法，该方法需要接收BlockException异常参数，不指定会默认调用系统默认的
* fallback：业务异常的兜底方法
* exceotionsToIgnore：数组，忽略的异常，出现该异常兜底方法不处理，自生自灭吧

### 问题
* 系统默认的，无法体现自己的业务要求
* 自定义的处理方法和业务代码耦合
* 每个业务都添加一个兜底的，代码膨胀
* 全局统一的处理的方法没有体现

### 问题解决
* blockHandlerClass：指定兜底方法所在的类

## 熔断
个人理解：
* 限流就是服务熔断触发的服务降级
* 指定业务异常的兜底方法只是单独的降级
* 同时设置降级和限流，没达到限流规则时，由业务异常的兜底方法执行，达到限流规则时，限流的兜底方法接管

## 整合Feign
```yml
feign:
    sentinel:
        enabled: true
```

## 持久化

### 需要持久化配置的模块
* cloudalibaba-sentinel-service8401

### pom
```xml
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

### yml
```yml
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery: #Nacos注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport: #dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认端口，如果被占用则从8719依次+1扫描
      datasource: #sentinel持久化
        dsl:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data_type: json
            rule-type: flow
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 然后需要在nacos添加对应的配置？？？我蒙了
我觉得应该让我的操作持久化，而不是获取配置好的。
具体如何在nacos配置


