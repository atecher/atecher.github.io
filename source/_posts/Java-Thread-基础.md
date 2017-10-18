---
title: Java-Thread-基础
date: 2017-09-13 19:15:33
categories: Java
tags:
    - Java
    - Thread
---

很多人都对其中的一些概念不够明确,如同步、并发等等,让我们先建立一个数据字典,以免产生误会。

<!-- more -->

## 概念

- 多线程:指的是这个程序(一个进程)运行时产生了不止一个线程
- 并行与并发:
    - 并行:多个cpu实例或者多台机器同时执行一段处理逻辑,是真正的同时。
    - 并发:通过cpu调度算法,让用户看上去同时执行,实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源,那么针对这个公用的资源往往产生瓶颈,我们会用TPS或者QPS来反应这个系统的处理能力。
    
- 线程安全:经常用来描绘一段代码。指在并发的情况之下,该代码经过多线程使用,线程的调度顺序不影响任何结果。这个时候使用多线程,我们只需要关注系统的内存,cpu是不是够用即可。反过来,线程不安全就意味着线程的调度顺序会影响最终结果,如不加事务的转账代码:
```java
void transferMoney(User from, User to, float amount){
  to.setMoney(to.getBalance() + amount);
  from.setMoney(from.getBalance() - amount);
}
```
- 同步:Java中的同步指的是通过人为的控制和调度,保证共享资源的多线程访问成为线程安全,来保证结果的准确。如上面的代码简单加入@synchronized关键字。在保证结果准确的同时,提高性能,才是优秀的程序。线程安全的优先级高于性能。

## 上下文切换

对于单核CPU来说(对于多核CPU,此处就理解为一个核),CPU在一个时刻只能运行一个线程,当在运行一个线程的过程中转去运行另外一个线程,这个叫做线程上下文切换(对于进程也是类似)。

由于可能当前线程的任务并没有执行完毕,所以在切换时需要保存线程的运行状态,以便下次重新切换回来时能够继续切换之前的状态运行。举个简单的例子:比如一个线程A正在读取一个文件的内容,正读到文件的一半,此时需要暂停线程A,转去执行线程B,当再次切换回来执行线程A的时候,我们不希望线程A又从文件的开头来读取。

因此需要记录线程A的运行状态,那么会记录哪些数据呢？因为下次恢复时需要知道在这之前当前线程已经执行到哪条指令了,所以需要记录程序计数器的值,另外比如说线程正在进行某个计算的时候被挂起了,那么下次继续执行的时候需要知道之前挂起时变量的值时多少,因此需要记录CPU寄存器的状态。所以一般来说,线程上下文切换过程中会记录程序计数器、CPU寄存器状态等数据。

说简单点的:对于线程的上下文切换实际上就是 **存储和恢复CPU状态的过程,它使得线程执行能够从中断点恢复执行。**

虽然多线程可以使得任务执行的效率得到提升,但是由于在线程切换时同样会带来一定的开销代价,并且多个线程会导致系统资源占用的增加,所以在进行多线程编程时要注意这些因素。

## 线程的实现

### 继承Thread类
在java.lang包中定义, 继承Thread类必须重写run()方法
创建好了自己的线程类之后,就可以创建线程对象了,然后通过start()方法去启动线程。注意,不是调用run()方法启动线程,run方法中只是定义需要执行的任务,如果调用run方法,即相当于在主线程中执行run方法,跟普通的方法调用没有任何区别,此时并不会创建一个新的线程来执行定义的任务。
```java
class MyThread extends Thread{
    private static int num = 0;
    public MyThread(){
        num++;
    }
    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}

public class Test {
    public static void main(String[] args)  {
        MyThread thread = new MyThread();
        thread.start();
    }
}
```

在上面代码中,通过调用start()方法,就会创建一个新的线程了。为了分清start()方法调用和run()方法调用的区别,请看下面一个例子:
```java
class MyThread extends Thread{
    private String name;
    public MyThread(String name){
        this.name = name;
    }
    @Override
    public void run() {
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}

public class Test {
    public static void main(String[] args)  {
        System.out.println("主线程ID:"+Thread.currentThread().getId());
        MyThread thread1 = new MyThread("thread1");
        thread1.start();
        MyThread thread2 = new MyThread("thread2");
        thread2.run();
    }
}

/*
运行结果:
主线程ID:1
name:thread2 子线程ID:1
name:thread1 子线程ID:8
*/
```
从输出结果可以得出以下结论:

