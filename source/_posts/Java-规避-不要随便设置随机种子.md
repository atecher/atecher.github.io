---
title: Java_规避_不要随便设置随机种子
date: 2017-07-28 11:58:48
categories: Java
tags:
    - Java
    - 规避
---

<!-- more -->

## 问题
随机数在太多的地方使用了，比如加密、混淆数据等，我们使用随机数是期望获得一个唯一的、不可仿造的数字，以避免产生相同的业务数据造成混乱。在Java项目中通常是通过Math.random方法和Random类来获得随机数的，我们来看一段代码：
```java
public class Client {
    public static void main(String[] args) {
        Random r = new Random();
        for(int i=1;i<4;i++){
            System.out.println("第"+i+"次："+r.nextInt());
        }  
     }  
}
```
代码很简单，我们一般都是这样获得随机数的，运行此程序可知：三次打印的随机数都不相同，即使多次运行结果也不同，这也正是我们想要随机数的原因。
我们再来看下面的程序：
```java
public class Client {
    public static void main(String[] args) {
        Random r = new Random(1000);
        for (int i = 1; i < 4; i++) {
            System.out.println("第" + i + "次：" + r.nextInt());
        }
    }
}

/*运行结果:
第1次：-1244746321
第2次：1060493871
第3次：-1826063944
*/
```

## 分析
计算机不同输出的随机数也不同，但是有一点是相同的：在同一台机器上，甭管运行多少次，所打印的随机数都是相同的，也就是说第一次运行，会打印出这三个随机数，第二次运行还是打印出这三个随机数，只要是在同一台硬件机器上，就永远都会打印出相同的随机数，似乎随机数不随机了，问题何在？

那是因为产生随机数的种子被固定了，在Java中，随机数的产生取决于种子，随机数和种子之间的关系遵从以下两个规则：
1. 种子不同，产生不同的随机数。
2. 种子相同，即使实例不同也产生相同的随机数。

看完上面两个规则，我们再来看这个例子，会发现问题就出在有参构造上，Random类的默认种子（无参构造）是System.nanoTime()的返回值（JDK 1.5版本以前默认种子是System. currentTimeMillis()的返回值），注意这个值是距离某一个固定时间点的纳秒数，不同的操作系统和硬件有不同的固定时间点，也就是说不同的操作系统其纳秒值是不同的，而同一个操作系统纳秒值也会不同，随机数自然也就不同了。（顺便说下，System.nanoTime不能用于计算日期，那是因为“固定”的时间点是不确定的，纳秒值甚至可能是负值，这点与System. currentTimeMillis不同。）

new Random(1000)显式地设置了随机种子为1000，运行多次，虽然实例不同，但都会获得相同的三个随机数。所以，除非必要，否则不要设置随机种子。

## 源码
```java
    /**
     * Creates a new random number generator. This constructor sets
     * the seed of the random number generator to a value very likely
     * to be distinct from any other invocation of this constructor.
     */
    public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }
    
    /**
     * Creates a new random number generator using a single {@code long} seed.
     * The seed is the initial value of the internal state of the pseudorandom
     * number generator which is maintained by method {@link #next}.
     *
     * <p>The invocation {@code new Random(seed)} is equivalent to:
     *  <pre> {@code
     * Random rnd = new Random();
     * rnd.setSeed(seed);}</pre>
     *
     * @param seed the initial seed
     * @see   #setSeed(long)
     */
    public Random(long seed) {
        if (getClass() == Random.class)
            this.seed = new AtomicLong(initialScramble(seed));
        else {
            // subclass might have overriden setSeed
            this.seed = new AtomicLong();
            setSeed(seed);
        }
    }
```

## 拓展
顺便提一下，在Java中有两种方法可以获得不同的随机数：通过java.util.Random类获得随机数的原理和Math.random方法相同，Math.random()方法也是通过生成一个Random类的实例，然后委托nextDouble()方法的，两者是殊途同归，没有差别。
Math.random()源码如下:
```java
    /**
     * Returns a {@code double} value with a positive sign, greater
     * than or equal to {@code 0.0} and less than {@code 1.0}.
     * Returned values are chosen pseudorandomly with (approximately)
     * uniform distribution from that range.
     *
     * <p>When this method is first called, it creates a single new
     * pseudorandom-number generator, exactly as if by the expression
     *
     * <blockquote>{@code new java.util.Random()}</blockquote>
     *
     * This new pseudorandom-number generator is used thereafter for
     * all calls to this method and is used nowhere else.
     *
     * <p>This method is properly synchronized to allow correct use by
     * more than one thread. However, if many threads need to generate
     * pseudorandom numbers at a great rate, it may reduce contention
     * for each thread to have its own pseudorandom-number generator.
     *
     * @return  a pseudorandom {@code double} greater than or equal
     * to {@code 0.0} and less than {@code 1.0}.
     * @see Random#nextDouble()
     */
    public static double random() {
        return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
    }
    
    private static final class RandomNumberGeneratorHolder {
        static final Random randomNumberGenerator = new Random();
    }
```

## 总结
注意　若非必要，不要设置随机数种子。
ref: 
[http://www.cnblogs.com/DreamDrive/p/5425094.html](http://www.cnblogs.com/DreamDrive/p/5425094.html)
