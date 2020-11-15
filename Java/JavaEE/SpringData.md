## JPA

### ORM思想

1.  建立实体类和表的关系
2.  建立实体类中的属性和表中字段的关系

所谓的ORM就是操作实体类就相当于操作数据库表。

不再重点关注：sql语句

实现了ORM的框架：Mybatis、Hibernate

### Hibernate框架

    Hibernate是一个开放源代码的对象关系映射框架，
        它对JDBC进行了轻量级的封装，
        它将POJO与数据库表建立映射关系，是一个全自动的orm框架

### JPA规范

    内部由接口和抽象类组成，是一套规范。

### JPA与Hibernate

JPA与Hibernate的关系就相当于JDBC和JDBC的驱动，由JPA提供规范，Hibernate实现。

### 入门案例

1.  pom依赖
```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.0.7.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-c3p0</artifactId>
            <version>5.0.7.Final</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
    </dependencies>
```

2.  jpa核心配置文件
* 位置：配置到类路径下的一个叫做META-INF的文件夹下
* 命名：persistence.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!--配置持久化单元
            name:持久化单元名称
            transaction-type：事务管理的方式
                        JTA：分布式事务管理，跨库的表操作
                        RESOURCE_LOCAL：本地事务管理，不跨库

    -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa的实现方式-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <!--数据库信息
                用户名：javax.persistence.jdbc.user
                密码：javax.persistence.jdbc.password
                驱动：javax.persistence.jdbc.driver
                数据库地址：javax.persistence.jdbc.url
            -->
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="286728"/>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql:///jpa"/>
            <!--可选配置：配置jpa实现方的配置信息
                显示sql:hibernate.show_sql
                自动创建数据库表:hibernate.hbm2dd1.auto
                                create:程序运行时创建数据库表（有表先删除，后创建）
                                update:运行时创建数据库表（有表不创建）
                                none:不创建表
            -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>

        </properties>
    </persistence-unit>
</persistence>
```

2.  创建实体类，使用注解配置实体类

**实体类与表的映射**

* @Entity   ：  生命该类是一个实体类
* @Table    ：  配置实体类和表的映射关系
    *name属性：对应数据库名

**属性与字段的映射**
* @Id   ：  声明主键配置
* @GeneratedValue   ：  配置主键的生成策略
    * GenerationType.IDENTITY：自增长
* @Column   ：  对应数据库字段名称

3.  测试
```java
//1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        //2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        //3.获取事务对象
        EntityTransaction tx = em.getTransaction();
        //4.开启事务
        tx.begin();
        //5.保存
        Customer customer = new Customer();
        customer.setCustName("lhf");
        customer.setCustIndustry("学习");
        em.persist(customer);
        //6.提交事务
        tx.commit();
        //7.释放资源
        em.close();
        factory.close();
```

### 主键生成策略

* @GeneratedValue
        strategy
            GenerationType.IDENTITY：自增长，底层数据库要支持自增长，mysql
            GenerationType.SEQUENCE：序列，oracle
            GenerationType.TABLE：通过一张表完成自增长，该表由JPA创建
            GenerationType.AUTO：自动选择上面最佳的方法

### 步骤分析
1.  加载配置文件创建工厂（实体管理器工厂）对象
    EPersistence：静态方法（根据持久化单元名称创建实体管理器工厂）
        createEntityManagerFactory：持久化单元名称
    作用：创建实体管理器工厂

2.  通过实体管理器工厂获取实体管理器
    EntityManagerFactory：获取EntityManager对象
    方法：createEntityManager
    * 内部维护了很多内容
        维护了数据库信息
        维护了缓存信息
        维护了所有实体管理器对象
        再创建EntityManagerFactory的过程中会根据配置创建数据库表
    * EntityManagerFactory的创建过程比较浪费资源
        特点：线程安全的对象
        可以使用静态代码块创建EntityManagerFactory来解决浪费资源的问题

3.  创建事务对象，开启事务
    EntityManager：实体类管理器
        beginTransaction：创建事务对象
        presist：保存
        merge：更新
        remove：删除
        find/getRefrence：根据id查询

    Transaction对象：事务
        begin：开启事务
        commit：提交事务
        rollback：回滚

4.  执行增删改查
5.  释放资源

### 增删改查
1. 查
    * find：查询结果封装类的字节码，id

    * getRefrence：查询结果封装类的字节码，id
        区别，getRefrence延时加载，得到一个动态代理对象，用到时才会查询。

2. 删
    * remove: 查询的对象
        需要先查询，然后把查询的结果放进去删除

3. 改
    * merge：查询的对象
    需要先查询，然后修改，最后提交

4. 增
    * persist：要保存的对象

### JPQL
JPA提供的查询语句，完成一些复杂操作。与sql类似。
    
    sql：查询的是表和表中的字段
    jpql：查询的是实体类和类中的属性

1. 查询全部
        sql：select * from cst_customer
        jsql:from Customer

```java
        String jpql = "from Customer";
        Query query = em.createQuery(jpql);
        List resultList = query.getResultList();
        for (Object o : resultList) {
            System.out.println(o);
        }
