---
title: Java_性能_不同的列表选择不同的遍历方法
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 性能
comments: false
---


<!-- more -->

## 标识接口
在Java中，RandomAccess和Cloneable、Serializable一样都是标识接口，不需要任何实现，只是用来表明其实现类具有某种特质的，实现了Cloneable表明可以被拷贝，实现了Serializable接口表明被序列化了，实现了RandomAccess则表明这个类可以随机存取。
ArrayList数组实现了RandomAccess接口（随机存取接口），标识着ArrayList是一个可以随机存取的列表，即元素之间没有关联，即两个位置相邻的元素之间没有相互依赖关系，可以随机访问和存储。
LinkedList类也是一个列表，它是有序存取的，实现了双向链表、每个数据节点都有单个数据项，前面节点的引用（Previous Node）、本节点元素（Node Element）、后续节点的引用（Next Node）。也就是说LinkedList两个元素本来就是有联系的，我知道你存在，你知道我存在。

## 场景:

我们来看一个场景，统计一个省的各科高考科目考试的平均分.
当然使用数据库中的一个SQL语句就能求出平均值,不过这个不再我们的考虑之列,这里只考虑使用纯Java的方式来解决.
看代码:
```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Random;

public class Client {
    public static void main(String[] args) {
        //学生数量,80万
        int stuNum = 800*1000;
        //List集合，记录所有学生的分数
        List<Integer> scores = new ArrayList<Integer>(stuNum);
        //写入分数
        for(int i=0;i<stuNum;i++){
            scores.add(new Random().nextInt(150));
        }
        //记录开始计算时间
        long start = System.currentTimeMillis();
        System.out.println("平均分是：" + average(scores));
        System.out.println("执行时间：" + (System.currentTimeMillis() -start) + "ms");
    }

    //计算平均数
    public static int average(List<Integer> list){
        int sum = 0;
        //遍历求和
        for(int i:list){
            sum +=i;
        }
        /*
        Java中的foreach()语法是iterator(迭代器)的变形用法,上面的foreach语法和下面的代码等价
        for(Iterator<Integer> i=list.iterator(); i.hasNext(); ){
            sum +=i.next();
        }
         */

        //除以人数，计算平均值
        return sum/list.size();
    }
}
/*
输出结果：
平均分是：74
执行时间：47ms
*/
```


## 遍历优化
仅仅求一个平均值就花费了47毫秒，考虑其他诸如加权平均值、补充平均值等的话，花费时间肯定更长。我们仔细分析一下arverage方法，加号操作是最基本操作，没有可以优化，我们可以尝试对List遍历进行优化。
List的遍历还有另外一种形式，即通过下表方式来遍历，如下：
```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Random;

public class Client {
    public static void main(String[] args) {
        //学生数量,800万
        int stuNum = 800*10000;
        //List集合，记录所有学生的分数
        List<Integer> scores = new ArrayList<Integer>(stuNum);
        //写入分数
        for(int i=0;i<stuNum;i++){
            scores.add(new Random().nextInt(150));
        }
        //记录开始计算时间
        long start = System.currentTimeMillis();
        System.out.println("平均分是：" + average(scores));
        System.out.println("执行时间：" + (System.currentTimeMillis() -start) + "ms");
    }

    //计算平均数
    public static int average(List<Integer> list){
        int sum = 0;
        //遍历求和
        for(int i = 0, size = list.size(); i < size; i++){
            sum += list.get(i);
        }
        //除以人数，计算平均值
        return sum/list.size();
    }
}
/*
运行结果如下:
平均分是：74
执行时间：58ms
*/
```
执行时间大幅提升，性能提升65%。
为什么会有如此提升呢？我们知道foreacher与下面代码等价：
```java
for(Iterator<Integer> i = list.iterator(); i.hasNext;){
    sum += i.next();
}
```

迭代器是23中设计模式的一种，提供一种方法访问一个容器对象中的各个元素，同时又无须暴露该对象的内部细节。也就是说对于ArrayList，需要先创建一个迭代器容器，然后屏蔽内部遍历细节，对外提供hasNext、next等方法。
问题是ArrayList实现了RandomAccess接口，表明元素之间本没有关系，为了使用迭代器就需要强制建立一种互相“知晓”的关系，比如上一个元素可以判断是否有下一个元素，以及下一个元素是什么等关系，这也就是通过foreach遍历耗时的原因。