1. thread1和thread2的线程ID不同,thread2和主线程ID相同,说明通过run方法调用并不会创建新的线程,而是在主线程中直接运行run方法,跟普通的方法调用没有任何区别；
2. 虽然thread1的start方法调用在thread2的run方法前面调用,但是先输出的是thread2的run方法调用的相关信息,说明新线程创建的过程不会阻塞主线程的后续执行。

### 实现Runnable接口

在Java中创建线程除了继承Thread类之外,还可以通过实现Runnable接口来实现类似的功能。实现Runnable接口必须重写其run方法。
下面是一个例子:
```java
public class Test {
    public static void main(String[] args)  {
        System.out.println("主线程ID:"+Thread.currentThread().getId());
        MyRunnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
    }
} 
class MyRunnable implements Runnable{
    public MyRunnable() {
    }
    @Override
    public void run() {
        System.out.println("子线程ID:"+Thread.currentThread().getId());
    }
}
```
Runnable的中文意思是“任务”,顾名思义,通过实现Runnable接口,我们定义了一个子任务,然后将子任务交由Thread去执行。注意,这种方式必须将Runnable作为Thread类的参数,然后通过Thread的start方法来创建一个新线程来执行该子任务。如果调用Runnable的run方法的话,是不会创建新线程的,这根普通的方法调用没有任何区别。

事实上,查看Thread类的实现源代码会发现Thread类是实现了Runnable接口的。

在Java中,这2种方式都可以用来创建线程去执行子任务,具体选择哪一种方式要看自己的需求。直接继承Thread类的话,可能比实现Runnable接口看起来更加简洁,但是由于Java只允许单继承,所以如果自定义类需要继承其他类,则只能选择实现Runnable接口。

### 使用ExecutorService、Callable、Future实现有返回结果的多线程

ExecutorService、Callable、Future这个对象实际上都是属于Executor框架中的功能类。想要详细了解Executor框架的可以访问 http://www.iteye.com/topic/366591 ,这里面对该框架做了很详细的解释。返回结果的线程是在JDK1.5中引入的新特征,有了这种特征我就不需要再为了得到返回值而大费周折了。

可返回值的任务必须实现Callable接口,类似的,无返回值的任务必须Runnable接口。执行Callable任务后,可以获取一个Future的对象,在该对象上调用get就可以获取到Callable任务返回的Object了,再结合线程池接口ExecutorService就可以实现传说中有返回结果的多线程了。下面提供了一个完整的有返回结果的多线程测试例子,在JDK1.5下验证过没问题可以直接使用。代码如下:
```java
/**
 * 有返回值的线程
 */
@SuppressWarnings("unchecked")
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("----程序开始运行----");
        Date date1 = new Date();

        int taskSize = 5;
        // 创建一个线程池  
        ExecutorService pool = Executors.newFixedThreadPool(taskSize);
        // 创建多个有返回值的任务  
        List<Future> list = new ArrayList<Future>();
        for (int i = 0; i < taskSize; i++) {
            Callable c = new MyCallable(i + " ");
            // 执行任务并获取Future对象  
            Future f = pool.submit(c);
            list.add(f);
        }
        // 关闭线程池  
        pool.shutdown();

        // 获取所有并发任务的运行结果  
        for (Future f : list) {
            // 从Future对象上获取任务的返回值,并输出到控制台  
            System.out.println(">>>" + f.get().toString());
        }

        Date date2 = new Date();
        System.out.println("----程序结束运行----,程序运行时间【"
                + (date2.getTime() - date1.getTime()) + "毫秒】");
    }
}

class MyCallable implements Callable<Object> {
    private String taskNum;

    MyCallable(String taskNum) {
        this.taskNum = taskNum;
    }

    public Object call() throws Exception {
        System.out.println(">>>" + taskNum + "任务启动");
        Date dateTmp1 = new Date();
        Thread.sleep(1000);
        Date dateTmp2 = new Date();
        long time = dateTmp2.getTime() - dateTmp1.getTime();
        System.out.println(">>>" + taskNum + "任务终止");
        return taskNum + "任务返回运行结果,当前任务时间【" + time + "毫秒】";
    }
}
```
代码说明:
上述代码中Executors类,提供了一系列工厂方法用于创先线程池,返回的线程池都实现了ExecutorService接口。
public static ExecutorService newFixedThreadPool(int nThreads)
创建固定数目线程的线程池。

