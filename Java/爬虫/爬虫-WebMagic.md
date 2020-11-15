## 介绍

        一款爬虫的框架，其底层所使用的就是上一篇博客所介绍的HttpClient和Jsoup，可以使我们更方便的开发爬虫。

        项目代码分为核心和扩展两部分。核心部分(webmagic-core)是一个精简的，模块化的爬虫实现，
        而扩展部分则包括一些便利的，实用性的功能。

        webmagic的设计目标是尽量模块化，并体现爬虫的功能特点。这部分提供非常简单，灵活的API，在基本不改变开发模式的情况下，编写一个爬虫。

        扩展部分(webmagic-extension)提供一些便捷的功能，例如注解模式编写爬虫等。
        同时内置了一些常用的组件，便于爬虫开发。

### 架构

**Spider是一个大的容器也是一个核心，将以下模块组织起来**

**1.    Downloader  下载**
**2.    PageProcessor  处理**
**3.    Scheduler  管理**
**4.    Pipeline  持久化**

### 流转对象

1.  Request
        Request是对URL地址的一层封装，一个Request对应一个URL地址。

        它是PageProcessor与Downloader交互的载体，也是PageProcessor控制Downlader唯一方式。

        除了URL本身外，它还包含了一个key-value结构的字段extra。可以保存特殊属性，然后在其他地方读取。例如附件上一个页面的一些信息等。

2.  Page
        Page代表了从Downloader下载到的一个页面————可能是html，也可能是json或者其它文本格式的内容。

        Page是WebMagic抽取过程的核心对象，它提供一些方法可供抽取，结果保存等。

3.  Resultltems
        Resultltems相当于一个Map，它保存PageProcessor处理的结果，供Pipeline使用。
        它的API与Map很类似，值得注意的是它有一个字段skip，若设置为true，则不应被Pipeline处理。

## 入门程序

### 坐标依赖
```xml
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-core</artifactId>
    <version>0.7.3</version>
</dependency>
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-extension</artifactId>
    <version>0.7.3</version>
</dependency>
```

>0.73版本不支持爬取只支持ssl v1.2的网站

>等待0.74版本发布或直接下载github上最新代码

### 程序
```java
public class WebTest implements PageProcessor {
    public void process(Page page) {
        page.putField("div",page.getHtml().css("css选择器"));
    }

    private Site site = Site.me();
    public Site getSite() {
        return site;
    }

    public static void main(String[] args) {
        Spider.create(new WebTest())
                .addUrl("需要解析的页面")
                .run();
    }
}
```

## 解析页面

**page.getHtml().**

1.      xpath
                使用xpath语法

2.      css     
                使用css选择器

3.      正则表达式

>三种抽取技术可以组合使用

### 获取结果

1.      get()
                返回一条String

2.      toString()
                同get(),返回一条String

3.      all()
                返回所有结果

### 获取链接

1.      links()
                获取链接，先解析页面，会获取选择器选择中的所有链接，与解析页面的方法组合使用

**获取到的链接添加进page.addTargetRequest()中循环解析**

## 数据存储
```java
public static void main(String[] args) {
        Spider.create(new WebTest())
                .addUrl("https://kuaibao.jd.com/?ids=229727844")
                .addPipeline(new FilePipeline("路径"))
                .run();
}
```
                new FilePipeline("路径"),可以自定义类实现Pipeline接口来保存数据

## 多线程
```java
public static void main(String[] args) {
        Spider.create(new WebTest())
                .addUrl("https://kuaibao.jd.com/?ids=229727844")
                .thread(线程数)
                .run();
}
```

## 配置
```java
private Site site = Site.me()
            .setCharset("utf8") //设置编码
            .setTimeOut(10000)  //设置超时时间
            .setRetrySleepTime(3000)    //设置重试的间隔时间
            .setRetryTimes(3);      //设置重试次数
```

|方法|说明|示例|
|:--|:--|:--|
|setCharset(String)|设置编码|site.setCharset("utf8")|
|setUserAgent(String)|设置UserAgent|site.setUserAgent("Spider")|
|setTimeOut(int)|设置超时时间|site.setTimeOut(3000)|
|setRetryTimes(int)|设置重复次数|site.setRetryTimes(3)|
|setCycleRetryTimes(int)|设置循环重试次数|site.setCycleRetryTimes(3)|
|addCookie(String,String)|添加一条cookie|site.addCookie("dotcomt_user","code4craft")|
|setDomain(String)|设置域名，设置域名才能添加cookie|site.setDomain("github.com")|
|addHeader(String,String)|添加一条Header|site.addHeader("Referer","https://github.com")|
|setHttpProxy(HttpHost)|设置http代理|site.setHttpProxy(new HttpHost("127.0.0.1",8080))|

### 启动配置
**Spider配置**

|方法|说明|示例|
|:--|:--|:--|
|create(PageProcessor)|创建Spider|Spider.create(new 当前类对象)|
|addUrl(String)|添加初始Url|spider.addUrl("http://lhf223.cn")|
|thread(int)|开启线程数|spider.thread(5)|
|run()|启动，会阻塞当前线程执行|spider.run()|
|start()/runAsync()|异步启动，当前线程继续执行|spider.start()|
|stop|停止爬虫|spider.stop()|
|addPipeline(Pipeline)|添加一个Pipeline，可以有多个|spider.addPipeline(new ConsolePipeline)|
|setScheduler(Scheduler)|设置Scheduler，只能有一个|spider.setScheduler(new RedisScheduler)|
|setDownloader(Downloader)|设置Downloader，只能有一个|spider.steDownloader(new SeleniumDownloader)|
|get(String)|同步调用，并直接取得结果|spider.get("网址")|
|getAll(String...)|同步调用，并直接取得一堆结果|spider.getAll("网址")|

## Scheduler

|类|说明|备注|
|:--|:--|:--|
|DuplicateRemovedScheduler|抽象基类，提供模板方法|继承它可以实现自己的功能|
|QueueScheduler|使用内存队列保存待抓取URL|比较耗费内存|
|PriorityScheduler|使用带有优先级的内存队列保存待抓取的URL|更加耗费内存，当设置request.priority之后，只能使用PriorityScheduler才可以使优先级生效|
|FileCacheQueueScheduler|使用文件保存抓取URL，可以再关闭程序下次启动时，从之前抓取到的URL继续抓取|需指定路径，会建立.urls.txt和.cursor.txt两个文件|
|RedisScheduler|使用Redis保存抓取队列，可进行多台及其同时合作抓取|需要安装并启动redis|

### URL去重

去重部分单独抽象成一个接口：DuplicateRemover，从而可以为同一个Scheduler选择不同的去重方式。

|类|说明|
|:--|:--|
|HashSetDuplicateRemover|使用HashSet来进行去重，占用内存大|
|BloomFilterDuplicateRemover|使用BloomFilter来去重，占用内存小，但是可能抓漏页面|

RedisScheduler是使用了Redis的set进行去重，其它默认使用HashSetDuplicateRemover去重。

如果使用BloomFilter，必须加入BloomFilter依赖
```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>16.0</version>
</dependency>
```