```

2. 分页查询
        sql:select * from cst_customer limit ?,?
        jpql:from Customer

```java
        String jpql = "select count(custId) from Customer ";
        Query query = em.createQuery(jpql);
        query.setFirstResult(0);
        query.setMaxResults(2);
        List resultList = query.getResultList();
        for (Object o : resultList) {
            System.out.println(o);
        }
```

3. 统计查询
        sql:select count(cst_Id) from cst_customer
        jpql:select count(custId) from Customer

```java
        String jpql = "select count(custId) from Customer ";
        Query query = em.createQuery(jpql);
        Object singleResult = query.getSingleResult();
        System.out.println(singleResult);
```

4. 条件查询
        sql:select * from cst_customer where cust_name LIKE "l%"
        jpql:from Customer where custName like ?

```java
        String jpql = "from Customer where custName like ?";
        Query query = em.createQuery(jpql);
        query.setParameter(1,"l%");
        Object singleResult = query.getSingleResult();
        System.out.println(singleResult);
```

5. 排序
        sql:select * from cst_customer order by cust_id desc
        jpql:from Customer order by custId desc

```java
        String jpql = "from Customer order by custId desc";
        Query query = em.createQuery(jpql);
        List resultList = query.getResultList();
        for (Object o : resultList) {
            System.out.println(o);
        }
```

## SpringDataJpa

### 介绍
SpringDataJpa是Spring基于ORM框架，JPA规范的基础上封装的一套JPA应用，可以是使开发者用极简的代码实习对数据库的访问和操作。

SpringDataJpa是对JPA的再封装，其底层实现还是Hibernate

### 快速入门
#### 1. pom依赖
```xml
<properties>
    <spring.version>5.0.2.RELEASE</spring.version>
    <hibernate.version>5.0.7.Final</hibernate.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <c3p0.version>0.9.1.2</c3p0.version>
    <mysql.version>5.1.6</mysql.version>
</properties>
 
<dependencies>
 
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
 
    <!-- spring beg -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.6.8</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${spring.version}</version>
    </dependency>
 
    <!-- spring对orm框架的支持包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
 
    <!-- hibernate beg -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>${hibernate.version}</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>${hibernate.version}</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.2.1.Final</version>
    </dependency>
 
    <!-- c3p0 beg -->
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>${c3p0.version}</version>
    </dependency>
 
    <!-- log beg -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
 
    <!-- mysql beg -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
 
    <!-- spring data jpa 的坐标-->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>1.9.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
 
    <!-- el beg 使用spring data jpa 必须引入 -->
    <dependency>
        <groupId>javax.el</groupId>
        <artifactId>javax.el-api</artifactId>
        <version>2.2.4</version>
    </dependency>
    <dependency>
        <groupId>org.glassfish.web</groupId>
        <artifactId>javax.el</artifactId>
        <version>2.2.4</version>
    </dependency>
