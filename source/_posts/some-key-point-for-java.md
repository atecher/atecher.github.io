---
title: Java的一些关键技术点
date: 2017-07-20 10:35:46
categories: "开发技巧"
tags: 
 - Java基础
---

基础这东西很重要，各个公司都很看重。基础这些东西无非几部分：逻辑思维，语言，操作系统，网络，数据结构和算法，再加上行业和领域的相关知识。这些都需要我们在平时积累和学习，今天这篇文章主要罗列一下Java语言中的一些技术点，内容会随着时间不断添加。
<!-- more -->

## Switch能否用string做参数

在jdk 7 之前，switch 只能支持 byte、short、char、int 这几个基本数据类型和其对应的封装类型。switch后面的括号里面只能放int类型的值，但由于byte，short，char类型，它们会 自动 转换为int类型（精精度小的向大的转化），所以它们也支持。

> 对于精度比int大的类型，比如long、float，doulble，不会自动转换为int，如果想使用，就必须强转为int，如(int)float;

### JDK7之前

```java

public void switchTest(int expression) {
    switch (expression){    // 括号里是一个表达式，结果是个整数
        case 1:     // 括号里是一个表达式，结果是个整数
            System.out.println("this is one");
            break;
        case 2:
            System.out.println("this is two");
            break;
        case 3:
            System.out.println("this is three");
            break;
        default:
            System.out.println("other");
    }
}

```

### JDK7之后
jdk7以后，整形，枚举类型，boolean，字符串都可以。

```java

public class TestString {

    static String string = "123";
    public static void main(String[] args) {
        switch (string) {
        case "123":
            System.out.println("123");
            break;
        case "abc":
            System.out.println("abc");
            break;
        default:
            System.out.println("defauls");
            break;
        }
    }
}

```

### Why

jdk7并没有新的指令来处理switch，而是通过调用switch中String.hashCode,将String转换为int从而进行判断。

## equals和==的区别
这个问题我们就不撸代码了，纯文字分析一下。

简单来讲：

1. ==是判断两者是不是同一个东西;
2. equals是判断两者是否一样，可能是同一个东西，也可以是两个东西长得完全一样。

详细来讲：

**如果比较对象是值变量**：只能使用==。因为基本类型不是对象，equals()是对象的方法。
**如果比较对象是引用型变量**：

- ==是判断两个引用是否指向同一个实例。
- equals是Object方法，默认使用了==进行比较。如果您自己写了一个类，没有重写equals方法，那对不住了--跟==没毛区别。
- 但是如果列位如果重写了equals方法，那么比较规则就由各位自己定义了。

## Java的基本数据类型以及封装类

java提供了九种基本数据类型，包括：boolean, byte, char, short,  int, long, float, double, void（存在异议）。
同时，java也提供了这些类型的封装类，分别为：Boolean, Byte, Character, Short, Integer, Long, Float, Double, Void。

### 为什么Java会这么做

在java中使用基本类型来存储语言支持的基本数据类型，这里没有采用对象，而是使用了传统的面向过程语言所采用的基本类在型，主要是从性能方面来考虑的：因为即使最简单的数学计算，**使用对象来处理也会引起一些开销**，而这些开销对于数学计算本来是毫无必要的。但是在java中，**泛型类包括预定义的集合，使用的参数都是对象类型**，无法直接使用这些基本数据类型，所以java又提供了这些基本类型的包装器。

### 有什么区别

- 基本数据类型只能按值传递，而封装类按引用传递。
- 基本类型在堆栈中创建；而对于对象类型，对象在堆中创建，对象的引用在堆栈中创建。基本类型由于在堆栈中，效率会比较高，但是可能会存在内存泄漏的问题。

### 基本数据类型介绍

Java基本数据类型分为两大类：boolean类型和数值类型。数值类型可分为整数类型和浮点类型，而其中字符类型可单独对待。所以Java只包含8种基本数据类型。

> 注意！字符串不是基本数据类型，字符串是一个类，是一个引用类型。这个在下一篇我们会仔细讨论它！

