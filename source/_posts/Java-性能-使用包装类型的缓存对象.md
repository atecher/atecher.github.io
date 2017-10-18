---
title: Java_性能_使用包装类型的缓存对象
date: 2017-07-28 11:58:48
categories: Java
tags:
    - Java
    - 性能
---

<!-- more -->

## Integer 自动装箱
本文将介绍 Java 中 Integer 缓存的相关知识。这是 Java5 中引入的一个有助于节省内存、提高性能的特性。首先看一个使用 Integer 的示例代码，展示了 Integer 的缓存行为。接着我们将学习这种实现的原因和目的。
```java
public class JavaIntegerCache {
	public static void main(String... strings) {
		Integer integer1 = 3;
		Integer integer2 = 3;
		if (integer1 == integer2)
			System.out.println("integer1 == integer2");
		else
			System.out.println("integer1 != integer2");

		Integer integer3 = 300;
		Integer integer4 = 300;
		if (integer3 == integer4)
			System.out.println("integer3 == integer4");
		else
			System.out.println("integer3 != integer4");
	}
}

/*执行结果
integer1 == integer2
integer3 != integer4
*/
```

在 Java 5 中，为 Integer 的操作引入了一个新的特性，用来节省内存和提高性能。整型对象在内部实现中通过使用相同的对象引用实现了缓存和重用。

上面的规则适用于整数区间 -128 到 +127。

这种 Integer 缓存策略仅在自动装箱（autoboxing）的时候有用，使用构造器创建的 Integer 对象不能被缓存。

Java 编译器把原始类型自动转换为封装类的过程称为自动装箱（autoboxing），这相当于调用 valueOf 方法
Integer a = 10; //this is autoboxing
Integer b = Integer.valueOf(10); //under the hood

现在我们知道了 JDK 源码中对应实现的部分在哪里了。我们来看看 valueOf 的源码。下面是 JDK 1.8.0 中的代码。

```java
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
在创建新的 Integer 对象之前会先在 IntegerCache.cache (是个Integer类型的数组)中查找。有一个专门的 Java 类来负责 Integer 的缓存。

## IntegerCache 类
IntegerCache 是 Integer 类中一个私有的静态类。我们来看看这个类，有比较详细的文档，可以提供我们很多信息。
```java
    /**
     * Cache to support the object identity semantics of autoboxing for values between
     * -128 and 127 (inclusive) as required by JLS.
     *
     * The cache is initialized on first usage.  The size of the cache
     * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
     * During VM initialization, java.lang.Integer.IntegerCache.high property
     * may be set and saved in the private system properties in the
     * sun.misc.VM class.
     */

    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

Javadoc 详细的说明这个类是用来实现缓存支持，并支持 -128 到 127 之间的自动装箱过程。最大值 127 可以通过 JVM 的启动参数 -XX:AutoBoxCacheMax=size 修改。 缓存通过一个 for 循环实现。从小到大的创建尽可能多的整数并存储在一个名为 cache 的整数数组中。这个缓存会在 Integer 类第一次被使用的时候被初始化出来。以后，就可以使用缓存中包含的实例对象，而不是创建一个新的实例(在自动装箱的情况下)。

实际上在 Java 5 中引入这个特性的时候，范围是固定的 -128 至 +127。后来在 Java 6 中，最大值映射到 java.lang.Integer.IntegerCache.high，可以使用 JVM 的启动参数设置最大值。这使我们可以根据应用程序的实际情况灵活地调整来提高性能。是什么原因选择这个 -128 到 127 这个范围呢？因为这个范围的整数值是使用最广泛的。 在程序中第一次使用 Integer 的时候也需要一定的额外时间来初始化这个缓存。

## Java 语言规范中的缓存行为
在 Boxing Conversion 部分的Java语言规范(JLS)规定如下：
如果一个变量 p 的值属于：-128至127之间的整数(§3.10.1这个估计是版本号吧)，true 和 false的布尔值 (§3.10.3)，’u0000′ 至 ‘u007f’ 之间的字符(§3.10.4)中时，将 p 包装成 a 和 b 两个对象时，可以直接使用 a == b 判断 a 和 b 的值是否相等。

## 其他缓存的对象
这种缓存行为不仅适用于Integer对象。我们针对所有整数类型的类都有类似的缓存机制。
有 ByteCache 用于缓存 Byte 对象
有 ShortCache 用于缓存 Short 对象
有 LongCache 用于缓存 Long 对象
有 CharacterCache 用于缓存 Character 对象
Byte，Short，Long 有固定范围: -128 到 127。对于 Character, 范围是 0 到 127。除了 Integer 可以通过参数改变范围外，其它的都不行。

## 学以致用
建议声明包装类型的时候，使用valueOf()生成，而不是通过构造函数生成。这样使用整型池，不仅仅提高了系统性能，同时节约了内存空间。

ref: 
[http://blog.csdn.net/qq_27093465/article/details/52473649](http://blog.csdn.net/qq_27093465/article/details/52473649)
[http://javapapers.com/java/java-integer-cache/](http://javapapers.com/java/java-integer-cache/)
