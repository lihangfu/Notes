# 1、多线程技能

==多线程是异步的，代码顺序并不能当成是线程的执行顺序，线程被调用的时机是随机的==

## 1.1 使用多线程

> 一个进程正在运行时至少会有一个线程正在运行。
>
> ```java
> public static void mian() #main方法的线程由JVM创建的。
> ```



* 继承Thread类
* 实现Runnable接口



从源码上不难发现，Thread实现了Runnable接口，它们之间具有多态关系。

继承Thread类的最大局限就是不支持多继承。需要多继承则可以实现Runnable接口。二者工作性质一样，没有本质上的区别。



==run方法里的内容为子线程执行内容，线程通过start方法告知CPU等待调度，调度时机是随机的，由CPU分配时间调度。==



注意：子线程是start方法启动，而不是调用run方法。直接调用run方法，那不还是顺序执行，而不是多线程了。



看一下原书对这里的解释：

> Thread.java类中的start方法通知“线程规划器”此线程已经准备就绪，等待调用线程对象的run方法。这个过程其实就是让系统安排一个时间来调用Thread中的run方法，也就是使线程得到运行，启动线程，具有异步执行的效果。如果调用代码thread的run方法就不啥异步执行了，而是同步，那么此线程对象并不交给“线程规划器”来进行处理，而是由main主线程来调用run方法，也就是必须run方法中的代码执行完成后才可以执行后面的代码。



==注意：执行start方法的顺序不代表线程启动的顺序。只是告诉系统等待启动，启动是随机的。==



关于使用Runnable接口，本质上还是创建Thread类：

```java
public class MyRunnable implements Runnable{
    @Override
    publlic void run(){
        System.out.println("运行中！")
    }
}
public class Run{
    public static void main(String[] args){
        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
        System.out.println("运行结束！")
    }
}
```



既然创建线程都是创建Thread类，那么创建线程的方法都在Thread的构造方法中了。

Thread类的构造方法中可以接收Runnable对象，而Thread又实现了Runnable对象，也就是说Thread类可以接收Thread对象。==将一个Thread对象中的run方法交由其他的线程进行调用。==

多个线程可以调用同一个Thread的run方法？那么线程安全问题就来了。



## 1.2 实例变量与线程安全问题

两种情况，不共享数据和共享数据。

**不共享数据**

```java
public class MyThread extends Thread{
    private int count = 5;
    ....
}
public class Run{
    public static void main(String[] args){
        MyThread a = new MyThread("A");
        MyThread b = new MyThread("B");
        MyThread b = new MyThread("C");
        a.start();
        b.start();
        c.start();
    }
}
```

这种情况下，每个Thread的count是不共享的，各种拥有各种的属性。



**共享变量**

```java
public class Run{
    public static void main(String[] args){
        MyThread mythread = new MyThread();
        Thread a = new Thread(mythread,"A");
        Thread b = new Thread(mythread,"B");
        Thread c = new Thread(mythread,"C");
        a.start();
        b.start();
        c.start();
    }
}
```

还记得上一节引出的安全问题吗？来了，将一个Thread的run方法交给多个线程执行时，多个线程对该Thread的属性进行操作，数据共享，便有了安全问题。



**synchronized**

通过synchronized关键字，使多个线程在执行run方法时，以排队的方式进行处理。

当一个线程调用run方法前，先判断run方法有没有被上锁，如果上锁，说明有其他线程正在调用run方法，必须等其他线程对run方法调用结束后才可以执行run方法。

这样就实现了排队调用run方法的目的，实现了线程安全。



synchronized可以在任意对象及方法上加锁，而加锁的这段代码称为“互斥区”或“临界区”。



当一个线程想要去执行同步方法里面的代码时，线程首先尝试去拿这把锁，如果能拿到这把锁，就可以执行synchronized里面的代码。如果拿不到，那么这个线程就会不断的尝试拿这把锁，直到能够拿到为止，而且是多个线程同时去争取这把锁。



**print**

通过源码可以发现，print方法是同步方法，有个细节问题。

```java
System.out.println("i=" + (i--));
```

上面这个代码在多线程访问时还是出现了线程安全的问题。

print不是同步方法吗？

是的，虽然print方法在内部是同步的，但是i--的操作却是在进入print之前发生的，所以有发生非线程安全问题的概率。



所以为了防止发生非线程安全问题，还是应该继续使用同步方法。



## 1.3 currentThread()方法

currentThread()方法是Thread的静态方法，可以返回代码段正在被哪个线程调用的信息。

```java
System.out.println(Thread.currentThread().getName());
// 返回当前执行线程的线程名称

this.getName();
//当前run方法所在Thread类的name属性

//二者不一定相同，比如
Mythread mythread = new Mythread();
Thread thread = new Thread(mythread);
thread.setName("A");
thread.start();

class Mythread extends Thread{
    @Override
    public void run() {
        super.run();
        System.out.println(Thread.currentThread().getName() + this.getName());
    }
}

//先输出了当前线程thread的名称，A
//然后再输出Mythread的Name。
```



## 1.4 isAlive()方法

方法isAlive()的功能是判断当前的线程是否处于活动状态。

什么是活动状态？

活动状态就是线程已经启动且尚未终止。线程正处于正在运行或者准备开始运行的状态，就认为线程是“存活”的。



==注意：如果将线程对象以构造参数的方式传递给Thread对象进行start时，运行结果跟上一小结一样，会有源自Thread.currentThread()和this的差异。==



## 1.5 sleep()方法

