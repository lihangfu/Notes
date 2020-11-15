## 介绍

## 入门程序

1.  pom依赖
```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

2.  编写代码
```java
        //1.创建HttpClient对象
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //2.获取get对象，参数为请求网址
        HttpGet httpGet = new HttpGet("http://www.lhf223.cn");
        CloseableHttpResponse response = null;
        try {
            //3.发出请求，获取响应数据
            response = httpClient.execute(httpGet);
            //4.判断响应状态码是否为200
            if (response.getStatusLine().getStatusCode()==200){
                //5.用utf-8解析数据
                HttpEntity entity = response.getEntity();
                String content = EntityUtils.toString(entity, "utf-8");
                System.out.println(content);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

## HttpClient

### Get
```java
        //1.创建HttpClient对象
        CloseableHttpClient httpClient = HttpClients.createDefault();
        //2.获取get对象，参数为请求网址
        HttpGet httpGet = new HttpGet("http://www.lhf223.cn");
        CloseableHttpResponse response = null;
        try {
            //3.发出请求，获取响应数据
            response = httpClient.execute(httpGet);
            //4.判断响应状态码是否为200
            if (response.getStatusLine().getStatusCode()==200){
                //5.用utf-8解析数据
                HttpEntity entity = response.getEntity();
                String content = EntityUtils.toString(entity, "utf-8");
                System.out.println(content);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

### GET带参
```java
        //获取URL
        URIBuilder uriBuilder = new URIBuilder("http://www.lhf223.cn");
        uriBuilder.setParameter("参数名","参数值");
        //获取get对象，参数为请求网址
        HttpGet httpGet = new HttpGet(uriBuilder.build());
```

### POST请求
```java
    HttpPost httpPost = new HttpPost("http://www.lhf223.cn");
```

### POST带参
```java
        //获取post对象，参数为请求网址
        HttpPost httpPost = new HttpPost("http://www.lhf223.cn");
        //声明List集合，封装表单中的参数
        List<NameValuePair> params = new ArrayList<NameValuePair>();
        params.add(new BasicNameValuePair("参数名1","参数值1"));
        params.add(new BasicNameValuePair("参数名2","参数值2"));
        //创建表单的Entity对象
        UrlEncodedFormEntity formEntity = new UrlEncodedFormEntity(params,"utf-8");
        //设置Entity对象到post中
        httpPost.setEntity(formEntity);
```

### 连接池
    如果每次请求都要创建HttpClient，会有频繁创建和销毁的问题，可以使用连接池解决。

```java
    public static void main(String[] args) {
        //创建连接池管理器
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
        //设置最大连接数
        cm.setMaxTotal(100);
        //设置每个主机的最大连接数，此主机是访问目标的主机
        cm.setDefaultMaxPerRoute(10);
        doGet(cm);
        doGet(cm);
    }

    private static void doGet(PoolingHttpClientConnectionManager cm) {
        //从连接池里获取对象
        CloseableHttpClient httpClient = HttpClients.custom().setConnectionManager(cm).build();
        HttpGet httpGet = new HttpGet("http://www.lhf223.cn");
        CloseableHttpResponse response = null;
        try {
            response = httpClient.execute(httpGet);
            if (response.getStatusLine().getStatusCode()==200){
                HttpEntity entity = response.getEntity();
                String string = EntityUtils.toString(entity, "utf-8");
                System.out.println(string.length());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if (response!=null){
                try {
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

### 参数
```java
        //配置请求信息
        RequestConfig config = RequestConfig.custom().setConnectTimeout(1000)//创建连接的最长时间，毫秒
            .setConnectionRequestTimeout(500)//设置获取连接的最长时间，毫秒
            .setSocketTimeout(10*1000)//数据传输的最长时间，毫秒
            .build();
        //设置到请求中
        httpGet.setConfig(config);
```

## Jsoup解析

### 介绍
        jsoup是一款java的html解析器，可直接解析某个url地址，html文本内容。它提供了一套非常省力的API，可以通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

### 主要功能
1.  从一个url，文件或字符串中解析html
2.  使用DOM或CSS选择器来查找，取出数据
3.  可操作HTML元素，属性，文本

### 主要依赖
```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.2</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.7</version>
</dependency>
```

### 解析URL
```java
    //解析url，第一个参数是url，第二个是访问时候的超时时间
    Document doc = Jsoup.parse(new URL("www"),1000);
    //使用标签选择器，获取标签体
    String title = doc.getElementsByTag("title").first().text();
```

        虽然使用jsoup可以替代httpclient直接发起请求解析数据，但是往往不会这么用，实际开发中，需要使用多线程，连接池，代理等等方式，而jsoup对这些的支持并不很好，所以我们一般把jsoup仅仅作为html解析工具使用。

### 解析字符串
```java
    //解析字符串
    Document doc = Jsoup.parse(content);
    String title = doc.getElementsByTag("title").first().text();
```

### 解析文件
```java
    //解析文件
    Document doc = Jsoup.parse(new File("路径"),"utf8");
    String title = doc.getElementsByTag("title").first().text();
```

### 使用Dom方式获取元素

|获取方式|获取方法|
|:--|:--|
|id|getElementById|
|标签|getElementByTag|
|class|getElementByClass|
|属性|getElementByAttribute或getElementByAttributeValue|

### 使用选择器获取元素

>doc.select("见下表");

|获取方式|举例|
|:--|:--|
|标签|span|
|id|#idname|
|class|.classname|
|属性名|[属性名]|
|属性名和属性值|[属性名=属性值]|

#### 选择器组合使用

|获取方式|举例|
|:--|:--|
|标签+id|span#idname|
|标签+class|span.classname|
|任意组合|span[属性名].classname|
|子元素|标识父 对应子，例如body div|
|直接子元素|标识父 > 对应子，例如body > div|
|所有子元素|标识父 > *，例如body > *|

### 从元素中获取数据

|获取内容|获取方法|
|:--|:--|
|获取id|id|
|获取ClassName|className或classNames|
|获取属性的值attr|attr("属性名")|
|获取所有属性的值attributes|attributes|
|获取文本内容text|text|