- **boolean**:数值只有true和false，不能用0代替。其他数值类型不能转换成boolean。包装类–Boolean
- **byte**:内存8位，无符号位时最大存储255，表数范围：-128~127。包装类–Byte
- **short**:内存16位，无符号位时最大存储65536，表数范围：-32768~32767。包装类–Short
- **int**:内存32位，无符号位时最大存储2的32次方减1，表数范围：负的2的31次方到正的2的31次方减1。包装类–Integer。
- **long**:内存64位，无符号位时最大存储2的64次方减1，表数范围：负的2的63次方到正的2的63次方减1。包装类–Long。
- **float**:内存32位，数据范围在3.4e-45~1.4e38，直接赋值时必须在数字后加上f或F。包装类–Float。
- **double**:内存64位，数据范围在4.9e-324~1.8e308，赋值时可以加d或D也可以不加。包装类–Double。
- **char**:16位，存储Unicode字符集，用单引号赋值。可以参与加减乘除运算的，也可以比较大小的！！包装类–Character。

### 封装类的共性

1. 带有基本值参数并创建包装类对象的构造函数.如可以利用Integer包装类创建对象:Integer obj=new Integer(145);
2. 带有字符串参数并创建包装类对象的构造函数.如:new Integer("-45.36");
3. 可生成对象基本值的typeValue方法,如:int num=obj.intValue();
4. 将字符串转换为基本值的 parseType方法,如Integer.parseInt(args[0]);
5. 因为有装进Map的几率，所以java设计了包装类里的哈希值，生成哈稀表代码的hashCode方法,如:obj.hasCode();
6. 对同一个类的两个对象进行比较的equals()方法,如:obj1.eauqls(obj2);
7. 生成字符串表示法的toString()方法,如:obj.toString().
8. 自动装包/拆包大大方便了基本类型数据和它们包装类地使用。

### 一些知识点

#### Integer

```java
public static void main(String[] args) {
        Integer a1 = 1;
        Integer a2 = 1;

        Integer b1 = 200;
        Integer b2 = 200;

        Integer c1 = Integer.valueOf(1);
//        Integer c2 = new Integer(1);      官方不推荐这种建对象的方法喔
        Integer c2 = Integer.valueOf(1);

        Integer d1 = Integer.valueOf(200);
        Integer d2 = Integer.valueOf(200);


        System.out.println("a1==a2?" + (a1 == a2));
        System.out.println("b1==b2?" + (b1 == b2));
        System.out.println("c1==c2?" + (c1 == c2));
        System.out.println("d1==d2?" + (d1 == d2));
    }

```

上面一段代码的运行结果就是我们要深思的东西啦，也是结合源码要懂的东西。

```
a1==a2? true 
b1==b2? false 
c1==c2? false 
d1==d2? false

```

第一个为什么是true呢，因为Integer的缓存机制嘛，刚刚我们看到的，缓存了[-128,127],这些可以直接取出。而剩余的为什么是false，因为他们都超过了缓存的那个范围，就建了个新对象咯。


## Object有哪些公用方法

Object是所有类的父类，任何类都默认继承Object。
Object类到底实现了哪些方法？

### clone方法

创建并返回此对象的一个副本
保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。
PS:浅复制是指当对象的字段值被复制时，字段引用的对象不会被复制
例如，如果一个对象有一个指向字符串的字段，并且我们对该对象做了一个浅复制，那么两个对象将引用同一个字符串

### getClass方法

final方法，获得运行时类型。

### toString方法

该方法用得比较多，一般子类都有覆盖，返回该对象字符串。

### finalize方法

该方法用于释放资源(由垃圾回收器调用)。因为无法确定该方法什么时候被调用，很少使用。

### equals方法

该方法是非常重要的一个方法。一般equals和==是不一样的，但是在Object中两者是一样的。子类一般都要重写这个方法。

### hashCode方法

该方法用于哈希查找，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。
一般必须满足obj1.equals(obj2)==true。可以推出obj1.hashCode()==obj2.hashCode()。
但是hashCode相等不一定就满足equals。
不过为了提高效率，应该尽量使上面两个条件接近等价。

### wait方法