</dependencies>
```

#### 2. spring配置文件整合jpa
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!--spring和spring data jpa 的配置-->
    <!--1.配置entityManagerFactory-->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--配置实体类所在的包-->
        <property name="packagesToScan" value="com.lhf.domain"/>
        <!--配置jpa的实现方式-->
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>
        <!--jpa的供应商适配器-->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!-- 配置是否自动创建数据库表 -->
                <property name="generateDdl" value="false" />
                <!-- 指定数据库类型 -->
                <property name="database" value="MYSQL" />
                <!-- 数据库方言：支持的特有语法 -->
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect" />
                <!-- 是否显示sql -->
                <property name="showSql" value="true" />
            </bean>
        </property>
        <!-- jpa的方言 ：高级的特性 -->
        <property name="jpaDialect" >
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
        </property>
        <!--注入jpa的配置信息
        加载jpa的基本配置信息和jpa实现方式(hibernate) 的配置信息
        hibernate.hbm2ddl.auto :自动创建数据库表
        create :每次都会 重新创建数据库表
        update:有表不会重新创建，没有表会重新创建表
        -->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.hbm2ddl.auto">create</prop>
            </props>
        </property>
    </bean>

    <!-- 2.创建数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="user" value="root"></property>
        <property name="password" value="286728"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///jpa?serverTimezone=UTC" ></property>
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
    </bean>

    <!-- 3.整合spring dataJpa-->
    <jpa:repositories base-package="com.lhf.dao" transaction-manager-ref="transactionManager"
                      entity-manager-factory-ref="entityManagerFactory" ></jpa:repositories>

    <!-- 4.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"></property>
    </bean>

    <!-- 4.txAdvice -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <!-- 5. 配置包扫描 -->
    <context:component-scan base-package="com.lhf" ></context:component-scan>
</beans>
```

#### 3. 编写实体类并配置
```java
package com.lhf.domain;

import javax.persistence.*;

@Entity
@Table(name = "cst_customer")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId;
    @Column(name = "cust_name")
    private String custName;
    @Column(name = "cust_source")
    private String custSource;
    @Column(name = "cust_level")
    private String custLevel;
    @Column(name = "cust_industry")
    private String custIndustry;
    @Column(name = "cust_phone")
    private String custPhone;
    @Column(name = "cust_address")
    private String custAddress;

    public Long getCustId() {
        return custId;
    }

    public void setCustId(Long custId) {
        this.custId = custId;
    }

    public String getCustName() {
        return custName;
    }

    public void setCustName(String custName) {
        this.custName = custName;
    }

    public String getCustSource() {
        return custSource;
    }

    public void setCustSource(String custSource) {
        this.custSource = custSource;
    }

    public String getCustLevel() {
        return custLevel;
    }

    public void setCustLevel(String custLevel) {
        this.custLevel = custLevel;
    }

    public String getCustIndustry() {
        return custIndustry;
    }

    public void setCustIndustry(String custIndustry) {
        this.custIndustry = custIndustry;
    }

    public String getCustPhone() {
        return custPhone;
    }

    public void setCustPhone(String custPhone) {
        this.custPhone = custPhone;
    }

    public String getCustAddress() {
        return custAddress;
    }

    public void setCustAddress(String custAddress) {
        this.custAddress = custAddress;
    }

    @Override
    public String toString() {
        return "Customer{" +
                "custId=" + custId +
                ", custName='" + custName + '\'' +
                ", custSource='" + custSource + '\'' +
                ", custLevel='" + custLevel + '\'' +
                ", custIndustry='" + custIndustry + '\'' +
                ", custPhone='" + custPhone + '\'' +
                ", custAddress='" + custAddress + '\'' +
                '}';
    }
}
```

#### 4. 编写dao层接口

* 只需要编写dao层接口，不需要编写实现类
* dao层接口规范
    1. 需要继承两个接口JpaRepository<实体类类型,主键类型>，JpaSpecificationExecutor<实体类类型>
    2. 需要提供相应的泛型

