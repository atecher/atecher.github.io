---
title: Java_性能_枚举项数量限定为64个以内
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 性能
comments: false
---

枚举项的数量为什么要限制在64个以内？

为了更好地使用枚举，Java提供了两个枚举集合：EnumSet和EnumMap，这两个集合使用的方法都比较简单，EnumSet表示其元素必须是某一枚举的枚举项，EnumMap表示Key值必须是某一枚举的枚举项，由于枚举类型的实例数量固定并且有限，相对来说EnumSet和EnumMap的效率会比其它Set和Map要高。

虽然EnumSet很好用，但是它有一个隐藏的特点，昆明Java培训机构的老师逐步分析。在项目中一般会把枚举用作常量定义，可能会定义非常多的枚举项，然后通过EnumSet访问、遍历，但它对不同的枚举数量有不同的处理方式。为了进行对比，我们定义两个枚举，一个数量等于64，一个是65（大于64即可，为什么是64而不是128,512呢，一会解释），代码如下：
```java
//普通枚举项，数量等于64
enum Const{
    A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,
    AA,BB,CC,DD,EE,FF,GG,HH,II,JJ,KK,LL,MM,NN,OO,PP,QQ,RR,SS,TT,UU,VV,WW,XX,YY,ZZ,
    AAA,BBB,CCC,DDD,EEE,FFF,GGG,HHH,III,JJJ,KKK,LLL
}
//大枚举，数量超过64
enum LargeConst{
    A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,
    AA,BB,CC,DD,EE,FF,GG,HH,II,JJ,KK,LL,MM,NN,OO,PP,QQ,RR,SS,TT,UU,VV,WW,XX,YY,ZZ,
    AAAA,BBBB,CCCC,DDDD,EEEE,FFFF,GGGG,HHHH,IIII,JJJJ,KKKK,LLLL,MMMM
}
```
Const的枚举项数量是64，LagrgeConst的枚举项数量是65,接下来我们希望把这两个枚举转换为EnumSet，然后判断一下它们的class类型是否相同，代码如下：
```java
public class Client89 {
    public static void main(String[] args) {
        EnumSet<Const> cs = EnumSet.allOf(Const.class);
        EnumSet<LargeConst> lcs = EnumSet.allOf(LargeConst.class);
        //打印出枚举数量
        System.out.println("Const的枚举数量："+cs.size());
        System.out.println("LargeConst的枚举数量："+lcs.size());
        //输出两个EnumSet的class
        System.out.println(cs.getClass());
        System.out.println(lcs.getClass());
   }
}
```
程序很简单，现在的问题是：cs和lcs的class类型是否相同？应该相同吧，都是EnumSet类的工厂方法allOf生成的EnumSet类，而且JDK API也没有提示EnumSet有子类。我们来看看输出结果：
很遗憾，两者不相等。就差一个元素，两者就不相等了？确实如此，这也是我们重点关注枚举项数量的原因。先来看看Java是如何处理的，首先跟踪allOf方法，其源码如下：
```java
    /**
     * Creates an enum set containing all of the elements in the specified
     * element type.
     *
     * @param <E> The class of the elements in the set
     * @param elementType the class object of the element type for this enum
     *     set
     * @return An enum set containing all the elements in the specified type.
     * @throws NullPointerException if <tt>elementType</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType) {
        EnumSet<E> result = noneOf(elementType);
        result.addAll();
        return result;
    }
```
allOf通过noneOf方法首先生成了一个EnumSet对象，然后把所有的枚举都加进去，问题可能就出在EnumSet的生成上了，我们来看看noneOf的源码：
```java
    /**
     * Creates an empty enum set with the specified element type.
     *
     * @param <E> The class of the elements in the set
     * @param elementType the class object of the element type for this enum
     *     set
     * @return An empty enum set of the specified type.
     * @throws NullPointerException if <tt>elementType</tt> is null
     */
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```

