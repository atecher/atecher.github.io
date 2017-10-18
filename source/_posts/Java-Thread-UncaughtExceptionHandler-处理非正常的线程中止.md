---
title: Java-Thread-UncaughtExceptionHandler-处理非正常的线程中止
date: 2017-09-13 19:15:33
categories: Java
tags:
    - Java
    - Thread
---

当单线程的程序发生一个未捕获的异常时我们可以采用try...catch进行异常的捕获,但是在多线程环境中,线程抛出的异常是不能用try...catch捕获的,这样就有可能导致一些问题的出现,比如异常的时候无法回收一些系统资源,或者没有关闭当前的连接等等。

<!-- more -->

首先来看一个示例:
```java
public class NoCaughtThread {
    public static void main(String[] args) {
        try {
            Thread thread = new Thread(new Task());
            thread.start();
        } catch (Exception e) {
            System.out.println("==Exception: "+e.getMessage());
        }
    }
}

class Task implements Runnable {
    @Override
    public void run() {
        System.out.println(3/2);
        System.out.println(3/0);
        System.out.println(3/1);
    }
}
```
运行结果:
```java
Exception in thread "Thread-0" java.lang.ArithmeticException: / by zero
    at com.exception.Task.run(NoCaughtThread.java:25)
    at java.lang.Thread.run(Unknown Source)
```

可以看到在多线程中通过try...catch试图捕获线程的异常是不可取的。

Thread的run方法是不抛出任何检查型异常的,但是它自身却可能因为一个异常而被中止,导致这个线程的终结。
首先介绍一下如何在线程池内部构建一个工作者线程,如果任务抛出了一个未检查异常,那么它将使线程终结,但会首先通知框架该现场已经终结。然后框架可能会用新的线程来代替这个工作线程,也可能不会,因为线程池正在关闭,或者当前已有足够多的线程能满足需要。当编写一个向线程池提交任务的工作者类线程类时,或者调用不可信的外部代码时（例如动态加载的插件）,使用这些方法中的某一种可以避免某个编写得糟糕的任务或插件不会影响调用它的整个线程。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class InitiativeCaught {
    public void threadDeal(Runnable r, Throwable t) {
        System.out.println("==Exception: "+t.getMessage());
    }
 
    class InitialtiveThread implements Runnable {
        @Override
        public void run() {
            Throwable thrown = null;
            try {
                System.out.println(3/2);
                System.out.println(3/0);
                System.out.println(3/1);
            } catch(Throwable e) {
                thrown e;
            } finally{
                threadDeal(this,thrown);
            }
        }
    }
 
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new InitiativeCaught().new InitialtiveThread());
        exec.shutdown();
    }
}
```
运行结果:
```
==Exception: / by zero
```

上面介绍了一种主动方法来解决未检测异常。在Thread ApI中同样提供了UncaughtExceptionHandle,它能检测出某个由于未捕获的异常而终结的情况。这两种方法是互补的,通过将二者结合在一起,就能有效地防止线程泄露问题。
如下:
```java
import java.lang.Thread.UncaughtExceptionHandler;
 
public class WitchCaughtThread {
    public static void main(String args[]) {
        Thread thread = new Thread(new Task());
        thread.setUncaughtExceptionHandler(new ExceptionHandler());
        thread.start();
    }
}
 
class ExceptionHandler implements UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("==Exception: "+e.getMessage());
    }
}
```
运行结果:
```
==Exception: / by zero
```
同样可以为所有的Thread设置一个默认的UncaughtExceptionHandler,通过调用Thread.setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh)方法,这是Thread的一个static方法。

如下:
```java
import java.lang.Thread.UncaughtExceptionHandler;
 
public class WitchCaughtThread {
    public static void main(String args[]) {
        Thread.setDefaultUncaughtExceptionHandler(new ExceptionHandler());
        Thread thread = new Thread(new Task());
        thread.start();
    }
}
 
class ExceptionHandler implements UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("==Exception: "+e.getMessage());
    }
}
```
运行结果:
```
==Exception: / by zero
```

如果采用线程池通过execute的方法去捕获异常,先看下面的例子:
```java
public class ExecuteCaught {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        Thread thread = new Thread(new Task());
        thread.setUncaughtExceptionHandler(new ExceptionHandler());
        exec.execute(thread);
        exec.shutdown();
    }
}
```
ExceptionHandler可参考上面的例子,运行结果:
```
Exception in thread "pool-1-thread-1" java.lang.ArithmeticException: / by zero
    at com.exception.Task.run(NoCaughtThread.java:25)
    at java.lang.Thread.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
    at java.lang.Thread.run(Unknown Source)
```
可以看到并未捕获到异常。
这时需要将异常的捕获封装到Runnable或者Callable中,如下所示:
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class ExecuteCaught {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new ThreadPoolTask());
        exec.shutdown();
    }
}
 
class ThreadPoolTask implements Runnable {
    @Override
    public void run() {
        Thread.currentThread().setUncaughtExceptionHandler(new ExceptionHandler());
        System.out.println(3/2);
        System.out.println(3/0);
        System.out.println(3/1);
    }
}
```
运行结果:
```
==Exception: / by zero
```
只有通过execute提交的任务,才能将它抛出的异常交给UncaughtExceptionHandler,而通过submit提交的任务,无论是抛出的未检测异常还是已检查异常,都将被认为是任务返回状态的一部分。如果一个由submit提交的任务由于抛出了异常而结束,那么这个异常将被Future.get封装在ExecutionException中重新抛出。
下面两个例子:
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class SubmitCaught {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.submit(new Task());
        exec.shutdown();
    }
}
```

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class SubmitCaught {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.submit(new ThreadPoolTask());
        exec.shutdown();
    }
}
```
运行结果都是: 1

这样可以证实我的观点。接下来通过这个例子可以看到捕获的异常:
```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class SubmitCaught {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        Future<?> future = exec.submit(new Task());
        exec.shutdown();
        try {
            future.get();
        } catch (InterruptedException | ExecutionException e) {
            System.out.println("==Exception: "+e.getMessage());
        }
    }
}
```
运行结果:
```
==Exception: java.lang.ArithmeticException: / by zero
```

ref: [http://www.importnew.com/18619.html](http://www.importnew.com/18619.html)

<!-- more -->

