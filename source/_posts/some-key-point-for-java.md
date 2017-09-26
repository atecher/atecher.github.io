---
title: Java的一些关键技术点
date: 2017-07-20 10:35:46
categories: "开发技巧"
tags: 
 - Java基础
---

基础这东西很重要，各个公司都很看重。基础这些东西无非几部分：逻辑思维，语言，操作系统，网络，数据结构和算法，再加上行业和领域的相关知识。这些都需要我们在平时积累和学习，今天这篇文章主要罗列一下Java语言中的一些技术点，内容会随着时间不断添加。
<!-- more -->

### Switch能否用string做参数

在jdk 7 之前，switch 只能支持 byte、short、char、int 这几个基本数据类型和其对应的封装类型。switch后面的括号里面只能放int类型的值，但由于byte，short，char类型，它们会 自动 转换为int类型（精精度小的向大的转化），所以它们也支持。

> 对于精度比int大的类型，比如long、float，doulble，不会自动转换为int，如果想使用，就必须强转为int，如(int)float;
#### JDK7之前

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

#### JDK7之后
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

#### Why

jdk7并没有新的指令来处理switch，而是通过调用switch中String.hashCode,将String转换为int从而进行判断。

### equals和==的区别
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

### Java的基本数据类型以及封装类
java提供了九种基本数据类型，包括：boolean, byte, char, short,  int, long, float, double, void。
同时，java也提供了这些类型的封装类，分别为：Boolean, Byte, Character, Short, Integer, Long, Float, Double, Void。