public static ExecutorService newCachedThreadPool()
创建一个可缓存的线程池,调用execute 将重用以前构造的线程(如果线程可用)。如果现有线程没有可用的,则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

public static ExecutorService newSingleThreadExecutor()
创建一个单线程化的Executor。

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
创建一个支持定时及周期性的任务执行的线程池,多数情况下可用来替代Timer类。

ExecutoreService提供了submit()方法,传递一个Callable,或Runnable,返回Future。如果Executor后台线程池还没有完成Callable的计算,这调用返回Future对象的get()方法,会阻塞直到计算完成。

## 线程的状态

在正式学习Thread类中的具体方法之前,我们先来了解一下线程有哪些状态,这个将会有助于后面对Thread类中的方法的理解。

{% asset_img Thread_State.png 线程状态 %}

- 创建(new)状态: 准备好了一个多线程的对象
- 就绪(runnable)状态: 调用了start()方法, 等待CPU进行调度
- 运行(running)状态: 执行run()方法
- 阻塞(blocked)状态: 暂时停止执行, 可能将资源交给其它线程使用
- 终止(dead)状态: 线程销毁

当需要新起一个线程来执行某个子任务时,就创建了一个线程。但是线程创建之后,不会立即进入就绪状态,因为线程的运行需要一些条件(比如内存资源,在前面的JVM内存区域划分一篇博文中知道程序计数器、Java栈、本地方法栈都是线程私有的,所以需要为线程分配一定的内存空间),只有线程运行需要的所有条件满足了,才进入就绪状态。

当线程进入就绪状态后,不代表立刻就能获取CPU执行时间,也许此时CPU正在执行其他的事情,因此它要等待。当得到CPU执行时间之后,线程便真正进入运行状态。

线程在运行状态过程中,可能有多个原因导致当前线程不继续运行下去,比如用户主动让线程睡眠(睡眠一定的时间之后再重新执行)、用户主动让线程等待,或者被同步块给阻塞,此时就对应着多个状态:time waiting(睡眠或等待一定的事件)、waiting(等待被唤醒)、blocked(阻塞)。

当由于突然中断或者子任务执行完毕,线程就会被消亡。

下面这副图描述了线程从创建到消亡之间的状态:
{% asset_img thread_status_from_new_2_dead.jpg thread_status_from_new_2_dead %}

下图是从别处摘来的线程状态转换, 可结合以供参考
{% asset_img Thread_State_trans.png 线程状态转换 %}

各种状态一目了然,值得一提的是”blocked”这个状态:
线程在Running的过程中可能会遇到阻塞(Blocked)情况

1. 调用join()和sleep()方法,sleep()时间结束或被打断,join()中断,IO完成都会回到Runnable状态,等待JVM的调度。
2. 调用wait(),使该线程处于等待池(wait blocked pool),直到notify()/notifyAll(),线程被唤醒被放到锁定池(lock blocked pool ),释放同步锁使线程回到可运行状态(Runnable)
3. 对Running状态的线程加同步锁(Synchronized)使其进入(lock blocked pool ),同步锁被释放进入可运行状态(Runnable)。

此外,在runnable状态的线程是处于被调度的线程,此时的调度顺序是不一定的。Thread类中的yield方法可以让一个running状态的线程转入runnable。

注:sleep和wait的区别:

- sleep是Thread类的方法,wait是Object类中定义的方法.
- Thread.sleep不会导致锁行为的改变, 如果当前线程是拥有锁的, 那么Thread.sleep不会让线程释放锁.
- Thread.sleep和Object.wait都会暂停当前的线程. OS会将执行时间分配给其它线程. 区别是, 调用wait后, 需要别的线程执行notify/notifyAll才能够重新获得CPU执行时间.

## 线程的常用方法