#### 5. 测试
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class JpaTest {
    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据id查询
     */
    @Test
    public void testFindOne(){
        Customer one = customerDao.findOne(1l);
        System.out.println(one);
    }
    /**
     * 保存或更新
     * 根据传递的对象是否有主键
     * 没有主键保存
     * 有主键，根据主键先查询数据，更新数据
     */
    @Test
    public void testSave(){
        Customer customer = new Customer();
        customer.setCustId(2l);
        customer.setCustName("ffk");
        customerDao.save(customer);
    }
    /**
     * 根据id删除
     * 先查询是否有，然后再删除
     */
    @Test
    public void testDelete(){
        customerDao.delete(2l);
    }
}
```

### 执行原理

1. 通过JdkDynamicAopProxy的invoke方法创建一个动态代理对象。
2. SimpleJpaRepository当中封装了JPA的操作（借助JPA的api完成数据库的CRUD）
3. 通过Hibernate完成数据库的操作（封装了JDBC）

### 复杂查询
1. 统计查询 count()
2. 判断是否存在 exists(id)
3. 按id查询 getOne(id) 延迟加载，需要事务支持

**Jpql查询**

1. 在接口定义方法
2. 使用注解配置该方法
3. @Query("jpql语句")
4. 可以使用占位符，占位符默认会按参数顺序赋值。
5. 也可以"from User where name=?2 and id=?1",加上参数索引确定占位符赋值
6. 更新操作需要再加一个@Modifying，需要事务支持
        "update Customer set custName =?2 where custId = ?1"
        该操作会自动回滚，需要在service层添加注解@Rollback(value=false)

**sql查询**
1. 在接口定义方法
2. 使用注解配置该方法
3. @Query("jpql语句")
    * value：jpql|sql
    * nativeQuery：false(使用jpql查询)|true(使用本地查询：sql查询)

**方法命名规则查询**
1. findBy开头：代表查询
    对象中的属性名（首字母大写）：根据属性名查询

2. findBy+属性名称+查询方式（Like|isnull）
    findByCustNameLike

3. 多条件查询
    findBy+属性名+查询方式+连接符（or|and）+属性名+查询方式
    精准匹配不用写查询方式

### 动态查询

**JpaSpecificationExecutor<T>接口的方法**
```java
    T findOne(Specification<T> var1);

    List<T> findAll(Specification<T> var1);

    Page<T> findAll(Specification<T> var1, Pageable var2);

    List<T> findAll(Specification<T> var1, Sort var2);

    long count(Specification<T> var1);
```

**Specification接口封装查询条件需要有实现类**
参数：
    root:查询的根对象（查询的任何属性都可以从根对象中获取）
    CriteriaQuery：顶层查询对象，自定义查询方式（了解：一般不用）
    CriteriaBuilder：查询的构造器，封装了很多的查询条件
```java
    @Test
    public void testSec(){
        //匿名内部类
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                //获取比较的属性
                Path<Object> name = root.get("custName");
                //比较的方式，封装了很多比较方式
                Predicate predicate = criteriaBuilder.equal(name, "lhf");
                return predicate;
            }
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }
```

**对于模糊匹配以及一些比较的其它查询，需要指定path的参数类型**
```java
Predicate p1 = criteriaBuilder.like(name.as(String.class), "lhf");
```

#### 多条件拼接
```java
    @Test
    public void testSec(){
        //匿名内部类
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                //获取比较的属性
                Path<Object> name = root.get("custName");
                Path<Object> id = root.get("custId");
                //比较的方式，封装了很多比较方式
                Predicate p1 = criteriaBuilder.equal(name, "lhf");
                Predicate p2 = criteriaBuilder.equal(id, 1l);
                //拼接查询条件and|or
                Predicate predicate = criteriaBuilder.and(p1, p2);
                return predicate;
            }
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }
```

#### 排序
```java
    Sort sort = new Sort(Sort.Direction.DESC,"custId");
    List<Customer> customer = customerDao.findAll(specification,sort);
```

#### 分页查询
```java
    Pageable pageable = new PageRequest(0,2);
    Page<Customer> page = customerDao.findAll(specification, pageable);
    System.out.println(page.getContent());//得到数据集合列表
    System.out.println(page.getTotalElements());//得到总条数
    System.out.println(page.getTotalPages());//得到总页数
```

### 多表关联
**双向关系**
@OneToMany：一对多的实体类对象集合
    targetEntity：多的字节码文件
@JoinColum：外键关联
    name：关联字段在本表的字段
    referencedColumnName：对应多的表的关联字段

@ManyToOne：对一的实体类对象
    targetEntity：一的字节码文件
@JoinColum：外键关联
    name：外键字段名
    referencedColumnName：对应一的表的字段

