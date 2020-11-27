## 1.ElasticSearch 简介



### 1.1 Lucene

> Lucene 是一个开源、免费、高性能、纯 Java 编写的全文检索工具包，Lucene 开发算是开源领域最好的全文检索工具包。

需要注意的是，Lucene 只是一个全文检索工具包，并非一个完整的搜索引擎。开发者可以基于Lucene开发出自己的搜索引擎。比较著名的、现成的解决方案有 Solr、在目前分布式和大数据环境下，ElasticSearch 更胜一筹。

Lucene 主要有以下特点：

* 使用简单
* 跨语言
* 强大的搜索引擎
* 索引速度快
* 索引文件兼容不同平台



### 1.2 ElasticSearch

> ElasticSearch 是一个分布式、可扩展、近实时性的高性能搜索引擎与数据分析引擎。ElasticSearch 也是基于 Java b编写，通过进一步封装 Lucene，将搜索的复杂性屏蔽起来，开发者只需要一套简单的 RESTful API 就可以操作全文检索。

ElasticSearch 在分布式环境下表现优异，这也是它受欢迎的原因之一，它支持 PB 级别的结构化或者非结构化数据的海量处理。

整体来说，ElasticSearch 有三大功能：

* 数据收集
* 数据分析
* 数据存储

ElasticSearch 主要有以下特点：

1. 分布式实时文件存储。
2. 实时分析的分布式搜索引擎。
3. 高可扩展的分布式搜索引擎。
4. 可插拔的插件支持。



## 2.ElasticSearch 安装

### 2.1 单节点安装

首先打开 Es 官网，找到 ElasticSearch：

* https://www.elastic.co/cn/downloads/elasticsearch

解压下载文件，解压后的目录含义如下：

| 目录    | 含义           |
| ------- | -------------- |
| modules | 依赖模块目录   |
| lib     | 第三方依赖库   |
| logs    | 输出日志目录   |
| plugins | 插件目录       |
| bin     | 可执行文件目录 |
| config  | 配置文件目录   |
| data    | 数据存储目录   |

启动方式：

* 进入到 bin 目录下，执行 elasticsearch 可执行文件
* 看到 satrted 表示启动成功。
* 默认监听的端口是 9200，浏览器访问可查看节点信息

节点名称以及集群的名字，可以自定义配置。config/elasticsearch.yml

```yaml
cluster.name: lhf
node.name: master
```

重启配置生效。



### 2.2 HEAD 插件安装

ElasticSearch-head 插件，可以通过可视化的方式查看集群信息。



#### 2.2.1 浏览器插件安装

浏览器插件商店搜索：Elasticsearch-head 进行安装



#### 2.2.2 下载插件安装

* git clone git://github.com/mobz/elasticsearch-head.git
* cd elasticsearch-head
* npm install
* npm run start
* open http://localhost:9100/

打开后发现没有数据，因为跨域的问题，修改 Es 配置文件：

```yaml
http.cors.enabled: true
http.cors.allow-orgin: "*"
```

修改配置后重启 Es 就可以获取数据了。



### 2.3 分布式安装

* 一主二从
* master：9200，slave：9201和9202

master 配置：

```yaml
node.master: true
network.host: 127.0.0.l
```

复制两份 Es 安装文件到不同目录，并修改其配置文件：

```yaml
cluster.name: lhf		# 相同集群下必须相同，通过名称寻找主机
node.name: slave01		# slave01 and slave02
network.host: 127.0.0.1
http.port: 9201			# 默认 9200
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]	# 主机地址
```

分别启动 slave01 和 slave02



### 2.4 Kibana 安装

Kibana 是一个 Elastic 公司推出的一个针对 es 的分析以及数据可视化平台，可以搜索、查看存放在 es 中的数据。

安装步骤如下：

1. 下载 Kibana https://www.elastic.co/cn/kibana
2. 解压
3. 配置 es 地址信息（如果 es 是默认地址端口，可不配置）
4. 执行 /bin/kibana 启动
5. 访问 localhost:5601



## 3. ElasticSearch 核心概念

### 3.1 集群（Cluster）

> 一个或者多个安装了 es 节点的服务器组织在一起，就是集群，这些节点共同持有数据，共同提供搜索服务。

一个集群有一个名字，这个名字是集群的唯一标识，该名字成为 cluster name，默认的集群名称是 elasticsearch，具有相同名称的节点才会组成一个集群。

可以在 config/elasticsearch.yml 文件中配置集群名称：

```yml
cluster.name: lhf
```

在集群中，节点的状态有三种：绿色、黄色、红色：

