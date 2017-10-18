---
title: Java_Thread_线程间共享数据的方式
date: 2017-09-13 15:52:33
categories: Java
tags:
    - Java
    - Thread
---



<!-- more -->

##  线程间共享数据的方式

### 目标
谈到多线程共享数据,理想情况下我们希望做到“同步”和“互斥”。

多线程共享数据通常的场景有以下两种:

### 场景一: 买票
卖票,我们都买过火车票。要买火车票我们可以去车站,也可以通过代售点(或网购),但不管有多少种方式火车票的总数是一定的。

#### 场景抽象:
对于卖票系统每个线程的核心执行的代码都相同(就是票数–)。

#### 解决方法:
只需创建一个Runnable,这个Runnable里有那个共享数据。

#### 代码模拟:
```java
public class Ticket implements Runnable{
 
    private int ticket = 10;
    public void run() {
        while(ticket>0){
            ticket--;
            System.out.println("当前票数为:"+ticket);
        }
    }
}

public class SellTicket {
    public static void main(String[] args) {
        Ticket t = new Ticket();
        new Thread(t).start();
        new Thread(t).start();
    }
}
```

### 场景二:银行存取款
比较常见的例子,银行问题,我们对账户可以存钱也可以取钱,怎么保证这样的数据共享呢？

#### 场景抽象:
每个线程执行的代码不同(比如上面的问题,对每个账户可以执行++操作和–操作),这时候需要用不同的Runnable对象,有如下两种方式来实现这些Runnable之间的数据共享

#### 解决方案:
有两种方法来解决此类问题:

- 将共享数据封装成另外一个对象中,然后将这个对象逐一传递给各个Runnable对象,每个线程对共享数据的操作方法也分配到那个对象身上完成,这样容易实现针对数据进行各个操作的互斥和通信
- 将Runnable对象作为偶一个类的内部类,共享数据作为这个类的成员变量,每个线程对共享数据的操作方法也封装在外部类,以便实现对数据的各个操作的同步和互斥,作为内部类的各个Runnable对象调用外部类的这些方法。

#### 代码模拟:
以一道面试题为例:
```java
/* 第一种解法 设计4个线程。,其中两个线程每次对j增加1,另外两个线程对j每次减1*/

public class MyData {
     private int j=0;
     public  synchronized void add(){
         j++;
         System.out.println("线程"+Thread.currentThread().getName()+"j为:"+j);
     }
     public  synchronized void dec(){
         j--;
         System.out.println("线程"+Thread.currentThread().getName()+"j为:"+j);
     }
     public int getData(){
         return j;
     }
}

public class AddRunnable implements Runnable{
    MyData data;
    public AddRunnable(MyData data){
        this.data= data;
    }
    public void run() {
        data.add();
    }
}

public class DecRunnable implements Runnable {
    MyData data;
    public DecRunnable(MyData data){
        this.data = data;
    }
    public void run() {
            data.dec();
    }
}

public class TestOne {
    public static void main(String[] args) {
        MyData data = new MyData();
        Runnable add = new AddRunnable(data);
        Runnable dec = new DecRunnable(data);
        for(int i=0;i<2;i++){
            new Thread(add).start();
            new Thread(dec).start();
        }
    }
}
```

#### 解法分析:

- 优点:

1. 这种解法代码写的有条理,简单易读,从main中很容易整理出思路
2. 将数据抽象成一个类,并将对这个数据的操作作为这个类的方法,这么设计可以和容易做到同步,只要在方法上加”synchronized“

- 不足:

代码写的比较繁琐,需要有多个类,不是那么简洁
个人观点:为了有条理个人比较喜欢这种写法。

```java
/* 第二种解法 */

public class MyData {
     private int j=0;
     public  synchronized void add(){
         j++;
         System.out.println("线程"+Thread.currentThread().getName()+"j为:"+j);
     }
     public  synchronized void dec(){
         j--;
         System.out.println("线程"+Thread.currentThread().getName()+"j为:"+j);
     }
     public int getData(){
         return j;
     }
}

public class TestThread {
    public static void main(String[] args) {
        final MyData data = new MyData();
        for(int i=0;i<2;i++){
            new Thread(new Runnable(){
                public void run() {
                    data.add();
                }
            }).start();
            new Thread(new Runnable(){
                public void run() {
                    data.dec();
                }
            }).start();
        }
    }
}
```

#### 解法分析:
与第一种方法的区别在于第二种方法巧妙的用了内部类共享外部类数据的思想,即把要共享的数据变得全局变量,这样就保证了操作的是同一份数据。同时内部类的方式使代码更加简洁。但是不如第一种解法条理那么清楚。

ref: [http://www.importnew.com/20861.html](http://www.importnew.com/20861.html)

