---
title: Java_性能_选择适合的字符串拼接方法
date: 2017-07-28 11:58:48
categories: Java
tags:
    - Java
    - 性能
---

<!-- more -->

## 问题

对一个字符串拼接有三种方法:加号,contact方法,StringBuffer或者StringBuilder的append方法,其中加号是最常用的.其他两种方式偶尔会出现在一些开源项目中,那么这三者有什么区别?
str += "c"; //加号拼接
str = str.concat("c"); //concat方法连接
以上是两种不同的字符串拼接方式,循环5万次后再检查执行的时间,加号方式执行的时间是1438毫秒,而concat方法的执行时间是703毫秒,时间相差一倍,如果使用StringBuilder方式,执行时间会更少.

```java
public class Client {
    public static final int MAX_LOOP = 50000;
    public static void main(String[] args) {
        doWithPlus();
        doWithConcat();
        doWithStringBuffer();
        
        String str ="abc";
        String str1 = str.concat("1");
        String str2 = "abc1";
        System.out.println(str1 == str2);
    }
    
    public static void doWithPlus(){
        String str = "a";
        long start = System.currentTimeMillis();
        for(int i=0;i<MAX_LOOP;i++){
            str += "c";
            //str = new StringBuilder(prefix).append("c").toString();
        }
        long finish = System.currentTimeMillis();
        System.out.println("doWithPlus:" + (finish - start) + "ms");
    }
    
    public static void doWithConcat(){
        String str = "a";
        long start = System.currentTimeMillis();
        for(int i=0;i<MAX_LOOP;i++){
            str = str.concat("c");
        }
        long finish = System.currentTimeMillis();
        System.out.println("doWithConcat:" + (finish - start) + "ms");
    }
    
    public static void doWithStringBuffer(){
        StringBuilder sb = new StringBuilder("a");
        long start = System.currentTimeMillis();
        for(int i=0;i<MAX_LOOP;i++){
            sb.append("c");
        }
        String str = sb.toString();
        long finish = System.currentTimeMillis();
        System.out.println("doWithStringBuffer:" + (finish - start) + "ms");
    }
}

/*
运行结果:
doWithPlus:1559ms
doWithConcat:748ms
doWithStringBuffer:2ms
false
*/
```

StringBuffer的append方法的执行时间是0毫秒.说明时间非常的短(毫秒不足以计时,可以使用纳秒进行计算).这个实验说明在字符串拼接的方式中,append方法最快,concat方法次之,加号最慢,这是为何呢?


## 三种方法区别

### "+"方法拼接字符串
虽然编译器对字符串的加号做了优化,它会使用StringBuilder的append方法进行追加,按道理来说,其执行时间应该也是0毫秒,不过它最终是通过toString方法转换成String字符串的,例子中"+"拼接的代码与如下代码相同:
```java
str = new StringBuilder(str).append("c").toString();
```
它与纯粹的使用StrignBuilder的append方法是不同的,意思每次循环都会创建一个StringBuilder对象,二是每次执行完毕都要调用toString方法将其转换为字符串------它的时间都耗费在这里了.

### concat方法拼接字符串
```java
public String concat(String str) {
    int otherLen = str.length();
    //如果追加的字符串长度为0,着返回字符串本身
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    //追加的字符串转化成字符数组,添加到buf中
    str.getChars(buf, len);
    //复制字符数组,产生一个新的字符串
    return new String(buf, true);
}
```
其整体看上去就是一个数组的拷贝,虽然在内存中的处理都是原子性操作,速度非常快,不过,注意看最后的return语句,每次的concat操作都会新创建一个String对象,这就是concat速度慢下来的真正原因,它创建了5万个String对象.

### append方法拼接字符串
StringBuilder的append方法直接由父类AbstractStringBuilder实现,其代码如下
```java
public AbstractStringBuilder append(String str) {
    if (str == null) str = "null";//如果是null值,则把null作为字符串处理
    int len = str.length();
    ensureCapacityInternal(count + len);//加长,并作数组拷贝
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```
整个append方法都在做字符组处理,加长,然后数组拷贝,这些都是基本数据处理,没有新建任何对象,所以速度也就最快了.
例子中是在最后通过StringBuffer的toString返回了一个字符串,也就是在5万次循环结束之后才生成了一个String对象.


## 总结
"+"非常符合我们的编码习惯,适合人类阅读,在大多数情况下都可以使用加号操作,只有在系统性能临界的时候才考虑使用concat或者apped方法.

而且很多时候,系统的80%的系能消耗是在20%的代码上,我们的精力应该更多的投入到算法和结构上.

ref: 
[http://www.cnblogs.com/DreamDrive/p/5660256.html](http://www.cnblogs.com/DreamDrive/p/5660256.html)