* 绿色：节点运行状态为健康状态。所有的主分片、副本分片都可以正常工作。
* 黄色：节点运行状态为警告状态，所有的主分片目前都可以直接运行，但是至少有一个副本分片是不能工作的。
* 红色：集群无法正常工作。

### 3.2 节点（Node）

> 集群中的一个服务器就是一个节点，节点中会存储数据，同时参与集群的索引以及搜索功能。一个节点想要加入一个集群，只需要配置一下集群的名称即可。默认情况下，如果我们启动了多个节点，多个节点还能彼此发现，自动组成一个集群，这是 es 默认提供的，但是这种方法并不可靠，有可能会发生脑裂现象。所以实际应用中，建议一定手动配置一下集群信息。



### 3.3 索引（Index）

* 名词：具有相似特征文档的集合。
* 动词：索引数据以及对数据进行索引操作。



 ### 3.4 类型（Type）

> 类型是索引上的分类或者分区。在 Es6 中之前，一个索引中可以有多个类型。从 Es7 开始，一个索引中只能有一个类型。在 Es 6.x 中，依然保持了兼容，依然支持单索引多类型结构，但是已经不建议这么使用。



### 3.5 文档（Document）

> 一个可以被索引的数据单元。例如一个用户的文档，一个产品的文档等等。文档都是 JSON 格式的。



### 3.6 分片（Shards）

> 索引都是存储在节点上的，但是受限于节点的空间大小以及数据处理能力，单个节点的处理效果可能不理想，此时我们可以对索引进行分片。当我们创建一个索引的时候，就需要指定分片的数量。每个分片本身也是一个功能完善并且独立的索引。
>
> 默认情况下，一个索引会自动创建 5 个分片，并且为每一个分片创建一个副本。



### 3.7 副本（Replicas）

> 副本也就是备份，是对主分片的一个备份。



### 3.8 Settings

> 集群中对索引的定义信息，例如索引的分片数、副本数等等。



### 3.9 Mapping

> Mapping 保存了定义索引字段的存储类型、分词方式、是否存储等信息。



### 3.10 Analyzer

> 字段分词方式的定义



### 3.11 ElasticSearch Vs 关系型数据库

| 关系型数据库         | ElasticSearch                 |
| -------------------- | ----------------------------- |
| 数据库               | 索引                          |
| 表                   | 类型                          |
| 行                   | 文档                          |
| 列                   | 字段                          |
| 表结构               | 映射                          |
| SQL                  | DSL(Domain Specific Language) |
| select * from xxx    | GET http://                   |
| update xxx set xx=xx | PUT http://                   |
| delete xxx           | DELETE http://                |
| 索引                 | 全文索引                      |



## 4. ElasticSearch 分词器

### 4.1 内置分词器

> ElasticSearch 核心功能就是数据检索，首先通过索引将文档写入 es。查询分析则分为两个步骤：
>
> 1. 词条化：分词器将输入的文本转为一个一个的词条流。
> 2. 过滤：比如停用词过滤器会从词条这去除不相干的词条（的，嗯，啊，呢）；另外还有同义词过滤器、小写过滤器等。

ElasticSearch 中内置了多种分词器可以供使用：

| 分词器               | 作用                                                     |
| -------------------- | -------------------------------------------------------- |
| Standard Analyzer    | 标准分词器，适用于英语等。                               |
| Simple Analyzer      | 简单分词器，基于分字母字符进行分词，单词会被转为小写字母 |
| Whitespace Analyzer  | 空格分词器，按照空格进行切分                             |
| Stop Analyzer        | 类似于简单分词器，但是增加了停用词的功能。               |
| Keyword Analyzer     | 关键词分词器，输入文本等于输出文本                       |
| Pattern Analyzer     | 理由正则表达式对文本进行切分，支持停用词                 |
| Language Analyzer    | 针对特定语言的分词器                                     |
| Fingerprint Analyzer | 指纹分析仪分词器，通过创建标记进行重复检测。             |



### 4.2 中文分词器

#### 4.2.1 安装

在 Es 中，使用较多的中文分词器是 [elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)，这是个 es 第三方插件，代码托管在 GitHub 上。

具体使用步骤如下：

1. 首先打开 https://github.com/medcl/elasticsearch-analysis-ik
2. 在 https://github.com/medcl/elasticsearch-analysis-ik/releases 找到对应版本的压缩包
3. 下载文件解压到 es/plugins/ik 目录下（需新建ik目录）
4. 重启 es 服务（日志中可以看到 ik 插件）

更多使用方法，可在官网查看。



#### 4.2.2 测试

1. 创建索引

