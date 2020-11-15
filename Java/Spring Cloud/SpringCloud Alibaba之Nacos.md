# 介绍
[SpringCloudAlibaba](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)

## 为什么叫Nacos

前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。

## 是什么

一个更易于构建云原生应用的动态服务发现，配置管理和服务管理平台。

注册中心+配置中心的组合
Eureka+Config+Bus

## 能干吗

* 替代Eureka做服务注册中心
* 替代Config做服务配置中心

# 搭建环境

* [下载](https://github.com/alibaba/nacos/releases)

* 执行 nacos\bin目录下的startup.cmd

* 成功访问 http://localhost:8848/nacos/

# 服务注册

确保环境成功启动！

## 搭建服务提供者

### 创建模块

* cloudalibaba-provider-payment9001
* cloudalibaba-provider-payment9002

二者除了端口其它完全一样

### pom
```xml
    <dependencies>
        <!--  SpringCloud alibaba nacos    -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 9001
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
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
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class, args);
    }
}

```

### controller
```java
package com.lhf.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: " + serverPort + "\t id" + id;
    }

}
```

## 搭建服务消费者

### 创建模块

* cloudalibaba-provider-payment9001

二者除了端口其它完全一样

### pom
```xml
    <dependencies>
        <!--  SpringCloud alibaba nacos    -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 83
spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
service-url:
  nacos-user-service: http://nacos-payment-provider
```

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class, args);
    }
}
```

### config
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

### controller
```java
package com.lhf.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderNacosController {


    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(serverURL + "/payment/nacos/" + id, String.class);
    }
}

```


## 服务注册中心对比
![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200320184920.png)

**切换CP**
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'

# 服务配置

## 环境搭建

* 参考服务发现的环境配置

## 配置文件命名规则

**prefix-spring.profile.active.file-exetension**

* prefix默认为spring.application.name的值
* spring.profile.active即为当前环境对应的profile,可以通过配置项spring.profile.active来配置。
* file-exetension为配置内容的数据格式，可以通过配置项spring.cloud.nacos.config.file-extension来配置

**${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}**

## 怎么玩

### 新建模块
* cloudalibaba-config-nacos-client3377

### pom
```xml
    <dependencies>
        <!--   nacos config     -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--  SpringCloud alibaba nacos    -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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

### bootstrap.yml
```yml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr: localhost:8848
        file-extension: yaml
```

### application.yml
```yml
spring:
  profiles:
    active: dev
```

### 说明

关于上述两个配置文件：
* bootstrap.yml:系统级配置文件，主要配置启动所必需配置，优先加载
* application.yml:用户级配置，从远端拉取配置内容，并与bootstrap.yml组成总体的配置文件
* 远端配置文件：根据命名规则，从上述两个配置文件获取名称，并在远端创建对应命名的配置文件，例如本案例：nacos-config-client-dev.yaml

### 主启动
```java
package com.lhf.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

### controller
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

    @GetMapping("/config/info")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

### 测试与动态刷新

访问controller的内容获取远程配置信息

**远程配置一旦修改，自动刷新**

## 分类配置
* 实际开发中，通常一个系统会准备：dev（开发环境），test（测试环境），prod（生产环境），如何保证指定环境启动时服务能正确读取到相应的配置文件？

* 一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境.....，那怎么对这些微服务配置进行管理呢?

![](https://cdn.jsdelivr.net/gh/LHF1998/Blogger@master/img/20200320202542.png)

* namespace：命名空间，用于区分部署环境
* group：分组，将在同一机房的集群放在一组，可以尽可能的组内互调，提升性能
* dataid：命名

### 配置的三种选择方式

* 通过dataid选择：通过修改application的spring.profiles.active来选择
* 通过group选择：通过添加spring.cloud.nacos.config.group来选择对应分组下的文件
* 通过namespace选择：通过添加spring.cloud.nacos.config.namespace=新建命名空间生成的序号

# 集群

## 集群模式
默认的nacos使用嵌入式数据库实现数据的存储derby可以切换成mysql。

用于生产环境，有nginx集群到cacos集群，为了实现配置的统一性，还需要共同的数据库支持。

## 搭建集群

### 下载
下载linux版本的nacosserver

### 数据库准备
* 从conf目录获取数据库脚本，并执行。
* 修改conf目录下的application.properties，添加以下内容
        db.num=1
        db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?
        characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
        db.user=root
        db.password= 123456

* 启动nacos，新建配置，查看数据库测试

### 下载Linux的nacos
解压安装

### 集群配置

* 配置application.properties使用数据库
* 配置cluster.conf,添加ip:端口，ip要hostname -i的ip
* 修改starup启动脚本，自定义启动端口
    * 仿造接收参数的格式，添加一个-p接收port
    * 修改nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
    * 改为nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
* 配置nginx，由它作为负载均衡器
    * 修改nginx.conf
    * upstream cluster{server 127.0.0.1:3333;server 127.0.0.1:4444;....}
    * server{listen 1111;....}
    * server{location / {proxy_pass http://cluster;}
    * 启动：./nginx -c /usr/local/nginx/conf/nginx.conf
* 测试：启动nginx，三台nacos测试












