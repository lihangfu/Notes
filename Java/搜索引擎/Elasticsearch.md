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

