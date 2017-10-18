---
title: Java_规避_使用valueOf前必须进行校验
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 规避
comments: false
---

每个枚举都是java.lang.Enum的子类,都可以访问Enum类提供的方法,比如hashCode(),name(),valueOf()等.....

其中valueOf()方法会把一个String类型的名称转变为枚举项,也就是枚举项中查找出字面值与该参数相等的枚举项,虽然这个方法很简单,但是JDK却做了一个对于开发人员来说并不简单的处理:

看代码:
```java
import java.util.Arrays;
import java.util.List;

public class Client {
    public static void main(String[] args) {
        //注意summer是小写
        List<String> params = Arrays.asList("Spring", "summer");
        for (String name : params) {
            //查找表面值与name相同的枚举项
            Season s = Season.valueOf(name);
            if (s != null) {
                // 有该枚举项时的处理
                System.out.println(s);
            } else {
                // 没有该枚举项时的逻辑处理
                System.out.println("无相关枚举项");
            }
        }
    }
}

enum Season {
    Spring, Summer, Autumn, Winter;
}
```
运行输出:
```
Spring
Exception in thread "main" java.lang.IllegalArgumentException: No enum constant cn.summerchill.test.Season.summer
    at java.lang.Enum.valueOf(Unknown Source)
    at cn.summerchill.test.Season.valueOf(Client.java:1)
    at cn.summerchill.test.Client.main(Client.java:12)
```
这段代码看起来很完美了,其中考虑到从String转换成枚举类型可能不成功的情况,比如没有匹配到指定的值,此时valueof的返回值应该为空,所以后面又紧跟着if....else判断输出.

但是运行结果抛出异常.报告是无效参数异常...也就说summer(小写s)午饭转换为Season枚举,无法转换那也不应该抛出IllegalArgumentException异常啊,一旦抛出这个异常,后续的代码就不能执行了,这才是要命的,

这与我们的习惯用法不一致,例如我们从List中查找一个元素,即使不存在也不会报错,顶多indexOf方法返回-1.

看源码:
```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name) {
        T result = enumType.enumConstantDirectory().get(name);//通过反射,从常量列表中查找.
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(//最后报无效参数异常
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }
```
valueOf方法先通过反射从枚举类的常量声明中查找,若找到就直接返回,若找不到就抛出无效参数异常.

valueOf方法本意是保护编码中的枚举安全性,使其不产生空枚举对象,简化枚举操作,但是又引入了一个我们无法避免的IllegalArgumentException异常.

可能有读者会所此处valueOf()方法的源代码不对,以上源代码是要输入两个参数,而我们的Season.valueOf()值传递一个String类型的参数.

真的是这样吗?是的,因为valueOf(String name)方法是不可见的,是JVM内置的方法,我们只有通过阅读公开的valueOf方法来了解其运行原理.

在Season枚举类中引用valueOf方法有三个: valueOf(String arg0): Season-Season, values():Season[], valueOf(Class<T> enumType, String name):T-Enum

但是在Enum的源码中只有一个valueOf()的方法: 其他两个方法都是JVM的内置方法...

问题清楚了,我们有两种方式可以解决处理此问题:
(1)使用try....catch捕获异常
```java
try {
    Season s = Season.valueOf(name);
    // 有该枚举项时的处理
    System.out.println(s);
} catch (Exception e) {
    System.out.println("无相关枚举项");
}
```
(2)扩展枚举类:
由于Enum类定义的方法基本上都是final类型的,所以不希望被覆写,那我们可以学习List和String,通过增加一个contains方法来判断是否包含指定的枚举项,然后再继续转换,看代码:
```java
enum Season {
    Spring, Summer, Autumn, Winter;
    public boolean contains(String _name){
        Season[] season = values();
        for(Season s:season){
            if(s.name().equals(_name)){
                return true;
            }
        }
        return false;

    }
}
```
Season枚举具备了静态方法contains()之后,就可以在valueOf前判断一下是否包含指定的枚举名称了,若包含则可以通过valueOf转换为Season枚举，若不包含则不转换.

总结代码:
```java
import java.util.Arrays;
import java.util.List;

public class Client {
    public static void main(String[] args) {
        // 注意summer是小写
        List<String> params = Arrays.asList("Spring", "summer");
        for (String name : params) {
            // 查找表面值与name相同的枚举项
//            Season s = Season.valueOf(name);
//            if (s != null) {
//                // 有该枚举项时的处理
//                System.out.println(s);
//            } else {
//                // 没有该枚举项时的逻辑处理
//                System.out.println("无相关枚举项");
//            }
            if (Season.contains(name)) {
                Season s = Season.valueOf(name);
                // 有该枚举项时的处理
                System.out.println(s);
            } else {
                System.out.println("无相关枚举项");
            }
        }
    }
}

enum Season {
    Spring, Summer, Autumn, Winter;

    // 是否包含指定名称的枚举项
    public static boolean contains(String name) {
        Season[] season = values(); // 所有的枚举值

        // 遍历查找
        for (Season s : season) {
            if (s.name().equals(name)) {
                return true;
            }
        }
        return false;
    }
}
```

ref:
[http://www.cnblogs.com/DreamDrive/p/5632706.html](http://www.cnblogs.com/DreamDrive/p/5632706.html)

<!-- more -->

