# 1. Lambda表达式的简介

## 1.1. Lambda表达式的概念

> lambda表达式，是Java8的一个新特性，也是Java8中最值得学习的新特性之一。

lambda表达式，从本质来讲，是一个匿名函数。可以使用这个匿名函数，实现接口中的方法。对接口进行非常简洁的实现，从而简化代码。

## 1.2. Lambda表达式对接口的要求

>  虽然说，lambda表达式可以在一定程度上简化接口的实现。但是，并不是所有接口都可以使用lambda表达式来简洁实现的。

lambda表达式毕竟只是一个匿名方法。当实现的接口方法过多或者过少的时候，lambda表达式都是不适用的。

lambda表达式，只能实现**函数式接口**

## 1.3. 函数式接口

### 1.3.1. 基础概念

> 如果说，一个接口中，要求实现类必须实现的抽象方法，有且只有一个！这样的接口，就是函数式接口。

```java
// 这个接口中，有且只有一个方法，是实现类必须实现的，因此是一个函数式接口。
interface Test1{
    void test();
}

// 这个接口中，实现类必须实现的方法有两个，因此不是一个函数式接口。
interface Test2{
    void test1();
    void test2();
}

// 这个接口中，实现类必须实现的方法是零个，因此不是一个函数式接口。
interface Test3{
    void test();
}

// 这个接口中，没有定义任何方法，但是可以从父接口中继承一个抽象方法，因此是一个函数式接口。
interface Test4 extends Test1{

}

// 这个接口中，虽然定义了两个方法，但是default方法子类不是必须实现的，因此是一个函数式接口。
interface Test5{
    void test5();
    default void test(){}
}

// 这个接口中，虽然定义了两个方法，但是toString方法是Object类中定义的方法，可以从父类继承，并不一定要重写，因此是一个函数式接口。
interface Test6{
    void test6();
    String toString();
}

// 不是。toString可以不重写
interface Test7{
    String toString();
}

// 是的只有test8必须实现
interface Test8{
    void test8();
    default void test1(){}
    static void test2(){}
    String toString();
}
```



### 1.3.2. @FunctionalInterface

>该注解用在接口之前，判断这个接口是否是一个函数式接口。如果是，则没有任何问题。如果不是，则会报错。功能类似于@Override。

```java
@FunctionalInterface
interface Test1{
    void test();
}
```



# 2. Lambda表达式的基础语法

 ## 2.1. Lambda表达式的基础语法

> lambda表达式，其实本质来讲，就是一个匿名函数。因此在写lambda表达式的时候，不需要关心方法名是什么。
> 实际上，我们在写lambda表达式的时候，也不需要关心返回值类型。
> 我们在写lambda表达式的时候，只需要关注两部分内容即可：**参数列表**和**方法体**



**lambda表达式的基础语法 ：**

```java
(参数) -> {
  	方法体  
};
```

**参数部分 ：**方法的参数列表，要求和实现的接口中的方法参数部分保持一致，包括参数数量和类型。

**方法体部分 ：**方法的实现部分，如果接口中定义的方法有返回值，则在实现的时候，注意返回值的返回。

**-> : **分割参数部分和方法体部分。



**lambda表达式的基本使用 ：**

```java
// 函数式接口名 变量名 = (参数) -> {方法体};
// 变量名.函数名();

// 1. 实现无参，无返回
NoneReturnNoneParameter lambda1 = () -> {
  System.out.println("这是一个无参，无返回值的方法");  
};
lambda1.test();

// 2. 实现有参，无返回
NoneReturnSingleParameter lambda2 = (int a) -> {
  System.out.println("这是一个参数，无返回值的方法，参数 a ="+ a);  
};
lambda2.test();

// 3. 实现多参，无返回
NoneReturnMutipleParameter lambda3 = (int a,int b) -> {
  System.out.println("这是一个多参，无返回值的方法，参数 a =" + a + "b =" + b);  
};
lambda3.test();

// 4. 实现无参，有返回
SingleReturnNoneParameter lambda4 = () -> {
  System.out.println("这是一个无参，有返回值的方法");
  return 100;
};
System.out.println(lambda4.test());

// 5. 实现有参，有返回
SingleReturnSingleParameter lambda5 = (int a) -> {
  System.out.println("这是一个参数，有返回值的方法，参数 a ="+ a); 
  return a;
};
System.out.println(lambda5.test());

// 6. 实现多参，有返回
SingleReturnMutipleParameter lambda6 = (int a,int b) -> {
  System.out.println("这是一个多参，有返回值的方法，参数 a =" + a + "b =" + b);
  return a+b;
};
System.out.println(lambda6.test());
```



