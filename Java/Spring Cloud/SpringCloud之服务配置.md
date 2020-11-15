# 介绍
我们将单体应用的业务拆分成一个个的子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的，动态的配置管理设施是必不可少的。

随着服务数量的增大，配置信息的修改会变得越来越麻烦。

# Config

## 是什么
SpringCloud Config为微服务架构中的微服务提高集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。

## 怎么玩
SpringCloud Config分为服务端和客户端两部分。

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容,并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并组可以通过git客户端工具来方便的管理和访问配置内容

## 能干吗
* 集中管理配置文件
* 不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
* 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取自己的配置的信息。
* 当配置发送变动时，服务不需要重启即可感知到配置的变化并应用新的配置
* 将配置信息以REST借口的形式暴露：post，curl访问刷新均可

**由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件),但最推荐的还是Git,而且使用的是http/https访问的形式**

## Config服务端配置与测试

### 新建模块

* cloud-config-center-3344

### pom
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/lihangfu/springcloud-config.git
          search-paths:
            - springcoud-config
      label: master

eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

### 测试
* http://localhost:3344/master/config-test.yml

### 配置读取规则
**只列出两种常用规则，更多参考官网**
/{lable}-{name}-{profiles}.yml

/{name}-{profiles}.yml

* label:分支（branch）
* name:服务名
* profiles:环境(dev.test/prod)

## Config客户端配置与测试

### 新建模块

* cloud-config-client-3355

### pom
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
* application.yml是用户级的资源配置项
* bootstrap.yml是系统级的，优先级更高

Spring Cloud会创建一个"Bootstrap Context" ,作为Spring应用的'Application Context'的父上下文。初始化的时候，'Bootstrap Context'负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的'Environment'。

Bootstrap 属性有高优先级,默认情况下，它们不会被本地配置覆盖。'Bootstrap context'和'Application Context'有着不同的约定,所以新增了一个bootstrap.ymI文件, 保证Bootstrap Context和Application Context配置的分离。

要将Client模块下的application.yml文件改为bootstrap.yml,这是很关键的,因为bootstrap.yml是比application.yml先加载的。 bootstrap.yml优先级高于application.yml

**bootstrap.yml**
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
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}
```

### controller
```java
package com.lhf.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

### 测试
* http://localhost:3355/configInfo
可以成功获取到配置信息

## 动态刷新
* 当github上的配置文件改变时，3344配置中心立马响应，而3355服务却不能立即响应

* 难道每次修改配置文件，都需要重新启动服务？噩梦

**修改3355模块，实现动态刷新**
* 修改pom，添加
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

* 改yml
```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

* controller加上@RefreshScope注解
```java
package com.lhf.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

**修改github配置文件后，需要手动：**
curI -X POST "http://localhost:3355/actuator/refresh"
进行刷新

### 问题
每次都要发送post更新，是不是不太方便，当然，运维可以写一个脚本，一键发送，但是我们有没有办法让，一次通知，处处生效那，那就是下一部分，服务总线得内容了。