* 发送请求 PUT http://localhost:9200/test
* 返回值

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "test"
}
```



2. 分词测试

* 发送请求 POST http://localhost/test/_analyze
* 请求参数 JSON

```json
{
    "analyzer": "ik_smart",
    "text": ""
}
```



#### 4.2.3 自定义扩展词库

##### 4.2.3.1 本地自定义

在 plugins/ik/config 目录下，新建 ext.dic 文件（文件名任意），在该文件中可以配置自定义词库。

* 新建 ext.dic 文件，并写入一个词`我的是`，多词换行写入
* 在 IKAnalyzer.cfg.xml 文件中配置

```xml
<entry key="ext_dict">ext.dic</entry>
```

* 重启 Es



##### 4.2.3.2 远程词库

也可以配置远程词库，支持热更新（无需重启 es）

热更新需要提供一个接口，接口返回扩展词即可。

* 可以使用 nginx 或者其它 http 服务器
* 例如使用 springboot 放入一个静态文件，在 IKAnalyzer.cfg.xml 中配置访问该静态文件
* 修改静态文件的时候需要重启 http 服务

该 http 请求需要返回两个头部(header)，一个是 `Last-Modified`，一个是 `ETag`，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库

具体细节 https://github.com/medcl/elasticsearch-analysis-ik



## 5. ElasticSearch 索引管理



### 5.1 新建索引

#### 5.1.1 通过 head 插件新建索引

在 head 插件中，选择索引选项卡，然后点击新建索引。新建索引时，需要填入索引名称、分片数以及副本数。

索引创建成功后可以在 head 插件中看到。

插件中数字代表分片，粗框代表主主分片，细框代表副本（点一下框，可以通过 primary 属性查看是主分片还是副本）。Kibana 只有一个分片和副本，所以只有 0。



#### 5.1.2 通过请求创建

* 可以通过 postman 发送请求，也可以通过 kibana 发送请求。kibana 有提示，利于编写。

* kibana

```
PUT book 			//创建索引，kibana 省略了前面的地址信息
```

* psotman

```
http://localhost:9200/book				// PUT请求
```



**注意：**

1. 索引名称不能有大写字母
2. 索引名不可重复



### 5.2 更新索引

#### 5.2.1 修改基础信息

创建索引后可修改其信息

* kibana

```
PUT book/_settings
{
	"number_of_replicas": 2		// 修改副本数
}
```



#### 5.2.2 修改索引的独写权限

索引创建成功后，可以向索引写入文档

* kibana

```
PUT book/_doc/1				// 1 文档 id
{
	"title": "三国演义"		// 
}
```

默认情况下，索引是具有独写权限的，当然这个独写权限也可以关闭。

例如关闭索引的写权限：

```
PUT book/_settings
{
	"blocks.write": true
}
```

关闭了之后，索引将无法写入文档，改成 false 可重新打开。

其它类似的权限有：

* blocks.write
* blocks.read
* blocks.read_only



### 5.3 查看索引

* 通过 head 插件查看索引信息
* 通过 kibana 查看

```
GET book/_settings
```

也可以同时查看多个索引信息：

```
GET book,test/_settings
```

也可以查看所有索引信息：

```
GET _all/_settings
```

### 5.4 删除索引

* head 插件可以删除
* kibana 删除

```
DELETE book
```



### 5.5 索引的打开/关闭

* 关闭

```
POST book/_close
```

* 开启

```
POST book/_open
```

同时关闭或打开多个或者所有索引：

```
POST book,test/_close	// 多个
POST _all/_close		// 所有
```



### 5.6 索引复制

> 索引复制只会复制数据，不会复制配置（分片和副本）

* kibana

```
POST _reindex
{
	"source": {"index": "book"},		// 要复制的索引
	"dest": {"index": "book_new"}		// 复制到新的索引
}
```

复制的时候，可以添加查询条件：

```
POST _reindex
{
	"source": {"index": "book","query":{}},		// 复制符合查询条件的数据
	"dest": {"index": "book_new"}		// 复制到新的索引
}
```

具体查询条件编写规则，在后面学习。



### 5.7 索引别名

可以为索引创建别名，如果这个别名是唯一的，该别名可以代替索引名称。

* 创建别名

```json
POST /_aliases
{
	"action": [{				// 数组，可同时操作多个
        "add": {
            "index": "book",	// 索引
            "alias": "book_alias"	// 别名
        }
    }]
}
```

* 移除别名

```json
POST /_aliases
{
	"action": [{
        "remove": {
            "index": "book",	// 索引
            "alias": "book_alias"	// 别名
        }
    }]
}
```

* 查询别名

```
GET /book/_alias		// 查询索引的别名
GET /book_alias/_alias	// 查询别名的索引
```

* 查询所有别名

```
GET /_alias
```



## 6. ElasticSearch 文档基本操作