|编号|方法|说明|
|:---|:---|:---|
|1  |public void start()|   使该线程开始执行；Java 虚拟机调用该线程的 run 方法。|
|2  |public void run()| 如果该线程是使用独立的 Runnable 运行对象构造的,则调用该 Runnable 对象的 run 方法；否则,该方法不执行任何操作并返回。|
|3  |public final void setName(String name)|    改变线程名称,使之与参数 name 相同。|
|4  |public final void setPriority(int priority)|   更改线程的优先级。|
|5  |public final void setDaemon(boolean on)|   将该线程标记为守护线程或用户线程。|
|6  |public final void join(long millisec)| 等待该线程终止的时间最长为 millis 毫秒。|
|7  |public void interrupt()|   中断线程。|
|8  |public final boolean isAlive()|    测试线程是否处于活动状态。|
|9  |public static void yield()|    暂停当前正在执行的线程对象,并执行其他线程。|
|10 |public static void sleep(long millisec)|   在指定的毫秒数内让当前正在执行的线程休眠(暂停执行),此操作受到系统计时器和调度程序精度和准确性的影响。|

### currentThread()方法
currentThread()方法可以返回代码段正在被哪个线程调用的信息。
```java
public class Run1{
    public static void main(String[] args){                 
    System.out.println(Thread.currentThread().getName());
    }
}
```

### sleep()方法
方法sleep()的作用是在指定的毫秒数内让当前“正在执行的线程”休眠(暂停执行)。这个“正在执行的线程”是指this.currentThread()返回的线程。
sleep方法有两个重载版本:
```java
sleep(long millis)     //参数为毫秒
sleep(long millis,int nanoseconds)    //第一参数为毫秒,第二个参数为纳秒
```
sleep相当于让线程睡眠,交出CPU,让CPU去执行其他的任务。
但是有一点要非常注意,sleep方法不会释放锁,也就是说如果当前线程持有对某个对象的锁,则即使调用sleep方法,其他线程也无法访问这个对象。看下面这个例子就清楚了:
```java
public class Test {
    private int i = 10;
    private Object object = new Object();
 
    public static void main(String[] args) throws IOException  {
        Test test = new Test();
        MyThread thread1 = test.new MyThread();
        MyThread thread2 = test.new MyThread();
        thread1.start();
        thread2.start();
    } 
 
    class MyThread extends Thread{
        @Override
        public void run() {
            synchronized (object) {
                i++;
                System.out.println("i:"+i);
                try {
                    System.out.println("线程"+Thread.currentThread().getName()+"进入睡眠状态");
                    Thread.currentThread().sleep(10000);
                } catch (InterruptedException e) {
                    // TODO: handle exception
                }
                System.out.println("线程"+Thread.currentThread().getName()+"睡眠结束");
                i++;
                System.out.println("i:"+i);
            }
        }
    }
}

/*
输出结果:

i:11
线程Thread-1进入睡眠状态
线程Thread-1睡眠结束
i:12
i:13
线程Thread-0进入睡眠状态
线程Thread-0睡眠结束
i:14
*/
```
从上面输出结果可以看出,当Thread-1进入睡眠状态之后,Thread-0并没有去执行具体的任务。只有当Thread-1执行完之后,此时Thread-1释放了对象锁,Thread-0才开始执行。

注意,如果调用了sleep方法,必须捕获InterruptedException异常或者将该异常向上层抛出。当线程睡眠时间满后,不一定会立即得到执行,因为此时可能CPU正在执行其他的任务。所以说调用sleep方法相当于让线程进入阻塞状态。

### yield()方法

调用yield方法会让当前线程交出CPU权限,让CPU去执行其他的线程。它跟sleep方法类似,同样不会释放锁。但是yield不能控制具体的交出CPU的时间,另外,yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

注意,调用yield方法并不会让线程进入阻塞状态,而是让线程重回就绪状态,它只需要等待重新获取CPU执行时间,这一点是和sleep方法不一样的。
代码:
```java
public class MyThread  extends Thread{
    @Override
    public void run() {
        long beginTime=System.currentTimeMillis();
        int count=0;
        for (int i=0;i<50000000;i++){
            count=count+(i+1);
            //Thread.yield();
        }
        long endTime=System.currentTimeMillis();
        System.out.println("用时:"+(endTime-beginTime)+" 毫秒！");
    }
}
 
public class Run {
    public static void main(String[] args) {
        MyThread t= new MyThread();
        t.start();
    }
}

/*
执行结果:

用时:3 毫秒！
如果将 //Thread.yield();的注释去掉,执行结果如下:

用时:16080 毫秒！
*/
```

