## 入门

啊，写了好几天，这是唯一一篇根据官方文档写的，可能有很多错误，先就到这，后面有时间再做学习。

[如何创建Struts 2 Web应用程序](https://struts.apache.org/getting-started/how-to-create-a-struts2-web-application.html)

### Hello World

#### 分析

**该案例需要运行在入门时搭建的struts基础上**

在struts web应用中，当你提交表单或者跳转时，会提交到相应的动作类，这些动作类起到controller的作用，然后做出结果，可以是返回一个页面或者一组数据。

以本例为例：

1.  模型：创建一个类存储欢迎消息
2.  视图：用来展示欢迎消息
3.  控制器：用来响应请求调用相应模型到视图层展示消息
4.  创建映射（struts.xml）以耦合Action类和视图(官网说法)

根据以上分析，我们得到以下信息：

#### 实现

1.  模型类
```java
public class MessageStore {
    private String message;
    
    public MessageStore() {
        message = "Hello Struts User";
    }

    public String getMessage() {
        return message;
    }
}
```

2.  控制器
```java
public class HelloWorldAction extends ActionSupport {
    private MessageStore messageStore;

    public String execute() {
        //初始化
        messageStore = new MessageStore() ;
        
        return SUCCESS;
    }

    public MessageStore getMessageStore() {
        return messageStore;
    }
}
```
struts会创建HelloWorldAction类，当用户发出相应动作时，会调用execute方法，创建消息类对象。

3.  视图
```html
<!DOCTYPE html>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!--struts的jsp标签-->
<%@ taglib prefix="s" uri="/struts-tags" %>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h2><s:property value="messageStore.message" /></h2>
  </body>
</html>
```
messageStore.message是struts调用getMessageStore获取messageStore对象，然后获取messageStore的message的值。

<s:property>是显示该字符串

4.  映射

**struts.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
		"-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
		"http://struts.apache.org/dtds/struts-2.5.dtd">
<struts>
    <constant name="struts.devMode" value="true" />

    <package name="basicstruts2" extends="struts-default">
        <action name="index">
            <result>/index.jsp</result>
        </action>
		
        <action name="hello" class="org.apache.struts.helloworld.action.HelloWorldAction" method="execute">
            <result name="success">/HelloWorld.jsp</result>
        </action>
    </package>
</struts>
```

5. 测试
```html
<!DOCTYPE html>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="s" uri="/struts-tags" %>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Basic Struts 2 Application - Welcome</title>
    </head>
    <body>
        <h1>Welcome To Struts 2!</h1>
        <p><a href="<s:url action='hello'/>">Hello World</a></p>
    </body>
</html>
```
Struts 标签会创建一个带有hello动作的url，该动作会映射到HelloWorldAction类及其execute方法，当用户发出该url时，调用execute方法并接受到success的返回值后跳转到HelloWorld.jsp，并在该页面中获取消息展示。

**执行mvn jetty:run运行程序**

### 代码是如何工作的
1.  web.xml中加载的过滤器org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter会将用户的请求路由到*.action。该StrutsPrepareAndExecuteFilter是框架入口点。
2.  框架查找对应名称的动作，并根据映射找到对应的Action类。框架实例化该类，并调用对应方法。
3.  对应方法会返回一个string类型的字符串，框架检查映射中对应字符串对应的页面，并渲染该页面
4.  页面渲染时处理数据，获取对应数据


## JSP标签

### 网址标签

1.  普通
```html
<a href="<s:url action='hello'/>">
```

2.  带参数
```html
<s:url action="hello" var="helloLink">
  <s:param name="userName">Bruce Phillips</s:param>
</s:url>

<p><a href="${helloLink}">Hello Bruce Phillips</a></p>
```

### 表单标签

#### 输入框标签
```html
<s:form action="hello">
  <s:textfield name="userName" label="Your name" />
  <s:submit value="Submit" />
</s:form>
```

#### 下拉框标签
```html
<s:select key="personBean.sport" list="sports" />
```

key的用法参考消息资源文件内的介绍，list提供下拉的选项列表，由action类中实现并提供get方法

```html
<s:select key="personBean.residency" list="states" listKey="stateAbbr" listValue="stateName" />
```
states是一个ArrayList，每个数据都是一个对象，对象里封装了stateAbbr和stateName的get方法。
stateAbbr作为name，stateName作为展示给用户的值。


#### 单选标签
```html
<s:radio key="personBean.gender" list="genders" />
```

key与list如同下拉框标签解释

#### 复选框标签
```html
<s:checkbox key="personBean.over21" />
```

同下拉框的key解析

```html
<s:checkboxlist key="personBean.carModels" list="carModelsAvailable" />
```
carModelsAvailable是一个ArrayList，必须实现其get方法





### 属性标签
```html
<s:property value="messageStore.message" />
```

引用action的java属性，需实现对应的get方法。

### 控制标签
```html
<s:if test="personBean.over21">
    <p>You are old enough to vote!</p>
</s:if>
<s:else>
   <p>You are NOT old enough to vote.</p>
</s:else>
```

当test属性计算为true时，才会执行if内的语句，否重执行else中的语句。

### 迭代器标签
```html
<table style="margin-left:15px">
    <s:iterator value="personBean.carModels">
        <tr><td><s:property /></td></tr>
    </s:iterator>
</table>
```

value属性必须一个集合（Array，List，Map）。

s:property是集合遍历出的单个元素。有value属性，元素只有一个字段时不用指定，有多个字段时需要指定，如下：
```xml
<table style="margin-left:15px">
    <s:iterator value="states" >	
        <tr><td><s:property value="stateAbbr" /></td> <td><s:property value="stateName" /></tr>
    </s:iterator>
</table>
```





### 文字标签

**消息资源文件系列中查看具体使用**

### 主题

当使用struts标签生成html时，样式和布局由struts主题决定，struts内置了三个主题：simple，xhtml，css_xhtml，
如未指定主题，则默认使用xhtml。

关于这三个主题介绍查询官网。

要覆盖整个表单的主题，请更改表单的“主题”属性。（例如，方便将ajax主题用于特定形式。）
```html
<s:form action="save" method="post" theme="xhtml">
</s:form>
```

要支持用户选择的主题，请在用户会话中设置主题。

要更改整个应用程序的主题，请修改struts.xml。
```xml
<constant name="struts.ui.theme" value="xhtml"></constant>
```

**关于自定义主题样式的方法，可自行查询官网，我觉得用处不大**

## 动作类

### 获取参数
1.  实现与参数名相同的set方法

2.  获取表单参数时，表单name为对象.属性，类中需实现相同对象名的set方法，类中要有相同属性名的set方法。

### 表单校验

#### 校验

struts中提供了一种在动作类中进行表单校验的功能，当用户提交表单到动作类时，会自动调用动作类的validate方法。

validate方法中用if语句来进行校验，当if为true时调用addFieldError方法，该方法是从ActionSupport继承，第一个参数为有错误的字段名，第二个为错误提示。

validate方法可有多个if来分别判断提示，有任何一个错误信息被添加都不会执行execute方法，并且返回字符串input到映射中。

**映射文件**
```xml
<result name="input">/register.jsp</result>
```

#### xml方式表单校验

包含验证规则的XML文件必须命名为ActionClassName-validation.xml，位于resource与包同目录结构

```xml
<!DOCTYPE validators PUBLIC "-//Apache Struts//XWork Validator 1.0.3//EN" "http://struts.apache.org/dtds/xwork-validator-1.0.3.dtd">
<validators>
    <validator type="requiredstring">
        <param name="fieldname">personBean.firstName</param>
        <message>First name is required.</message>
    </validator>
</validators>
```

可以有一个或者多个验证节点，type指定使用Struts框架使用哪个验证器。（关于验证器，后面再介绍）
param name =“ fieldname”节点用于告诉框架将规则应用于哪个表单字段条目。
message告诉页面验证失败反馈什么消息。

**使用正则表达式验证**
```xml
<validator type="regex">
    <param name="fieldname">personBean.phoneNumber</param>
    <param name="regex"><![CDATA[\d{3}-\d{3}-\d{4}]]></param>
    <message>Phone number must be entered as 999-999-9999.</message>
</validator>
```

param name="regex"节点用于指定将应用于用户输入的正则表达式。注意正则表达式如何包含在CDATA节中。

**使用OGNL表达式验证**

**FieldExpression验证器**
```xml
<validator type="fieldexpression">
    <param name="fieldname">personBean.carModels</param>
    <param name="expression"><![CDATA[personBean.carModels.length > 0]]></param>
    <message>You must select at least one car model.</message>
</validator>
```

param name=”expression”节点包含一个OGNL表达式，其值为true或false。
如果OGNL表达式的计算结果不为true，则将不允许用户输入。



#### 错误提示

当表单校验失败返回原网页时，如果原网页是使用struts标签生成的表单，会自动根据addFieldError生成错误信息。

可以在jsp中使用<s:head />来使用样式

### 消息资源文件

额，说实话，刚看到这个标题以及描述时，我是懵逼的，我不明白这个功能的用处高明在哪里，但是为了深度的学习struts，我还是学习了下去。

#### 消息资源属性文件

按照官网的解释来说就是，创建一个与动作类同名的properties文件，该文件在resources的路径与对应动作类相同。里面放上一些键值对。

用户访问该动作类的input方法，该方法由父类实现了，默认返回input，然后转向视图，此时视图层可以调用properties中的键值对。

也就是说properties中的键值对是在动作类执行后实现的。

官网给出的视图中使用键值对的方法为：
```html
<s:textfield key="personBean.firstName"  />
```
此时key相当于name，而value相当于lable

**注意，此时这个表单页面是通过properties同名动作类的输入方法而呈现的，而非直接访问**

#### 文字标签
```html
<h3><s:text name="thankyou" /></h3>
```
上述标签可以获取properties内的对应键的值，当然也必须是从properties同名动作类跳转过来的

**Register.properties**
        thankyou=Thank you for registering %{personBean.firstName}.

%{personBean.firstName}该语句会从同名动作类中的get方法中获取personBean，再从personBean的get方法中获取firstName，拼接到properties文件中。

#### 包级别properties

上诉的properties只能在对应的同名动作类实现之后才可以被视图使用。

如果想要在多个动作类实现后的视图使用。需要使用包级别properties。

只需要在动作类所在包，或者上级目录创建package.properties

然后在视图层使用文字标签获取

该级别的属性只对，同级以及同级子目录下的动作类跳转的视图可用

#### 全局属性

在所有动作类的视图都可以使用。

在resources根目录创建properties，名称可任意。

需要在struts.xml指定为全局属性
```xml
<constant name="struts.custom.i18n.resources" value="global" />
```

此properties的键值对可在所有动作类执行之后的视图呈现

#### 国际化

**暂时没有感觉到什么实用性，不作说明，具体查询官网**

[国际化](https://struts.apache.org/getting-started/message-resource-files.html)

### 动作类映射

**使用通配符**
```xml
<action name="*Person" class="org.apache.struts.tutorials.wildcardmethod.action.PersonAction" method="{1}">
    <result name="success">view.jsp</result>
    <result name="input">input.jsp</result>
</action>
```

*号为通配符，{1}为占位符。

该映射会处理任何以Person结尾的动作，{1}会匹配与Person前的字符所对应的方法。

例如createPerson.action对应create方法，而Person.action则对应默认的execute方法。




## 异常处理

Struts 2框架，可以在struts.xml中指定该框架应如何处理未捕获的异常。

处理逻辑可以应用于所有动作（全局异常处理）或特定动作。

### 全局异常处理

要启用全局异常处理，需要在strust.xml进行配置
```xml
<global-results>
    <result name="securityerror">/securityerror.jsp</result>
    <result name="error">/error.jsp</result>
</global-results>

<global-exception-mappings>
    <exception-mapping exception="org.apache.struts.register.exceptions.SecurityBreachException" result="securityerror" />
    <exception-mapping exception="java.lang.Exception" result="error" />
</global-exception-mappings>
```

上述配置告诉struts框架，如果程先抛出了指定类型的未捕获异常（或该类型的子代）。
将返回错误指令，错误指令会映射到相应的视图上。


### 特定动作的异常处理
```xml
<action name="actionspecificexception" class="org.apache.struts.register.action.Register" method="throwSecurityException">
   <exception-mapping exception="org.apache.struts.register.exceptions.SecurityBreachException" result="login" />
   <result>/register.jsp</result>
   <result name="login">/login.jsp</result>
</action>
```

如果发送指定异常而且未被捕获，将返回一个字符串，根据字符串跳转到指定视图。

特定动作的异常处理优先级高于全局

### 纪录异常

配置struts纪录任何未捕获的异常。

```xml
<interceptors>
    <interceptor-stack name="appDefaultStack">
        <interceptor-ref name="defaultStack">
            <param name="exception.logEnabled">true</param>
            <param name="exception.logLevel">ERROR</param>
        </interceptor-ref>
    </interceptor-stack>
</interceptors>

<default-interceptor-ref name="appDefaultStack" />
```

三个参数值以启用日志记录（logEnabled），要使用的日志级别（logLevel）和要在日志消息中指定的日志类别（logCategory）。

上述的拦截器节点配置了一个名为appDefaultStack的新堆栈，基于struts的默认拦截器。

默认记录到log4j2.xml有关的日志设置。

### 显示异常

在视图里利用标签显示错误信息
```html
<h4>Exception Name: <s:property value="exception" /> </h4>

<h4>Exception Details: <s:property value="exceptionStack" /></h4>
```

## 整合spring

1.  添加插件,会自动下载spring依赖
```xml
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-spring-plugin</artifactId>
    <version>2.5.14.1</version>
</dependency>
```

2.  web.xml开启监听
```xml
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
```

3.  配置struts.xml交由spring创建action
```xml
<struts>
  <constant name="struts.objectFactory" value="spring" />
  ... 
</struts>
```

4.  配置applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    ...
</beans>

完成以上操作，在applicationContext.xml中创建action以及其它对象，struts.xml中进行映射。




