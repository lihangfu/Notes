## 常用插件

### 1、.ignore

> 生成各种ignore文件，一键创建git ignore文件的模板



### 2、Translation

> 翻译插件



### 3、RestfulTool

> 查看所有 controller 接口



### 4、lombok

> 支持lombok的各种注解，从此不用写getter setter这些 可以把注解还原为原本的java代码



### 5、p3c

> 阿里巴巴出品的java代码规范插件
>
> 可以扫描整个项目 找到不规范的地方 并且大部分可以自动修复



## 代码模板

### 1、mapper

* File -> New -> Edit File Templates
* 选择 Files 点击 + 号
* Name ：mapper ，Extension：xml
* 内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="#[[$namespace$]]#">
</mapper>
```



### 2、类模板（自动生成类描述信息注释）

* File -> New -> Edit File Templates
* 选择 Files 找到 Class
* 内容

```java
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
/**
*@program: ${PROJECT_NAME}
*@description: ${description}
*@author: lhf
*@create: ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}
*/
public class ${NAME} {
}
```