方法sleep()的作用是在指定的毫秒数内让当前"正在执行的线程"休眠（暂停执行）。

这个“正在执行的线程”是指this.currentThread()返回的线程。



## 1.6 getId()方法

getId()方法的作用是取得线程的唯一标识。



## 1.7 停止线程

停止一个线程意味着在线程处理完任务之前停掉正在做的操作，也就是放弃当前操作。

看起来好像很简单？但是必须做好防范措施，以便达到预期的效果。

在Java中有以下3种方法可以终止正在运行的线程：

* 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
* 使用stop方法强行终止线程，但是不推荐使用，是作废过期方法，可能产生不可预料的结果。
* 使用interrupt方法中断线程。



### 1.7.1 停止不了的线程

interrupt方法停止线程，不像for+break语句那样，马上停止。

调用interrupt方法仅仅是在当前线程中打了一个停止的标记，并不是真的停止线程。



### 1.7.2 判断线程是否是停止状态

在停止线程钱，先要知道线程的状态是不是停止的。

1. this.interrupted()：测试当前线程是否已经中断状态。静态方法。执行后具有将状态标志清除为false的功能。
2. this.isInterrupted()：测试线程Thread对象是否已经中断状态。但是不清楚状态标志。

二者区别就是前一个是静态方法，只能判断当前正在执行该语句的线程是否中断，而第二个则可以判断子线程。

而且interrupted会清楚中断状态，而isInterrupted则不会。



### 1.7.3 能停止的线程——异常法

原理就是检测线程状态，中断抛出异常结束。

```java
class Mythread extends Thread{
    @Override
    public void run() {
        super.run();
        try {
            for (int i = 0; i < 500000; i++) {
                if(this.isInterrupted()){
                    System.out.println("线程中断~~~~~");
                    throw new InterruptedException();
                }
                System.out.println("i=" + (i+1));
            }
        } catch (InterruptedException e) {
                e.printStackTrace();
        }
    }
}
```



### 1.7.4 在沉睡中停止

如果线程在sleep状态下，改变状态停止。会被catcha捕获。

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Mythread mythread = new Mythread();
        mythread.start();
        Thread.sleep(2000);
        mythread.interrupt();
    }
}
class Mythread extends Thread{
    @Override
    public void run() {
        super.run();
        try {
            Thread.sleep(200000);
        } catch (InterruptedException e) {
            System.out.println("线程中断"+this.isInterrupted());
            e.printStackTrace();
        }
    }
}
```



==结论就是，在sleep状态下停止某一线程，会进入catch语句，并且清楚停止状态，使之变成false。==



### 1.7.5 能停止的线程——暴力停止

使用stop方法停止线程则是非常暴力的。

不推荐使用！

stop方法会抛出ThreadDeath异常，此异常不需要显示的捕捉。

stop方法停止线程可能会导致一些清理性的工作得不到完成。

另外一个情况就是对锁定的对象进行了“解锁”，导致数据得不到同步的处理，出现了了数据不一致的问题。



### 1.7.6 使用return停止线程

将方法interrupt与return结合使用也能实现停止线程的效果。

判断isInterrupted然后进行return。

不过还是推荐使用“抛异常”的方法来实现线程的停止。

因为在catch块中还可以将异常向上抛，使线程停止的事件得以传播。



## 1.8 暂停线程

暂停线程意味着此线程还可以恢复运行。

可以使用~~suspend~~方法暂停线程，

使用~~resume~~方法恢复线程执行。

> 暂停线程有一定的安全问题

在同步代码块中暂停线程会导致对同步对象的独占。因为暂停线程并不会释放锁。



**还有一个特别细的问题**

`System.out.ptintln()`方法内部是线程安全的，如果线程在该方法内被暂停，导致同步锁不能释放

会导致其他线程无法进入该方法，无法打印信息。

~~suspend~~方法是过期作废的方法，但还是有必要研究它过期作废的原因。



除了上诉问题，也会出现多线程最普遍的问题——数据不同步

比如修改数据一部分后线程暂停，前后数据就会不同。



## 1.9 yield方法

yield方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU执行时间。

但放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片。



## 1.10 线程的优先级

> 在操作系统中，线程可以划分优先级，优先级较高的线程得到的CPU资源较多，也就是CPU优先执行优先级较高的线程对象中的任务。

设置线程优先级有助于帮"线程规划器"确定在下一次选择哪一个线程来优先执行。

设置线程的优先级使用`setPriority()`方法

线程优先级分为1~10这十个等级，小于1或大于10，则抛出异常。



> 线程的优先级具有继承性：比如A线程启动B线程，则B线程的优先级与A是一样的。



**线程的优先级具体体现在什么地方那？**

高优先级的线程总是大部分先执行完，但不代表高优先级的线程全部先执行。

高优先级的线程可能会优先得到执行，但不绝对，只是可能。低优先级也可能先执行。

线程的优先级与代码的执行顺序无关，线程的优 先级具有一定的规则性，也就是CPU尽量将执行资源让给优先级比较高的线程。让他得到更多的机会运行，运行的更快。



## 1.11 守护线程

> 在Java中有两种线程，一种用户线程，另一种是守护线程。                                                                                                                                                                                                                                                                                                                                                                                                              

守护线程是一种特殊的线程，它的特性有“陪伴”的含义，当进程中不存在非先守护线程了，则守护线程自动销毁。

典型的守护线程就是垃圾回收线程，当进程中没有非守护线程了

则垃圾回收线程也就没有存在必要了，自动销毁。