## 2.2. Lambda表达式的语法进阶

> 再上诉代码中，的确可以使用lambda表达式实现接口，但是依然不够简洁，有简化空间

### 2.2.1. 参数部分的精简

* 参数的类型

  * 参数的类型和数量与实现方法一致即可，类型可以不写。

  * 注意：要不写就都别写，别写两个参数类型，不写两个参数类型

  * ```java
    SingleReturnMutipleParameter lambda6 = (a,b) -> {
      System.out.println("这是一个多参，有返回值的方法，参数 a =" + a + "b =" + b);
      return a+b;
    };
    ```

* 参数的小括号

  * 如果方法的参数列表 **有且只有一个**，此时，参数列表的小括号可以省略不写。

  * 注意：参数多了少了都不能省略，且省略括号时类型也必须省略

  * ```java
    NoneReturnSingleParameter lambda2 = a -> {
      System.out.println("这是一个参数，无返回值的方法，参数 a ="+ a);  
    };
    ```

### 2.2.2. 方法体部分的精简

当方法体部分 **有且仅有一条** 语句时，大括号可以省略不写。

**注意 ：**有返回值时，仅有的语句必须省略 return 关键字。



# 3. 函数应用

> lambda表达式是为了简化接口的实现的。在lambda表达式中，不应该出现比较复杂的逻辑。如果在lanbda表达式中出现了过于复杂的逻辑，会对程序的可读性造成非常大的影响。如果在lambda表达式中需要处理的逻辑比较复杂，一般情况会单独的写一个方法。在lamibda表达式中直接引用这个方法即可。

> 或者，在有些情况下，我们需要在lambda表达式中实现的逻辑，在另一个地方已经写好了。此时我们既不需要再单独写一遍，只需要直接引用这个已经存在的方法即可。

**函数引用 ：**引用一个已经存在的方法，使其替代lambda表达式完成接口的实现。



## 3.1. 静态方法的引用

* 语法：
  * 类::静态方法
* 注意事项：
  * 在引用的方法后面，不用添加小括号。
  * 引用的这个方法，参数（类型，数量）和返回值，必须跟接口中定义的一致。
* 示例:

```java
public class Lambda1 {
    private static interface Calculate{
        int test(int a,int b);
    }
    public static void main(String[] args) {
        // 静态方法的引用
        Calculate calculate = Lambda1::calculate;
        System.out.println(calculate.test(20, 20));
    }
    // 静态方法
    public static int calculate(int a,int b){
        if(a>b) return a-b;
        else if(a<b) return b-a;
        return a+b;
    }
}
```



## 3.2. 非静态方法的引用

* 语法：
  * 对象::非静态方法
* 注意事项：
  * 在引用的方法后面，不用添加小括号。
  * 引用的这个方法，参数（类型，数量）和返回值，必须跟接口中定义的一致。
* 示例：

```java
public class Lambda1 {
    private static interface Calculate{
        int test(int a,int b);
    }
    public static void main(String[] args) {
        Calculate calculate = new Lambda1()::calculate;
        System.out.println(calculate.test(20, 20));
    }
    public int calculate(int a,int b){
        if(a>b) return a-b;
        else if(a<b) return b-a;
        return a+b;
    }
}
```



