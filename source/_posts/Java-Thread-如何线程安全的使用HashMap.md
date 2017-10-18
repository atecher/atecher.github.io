---
title: Java-Thread-如何线程安全的使用HashMap
date: 2017-09-18 10:57:26
categories: Java
tags:
    - Java
    - Thread
---

总说 HashMap 是线程不安全的,不安全的,不安全的,那么到底为什么它是线程不安全的呢？要回答这个问题就要先来简单了解一下 HashMap 源码中的使用的存储结构(这里引用的是 Java 8 的源码,与7是不一样的)和它的扩容机制。

<!-- more -->

### HashMap的内部存储结构
下面是 HashMap 使用的存储结构:
```java
transient Node<K,V>[] table;
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
```

可以看到 HashMap 内部存储使用了一个 Node 数组(默认大小是16),而 Node 类包含一个类型为 Node 的 next 的变量,也就是相当于一个链表,所有根据 hash 值计算的 bucket 一样的 key 会存储到同一个链表里(即产生了冲突),大概就是下面图的样子
{% asset_img HashMap内部存储结果.png %}

需要注意的是,在 Java 8 中如果 hash 值相同的 key 数量大于指定值(默认是8)时使用平衡树来代替链表,这会将get()方法的性能从O(n)提高到O(logn)。具体的可以看我的另一篇博客[Java 8中HashMap和LinkedHashMap如何解决冲突](https://yemengying.com/2016/02/03/%E8%AF%91-Java%E4%B8%ADHashMap%E5%92%8CLinkedHashMap%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E5%86%B2%E7%AA%81/)。

### HashMap的自动扩容机制
HashMap 内部的 Node 数组默认的大小是16,假设有100万个元素,那么最好的情况下每个 hash 桶里都有62500个元素,这时get(),put(),remove()等方法效率都会降低。为了解决这个问题,HashMap 提供了自动扩容机制,当元素个数达到数组大小 loadFactor 后会扩大数组的大小,在默认情况下,数组大小为16,loadFactor 为0.75,也就是说当 HashMap 中的元素超过16*0.75=12时,会把数组大小扩展为2*16=32,并且重新计算每个元素在新数组中的位置。如下图所示
{% asset_img resizing_of_java_hashmap.jpg %}
从图中可以看到没扩容前,获取 EntryE 需要遍历5个元素,扩容之后只需要2次

为什么线程不安全

个人觉得 HashMap 在并发时可能出现的问题主要是两方面,首先如果多个线程同时使用put方法添加元素,而且假设正好存在两个 put 的 key 发生了碰撞(根据 hash 值计算的 bucket 一样),那么根据 HashMap 的实现,这两个 key 会添加到数组的同一个位置,这样最终就会发生其中一个线程的 put 的数据被覆盖。第二就是如果多个线程同时检测到元素个数超过数组大小* loadFactor ,这样就会发生多个线程同时对 Node 数组进行扩容,都在重新计算元素位置以及复制数据,但是最终只有一个线程扩容后的数组会赋给 table,也就是说其他线程的都会丢失,并且各自线程 put 的数据也丢失。
关于 HashMap 线程不安全这一点,《Java并发编程的艺术》一书中是这样说的:
```
HashMap 在并发执行 put 操作时会引起死循环,导致 CPU 利用率接近100%。因为多线程会导致 HashMap 的 Node 链表形成环形数据结构,一旦形成环形数据结构,Node 的 next 节点永远不为空,就会在获取 Node 时产生死循环。
```

注: 死循环并不是发生在 put 操作时,而是发生在扩容时。详细的解释可以看下面几篇博客:

- [酷壳-Java HashMap的死循环](http://coolshell.cn/articles/9606.html)
- [HashMap在java并发中如何发生死循环](http://firezhfox.iteye.com/blog/2241043)
- [How does a HashMap work in JAVA](http://coding-geek.com/how-does-a-hashmap-work-in-java/)

## 如何线程安全的使用HashMap
了解了 HashMap 为什么线程不安全,那现在看看如何线程安全的使用 HashMap。这个无非就是以下三种方式:

- Hashtable
- ConcurrentHashMap
- Synchronized Map

例子:
```java
//Hashtable  
Map<String, String> hashtable = new Hashtable<>();  
//synchronizedMap  
Map<String, String> synchronizedHashMap = Collections.synchronizedMap(new HashMap<String, String>());  
//ConcurrentHashMap  
Map<String, String> concurrentHashMap = new ConcurrentHashMap<>();
```

### Hashtable

Hashtable 源码中是使用 synchronized 来保证线程安全的,比如下面的 get 方法和 put 方法:
```java
public synchronized V get(Object key) {  
   // 省略实现  
}
public synchronized V put(K key, V value) {  
// 省略实现  
}
```
所以当一个线程访问 HashTable 的同步方法时,其他线程如果也要访问同步方法,会被阻塞住。举个例子,当一个线程使用 put 方法时,另一个线程不但不可以使用 put 方法,连 get 方法都不可以,好霸道啊！！！so~~,效率很低。

### ConcurrentHashMap

ConcurrentHashMap (以下简称CHM)是 JUC 包中的一个类,Spring 的源码中有很多使用 CHM 的地方。之前已经翻译过一篇关于 ConcurrentHashMap 的博客,[如何在java中使用ConcurrentHashMap](http://yemengying.com/2015/11/06/%E3%80%90%E8%AF%91%E3%80%91%E5%A6%82%E4%BD%95%E5%9C%A8java%E4%B8%AD%E4%BD%BF%E7%94%A8ConcurrentHashMap/),里面介绍了 CHM 在 Java 中的实现,CHM 的一些重要特性和什么情况下应该使用 CHM。需要注意的是,上面博客是基于 Java 7 的,和8有区别,在8中 CHM 摒弃了 Segment（锁段）的概念,而是启用了一种全新的方式实现,利用 CAS 算法,有时间会重新总结一下。

### SynchronizedMap
```java
// synchronizedMap方法  
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {  
   return new SynchronizedMap<>(m);  
}
// SynchronizedMap类  
private static class SynchronizedMap<K,V>  
implements Map<K,V>, Serializable {  
private static final long serialVersionUID = 1978198479659022715L;  
private final Map<K,V> m;     // Backing Map  
final Object      mutex;        // Object on which to synchronize  
SynchronizedMap(Map<K,V> m) {  
   this.m = Objects.requireNonNull(m);  
   mutex = this;  
}  
SynchronizedMap(Map<K,V> m, Object mutex) {  
   this.m = m;  
   this.mutex = mutex;  
}  
public int size() {  
   synchronized (mutex) {return m.size();}  
}  
public boolean isEmpty() {  
   synchronized (mutex) {return m.isEmpty();}  
}  
public boolean containsKey(Object key) {  
   synchronized (mutex) {return m.containsKey(key);}  
}  
public boolean containsValue(Object value) {  
   synchronized (mutex) {return m.containsValue(value);}  
}  
public V get(Object key) {  
   synchronized (mutex) {return m.get(key);}  
}  
public V put(K key, V value) {  
   synchronized (mutex) {return m.put(key, value);}  
}  
public V remove(Object key) {  
   synchronized (mutex) {return m.remove(key);}  
}  
// 省略其他方法  
}
```
从源码中可以看出调用 synchronizedMap() 方法后会返回一个 SynchronizedMap 类的对象,而在 SynchronizedMap 类中使用了 synchronized 同步关键字来保证对 Map 的操作是线程安全的。

### 性能对比

这是要靠数据说话的时代,所以不能只靠嘴说 CHM 快,它就快了。写个测试用例,实际的比较一下这三种方式的效率,下面的代码分别通过三种方式创建 Map 对象,使用 ExecutorService 来并发运行5个线程,每个线程添加/获取500K个元素。
```java
public class CrunchifyConcurrentHashMapVsSynchronizedMap {  
    public final static int THREAD_POOL_SIZE = 5;  
    public static Map<String, Integer> crunchifyHashTableObject = null;  
    public static Map<String, Integer> crunchifySynchronizedMapObject = null;  
    public static Map<String, Integer> crunchifyConcurrentHashMapObject = null;  
    public static void main(String[] args) throws InterruptedException {  
        // Test with Hashtable Object  
        crunchifyHashTableObject = new Hashtable<>();  
        crunchifyPerformTest(crunchifyHashTableObject);  
        // Test with synchronizedMap Object  
        crunchifySynchronizedMapObject = Collections.synchronizedMap(new HashMap<String, Integer>());  
        crunchifyPerformTest(crunchifySynchronizedMapObject);  
        // Test with ConcurrentHashMap Object  
        crunchifyConcurrentHashMapObject = new ConcurrentHashMap<>();  
        crunchifyPerformTest(crunchifyConcurrentHashMapObject);  
    }  
    public static void crunchifyPerformTest(final Map<String, Integer> crunchifyThreads) throws InterruptedException {  
        System.out.println("Test started for: " + crunchifyThreads.getClass());  
        long averageTime = 0;  
        for (int i = 0; i < 5; i++) {  
            long startTime = System.nanoTime();  
            ExecutorService crunchifyExServer = Executors.newFixedThreadPool(THREAD_POOL_SIZE);  
            for (int j = 0; j < THREAD_POOL_SIZE; j++) {  
                crunchifyExServer.execute(new Runnable() {  
                    @SuppressWarnings("unused")  
                    @Override  
                    public void run() {  
                        for (int i = 0; i < 500000; i++) {  
                            Integer crunchifyRandomNumber = (int) Math.ceil(Math.random() * 550000);  
                            // Retrieve value. We are not using it anywhere  
                            Integer crunchifyValue = crunchifyThreads.get(String.valueOf(crunchifyRandomNumber));  
                            // Put value  
                            crunchifyThreads.put(String.valueOf(crunchifyRandomNumber), crunchifyRandomNumber);  
                        }  
                    }  
                });  
            }  
            // Make sure executor stops  
            crunchifyExServer.shutdown();  
            // Blocks until all tasks have completed execution after a shutdown request  
            crunchifyExServer.awaitTermination(Long.MAX_VALUE, TimeUnit.DAYS);  
            long entTime = System.nanoTime();  
            long totalTime = (entTime - startTime) / 1000000L;  
            averageTime += totalTime;  
            System.out.println("2500K entried added/retrieved in " + totalTime + " ms");  
        }  
        System.out.println("For " + crunchifyThreads.getClass() + " the average time is " + averageTime / 5 + " ms\n");  
    }  
}
```
测试结果:
```
Test started for: class java.util.Hashtable  
2500K entried added/retrieved in 2018 ms  
2500K entried added/retrieved in 1746 ms  
2500K entried added/retrieved in 1806 ms  
2500K entried added/retrieved in 1801 ms  
2500K entried added/retrieved in 1804 ms  
For class java.util.Hashtable the average time is 1835 ms  
Test started for: class java.util.Collections$SynchronizedMap  
2500K entried added/retrieved in 3041 ms  
2500K entried added/retrieved in 1690 ms  
2500K entried added/retrieved in 1740 ms  
2500K entried added/retrieved in 1649 ms  
2500K entried added/retrieved in 1696 ms  
For class java.util.Collections$SynchronizedMap the average time is 1963 ms  
Test started for: class java.util.concurrent.ConcurrentHashMap  
2500K entried added/retrieved in 738 ms  
2500K entried added/retrieved in 696 ms  
2500K entried added/retrieved in 548 ms  
2500K entried added/retrieved in 1447 ms  
2500K entried added/retrieved in 531 ms  
For class java.util.concurrent.ConcurrentHashMap the average time is 792 ms 
```

ref:
[http://yemengying.com/2016/05/07/threadsafe-hashmap/](http://yemengying.com/2016/05/07/threadsafe-hashmap/)
