---
title: Java_性能_推荐使用枚举定义常量
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 改善
comments: false
---

枚举和注解都是在Java1.5中引入的,虽然他们是后起之秀,但是功能不容小觑,枚举改变了常量的声明方式,注解耦合了数据和代码.

常量的声明是每一个项目中不可或缺的，在Java1.5之前，我们只有两种方式的声明：类常量和接口常量。不过，在1.5版之后有了改进，即新增了一种常量声明方式，枚举常量。代码如下： 
```java
enum Season{ 
    Spring,Summer,Autumn,Winter; 
}
```
JLS(Java Language Specification,Java语言规范)提倡枚举项全都大写,字母之间用下划线分隔.这也是从常量的角度考虑的.

<!-- more -->

## 枚举常量的优势

那么枚举常量与我们的经常使用的类常量和静态常量比有什么优势呢?
枚举的优点主要表现在以下四个方面.

### 枚举常量更简单 
先把Season枚举翻译成接口，代码如下：
```java
interface Season{ 
    int Sprint = 0; 
    int Summer = 1; 
    int Autumn = 2; 
    int Winter = 3; 
} 
```
首先对比以下两者的定义,枚举常量只需要定义每个枚举项，不需要定义枚举值，而接口常量（或类常量）则必须定义值，否则编译通不过,即使我们不需要关注其值是多少也必须定义；其次,虽然两个引用的方式相同（都是“类名.属性”，如Season.Sprint），但是枚举表示的是一个枚举项，字面含义是春天，而接口常量却是一个int类型,虽然其字面含义也是春天,但在运算中我们势必要关注其int值.

### 枚举常量属于稳态型 
例如:我们要给外星人描述一下地球上的春夏秋冬是什么样子的,使用接口常量应该是这样写.
```java
public void describe(int s){ 
    //s变量不能超越边界，校验条件 
    if(s >= 0 && s <4){ 
        switch(s){ 
            case Season.Summer: 
                System.out.println("Summer is very hot!"); 
                break; 
            case Season.Winter: 
                System.out.println("Winter is very cold!"); 
                break; 
        ..... 
        } 
    } 
}
```
我们需要用switch语句判断是哪一个常量,然后输出.但问题是我们得对输入值进行检查,确定是否越界，如果常量非常庞大，校验输入就是一件非常麻烦的事情，但这是一个不可逃避的过程,特别是如果我们的校验条件不严格,虽然可以编译照样通过,但是运行期就会产生无法预知的后果.

我们再来看看枚举常量是否能够避免校验问题，代码如下：
```java
public void describe(Season s){ 
    switch(s){ 
        case Season.Summer: 
            System.out.println("Summer is very hot!"); 
            break; 
        case Season.Winter: 
            System.out.println("Winter is very cold!"); 
            break; 
       ...... 
    } 
}
```
不用校验，已经限定了是Season枚举，所以只能是Season类的四个实例。这也是我们看重枚举的地方：在编译期间限定类型，不允许发生越界的情况。  

### 枚举具有内置方法 
有一个很简单的问题:如果要列出所有的季节常量,如何实现?接口常量或者类常量可以通过反射来实现,这没错,只是虽然能实现,但会非常繁琐.但是对于枚举就可以非常简单的实现.
```java
public static void main(String[] args){ 
    for(Season s:Season.values()){ 
            System.out.println(s); 
    } 
} 
```
通过values()方法获得所有的枚举项.这得益于枚举内置的方法,每个枚举都是java.lang.Enum的子类，该基类提供了诸如获得排序值的ordinal方法、compareTo比较方法等，大大简化了常量的访问。

### 枚举可以自定义方法 
这一点似乎不是枚举的优点，类常量也可以有自己的方法，但关键是枚举常量不仅仅可以定义静态方法，还可以定义非静态方法，而且还能够从根本上杜绝常量类被实例化。比如我们在定义获取最舒服的季节，使用枚举的代码如下： 
```java
enum Season{ 
    Spring,Summer,Autumn,Winter; 
    //最舒服的季节 
    public static Season getComfortableSeason(){ 
        return Spring; 
    } 
}
```
我们知道每个枚举项都是该枚举的一个实例,对于我们的例子来说,也就表示Spring其实是Season的一个实例,Summer也是其中的一个实例.那我们再枚举中定义的静态方法既可以在类(Season类)中引用,也可以在实例(也就是枚举项Spring,Summer,Autumn,Winter)中引用,看如下代码:
```java
enum Season{ 
    Spring,Summer,Autumn,Winter; 
    //最舒服的季节 
    public static Season getComfortableSeason(){ 
        return Spring; 
    } 
}

public class Client {
    public static void main(String[] args) {
        System.out.println("The most comfortable season is " + Season.getComfortableSeason());
        System.out.println("kxh test " + Season.getComfortableSeason());
    }
}
```
那如果使用类常量要如何实现呢?代码如下:
```java
class Season{ 
    public final static int Spring = 0; 
    public final static int Summer = 1; 
    public final static int Autumn = 2; 
    public final static int Winter = 3; 
 
    //最舒服的季节 
    public static int getComfortableSeason(){ 
        return Spring; 
    } 
}
```
想想看,我们要怎么才能打印出"The most comfortable season is Spring" 这句话呢? 除了使用switch判断外没有其他更好的办法了.

虽然枚举在很多方面都比接口常量和类常量好用，但是它有一点比不上接口常量和类常量的，就是继承，枚举类型是不能有继承的，也就是说一个枚举常量定义完毕后，除非修改重构，否则无法做扩展。 

## 建议 
在项目开发中，推荐使用枚举常量代替接口常量或类常量。

ref:
[http://www.cnblogs.com/DreamDrive/p/5419555.html](http://www.cnblogs.com/DreamDrive/p/5419555.html)