## 3.3. 引用构造方法

* 使用场景
  * 如果某一个函数式接口中定义的方法，仅仅是为了得到一个类的对象。此时我们就可以使用构造方法的引用，简化这个方法的实现。
* 语法
  * 类名::new
* 注意事项
  * 可以通过接口中的方法的参数，区分引用不同的构造方法。
* 示例：

```java 
public class Lambda1 {

    private static class Person{
        private String name;
        private int age;
        public Person() {
        }
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }
    private static interface getPerson1{
        Person get();
    }
    private static interface getPerson2{
        Person get(String name,int age);
    }
    public static void main(String[] args) {
        getPerson1 person1 =  Person::new;
        getPerson2 person2 =  Person::new;
    }
}
```



## 3.4. 对象方法的特殊引用

> 如果在使用lambda表达式，实现某些接口时。lambda表达式中包含了某一个对象，此时方法中，直接使用这个对象调用它的某一个方法就可以完成整体的逻辑。其它的参数，可以作为调用方法的参数。此时，可以对这种实现进行简化。

* 使用情况：
  * 参数：其中有一个对象x
  * 方法体：使用对象x的某一个方法完成整体的逻辑
* 示例：

```java
public class Lambda1 {
    private static class Person{
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
    private static interface getPerson1{
        String get(Person person);
    }
    private static interface getPerson2{
        void set(Person person,String value);
    }
    public static void main(String[] args) {
        Person person = new Person();
        person.setName("ssss");
        getPerson1 person1 =  Person::getName;
        getPerson2 person2 =  Person::setName;
        System.out.println(person1.get(person));
        person2.set(person,"hhhhh");
        System.out.println(person1.get(person));
    }
}
```



# 4. 集合流的简介

## 4.1. 集合的流式编程的简介

> Stream是JDK1.8之后出现的新特性，也是JDK1.8新特性中最值得学习的两种新特性之一。(另外一个是lambda表达式)。
> Stream是对集合操作的增强，流不是集合的元素，不是一种数据结构，不负责数据的存储的。流更像是一个迭代器，可以单向的追历一个集合中的每一个元素，并且不可循环



## 4.2. 为什么要使用集合的流式编程

有些时候，对集合中的元素进行操作的时候，需要使用到其他操作的结果。在这个过程中，集合的流式编程可以大幅度的简化代码的数量，将数据源中的数据，读现到一个流中，可以对这个流中的数据进行操作（删除、过滤、映射...)。每次的操作结果也是一个流对象，可以对这个流再进行其他的操作。



## 4.3. 使用流式编程的步骤

通常情况下，对集合中的数据使用流式编程，需要经过以下三步。

1. 获取数据源，将数据源中的数据读取到流中。
2. 对流中的数据进行各种各样的处理。
3. 对流中的数据进行整合处理。



​			在上述三个过程中，过程2中，有若干方法，可以对流中的数据进行各种各样的操作，并且返回流对像本身，这样的操作，被称为 —— 中间操作。过程3中，有若干方法，可以对流中的数据进行各种处理，并关闭流，这样的操作，被称为 —— 最终操作。

​			在中间操作和最终操作中，基本上所有的方法参数都是函数式接口，可以使用lambda表达式来实现。使用集合的流式编程，来简化代码量，是需要对lanbda表达式做到熟练掌握。



# 5. 数据源的获取

## 5.1. 数据源的简介

> 数据源，顾名思义，既是流中的数据的来源，是集合的流式编程的第一步，将数据源中的数据读取到流中，进行处理。注意:将数据读取到流中进行处理的时候，与数据源中的数据没有关系。也就是说，中间操作对流中的数据进行处理、过滤、映射、排序 . . . ，此时是不会影响数据源中的数据的。



## 5.2. 数据源的获取

​		这个过程，其实是将一个容器中的数据，读取到一个流中。因此无论什么容器作为数据源，读取到流中的方法返回值一定是一个Stream。

