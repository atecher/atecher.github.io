---
title: Java_改善_枚举和注解结合使用威力更大
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 改善
comments: false
---

注解的写法和接口很类似,都采用了关键字interface,而且都不能有实现代码,常量定义默认都是pulbic static final类型的.
他们的主要不同点是:注解在interface前加上@字符,而且不能继承,不能实现,这经常会给我们的开发带来一些障碍.

分析一个ACL(Access Contorl List ,访问控制列表)设计案例..看看如何避免这些障碍.
ACL中有三个重要的元素:
1. 资源,有哪些信息是要被控制起来的.
2. 权限级别,不同的访问者在规划在不同的级别中.
3. 控制器(也叫鉴权人),控制不同的级别访问不同的资源.

鉴权人是整个ACL的实际核心,我们从最主要的鉴权人开始,看代码:
```java
//鉴权者接口
interface Identifier {
    //无权访问时的礼貌语
    String REFUSE_WORD = "您无权访问";
    // 鉴权
    public boolean identify();
}
```
这是一个鉴权人的接口,定义了一个常量和一个鉴权方法,接下来应该实现该鉴权方法,但问题是我们的权限级别和鉴权方法之间是紧耦合的,若分拆成两个类显得有点啰嗦,怎么办?我们可以直接定义一个枚举来实现.
```java
//常用鉴权者
enum CommonIdentifier implements Identifier {
    //权限级别
    Reader, Author, Admin;

    //实现鉴权
    public boolean identify() {
        return false;
    }
}
```

定义了一个通用鉴权者,使用的是枚举类型,并且实现了鉴权者接口,现在就剩下资源定义了,这很容易定义,资源就是我们写的类,方法等,之后再通过配置来决定哪些类,方法允许什么级别的访问,这里的问题是:怎么把资源和权限级别关联起来呢?

使用XML配置文件?是个方法,但是对于我们的示例程序来说显得太过繁重,使用注解会更简洁些.需要首先定义出权限级别的注解,代码如下:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Access {
    //确定什么级别可以访问
    CommonIdentifier level() default CommonIdentifier.Admin;
}
```

该注解是标注在类上面的,并且会保留到运行期,我们定义一个资源类,代码如下:
```java
//商业逻辑，默认访问权限是Admin
@Access(level = CommonIdentifier.Author)
class Foo {

}
```

Foo类只能是作者级别的人的访问,场景定义完毕,看如何模拟ACL的实现...看代码:
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class Client {
    public static void main(String[] args) {
        //初始化商业逻辑
        Foo b = new Foo();
        //获取注解
        Access access = b.getClass().getAnnotation(Access.class);
        //没有Access注解或者鉴权失败
        if (access == null || !access.level().identify()) {
            //没有Access注解或者鉴权失败
            System.out.println(access.level().REFUSE_WORD);
        }

    }

}

//商业逻辑，默认访问权限是Admin
@Access(level = CommonIdentifier.Author)
class Foo {

}

//鉴权者接口
interface Identifier {
    //无权访问时的礼貌语
    String REFUSE_WORD = "您无权访问";
    // 鉴权
    public boolean identify();
}

//常用鉴权者
enum CommonIdentifier implements Identifier {
    //权限级别
    Reader, Author, Admin;

    //实现鉴权
    public boolean identify() {
        return false;
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Access {
    //确定什么级别可以访问
    CommonIdentifier level() default CommonIdentifier.Admin;
}

/*
打印输出:
您无权访问
*/
```

ref:
[http://www.cnblogs.com/DreamDrive/p/5640900.html](http://www.cnblogs.com/DreamDrive/p/5640900.html)

<!-- more -->