看到这里，恍然大悟，Java原来是如此处理的：当枚举项数量小于等于64时，创建一个RegularEnumSet实例对象，大于64时则创建一个JumboEnumSet实例对象。
为什么要如此处理呢？这还要看看这两个类之间的差异，首先看RegularEnumSet类，源码如下：
```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 3411599620347842686L;
    /**
     * Bit vector representation of this set.  The 2^k bit indicates the
     * presence of universe[k] in this set.
     */
    private long elements = 0L;

    RegularEnumSet(Class<E>elementType, Enum<?>[] universe) {
        super(elementType, universe);
    }

    void addRange(E from, E to) {
        elements = (-1L >>>  (from.ordinal() - to.ordinal() - 1)) << from.ordinal();
    }

    void addAll() {
        if (universe.length != 0)
            elements = -1L >>> -universe.length;
    }
    //其它代码略
}
```
我们知道枚举项的排序值ordinal是从0、1、2......依次递增的，没有重号，没有跳号，RegularEnumSet就是利用这一点把每个枚举项的ordinal映射到一个long类型的每个位置上的，注意看addAll方法的elements元素，它使用了无符号右移操作，并且操作数是负值，位移也是负值，这表示是负数(符号位是1)的"无符号左移"：符号位为0，并补充低位，简单的说，Java把一个不多于64个枚举项映射到了一个long类型变量上。这才是EnumSet处理的重点，其他的size方法、contains方法等都是根据elements方法等都是根据elements计算出来的。想想看，一个long类型的数字包含了所有的枚举项，其效率和性能能肯定是非常优秀的。
我们知道long类型是64位的，所以RegularEnumSet类型也就只能负责枚举项的数量不大于64的枚举(这也是我们以64来举例，而不以128,512举例的原因)，大于64则由JumboEnumSet处理，我们看它是怎么处理的：
```java
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private static final long serialVersionUID = 334349849919042784L;

    /**
     * Bit vector representation of this set.  The ith bit of the jth
     * element of this array represents the  presence of universe[64*j +i]
     * in this set.
     */
    private long elements[];

    // Redundant - maintained for performance
    private int size = 0;

    JumboEnumSet(Class<E>elementType, Enum<?>[] universe) {
        super(elementType, universe);
        elements = new long[(universe.length + 63) >>> 6];
    }

    void addRange(E from, E to) {
        int fromIndex = from.ordinal() >>> 6;
        int toIndex = to.ordinal() >>> 6;

        if (fromIndex == toIndex) {
            elements[fromIndex] = (-1L >>>  (from.ordinal() - to.ordinal() - 1))
                            << from.ordinal();
        } else {
            elements[fromIndex] = (-1L << from.ordinal());
            for (int i = fromIndex + 1; i < toIndex; i++)
                elements[i] = -1;
            elements[toIndex] = -1L >>> (63 - to.ordinal());
        }
        size = to.ordinal() - from.ordinal() + 1;
    }

    void addAll() {
        for (int i = 0; i < elements.length; i++)
            elements[i] = -1;
        elements[elements.length - 1] >>>= -universe.length;
        size = universe.length;
    }
    //其它代码略
}
```
JumboEnumSet类把枚举项按照64个元素一组拆分成了多组，每组都映射到一个long类型的数字上，然后该数组再放置到elements数组中，简单来说JumboEnumSet类的原理与RegularEnumSet相似，只是JumboEnumSet使用了long数组容纳更多的枚举项。不过，这样的程序看着会不会觉得郁闷呢？其实这是因为我们在开发中很少使用位移操作。大家可以这样理解：RegularEnumSet是把每个枚举项映射到一个long类型数字的每个位上，JumboEnumSet是先按照64个一组进行拆分，然后每个组再映射到一个long类型数字的每个位上。

从以上分析可知，EnumSet提供的两个实现都是基本的数字类型操作，其性能肯定比其他的Set类型要好的多，特别是Enum的数量少于64的时候，那简直就是飞一般的速度。

注意：枚举项数量不要超过64，否则建议拆分。


ref:
[http://km.java.tedu.cn/news/163367.html](http://km.java.tedu.cn/news/163367.html)

<!-- more -->