```java
// 1．通过Collection接口中的stream()方法获取数据源为Collection的流
Stream<Integer> stream= list.stream();
// 2．通过Collection接口的parallelstream()方法获取数据源为Collection的流
Stream<Integer> stream = list.parallelStream();
// 3．通过Arrays工具类中的stream()方法获取数据源为数组(Integer数组)的流
Stream<Integer> stream = Arrays.stream(array);
// 4．通过Arrays工具类中的stream()方法获取数据源为数组(int数组)的流
IntStream stream = Arrays.stream(array);
```

> 关于`stream()`和`parallelStream()`
> 他们都是Collection集合获取数据源的方法，不同点在于stream()方法获取的数据源是串行的，parallelStream()获取的数据源是并行的，parallelStream()内部集成了多个线程对流中的数据进行操作，效率更高。



# 6. 最终操作

## 6.1. 最终操作的简介

> 将流中的数据整合到一起，可以存入一个集合，也可以直接对流中的数据进行遍历、数据统计...，通过最终操作，需要掌握如何从流中提取出来我们想要的信息。
> **注意事项∶**最终操作，之所以叫最终操作，是因为，在最终操作执行结束后，会关闭这个流，流中的所有数据都会销毁。如果使用一个已经关闭了的流，会出现异常。



## 6.2. collect 收集

​			将流中的数据收集到一起，对这些数据进行一些处理。最常见的处理，就是将流中的数据存入一个集合。collect方法的参数，是一个Collector接口，而且这个接口并不是一个函数式接口。实现这个接口，可以自定义收集的规则。但是，绝大部分情况下，不需要自定义。

​			直接使用`Collectors`工具类提供的方法即可。

```java

List<Integer> collect = list.stream().collect(Collectors.toList());
Set<Integer> collect = list.stream().collect(Collectors.toSet());
Map<String, Integer> collect = list.stream().collect(Collectors.toMap(Objects::toString, i -> i));

```



## 6.3. reduce 聚合

​			将流中的数据按照—定的规则聚合起来。

 ```java
// 将流的元素，逐一带入到这个方法中，进行运算
// 最终的运算结果，得到的其实是一个Optional类型，需要使用get()获取到里面的数据
// Integer integer = list.stream().reduce((a, b) -> a + b).get();
Integer integer = list.stream().reduce(Integer::sum).get();
 ```



## 6.4. count 统计

​			统计流中的元素数量。 

```java
long count = list.stream().count();
```



## 6.5. forEach 遍历

​			迭代、遍历流中的数据。

```java
list.stream().forEach(System.out::println);
```



## 6.6. max & min 最值

​			获取流中的最大的元素、最小的元素。

```java
// 获取最大值
Integer max = list.stream().max(Integer::compareTo).get();
// 获取最小值
Integer max = list.stream().min(Integer::compareTo).get();
```



## 6.7. Matching 匹配

* allMathch：只有当流中所有的元素，都匹配指定的规则，才会返回 true
* anyMatch：只要流中有任意的数据，满足指定的规则，都会返回 true
* noneMatch：只有当流中的所有的元素，都不满足指定的规则，才会返回 true

```java
// 判断流中是否所有的元素都大于 50
boolean ans = list.stream().allMatch(ele -> ele > 50);

// 判断流中是否有大于 50 的数据
boolean ans = list.stream().anyMatch(ele -> ele > 50);

// 判断流中是否没有奇数
boolean ans = list.stream().noneMatch(ele -> ele % 2 != 0);
```



## 6.8. find 获取

* findFirst：从流中获取一个元素（一般情况下，是获取的开通的元素）
* findAny：从流中获取一个元素（一般情况下，是获取的开头的元素）

这两个方法，绝大部分情况下，是完全相同的，但是在多线程的环境下，findAny和find返回的结果可能不一样。

```java
Integer ans = list.parallelStream().findFirst().get();

Integer ans = list.parallelStream().findAny().get();
```



## 6.9. IntStream的最终操作

