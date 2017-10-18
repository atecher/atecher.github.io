---
title: Java-LinkedList-LinkedList的局限
date: 2017-09-29 00:00:00
categories: Java
tags:
    - Java
    - LinkedList
---

java.util.LinkedList是双向链表,这个大家都知道,比如Java的基础面试题喜欢问ArrayList和LinkedList的区别,在什么场景下用。
大家都会说LinkedList随机增删多的场景比较合适,而ArrayList的随机访问多的场景比较合适。
更进一步,我有时候会问,LinkedList.remove(Object)方法的时间复杂度是什么？有的人回答对了,有的人回答错了。回答错的应该是没有读过源码。
理论上说,双向链表的删除的时间复杂度是O(1),你只需要将要删除的节点的前节点和后节点相连,然后将要删除的节点的前节点和后节点置为null即可,

<!-- more -->

先看看删除操作
```java
    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```
这个操作的时间复杂度可以认为是O(1)级别的。但是LinkedList的实现是一个通用的数据结构,因此没有暴露内部的节点 Node 对象,remove(Object)传入的Object其实是节点存储的value,这里还需要一个查找过程:
```java
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            // 查找节点 Node
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```
因此,显然,LinkedList.remove(Object)方法的时间复杂度是O(n)+O(1),结果仍然是O(n)的时间复杂度,而非推测的O(1)复杂度。最坏情况下要删除的元素是最后一个,你都要比较 N-1 次才能找到要删除的元素。

既然如此,说LinkedList适合随机删减有个前提,链表的大小不能太大,如果链表元素非常多,调用remove(Object)去删除一个元素的效率肯定有影响,一个简单测试,插入100万数据,随机删除1000个元素:
```java
    public static void main(String[] args) {
		final List<Integer> list = new LinkedList<Integer>();
		final int count = 1000000;

		for (int i = 0; i < count; i++) {
			list.add(i);
		}

		final Random rand = new Random();
		long start = System.nanoTime();
		// remove(Object) 耗时 3.196757626
		/*for (int i = 0; i < 1000; i++) {
			//这里要强制转型为Integer,否则调用的是remove(int)
			list.remove((Integer) rand.nextInt(count));
		}*/

		// remove(int) 耗时 1.182296505
		for (int i = 0; i < 1000; i++) {
            // 随机数范围要递减,防止数组越界
			list.remove(rand.nextInt(list.size() - 1));
		}

		System.out.println((System.nanoTime() - start) / Math.pow(10, 9));
	}
```
在我的机器上耗时近 3.19 秒,删除1000个元素耗时 3.19 秒,耗时很长？注意到上面的注释,产生的随机数强制转为Integer对象,否则调用的是 remove(int)方法,而非remove(Object)。如果我们调用remove(int)根据索引来删除,换成remove(int)效率提高不少,这是因为 remove(int)的实现很有技巧,它首先判断索引位置在链表的前半部分还是后半部分,如果是前半部分则从head往前查找,如果在后半部分,则从 head往后查找（LinkedList的实现是一个环）:
```java
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

    Node<E> node(int index) {
        // assert isElementIndex(index);

        // size >> 1 判断索引位置是前半部分还是后半部分
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
最坏情况下要删除的节点在中点左右,查找的次数仍然达到n/2次,但是注意到这里没有比较的开销,并且比remove(Object)最坏情况下n次查找还是好很多。

总结下,LinkedList的两个remove方法,remove(Object)和remove(int)的时间复杂度都是O(n),在链表元素很多并且没有索引可用的情况下,LinkedList也并不适合做随机增删元素。在对性能特别敏感的场景下,还是需要自己实现专用的双向链表结构,真正实现 O(1)级别的随机增删。更进一步,jdk5引入的ConcurrentLinkedQueue是一个非阻塞的线程安全的双向队列实现,同样有本文提到的问题,有兴趣可以测试一下在大量元素情况下的并发随机增删,效率跟自己实现的特定类型的线程安全的链表差距是惊人的。

题外,ArrayList比LinkedList更不适合随机增删的原因是多了一个数组移动的动作,假设你删除的元素在m,那么除了要查找m次之外,还需要往前移动 n-m-1 个元素。

ref:
[http://jm.taobao.org/2010/09/16/322/](http://jm.taobao.org/2010/09/16/322/)
