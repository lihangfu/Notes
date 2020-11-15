## Swagger简介

## 前后端分离

Vue + SpringBoot

后端时代：前端只用管理静态页面；html==>后端。模板引擎 JPS => 后端是主力



前后端分离式时代：

* 后端：后端控制层，服务层，数据访问层
* 前端：前端控制层，视图层
  * 伪造后端数据进行测试
* 前后端如何交互？===>API
* 前后端相对独立，松耦合；
* 前后端分开部署，无需在一个服务器。



问题：

* 前后端集成联调，前端人员与后端人员无法做到“及时协商，尽早解决”，最终导致问题集中爆发；

解决方案：

* 首先指定计划的提纲，实时更新最新API，降低集成的风险；
* 早些年：制定word计划文档；
* 前后端分离：
  * 前端测试后端接口：postman
  * 后端提供接口：需要实时更新最新的消息及改动！



## Swagger

* 号称世界上最流行的Api框架；
* RestFul Api 文档在线自动生成工具 => Api文档与Api定义同步更新
* 直接运行，可以在线测试Api接口！
* 支持多种语言



**官网：** https://swagger.io/ 



在项目中使用Swagger需要 springbox；

* swagger2
* ui



## SpringBoot集成Swagger

1. 新建一个SpringBoot的Web项目

2. 导入相关依赖

3. 编写一个HelloWorld的接口

4. 编写SwaggerConfig

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig {
       
   }
   ```

   





## 配置Swagger

Swagger

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean		//获取swagger的docket实例
    public Docket webApiConfig(){
        return new Docket(DocumentationType.SWAGGER_2)		//分析源码，使用最新版
                .groupName("webApi")						//基础信息
                .apiInfo(webApiInfo())
                .select()									//开启扫描接口配置，最后必须build
                .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                .paths(Predicates.not(PathSelectors.regex("/error.*")))
                .build();
    }

    private ApiInfo webApiInfo(){						    //配置基础信息
        return new ApiInfoBuilder()
                .title("文档title")
                .description("文档描述")
                .version("1.0")		//版本
                .contact(new Contact("java", "http://lhf223.cn", "280001404@qq.com")) //作者信息
                .build();
    }
}
```





## 配置扫描接口及开关

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean		//获取swagger的docket实例
    public Docket webApiConfig(){
        return new Docket(DocumentationType.SWAGGER_2)		//分析源码，使用最新版
                .groupName("webApi")						//基础信息
                .apiInfo(webApiInfo())
                .select()									//开启扫描接口配置，最后必须build
            	//RequestHandlerSelectors:配置扫描接口的方式
                    //basePackage:指定要扫描的包
                    //any():扫描全部
                    //none():不扫描
                    //withClassAnnotation:扫描类上的注解，需要提供注解类字节码文件
            		//withMethodAnnotation:扫描方法上的注解，需要提供注解类字节码文件
            	.apis(RequestHandlerSelectors.basePackage("包路径"))
            	//paths:过滤，不扫描哪些
                .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                .paths(Predicates.not(PathSelectors.regex("/error.*")))
                .build();
    }
```



**配置多个分组；创建多个Docket实例即可**



```
docker run -d --name=kong-database --network=kong-net -p 5432:5432 -e POSTGRES_USER=kong -e POSTGRES_DB=kong postgres:9.6 
```

