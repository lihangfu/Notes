## 什么是全文检索
        非结构化的数据建立索引，然后根据索引找到索引所在文件的这么一个过程。
        索引一次创建可以多次使用。可以加快查询速度。

**多用于搜索引擎，站内搜索，只要有搜索的场景就可以使用全文检索**

## Lucene

**Lucene是一个基于Java开发全文检索工具包**

### 流程

1. 创建索引
    1.  获得文档
            原始文档：要基于哪些数据来进行搜索，那么这些数据就是原始文档。
            搜索引擎：使用爬虫获得原始文档。
            站内搜索：数据库中的数据。

    2.  构建文档对象
            对应每个原始文档创建一个document对象，每个网页啊，每条数据啊。

            每个document对象中包含多个域（field），域中保存的就是原始文档的数据。
                域的名称
                域的值

            每个文档都有一个唯一的编号，就是文档id
    
    3.  分析文档
            就是分词的过程

            按规则进行划分，不区分大小写，去除标点符号，去除无意义词也叫停用词

            每个关键词都封装成一个Term对象，其中包含：
                关键词所在的域
                关键词本身
            
            不同的域中拆分出相同的关键词是不同的Term

    4.  创建索引
            基于关键词列表创建一个索引，保存到索引库中。
            索引库：
                    索引
                    document对象
                    关键词和文档对应关系

    **通过词语找文档，这种索引的结构叫倒排索引结构**

2.  查询索引
    1.  用户查询接口
            输入查询条件的地方，例如搜索框
        
    2.  把查询内容封装成一个查询对象
            要查询的域
            要搜索的关键词
    
    3.  执行查询
            根据要查询的关键词到对应的域上进行搜索

            找到关键词，根据关键词找到对应文档
    4.  结果渲染
            根据文档id找到文档对象

            对关键词进行高亮显示

            分页处理等

### 快速入门

1.  pom坐标

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-core</artifactId>
        <version>7.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-analyzers-common</artifactId>
        <version>7.4.0</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>
```

2.  创建索引库
    1.  创建Director对象，指定索引库保存的位置
    2.  基于Director对象创建一个IndexWriter对象
    3.  读取磁盘上的文件，对应每个文件创建一个文档对象
    4.  向文档对象中添加域
    5.  把文档对象写入索引库
    6.  关闭indexWriter对象

```java
        //1.创建Director对象，指定索引库保存的位置
        Directory directory = FSDirectory.open(new File("索引文件保存路径").toPath());
        //2.基于Director对象创建一个IndexWriter对象
        IndexWriter indexWriter = new IndexWriter(directory,new IndexWriterConfig());
        //3.读取磁盘上的文件，对应每个文件创建一个文档对象
        File dir = new File("原始文档路径");
        File[] files = dir.listFiles();
        for (File f:files){
            //获取文件名
            String fileName = f.getName();
            //获取文件路径
            String filePath = f.getPath();
            //获取文件内容
            String fileContent = FileUtils.readFileToString(f,"utf-8");
            获取文件大小
            long fileSize = FileUtils.sizeOf(f);
            //创建域对象
            Field fieldName = new TextField("name",fileName,Field.Store.YES);
            Field fieldPath = new TextField("path",filePath,Field.Store.YES);
            Field fieldContent = new TextField("content",fileContent,Field.Store.YES);
            Field fieldSize = new TextField("size",fileSize+"",Field.Store.YES);
            //创建文档对象并添加域
            Document document = new Document();
            document.add(fieldName);
            document.add(fieldPath);
            document.add(fieldContent);
            document.add(fieldSize);
            //添加文档对象
            indexWriter.addDocument(document);
        }
        //关闭
        indexWriter.close();
```

3. 查询索引库
    1.  创建一个Director对象，指定索引库的位置
    2.  创建一个IndexReader对象
    3.  创建一个IndexSearcher对象，构造方法中的参数是IndexReader对象
    4.  创建一个Query对象，得到一个TermQuery
    5.  执行查询，得到一个TopDocs对象
    6.  取查询结果的总记录数
    7.  取文档列表
    8.  打印文档内容
    9.  关闭IndexReader对象

```java
        //1.创建一个Director对象，指定索引库的位置
        Directory directory = FSDirectory.open(new File("索引文件保存路径").toPath());
        //2.创建一个IndexReader对象
        IndexReader indexReader = DirectoryReader.open(directory);
        //3.创建一个IndexSearcher对象，构造方法中的参数是IndexReader对象
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);
        //4.创建一个Query对象，得到一个TermQuery
        Query query = new TermQuery(new Term("content","spring"));
        //5.执行查询，得到一个TopDocs对象
        TopDocs topDocs = indexSearcher.search(query, 10);
        //6.取查询结果的总记录数
        System.out.println("查询的总记录数："+topDocs.totalHits);
        //7.取文档列表
        ScoreDoc[] scoreDocs = topDocs.scoreDocs;
        //8.打印文档内容
        for(ScoreDoc doc:scoreDocs){
            int docId = doc.doc;
            Document document = indexSearcher.doc(docId);
            System.out.println(document.get("name"));
            System.out.println(document.get("path"));
            System.out.println(document.get("size"));
            System.out.println(document.get("content"));
        }
        //9.关闭IndexReader对象
        indexReader.close();