wait方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。
wait()方法一直等待，直到获得锁或者被中断。
wait(long timeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。
调用该方法后当前线程进入睡眠状态，直到以下事件发生。

1. 其他线程调用了该对象的notify方法。
2. 其他线程调用了该对象的notifyAll方法。
3. 其他线程调用了interrupt中断该线程。
4. 时间间隔到了。

此时该线程就可以被调度了，如果是被中断的话就抛出一个InterruptedException异常。

### notify方法

该方法唤醒在该对象上等待的某个线程。

### notifyAll方法

该方法唤醒在该对象上等待的所有线程。

##  Java的四种引用，强弱软虚，用到的场景

### 强引用

强引用不会被GC回收，并且在java.lang.ref里也没有实际的对应类型，平时工作接触的最多的就是强引用。
Object obj = new Object();这里的obj引用便是一个强引用。
如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。
当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

### 软引用

如果一个对象只具有软引用，那就类似于可有可物的生活用品。
如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。
只 要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。 
软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

### 弱引用

弱引用（weak reference）在强度上弱于软引用，通过类WeakReference来表示。
它的作用是引用一个对象，但是并不阻止该对象被回收。如果使用一个强引用的话，只要该引用存在，那么被引用的对象是不能被回收的。
弱引用则没有这个问题。在垃圾回收器运行的时候，如果一个对象的所有引用都是弱引用的话，该对象会被回收。
弱引用的作用在于解决强引用所带来的对象之间在存活时间上的耦合关系。
弱引用最常见的用处是在集合类中，尤其在哈希表中。哈希表的接口允许使用任何Java对象作为键来使用。
当一个键值对被放入到哈希表中之后，哈希表对象本身就有了对这些键和值对象的引用。
如果这种引用是强引用的话，那么只要哈希表对象本身还存活，其中所包含的键和值对象是不会被回收的。
如果某个存活时间很长的哈希表中包含的键值对很多，最终就有可能消耗掉JVM中全部的内存。
对于这种情况的解决办法就是使用弱引用来引用这些对象，这样哈希表中的键和值对象都能被垃圾回收。
Java中提供了WeakHashMap来满足这一常见需求。

### 虚引用

在介绍幽灵引用之前，要先介绍Java提供的**对象终止化机制**（finalization）。在Object类里面有个**finalize**方法，其设计的初衷是在一个对象被真正回收之前，可以用来执行一些清理的工作。
因为Java并没有提供类似C++的析构函数一样的机制，就通过 finalize方法来实现。但是问题在于垃圾回收器的运行时间是不固定的，所以这些清理工作的实际运行时间也是不能预知的。
幽灵引用（phantom reference）可以解决这个问题。在创建幽灵引用PhantomReference的时候必须要指定一个引用队列。
当一个对象的finalize方法已经被调用了之后，这个对象的幽灵引用会被加入到队列中。
通过检查该队列里面的内容就知道一个对象是不是已经准备要被回收了。
幽灵引用及其队列的使用情况并不多见，主要用来实现比较精细的内存使用控制，这对于**移动设备**来说是很有意义的。
程序可以在确定一个对象要被回收之后，再申请内存创建新的对象。**通过这种方式可以使得程序所消耗的内存维持在一个相对较低的数量。**

## Hashcode的作用

Java中的集合（Collection）有两类，一类是List，再有一类是Set。
前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。 
那么这里就有一个比较严重的问题了：要想保证元素不重复，可两个元素是否重复应该依据什么来判断呢？ 
这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。 
也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法。这显然会大大降低效率。  
于是，Java采用了哈希表的原理。 
哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。
put方法是用来向HashMap中添加新的元素，从put方法的具体实现可知，会先调用hashCode方法得到该元素的hashCode值，然后查看table中是否存在该hashCode值，如果存在则调用equals方法重新确定是否存在该元素，如果存在，则更新value值，否则将新的元素添加到HashMap中。从这里可以看出，hashCode方法的存在是为了减少equals方法的调用次数，从而提高程序效率。

因此有人会说，可以直接根据hashcode值判断两个对象是否相等吗？肯定是不可以的，因为不同的对象可能会生成相同的hashcode值。虽然不能根据hashcode值判断两个对象是否相等，但是可以直接根据hashcode值判断两个对象不等，如果两个对象的hashcode值不等，则必定是两个不同的对象。如果要判断两个对象是否真正相等，必须通过equals方法。

　1. 也就是说对于两个对象，如果调用equals方法得到的结果为true，则两个对象的hashcode值必定相等；
　2. 如果equals方法得到的结果为false，则两个对象的hashcode值不一定不同；
　3. 如果两个对象的hashcode值不等，则equals方法得到的结果必定为false；
　4. 如果两个对象的hashcode值相等，则equals方法得到的结果未知。
　　

## ArrayList、Vector、LinkedList

ArrayList,LinkedList,Vestor这三个类都实现了java.util.List接口，但它们有各自不同的特性，主要如下： 

### 同步性 

ArrayList,LinkedList是不同步的，而Vestor是同步的。所以如果不要求线程安全的话，可以使用ArrayList或LinkedList，可以节省为同步而耗费的开销。但在多线程的情况下，有时候就不得不使用Vector了。当然，也可以通过一些办法包装ArrayList,LinkedList，使他们也达到同步，但效率可能会有所降低。 

### 数据增长 

从内部实现机制来讲ArrayList和Vector都是使用Objec的数组形式来存储的。当你向这两种类型中增加元素的时候，如果元素的数目超出了内部数组目前的长度它们都需要扩展内部数组的长度，Vector缺省情况下自动增长原来一倍的数组长度，ArrayList是原来的50%,所以最后你获得的这个集合所占的空间总是比你实际需要的要大。所以如果你要在集合中保存大量的数据那么使用Vector有一些优势，因为你可以通过设置集合的初始化大小来避免不必要的资源开销。 

### 检索、插入、删除对象的效率 

ArrayList和Vector中,从指定的位置(index)检索一个对象，或在集合的末尾插入、删除一个对象的时间是一样的，可表示为O(1)。
但是，如果在集合的其他位置增加或移除元素那么花费的时间会呈线形增长:O(n-i),其中n代表集合中元素的个数，i代表元素增加或移除元素的索引位置。
为什么会这样呢？因为在进行上述操作的时候集合中第i和第i个元素之后的所有元素都要执行(n-i)个对象的位移操作。LinkedList中，在插入、删除集合中任何位置的元素所花费的时间都是一样的—O(1)，但它在索引一个元素的时候比较慢，为O(i),其中i是索引的位置。 

***
一般大家都知道ArrayList和LinkedList的大致区别：

1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。

ArrayList和LinkedList是两个集合类，用于存储一系列的对象引用(references)。例如我们可以用ArrayList来存储一系列的String或者Integer。那么ArrayList和LinkedList在性能上有什么差别呢？什么时候应该用ArrayList什么时候又该用LinkedList呢？

#### 时间复杂度 

首先一点关键的是，ArrayList的内部实现是基于基础的对象数组的，因此，它使用get方法访问列表中的任意一个元素时(random access)，它的速度要比LinkedList快。LinkedList中的get方法是按照顺序从列表的一端开始检查，直到另外一端。对LinkedList而言，访问列表中的某个指定元素没有更快的方法了。 
假设我们有一个很大的列表，它里面的元素已经排好序了，这个列表可能是ArrayList类型的也可能是LinkedList类型的，现在我们对这个列表来进行二分查找(binary search)，比较列表是ArrayList和LinkedList时的查询速度，看下面的程序：

```java

package com.mangocity.test;   
import java.util.LinkedList;   
import java.util.List;   
import java.util.Random;   
import java.util.ArrayList;   
import java.util.Arrays;   
import java.util.Collections;   
public class TestList {   
     public static final int N=50000;   
     public static List values;   
     static {   
         Integer vals[]=new Integer[N];   
         Random r=new Random();   
         for(int i=0,currval=0;i<N;i++){   
             vals=new Integer(currval);   
             currval+=r.nextInt(100)+1;   
         }   
         values=Arrays.asList(vals);   
     }   
  
     static long timeList(List lst){   
         long start=System.currentTimeMillis();   
         for(int i=0;i<N;i++){   
             int index=Collections.binarySearch(lst, values.get(i));   
             if(index!=i)   
                 System.out.println("***错误***");   
         }   
         return System.currentTimeMillis()-start;   
     }   
     public static void main(String args[]){   
         System.out.println("ArrayList消耗时间："+timeList(new ArrayList(values)));   
         System.out.println("LinkedList消耗时间："+timeList(new LinkedList(values)));   
     }   
}

```
   

我得到的输出 是：

```
ArrayList消耗时间：15 
LinkedList消耗时间：2596 
```

这个结果不是固定的，但是基本上ArrayList的时间要明显小于LinkedList的时间。因此在这种情况下不宜用LinkedList。二分查找法使用的随机访问(random access)策略，而LinkedList是不支持快速的随机访问的。对一个LinkedList做随机访问所消耗的时间与这个list的大小是成比例的。而相应的，在ArrayList中进行随机访问所消耗的时间是固定的。

这是否表明ArrayList总是比LinkedList性能要好呢？这并不一定，在某些情况下LinkedList的表现要优于ArrayList，有些算法在LinkedList中实现时效率更高。比方说，利用 Collections.reverse方法对列表进行反转时，其性能就要好些。

看这样一个例子，加入我们有一个列表，要对其进行大量的插入和删除操作，在这种情况下LinkedList就是一个较好的选择。请看如下一个极端的例子，我们重复的在一个列表的开端插入一个元素： 


```java
package com.mangocity.test;   
  
import java.util.*;   
public class ListDemo {   
     static final int N=50000;   
     static long timeList(List list){   
     long start=System.currentTimeMillis();   
     Object o = new Object();   
     for(int i=0;i<N;i++)   
         list.add(0, o);   
     return System.currentTimeMillis()-start;   
     }   
     public static void main(String[] args) {   
         System.out.println("ArrayList耗时："+timeList(new ArrayList()));   
         System.out.println("LinkedList耗时："+timeList(new LinkedList()));   
     }   
}
```
这时我的输出结果是：
```
ArrayList耗时：2463 
LinkedList耗时：15 
```

这和前面一个例子的结果截然相反，当一个元素被加到ArrayList的最开端时，所有已经存在的元素都会后移，这就意味着数据移动和复制上的开销。相反的，将一个元素加到LinkedList的最开端只是简单的未这个元素分配一个记录，然后调整两个连接。在LinkedList的开端增加一个元素的开销是固定的，而在ArrayList的开端增加一个元素的开销是与ArrayList的大小成比例的。

#### 空间复杂度

在LinkedList中有一个私有的内部类，定义如下： 

```java
private static class Entry {   
     Object element;   
     Entry next;   
     Entry previous;   
}   
```
每个Entry对象 reference列表中的一个元素，同时还有在LinkedList中它的上一个元素和下一个元素。一个有1000个元素的LinkedList对象将有1000个链接在一起的Entry对象，每个对象都对应于列表中的一个元素。这样的话，在一个LinkedList结构中将有一个很大的空间开销，因为它要存储这1000个Entity对象的相关信息。

ArrayList使用一个内置的数组来存储元素，这个数组的起始容量是10.当数组需要增长时，新的容量按 如下公式获得：新容量=(旧容量*3)/2+1，也就是说每一次容量大概会增长50%。这就意味着，如果你有一个包含大量元素的ArrayList对象， 那么最终将有很大的空间会被浪费掉，这个浪费是由ArrayList的工作方式本身造成的。如果没有足够的空间来存放新的元素，数组将不得不被重新进行分 配以便能够增加新的元素。对数组进行重新分配，将会导致性能急剧下降。如果我们知道一个ArrayList将会有多少个元素，我们可以通过构造方法来指定容量。我们还可以通过trimToSize方法在ArrayList分配完毕之后去掉浪费掉的空间。


#### 总结 

ArrayList和LinkedList在性能上各 有优缺点，都有各自所适用的地方，总的说来可以描述如下：

1．对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。对ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对LinkedList而言，这个开销是 统一的，分配一个内部Entry对象。 
2．在ArrayList的中间插入或删除一个元素意味着这个列表中剩余的元素都会被移动；而在LinkedList的中间插入或删除一个元素的开销是固定的。

3．LinkedList不 支持高效的随机元素访问。
4．ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间 

可以这样说：当操作是在一列 数据的后面添加数据而不是在前面或中间,并且需要随机地访问其中的元素时,使用ArrayList会提供比较好的性能；当你的操作是在一列数据的前面或中 间添加或删除数据,并且按照顺序访问其中的元素时,就应该使用LinkedList了。 

所以，如果只是查找特定位置的元素或只在集合的末端增加、移除元素，那么使用Vector或ArrayList都可以。如果是对其它指定位置的插入、删除操作，最好选择LinkedList。

## String、StringBuffer和StringBuilder

### String
String：字符串常量，字符串长度不可变。Java中String是immutable（不可变）的。
String类的包含如下定义：
```java
/** The value is used for character storage. */  
private final char value[];  
  
/** The offset is the first index of the storage that is used. */  
private final int offset;  
  
/** The count is the number of characters in the String. */  
private final int count;  

```

用于存放字符的数组被声明为final的，因此只能赋值一次，不可再更改。

### StringBuffer（JDK1.0）

StringBuffer：字符串变量（Synchronized，即线程安全）。如果要频繁对字符串内容进行修改，出于效率考虑最好使用StringBuffer，如果想转成String类型，可以调用StringBuffer的toString()方法。

Java.lang.StringBuffer线程安全的可变字符序列。在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。可将字符串缓冲区安全地用于多个线程。

StringBuffer上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。每个方法都能有效地将给定的数据转换成字符串，然后将该字符串的字符追加或插入到字符串缓冲区中。append 方法始终将这些字符添加到缓冲区的末端；而insert方法则在指定的点添加字符。例如，如果z引用一个当前内容是“start”的字符串缓冲区对象，则此方法调用z.append("le")会使字符串缓冲区包含“startle”，而z.insert(4, "le")将更改字符串缓冲区，使之包含“starlet”。

### StringBuilder（JDK5.0）

StringBuilder：字符串变量（非线程安全）。在内部，StringBuilder对象被当作是一个包含字符序列的变长数组。

java.lang.StringBuilder是一个可变的字符序列，是JDK5.0新增的。此类提供一个与StringBuffer兼容的API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。

其构造方法如下：

构造方法 | 描述
 ----- | :-------------
StringBuilder()    | 创建一个容量为16的StringBuilder对象（16个空元素）
StringBuilder(CharSequence cs)  | 创建一个包含cs的StringBuilder对象，末尾附加16个空元素 
StringBuilder(int initCapacity) | 创建一个容量为initCapacity的StringBuilder对象 
StringBuilder(String s) | 创建一个包含s的StringBuilder对象，末尾附加16个空元素 

在大部分情况下，StringBuilder > StringBuffer。这主要是由于前者不需要考虑线程安全。

### 三者区别
String 类型和StringBuffer的主要性能区别：String是不可变的对象,因此在每次对String类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象，所以经常改变内容的字符串最好不要用String，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，性能就会降低。

使用StringBuffer类时，每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用。所以多数情况下推荐使用StringBuffer，特别是字符串对象经常改变的情况下。

在某些特别情况下，String对象的字符串拼接其实是被JavaCompiler编译成了StringBuffer对象的拼接，所以这些时候String对象的速度并不会比StringBuffer对象慢，例如：
```java
String s1 = "This is only a" + " simple" + " test";  
StringBuffer Sb = new StringBuilder("This is only a").append(" simple").append(" test");  

```

生成 String s1对象的速度并不比 StringBuffer慢。其实在Java Compiler里，自动做了如下转换：
Java Compiler直接把上述第一条语句编译为：
```java
String s1 = "This is only a simple test";  
```

所以速度很快。但要注意的是，如果拼接的字符串来自另外的String对象的话，Java Compiler就不会自动转换了，速度也就没那么快了，例如：

```java
String s2 = "This is only a";  
String s3 = " simple";  
String s4 = " test";  
String s1 = s2 + s3 + s4;  
```

这时候，Java Compiler会规规矩矩的按照原来的方式去做，String的concatenation（即+）操作利用了StringBuilder（或StringBuffer）的append方法实现，此时，对于上述情况，若s2，s3，s4采用String定义，拼接时需要额外创建一个StringBuffer（或StringBuilder），之后将StringBuffer转换为String；若采用StringBuffer（或StringBuilder），则不需额外创建StringBuffer。

### 使用策略

1. 基本原则：如果要操作少量的数据，用String；单线程操作大量数据，用StringBuilder；多线程操作大量数据，用StringBuffer。

2. 不要使用String类的"+"来进行频繁的拼接，因为那样的性能极差的，应该使用StringBuffer或StringBuilder类，这在Java的优化上是一条比较重要的原则。例如：


```java
String result = "";  
for (String s : hugeArray) {  
    result = result + s;  
}  
  
// 使用StringBuilder  
StringBuilder sb = new StringBuilder();  
for (String s : hugeArray) {  
    sb.append(s);  
}  
String result = sb.toString();  
```
当出现上面的情况时，显然我们要采用第二种方法，因为第一种方法，每次循环都会创建一个String result用于保存结果，除此之外二者基本相同（对于jdk1.5及之后版本）。

3. 为了获得更好的性能，在构造StringBuffer或StringBuilder时应尽可能指定它们的容量。当然，如果你操作的字符串长度（length）不超过16个字符就不用了，当不指定容量（capacity）时默认构造一个容量为16的对象。不指定容量会显著降低性能。

4. StringBuilder一般使用在方法内部来完成类似"+"功能，因为是线程不安全的，所以用完以后可以丢弃。StringBuffer主要用在全局变量中。

5. 相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。而在现实的模块化编程中，负责某一模块的程序员不一定能清晰地判断该模块是否会放入多线程的环境中运行，因此：除非确定系统的瓶颈是在StringBuffer上，并且确定你的模块不会运行在多线程模式下，才可以采用StringBuilder；否则还是用StringBuffer。

## Map、Set、List、Queue、Stack的特点与用法

这个篇幅太长，另起一篇:[Map、Set、List、Queue、Stack的特点与用法][1]。







[1]:/2017/09/27/Map-Set-List-Queue-Stack/




















