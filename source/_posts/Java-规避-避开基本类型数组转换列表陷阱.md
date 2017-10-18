---
title: Java_规避_避开基本类型数组转换列表陷阱
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 规避
comments: false
---


<!-- more -->

## 问题
开发中经常用到Arrays和Collections这两个工具类. 在数组和列表之间进行切换.非常方便.但是也会遇到一些问题.
看代码:
```java
import java.util.Arrays;
import java.util.List;
public class Client {
    public static void main(String[] args) {
        int[] data = {1,2,3,4,5};
        List list = Arrays.asList(data);
        System.out.println("列表中的元素数量是：" + list.size());
    }
}

/*
运行结果: 
列表中的元素数量是：1
*/
```

## 分析
为什么不是5? 事实上data确实是一个有5个元素的int类型数组,只是通过asList转换列表之后就只有一个元素了.
看Arrays.asList的方法说明:输入一个变长参数,返回一个固定长度的列表.
```java
    /**
     * Returns a fixed-size list backed by the specified array.  (Changes to
     * the returned list "write through" to the array.)  This method acts
     * as bridge between array-based and collection-based APIs, in
     * combination with {@link Collection#toArray}.  The returned list is
     * serializable and implements {@link RandomAccess}.
     *
     * <p>This method also provides a convenient way to create a fixed-size
     * list initialized to contain several elements:
     * <pre>
     *     List&lt;String&gt; stooges = Arrays.asList("Larry", "Moe", "Curly");
     * </pre>
     *
     * @param a the array by which the list will be backed
     * @return a list view of the specified array
     */
    @SafeVarargs
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```
asList方法输入的是一个泛型变长参数,我们知道基本类型是不能泛型化的,也就是说8个基本类型不能作为泛型参数,要想作为泛型参数就必须使用其所对应的包装类型,那前面的例子传递了一个int类型的数组,程序为何没有编译报错?
Java中数组是一个对象,它是可以泛型化的,也就说例子中是把一个int类型的数组作为了T的类型,所以转换后在List中就只有一个类型为int数组的元素了.打印出来
```java
import java.util.Arrays;
import java.util.List;

public class Client {
    public static void main(String[] args) {
        int[] data = {1,2,3,4,5};
        List list = Arrays.asList(data);
        System.out.println("元素类型：" + list.get(0).getClass());
        System.out.println("前后是否相等："+data.equals(list.get(0)));
    }
}
/*
运行输出:
元素类型：class [I
前后是否相等：true
*/
```
放在列表中的元素是一个int数组,为什么"元素类型"后的class是"[I"?  因为JVM不可能输出Array类型,因为Array是属于java.lang.reflect包的,它是通过反射访问数组元素的工具类.在Java中任何一个数组的类都是"[I"(如果是double对应"[D",float对应的是"[F"),究其原因就是Java中并没有定义数组这个类,它是编译器编译的时候生成的,是一个特殊的类,在JDK的帮助中也没有任何数组类的信息.


## 解决
修改方案,直接使用包装类即可,代码如下:
```java
import java.util.Arrays;
import java.util.List;

public class Client {
    public static void main(String[] args) {
        Integer[] data = {1,2,3,4,5};
        List list = Arrays.asList(data);
        System.out.println("列表中的元素数量是：" + list.size());
    }
}
/*
运行输出:
列表中的元素数量是：5
*
```

仅仅把int变成Integer,即可让输出的元素数量变成5,需要说明的是,不仅仅是int类型的数组有这个问题,其他7个基本类型的数组也都存在相似的问题.
在把基本类型数组转换成列表时,要特别小心asList方法的陷阱,避免出现程序逻辑混乱的情况.


## 建议
原始类型数组不能作为asList的输入参数,否则会引起程序逻辑混乱.

ref:
[http://www.cnblogs.com/DreamDrive/p/5641065.html](http://www.cnblogs.com/DreamDrive/p/5641065.html)
