## 简介

SpringBoot对Spring的缺点进行的改善和优化，基于约定优于配置的思想，可以让开发人员不必在配置与逻辑业务之间进行思维的切换，全身心的投入到逻辑业务的代码编写中，从而大大提高了开发的效率。

### 特点

* 为基于Spring的开发提供更快的入门体验
* 开箱即用，没有代码生成，也无需XML配置。同时也可以修改默认值来满足特定的需求
* 提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等
* SpringBoot不是对Spring功能上的增强，而是提供了一种快速使用Spring的方式

### 核心功能

* 起步依赖
    起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依
    赖，这些东西加在一起即支持某项功能。
    简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。
* 自动配置
    Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定
    Spring配置应该用哪个，不该用哪个。该过程是Spring自动完成的。

## 快速入门

1.  pom坐标
        
    spirngboot要求项目继承springboot的起步依赖
    
    ```xml
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.1.RELEASE</version>
        </parent>
    ```

     SpringBoot要集成SpringMVC进行Controller的开发，所以项目要导入web的启动依赖
    
    ```xml
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                </dependency>
        </dependencies>
    ```

2.  编写引导类
    ```java
    //声明该类是一个springboot的引导类
    @SpringBootApplication
    public class MySpringBootApplication {
        //main方法，程序的入口
        public static void main(String[] args) {
            //run方法 标识运行springboot的引导类，参数是springboot引导类的字节码文件
            SpringApplication.run(MySpringBootApplication.class);
        }
    }
    ```

3.  编写Controller
    ```java
    @Controller
    public class StartController {
        @RequestMapping("/start")
        @ResponseBody
        public String start(){
        return "hello springboot";
        }
    }
    ```

4.  启动浏览器测试http://localhost:8080/start

### 热部署

添加热部署功能坐标,修改代码不需要重复启动项目

```xml
<!--热部署配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

**idea本身不会自动编译，需要[设置](https://www.cnblogs.com/maoxy/p/11752039.html "")** 

**idea可以快速创建springboot项目**

## 配置
SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用application.properties或者application.yml（application.yaml）进行配置。

加载顺序，后加载的会替换先加载的值
* application.yml
* application.yaml
* application.properties

**[SpringBoot配置信息查询](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#common-application-properties "")**

### yml配置

**通过空格控制层级关系，相同的空格为同一级，值之前要有空格**

1.  配置普通数据

        name: 张三

2.  对象配置

        person:
            name: 张三
            age: 18
            addr: 北京
        
        server:
            port: 8081

        map:
            key1: value1
            key2: value2

        #或者行内对象配置
        person: {name: haohao,age: 31,addr: beijing}

3.  配置数组集合

        city:
          - beijing
          - tianjin
          - shanghai
          - chongqing
        
        #或者行内数组
        city: [beijing,tianjin,shanghai,chongqing]

        #集合中的元素是对象形式
        student:
          - name: zhangsan
            age: 18
            score: 100
          - name: lisi
            age: 28
            score: 88
          - name: wangwu
            age: 38
            score: 90

### 获取自定义的配置信息

1.  @Value("${kye}")

2.  @ConfigurationProperties(prefix="配置文件中的key的前缀")可以将配置文件中的配置自动与实体进行映射

        该注解用于类上，对应属性要实现getset方法，属性名与配置相同

## 整合

### 整合Mybatis

1.  pom依赖
```xml
<!--mybatis起步依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
</dependency>
<!-- MySQL连接驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2.  添加数据库连接信息application.properties
```
    #DB Configuration:
    spring.datasource.driverClassName=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?
    useUnicode=true&characterEncoding=utf8
    spring.datasource.username=root
    spring.datasource.password=root

    #spring集成Mybatis环境
    #pojo别名扫描包
    mybatis.type-aliases-package=com.itheima.domain
    #加载Mybatis映射文件
    mybatis.mapper-locations=classpath:mapper/*Mapper.xml
```

### 整合Junit

1.  pom依赖
```xml
<!--测试的起步依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

2.  测试类上使用注解
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = MySpringBootApplication.class)   //引导类
```