```

### 中文分析器

#### 分析器

**默认使用标准分析器StandardAnalyzer**

1.  查看分析器分析结果
        使用Analyzer对象的tokenStream方法返回一个TokenStream对象。词对象中包含了最终分词结果。
    
```java
        //1.创建一个Analyzer对象，StandardAnalyzer对象
        Analyzer analyzer = new StandardAnalyzer();
        //2.使用分析器的tokenStream方法获得一个TokenStream对象
        TokenStream tokenStream = analyzer.tokenStream("", "要分析的字符串");
        //3.向tokenStream对象中设置一个引用，相当于一个指针
        CharTermAttribute charTermAttribute = tokenStream.addAttribute(CharTermAttribute.class);
        //4.调用tokenStream的reset方法。如果不调用抛异常
        tokenStream.reset();
        //5.使用while循环遍历tokenStream对象
        while (tokenStream.incrementToken()){
            System.out.println(charTermAttribute.toString());
        }
        //6.关闭tokenStream对象
        tokenStream.close();
```

**标准分析器对中文无法分析成相应的词语**

#### 中文分析器

**使用的是IKAnalyzer**

Github地址：https://github.com/magese/ik-analyzer-solr

>导入对应坐标，复制出配置文件并进行修改IKAnalyzer.cfg.xml

在创建索引时如何使用：

```java
//1.创建Director对象，指定索引库保存的位置
        Directory directory = FSDirectory.open(new File("索引文件保存路径").toPath());
        //2.基于Director对象创建一个IndexWriter对象
        IndexWriter indexWriter = new IndexWriter(directory,new IndexWriterConfig(new IKAnalyzer()));  //此处修改分析器
        //3.读取磁盘上的文件，对应每个文件创建一个文档对象
        File dir = new File("原始文档路径");
        File[] files = dir.listFiles();
        for (File f:files){
            //获取文件名
            String fileName = f.getName();
            //获取文件路径
            String filePath = f.getPath();
            //获取文件内容
            String fileContent = FileUtils.readFileToString(f,"utf-8");
            获取文件大小
            long fileSize = FileUtils.sizeOf(f);
            //创建域对象
            Field fieldName = new TextField("name",fileName,Field.Store.YES);
            Field fieldPath = new TextField("path",filePath,Field.Store.YES);
            Field fieldContent = new TextField("content",fileContent,Field.Store.YES);
            Field fieldSize = new TextField("size",fileSize+"",Field.Store.YES);
            //创建文档对象并添加域
            Document document = new Document();
            document.add(fieldName);
            document.add(fieldPath);
            document.add(fieldContent);
            document.add(fieldSize);
            //添加文档对象
            indexWriter.addDocument(document);
        }
        //关闭
        indexWriter.close();
```

### 索引库的维护

#### 添加索引

##### Field域的属性

* 是否分析：是否对域的内容进行分词处理。前提是我们要对域的内容进行查询
* 是否索引：将Field分析后的词或整个Field值进行索引，只有索引方可搜索到
* 是否存储：将Field值存储在文档中，存储在文档中的Field才可以从Document中获取

**是否存储的标准：是否要将内容展示给用户**

|Field类|数据类型|Analyzed是否分析|Indexed是否索引|Stored是否存储|说明|
|:-|:-|:-|:-|:-|:-|
|StringField(FieldName, FieldValue,Store.YES))|字符串|N|Y|Y或N|这个Field用来构建一个字符串Field，但是不会进行分析，会将整个串存储在索引中，比如(订单号,姓名等)是否存储在文档中用Store.YES或Store.NO决定|
|LongPoint(String name, long... point)|Long型|Y|Y|N|可以使用LongPoint、IntPoint等类型存储数值类型的数据。让数值类型可以进行索引。但是不能存储数据，如果想存储数据还需要使用StoredField。|
|StoredField(FieldName, FieldValue)|重载方法，支持多种类型|N|N|Y|这个Field用来构建不同类型Field不分析，不索引，但要Field存储在文档中|
|TextField(FieldName, FieldValue, Store.NO)或TextField(FieldName, reader)|字符串或流|Y|Y|Y或N|如果是一个Reader, lucene猜测内容比较多,会采用Unstored的策略|

##### 添加

>添加方法与创建相同，将新的文档加入即可。

#### 删除

##### 全部删除

```java
    indexWriter.deleteAll();
    indexWriter.close();
```
**重建使使用，不要轻易使用**

##### 使用关键词删除文档索引

```java
    indexWriter.deleteDocuments(new Term("name","关键词"));
    indexWriter.colse();
```

#### 更新

**原理：先删除后重新添加**

```java
    indexWriter.updateDocment(new Term("name","要删除的"),新的document);
    indexWriter.colse();
```

### 索引库的查询

1.  使用Query的子类

    1.  TermQuery
            根据关键词查询
            需要指定要查询的域及要查询的关键词

    2.  RangeQuery
            范围查询
            使用LongPoint.newRangeQuery获取该对象
            需要指定查询的域以及范围

2.  使用QueryPaser进行查询

    可以对要查询的内容先分词，然后基于分词的结果进行查询

    1.  pom坐标
    ```xml
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-queryparser</artifactId>
        <version>7.4.0</version>
    </dependency>
    ```

    2.  创建QueryPaser对象，传入默认搜索域和分析器

    3.  调用parse方法，传入要查询的字符串，得到query对象

    4.  执行查询

## 总结

lunece是一个很基础的全文检索技术，虽然很少再有项目直接使用它来做全文检索，但是一些全文检索技术底层依然是用lunece来实现，所以还是很有必要学习一下的，后续有时间再学习其他全文检索技术。完结


