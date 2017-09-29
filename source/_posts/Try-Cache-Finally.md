---
title: java中finally与return的执行顺序
date: 2016-09-27 11:33:10
categories: "Java"
tags: 
     - Java语法
---

关于finally语句什么时候执行，是不是一定执行，网上有很多不同的说法，那今天我们就实验一下，下面会对这些问题分别测试，有问题大家可以分享一下，共同讨论。

<!-- more -->

## finally语句是不是一定会被执行

我做了一下试验，至少有两种情况是不会执行的：

> 1. 如果没有执行到try catch语句就返回了（这个是废话）
> 2. 如果try语句块中有System.exit(0)，执行到这句时JVM直接被终止了，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到。

## finally和return谁先执行

### finally语句在return语句执行之后return返回之前执行

先上例子：
```java
/**
 * 测试finally和return 哪个先执行
 */
public class FinallyTest {
    int i = 0;

    public int finallyTest() {
        try {
            System.out.println("run finallTest");
            return i += 10;
        } catch (Exception e) {
            System.out.println("run exception");
            return i;
        } finally {
            System.out.println("run finally");
            if (i > 0) {
                System.out.println("b>0, b = " + i);
            }
        }
    }

    public static void main(String[] args) {
        FinallyTest test = new FinallyTest();
        System.out.println(test.finallyTest());
    }
}

```

运行结果：

```
run finallTest
run finally
b>0, b = 10
10
```

再来一个：
```java
public class FinallyTest {
    
    public String returnTest(){
        System.out.println("tun return");

        return "finish";
    }

    public String finallyTest(){
        try{
            System.out.println("run finallTest");
            return returnTest();
        }catch (Exception e){
            System.out.println("run exception");
            return "exception";
        }finally {
            System.out.println("run finally");
        }
    }

    public static void main(String[] args) {
        FinallyTest test=new FinallyTest();
        System.out.println(test.finallyTest());
    }
}
```

运行结果：

```
run finallTest
tun return
run finally
finish
```

**结论：**
>以上两个例子说明了：return执行完并没有立即返回，执行finally语句之后才返回结果。

### finally中的return会覆盖try中的return

```java
/**
 * 验证：finally中的return会覆盖try中的return
 */
public class FinallyTest2 {
    public String finallyTest(){
        try{
            System.out.println("run finallTest");
            return "finish";
        }catch (Exception e){
            System.out.println("run exception");
            return "exception";
        }finally {
            System.out.println("run finally");
            return "finally";
        }
    }

    public static void main(String[] args) {
        FinallyTest2 test=new FinallyTest2();
        System.out.println(test.finallyTest());
    }
}
```

运行结果：

```
run finallTest
run finally
finally
```

>可以看到try中返回的是finish,finally中返回的是finally，最终返回结果为finally。

### finally中是否能改变return的内容？

先看两个例子：
```java
public class FinallyTest3 {

    int returnValue=0;

    public int finallyTest(){
        try{
            System.out.println("run finallTest");
            return returnValue+=10;
        }catch (Exception e){
            System.out.println("run exception");
        }finally {
            System.out.println("run finally");
            returnValue+=20;
        }
        return returnValue;
    }

    public static void main(String[] args) {
        FinallyTest3 test=new FinallyTest3();
        System.out.println(test.finallyTest());
    }

}
```

运行结果：
```
run finallTest
run finally
10
```

我们看到return的值并没有发生变化。

```java
public Map<String,String> finallyTest(){
        Map<String,String> result=new HashMap<>();
        try{
            System.out.println("run finallTest");
            result.put("KEY","finish");
            return result;
        }catch (Exception e){
            System.out.println("run exception");
        }finally {
            System.out.println("run finally");
            result.put("KEY","finally");
            result = null;
        }
        return result;
    }

    public static void main(String[] args) {
        FinallyTest4 test=new FinallyTest4();
        Map<String,String> result = test.finallyTest();
        System.out.println(result.get("KEY"));
    }
```

运行结果：
```
run finallTest
run finally
finally
```

而这个例子，返回的结果发生变化了。
这是说明java对这部分的处理有问题吗？并不是，这个涉及到Java到底是传值还是传址的问题了。简要的说：java只有传值没有传址，这也是为什么result = null这句不起作用。


>最后总结：finally块的语句在try或catch中的return语句执行之后返回之前执行且finally里的修改语句可能影响也可能不影响try或catch中 return已经确定的返回值，若finally里也有return语句则覆盖try或catch中的return语句直接返回。