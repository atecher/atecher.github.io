---
title: Java_改善_用枚举实现工厂方法模式更简洁
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 改善
comments: false
---

<!-- more -->

## 工厂方法模式
(Factory Method Patter)是"创建对象的接口",让子类决定实例化哪一个类,并使一个类的实例化延迟到其子类.工厂方法模式在我们的开发工作中,经常会用到.

下面以汽车制造为例,看看一般的工厂方法模式是如何实现的,代码如下:
```java
public class Client {
    public static void main(String[] args) {
        //生产车辆
        Car car = CarFactory.createCar(FordCar.class);
    }
}

//抽象产品
interface Car {};
//具体产品类
class FordCar implements Car {};
//具体产品类
class BuickCar implements Car {};
//工厂类
class CarFactory {
    //生产汽车
    public static Car createCar(Class<? extends Car> c) {
        try {
            return (Car) c.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
这是最原始的工厂方法模式,有两个产品"福特骑车和别克骑车,然后通过工厂方法模式来生产,有了工厂方法模式,我们就不用关心一辆车具体是怎么生成的了,只要告诉工厂"给我生产一辆福特骑车"就可以了,下面是产出一辆福特骑车时客户端的代码:
```java
Car car = CarFactory.createCar(FordCar.class);
```
这就是我们经常使用的工厂方法模式,但经常使用不代表就是最优秀的,最简洁的.

此处在介绍一种通过枚举实现工厂方法模式的方案,谁优谁劣自行评价.枚举实现工厂方法模式有两种方法:

## 枚举非静态方法实现工厂方法模式

我们知道每个枚举项都是该枚举的实例对象,那是不是定义一个方法可以生成每个枚举项的对应产品来实现此模式呢?代码如下:
```java
public class Client {
    public static void main(String[] args) {
        Car car = CarFactory.BuickCar.create();
    }
}

interface Car {};
class FordCar implements Car {};
class BuickCar implements Car {};

enum CarFactory {
    //定义工厂类能生产汽车的类型
    FordCar, BuickCar;
    //生产汽车
    public Car create() {
        switch (this) {
        case FordCar:
            return new FordCar();
        case BuickCar:
            return new BuickCar();
        default:
            throw new AssertionError("无效参数");
        }
    }
}
```
create是一个非静态方法,也就是只有通过FordCar,BuickCar枚举项才能访问,采用这种方式实现工厂方法模式时,客户端要产生一辆汽车就很简单了.代码如下:
```java
Car car = CarFactory.BuickCar.create();
```

## 通过抽象方法生成产品

枚举类型虽然不能继承,但是可以用abstract修饰其方法,此时就标识该枚举是一个抽象枚举,需要每个枚举项自行实现该方法,也就说枚举项的类型是该枚举的一个子类,看代码:
```java
public class Client {
    public static void main(String[] args) {
        Car car = CarFactory.BuickCar.create();
    }
}

interface Car {};
class FordCar implements Car {};
class BuickCar implements Car {};

enum CarFactory {
    FordCar {
        public Car create() {
            return new FordCar();
        }
    },
    BuickCar {
        public Car create() {
            return new BuickCar();
        }
    };
    //抽象生产方法
    public abstract Car create();
}
```
首先定义一个抽象制造方法create,然后 每个枚举项自行实现,这种方式编译后会产生两个CarFactory的匿名子类,因为每个枚举项都 要实现抽象create方法,客户端的调用与上一个方案相同,不再赘述.

## 使用枚举类型的工厂方法模式的优点:

1. 避免错误调用的发生
一般工厂方法模式中的生产方法(也就是createCar方法)可以接收三种类型的参数:类型参数(Class),String参数(生产方法中判断String参数是需要生产什么产品),int参数(根据int值判断需要生产什么类型的产品).
这三种参数都是宽泛的数据类型,很容易产生错误.比如边界问题,null值问题,而且出现这类错误编译器还不会报警.例如:Car car = CarFactory.createCar(Car.class);
Car是一个接口,完全合乎createCar方法的要求,所以它在编译时不会报任何错误,但一运行起来就会报java.lang.InstantiationException异常,而使用枚举类型的工厂方法模式就不存在该问题.不需要传递任何参数,只需要选择好生产什么类型的产品就可以了.
2. 性能好,使用便捷.
枚举类型的计算是以int类型的计算为基础的,这是最基本的操作,性能当然快.
3. 降低类间的耦合
不管生产方法接收的是Class,String还是int参数,都会成为客户端类的负担.这些类并不是客户端需要的,而是因为工厂方法的限制必须输入的.

ref:
[http://www.cnblogs.com/DreamDrive/p/5633233.html](http://www.cnblogs.com/DreamDrive/p/5633233.html)