### start()方法
start()用来启动一个线程,当调用start方法后,系统才会开启一个新的线程来执行用户定义的子任务,在这个过程中,会为相应的线程分配需要的资源。

### run()方法
run()方法是**不需要用户来调用**的,当通过start方法启动一个线程之后,当线程获得了CPU执行时间,便进入run方法体去执行具体的任务。注意,继承Thread类必须重写run方法,在run方法中定义具体要执行的任务。

### getId()
getId()的作用是取得线程的唯一标识

### isAlive()方法
方法isAlive()的功能是判断当前线程是否处于活动状态
什么是活动状态呢？活动状态就是线程已经启动且尚未终止。线程处于正在运行或准备开始运行的状态,就认为线程是“存活”的。

### join()方法
在很多情况下,主线程创建并启动了线程,如果子线程中药进行大量耗时运算,主线程往往将早于子线程结束之前结束。这时,如果主线程想等待子线程执行完成之后再结束,比如子线程处理一个数据,主线程要取得这个数据中的值,就要用到join()方法了。方法join()的作用是等待线程对象销毁。
```java
public class Thread4 extends Thread{
    public Thread4(String name) {
        super(name);
    }
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName() + "  " + i);
        }
    }
    public static void main(String[] args) throws InterruptedException {
        // 启动子进程
        new Thread4("new thread").start();
        for (int i = 0; i < 10; i++) {
            if (i == 5) {
                Thread4 th = new Thread4("joined thread");
                th.start();
                th.join();
            }
            System.out.println(Thread.currentThread().getName() + "  " + i);
        }
    }
}
```
执行结果:
```
main  0
main  1
main  2
main  3
main  4
new thread  0
new thread  1
new thread  2
new thread  3
new thread  4
joined thread  0
joined thread  1
joined thread  2
joined thread  3
joined thread  4
main  5
main  6
main  7
main  8
main  9
```

由上可以看出main主线程等待joined thread线程先执行完了才结束的。如果把th.join()这行注释掉,运行结果如下:
```
main  0
main  1
main  2
main  3
main  4
main  5
main  6
main  7
main  8
main  9
new thread  0
new thread  1
new thread  2
new thread  3
new thread  4
joined thread  0
joined thread  1
joined thread  2
joined thread  3
joined thread  4
```

### getName和setName
用来得到或者设置线程名称。

### getPriority和setPriority
用来获取和设置线程优先级。

### setDaemon和isDaemon
用来设置线程是否成为守护线程和判断线程是否是守护线程。

守护线程和用户线程的区别在于:守护线程依赖于创建它的线程,而用户线程则不依赖。举个简单的例子:如果在main线程中创建了一个守护线程,当main方法运行完毕之后,守护线程也会随着消亡。而用户线程则不会,用户线程会一直运行直到其运行完毕。在JVM中,像垃圾收集器线程就是守护线程。

在上面已经说到了Thread类中的大部分方法,那么Thread类中的方法调用到底会引起线程状态发生怎样的变化呢？下面一幅图就是在上面的图上进行改进而来的:
{% asset_img thread_status_from_new_2_dead_with_methods.jpg thread_status_from_new_2_dead_with_methods %}

ps:
Thread类最佳实践:
写的时候最好要设置线程名称 Thread.name,并设置线程组 ThreadGroup,目的是方便管理。在出现问题的时候,打印线程栈 (jstack -pid) 一眼就可以看出是哪个线程出的问题,这个线程是干什么的。

## 停止线程

停止线程是在多线程开发时很重要的技术点,掌握此技术可以对线程的停止进行有效的处理。
停止一个线程可以使用Thread.stop()方法,但最好不用它。该方法是不安全的,已被弃用。
在Java中有以下3种方法可以终止正在运行的线程:

- 使用退出标志,使线程正常退出,也就是当run方法完成后线程终止
- 使用stop方法强行终止线程,但是不推荐使用这个方法,因为stop和suspend及resume一样,都是作废过期的方法,使用他们可能产生不可预料的结果。
- 使用interrupt方法中断线程,但这个不会终止一个正在运行的线程,还需要加入一个判断才可以完成线程的停止。

## 暂停线程
interrupt()方法


## 线程的优先级