> 注意：IntStream和DoubleStream等一样的方法。

```java
int[] array = new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
IntStream stream = Arrays.stream(array);
int max = stream.max().getAsInt();					// 获取最大值
int min = stream.min().getAsInt();					// 获取最小值
int sum = stream.sum();								// 获取数据和
int count = stream.count();							// 获取流中的数据数量
double average = stream.average().getAsDouble();	// 获取流中数据的平均值
IntSummarystatistics intSummaryStatistics = stream.summaryStatistics();	// 获取流中数据的分析结果，包括上述所有的值。
```



# 7. 中间操作

## 7.1. 中间操作的简介

> 将数据从数据源中读取到流中，中间操作，就是对流中的数据进行各种各样的操作、处理。中间操作可以连续操作，每一个操作的返回值都是一个stream对象，可以继续进行其他的操作。直到最终操作。



## 7.2. filter 过滤

> 条件过滤，仅保留流中满足指定条件的数据，其它不满足的数据都会被删除。

```java
list.stream().filter(ele -> ele.length() > 5).forEach(System.out::println);
```



## 7.3. distinct 去重

> 去重集合中重复的元素。这个方法没有参数。去重的规则与HashSet相同。

```java
list.stream().distinct().forEach(System.out::println);
```



## 7.4. sorted 排序

> 将流中数据进行排序

```java
// 按照流中的元素对应的类实现的 Comparable 接口中的方法实现排序
list.stream().sorted().forEach(System.out::println);
// 将流中的数据，按照指定的规则进行排序
list.stream().sorted((e1,e2) -> e1.length()-e2.length()).forEach(System.out::println);
```



## 7.5. limit & skip

> limit : 限制，表示截取流中指定数量的数据
>
> skip：跳过，可以跟限制配合使用

```java
// 获取流中第三个到第五个
list.stream().skip(2).limit(3).forEach(System.out::println);
// 获取流中第三个到第五个
list.stream().limit(5).skip(2).forEach(System.out::println);
```



## 7.6. map & flatMap

> map：对流中的数据进行映射，用新的数据替换旧的数据。

```java
list.stream().map(ele -> ele + ".txr").forEach(System.out::println);
```

map最主要，就是来做元素的替换。其实map是一个元素的映射。

flatMap也是元素的映射，flatMao是扁平化映射。

扁平化映射：一般在map映射完成后，流中的数据是一个容器，而我们需要对容器中的数据进行处理，此时使用扁平化映射，可以将流中的容器中的数据，直接读取到流中。

```java
// 1. 实例化一个字符串数组
String[] array = {"hello","world"};
// 2. 将字符串数组中的数据读取到流中
Stream<String> stream = Arrays.stream(array);
// 3. 需求：统计字符串数组中所有出现的字符
stream.map(s -> s.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .forEach(System.out::println);
```



# 8. Collectors工具类

## 8.1. 概念

> Collectors是一个工具类，里面封装了很多方法，可以很方便的获取到一个Collector接口的实现类对象，从而可以使用`col1ect()`方法，对流中的数据，进行各种各样的处理、整合。



## 8.2. 常用方法

| 方法                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| Collectors.toList() | 将流中的数据，聚合到一个 List 集合中                         |
| Collectors.toSet()  | 将流中的数据，聚合到一个 Set 集合中                          |
| Collectors.toMap()  | 将流中的数据，聚合到一个 Map 集合中                          |
| maxBy()             | 按照指定的规则，找到流中最大的元素，等同于 max               |
| minBy()             | 按照指定的规则，找到流中最小的元素，等同于 min               |
| joining()           | 将流中的数据评价成一个字符串，注意：只能操作流中是String的数据 |
| summingInt()        | 将流中的数据，映射成 int 类型的数据，并求和                  |
| averagingInt()      | 将流中的数据，映射成 int 类型的数据，并求平均值              |
| summarizingInt()    | 将流中的数据，映射成 int 类型的数据，并获取描述信息          |