Java为ArrayList类加上了RandomAccess接口,就是在告诉我们"ArrayList是随机存取的,采用下标方式遍历列表速度会更快".

但是为什么不把RadomAccess加到所有的List实现类上呢?
那是因为有些List实现类是不能随机存取的,而是有序存取的,比如LinkedList类,LinkedList也是一个列表,但是它实现了双向链表,每个数据节点中都有三个数据项:前节点的引用(Previous Node),本节点元素(Node Element),后继节点的引用(Next Node),这是数据结构的节本知识,也就是在LinkedList中的两个元素本来就是有关联的,我知道你的存在,你也知道我的存在.

综上对于LinkedList由分析讲述，元素之间已经有关联了，使用foreach也就是迭代器方式是不是更高呢？代码如下
```java
import java.util.LinkedList;
import java.util.List;
import java.util.Random;

public class Client {
    public static void main(String[] args){
        //学生数量，80万
        int stuNum = 800 * 10000;
        //List集合，记录所有学生分数
        List<Integer> scores = new LinkedList<Integer>();

        //写入分数
        for(int i = 0; i < stuNum; i++){
            scores.add(new Random().nextInt(150));
        }

        //记录开始计算时间
        long start = System.currentTimeMillis();
        System.out.println("平均分是：" + average(scores));
        System.out.println("执行时间：" + (System.currentTimeMillis() - start) + "ms");
    }
    public static int average(List<Integer> list){
        int sum = 0;
        //foreach遍历求和
        for(int i : list){
            sum += i;
        }
        //除以人数，计算平均值
        return sum/list.size();
    }
}
/*
运行结果:
平均分是：74
执行时间：118ms
*/
```
可能这个数据量不是很适合.....用八十万量的数据量LinkedList使用foreach的速度和ArrayList使用普通for循环的速度差不多.....
可以测试使用下标的方式遍历LinkedList中的元素:
其实不用测试,效率真的非常低,直接看源代码:
```java
public E get(int index){
    return entry(index).element;
}
```
由entry方法查找指定下标的节点，然后返回其包含的元素，看entry方法：
```java
private Entry<E> entry(int index){
    //检查下标是否越界
    Entry<E> e = header;
    if(index < (size >> 1)){
        //如果下标小于中间值，则从头节点开始搜索
        for(int i = 0; i <= index; I++){
        e = e.next;
    }
    }else{
        //如果下标大于等于中间值，则从尾节点反向遍历
        for(int i = size; i > index; i++){
            e = e.previous;
        }
    }
    return e;
}
```
程序会先判断输入的下标与中间值(size右移一位,也就是除以2了)的关系,小于中间值则从头开始正向搜索,大于中间值则从尾节点反向搜索,想想看,每一次的get方法都是一个遍历,"性能"两字从何说起呢!
明白了随机存取列表和有序存取列表的区别,average方法就必须重构,以便实现不同的列表采用不同的遍历方式.代码如下:
```java
import java.util.LinkedList;
import java.util.List;
import java.util.Random;
import java.util.RandomAccess;

public class Client {
    public static void main(String[] args) {
        // 学生数量,80万
        int stuNum = 80 * 10000;
        // List集合，记录所有学生的分数
        List<Integer> scores = new LinkedList<Integer>();
        // 写入分数
        for (int i = 0; i < stuNum; i++) {
            scores.add(new Random().nextInt(150));
        }

        // 记录开始计算时间
        long start = System.currentTimeMillis();
        System.out.println("平均分是：" + average(scores));
        System.out.println("执行时间：" + (System.currentTimeMillis() - start)
                + "ms");
    }

    // 计算平均数
    public static int average(List<Integer> list) {
        int sum = 0;
        if (list instanceof RandomAccess) {
            //可以随机存取，则使用下标遍历
            for (int i = 0, size = list.size(); i < size; i++) {
                sum += list.get(i);
            }
        } else {
            //有序存取，使用foreach方式
            for (int i : list) {
                sum += i;
            }
        }
        // 除以人数，计算平均值
        return sum / list.size();
    }
}
```
这样无论是随机存取列表还是有序列表,程序都可以提供快速的遍历.
列表遍历也不是那么简单的,适时选择最优的遍历方式,不要固化为一种.
ref:
[http://www.cnblogs.com/DreamDrive/p/5647953.html](http://www.cnblogs.com/DreamDrive/p/5647953.html)