在操作系统中,线程可以划分优先级,优先级较高的线程得到的CPU资源较多,也就是CPU优先执行优先级较高的线程对象中的任务。
设置线程优先级有助于帮“线程规划器”确定在下一次选择哪一个线程来优先执行。
设置线程的优先级使用setPriority()方法,此方法在JDK的源码如下:
```java
public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess();
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
        throw new IllegalArgumentException();
    }
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority();
        }
        setPriority0(priority = newPriority);
    }
}
```
在Java中,线程的优先级分为1~10这10个等级,如果小于1或大于10,则JDK抛出异常throw new IllegalArgumentException()。
JDK中使用3个常量来预置定义优先级的值,代码如下:
```java
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

线程优先级特性:

- 继承性
比如A线程启动B线程,则B线程的优先级与A是一样的。
- 规则性
高优先级的线程总是大部分先执行完,但不代表高优先级线程全部先执行完。
- 随机性
优先级较高的线程不一定每一次都先执行完。

## 守护线程

在Java线程中有两种线程,一种是User Thread(用户线程),另一种是Daemon Thread(守护线程)。
Daemon的作用是为其他线程的运行提供服务,比如说GC线程。其实User Thread线程和Daemon Thread守护线程本质上来说去没啥区别的,唯一的区别之处就在虚拟机的离开:如果User Thread全部撤离,那么Daemon Thread也就没啥线程好服务的了,所以虚拟机也就退出了。

守护线程并非虚拟机内部可以提供,用户也可以自行的设定守护线程,方法:public final void setDaemon(boolean on) ；但是有几点需要注意:

- thread.setDaemon(true)必须在thread.start()之前设置,否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。 (备注:这点与守护进程有着明显的区别,守护进程是创建后,让进程摆脱原会话的控制+让进程摆脱原进程组的控制+让进程摆脱原控制终端的控制；所以说寄托于虚拟机的语言机制跟系统级语言有着本质上面的区别)
- 在Daemon线程中产生的新线程也是Daemon的。 (这一点又是有着本质的区别了:守护进程fork()出来的子进程不再是守护进程,尽管它把父进程的进程相关信息复制过去了,但是子进程的进程的父进程不是init进程,所谓的守护进程本质上说就是“父进程挂掉,init收养,然后文件0,1,2都是/dev/null,当前目录到/”)
- 不是所有的应用都可以分配给Daemon线程来进行服务,比如读写操作或者计算逻辑。因为在Daemon Thread还没来的及进行操作时,虚拟机可能已经退出了。

## 每个对象都有的方法(机制)

synchronized, wait, notify 是任何对象都具有的同步工具。让我们先来了解他们
monitor
{% asset_img a_java_monitor.png monitor %}

他们是应用于同步问题的人工线程调度工具。讲其本质,首先就要明确monitor的概念,Java中的每个对象都有一个监视器,来监测并发代码的重入。在非多线程编码时该监视器不发挥作用,反之如果在synchronized 范围内,监视器发挥作用。

wait/notify必须存在于synchronized块中。并且,这三个关键字针对的是同一个监视器(某对象的监视器)。这意味着wait之后,其他线程可以进入同步块执行。

当某代码并不持有监视器的使用权时(如图中5的状态,即脱离同步块)去wait或notify,会抛出java.lang.IllegalMonitorStateException。也包括在synchronized块中去调用另一个对象的wait/notify,因为不同对象的监视器不同,同样会抛出此异常。

- synchronized单独使用:
    - 代码块:如下,在多线程环境下,synchronized块中的方法获取了lock实例的monitor,如果实例相同,那么只有一个线程能执行该块内容
    ```java
    public class Thread1 implements Runnable {
       Object lock;
       public void run() {  
           synchronized(lock){
             ..do something
           }
       }
    }
    ```

    - 直接用于方法: 相当于上面代码中用lock来锁定的效果,实际获取的是Thread1类的monitor。更进一步,如果修饰的是static方法,则锁定该类所有实例
    ```java
    public class Thread1 implements Runnable {
       public synchronized void run() {  
            ..do something
       }
    }
    ```
- synchronized, wait, notify结合:典型场景生产者消费者问题
    ```java
    /**
    * 生产者生产出来的产品交给店员
    */
    public synchronized void produce() {
        if(this.product >= MAX_PRODUCT) {
            try {
                wait();  
                System.out.println("产品已满,请稍候再生产");
            } catch(InterruptedException e) {
                e.printStackTrace();
            }
            return;
        }
        this.product++;
        System.out.println("生产者生产第" + this.product + "个产品.");
        notifyAll();   //通知等待区的消费者可以取出产品了
    }
 
    /**
    * 消费者从店员取产品
    */
    public synchronized void consume() {
        if(this.product <= MIN_PRODUCT) {
            try {
                wait(); 
                System.out.println("缺货,稍候再取");
            } catch (InterruptedException e)  {
                e.printStackTrace();
            }
            return;
        }
        System.out.println("消费者取走了第" + this.product + "个产品.");
        this.product--;
        notifyAll();   //通知等待去的生产者可以生产产品了
    }
    ```
    
volatile

多线程的内存模型:main memory(主存)、working memory(线程栈),在处理数据时,线程会把值从主存load到本地栈,完成操作后再save回去(volatile关键词的作用:每次针对该变量的操作都激发一次load and save)。
{% asset_img main_memory.png main_memory %}
针对多线程使用的变量如果不是volatile或者final修饰的,很有可能产生不可预知的结果(另一个线程修改了这个值,但是之后在某线程看到的是修改之前的值)。其实道理上讲同一实例的同一属性本身只有一个副本。但是多线程是会缓存值的,本质上,volatile就是不去缓存,直接取值。在线程安全的情况下加volatile会牺牲性能。

## 如何获取线程中的异常
{% asset_img thread_exception_catch.png thread_exception_catch %}

不能用try,catch来获取线程中的异常

## 高级多线程控制类

以上都属于内功心法,接下来是实际项目中常用到的工具了,Java1.5提供了一个非常高效实用的多线程包:java.util.concurrent, 提供了大量高级工具,可以帮助开发者编写高效、易维护、结构清晰的Java多线程程序。

### ThreadLocal类

用处:保存线程的独立变量。对一个线程类(继承自Thread)
当使用ThreadLocal维护变量时,ThreadLocal为每个使用该变量的线程提供独立的变量副本,所以每一个线程都可以独立地改变自己的副本,而不会影响其它线程所对应的副本。常用于用户登录控制,如记录session信息。

实现:每个Thread都持有一个TreadLocalMap类型的变量(该类是一个轻量级的Map,功能与map一样,区别是桶里放的是entry而不是entry的链表。功能还是一个map。)以本身为key,以目标为value。
主要方法是get()和set(T a),set之后在map里维护一个threadLocal -> a,get时将a返回。ThreadLocal是一个特殊的容器。

### 原子类(AtomicInteger、AtomicBoolean……)
如果使用atomic wrapper class如atomicInteger,或者使用自己保证原子的操作,则等同于synchronized
```java
//返回值为boolean
AtomicInteger.compareAndSet(int expect,int update)
```
该方法可用于实现乐观锁,考虑文中最初提到的如下场景:a给b付款10元,a扣了10元,b要加10元。此时c给b2元,但是b的加十元代码约为:
```java
if(b.value.compareAndSet(old, value)){
   return ;
}else{
   //try again
   // if that fails, rollback and log
}
```
AtomicReference
对于AtomicReference 来讲,也许对象会出现,属性丢失的情况,即oldObject == current,但是oldObject.getPropertyA != current.getPropertyA。
这时候,AtomicStampedReference就派上用场了。这也是一个很常用的思路,即加上版本号

### Lock类

lock: 在java.util.concurrent包内。共有三个实现:

- ReentrantLock
- ReentrantReadWriteLock.ReadLock
- ReentrantReadWriteLock.WriteLock

主要目的是和synchronized一样, 两者都是为了解决同步问题,处理资源争端而产生的技术。功能类似但有一些区别。

区别如下:

- lock更灵活,可以自由定义多把锁的枷锁解锁顺序(synchronized要按照先加的后解顺序)
- 提供多种加锁方案,lock 阻塞式, trylock 无阻塞式, lockInterruptily 可打断式,还有trylock的带超时时间版本。
- 本质上和监视器锁(即synchronized是一样的)
- 能力越大,责任越大,必须控制好加锁和解锁,否则会导致灾难。
- 和Condition类的结合。
- 性能更高,对比如下图:
synchronized和Lock性能对比
{% asset_img synchronize_Lock_performance.png synchronize_Lock_performance %}

ReentrantLock
可重入的意义在于持有锁的线程可以继续持有,并且要释放对等的次数后才真正释放该锁。
使用方法是:

```java
//1.先new一个实例
static ReentrantLock r=new ReentrantLock();

//2.加锁
r.lock()//或r.lockInterruptibly();
/*此处也是个不同,后者可被打断。当a线程lock后,b线程阻塞,此时如果是lockInterruptibly,那么在调用b.interrupt()之后,b线程退出阻塞,并放弃对资源的争抢,进入catch块。(如果使用后者,必须throw interruptable exception 或catch)*/

//3.释放锁
r.unlock()

/*必须做！何为必须做呢,要放在finally里面。以防止异常跳出了正常流程,导致灾难。这里补充一个小知识点,finally是可以信任的:经过测试,哪怕是发生了OutofMemoryError,finally块中的语句执行也能够得到保证。*/
```

ReentrantReadWriteLock

可重入读写锁(读写锁的一个实现)
```java
ReentrantReadWriteLock lock = new ReentrantReadWriteLock()
ReadLock r = lock.readLock();
WriteLock w = lock.writeLock();
```
两者都有lock,unlock方法。写写,写读互斥；读读不互斥。可以实现并发读的高效线程安全代码

### 容器类

这里就讨论比较常用的两个:

- BlockingQueue
- ConcurrentHashMap

BlockingQueue
阻塞队列。该类是java.util.concurrent包下的重要类,通过对Queue的学习可以得知,这个queue是单向队列,可以在队列头添加元素和在队尾删除或取出元素。类似于一个管　　道,特别适用于先进先出策略的一些应用场景。普通的queue接口主要实现有PriorityQueue(优先队列),有兴趣可以研究

BlockingQueue在队列的基础上添加了多线程协作的功能:
{% asset_img BlockingQueue.png BlockingQueue %}

除了传统的queue功能(表格左边的两列)之外,还提供了阻塞接口put和take,带超时功能的阻塞接口offer和poll。put会在队列满的时候阻塞,直到有空间时被唤醒；take在队　列空的时候阻塞,直到有东西拿的时候才被唤醒。用于生产者-消费者模型尤其好用,堪称神器。

常见的阻塞队列有:

- ArrayListBlockingQueue
- LinkedListBlockingQueue
- DelayQueue
- SynchronousQueue

ConcurrentHashMap
高效的线程安全哈希map。请对比hashTable , concurrentHashMap, HashMap

### 管理类

管理类的概念比较泛,用于管理线程,本身不是多线程的,但提供了一些机制来利用上述的工具做一些封装。
了解到的值得一提的管理类:ThreadPoolExecutor和 JMX框架下的系统级管理类 ThreadMXBean
ThreadPoolExecutor
如果不了解这个类,应该了解前面提到的ExecutorService,开一个自己的线程池非常方便:
```java
ExecutorService e = Executors.newCachedThreadPool();
ExecutorService e = Executors.newSingleThreadExecutor();
ExecutorService e = Executors.newFixedThreadPool(3);
// 第一种是可变大小线程池,按照任务数来分配线程,
// 第二种是单线程池,相当于FixedThreadPool(1)
// 第三种是固定大小线程池。
// 然后运行
e.execute(new MyRunnableImpl());
```
该类内部是通过ThreadPoolExecutor实现的,掌握该类有助于理解线程池的管理,本质上,他们都是ThreadPoolExecutor类的各种实现版本。请参见javadoc:
{% asset_img ThreadPoolExecutor.png ThreadPoolExecutor %}

ThreadPoolExecutor参数解释
翻译一下:
corePoolSize:池内线程初始值与最小值,就算是空闲状态,也会保持该数量线程。
maximumPoolSize:线程最大值,线程的增长始终不会超过该值。
keepAliveTime:当池内线程数高于corePoolSize时,经过多少时间多余的空闲线程才会被回收。回收前处于wait状态
unit:时间单位,可以使用TimeUnit的实例,如TimeUnit.MILLISECONDS
workQueue:待入任务(Runnable)的等待场所,该参数主要影响调度策略,如公平与否,是否产生饿死(starving)
threadFactory:线程工厂类,有默认实现,如果有自定义的需要则需要自己实现ThreadFactory接口并作为参数传入。

ref: 
[http://www.importnew.com/21089.html](http://www.importnew.com/21089.html)
[http://www.importnew.com/21136.html](http://www.importnew.com/21136.html)
