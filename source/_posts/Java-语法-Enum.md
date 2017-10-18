---
title: Java_语法_Enum
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 语法
comments: false
---

JDK1.5引入了新的类型——枚举。在 Java 中它虽然算个“小”功能，却给我的开发带来了“大”方便。

<!-- more -->

## 用法一：常量

在JDK1.5 之前，我们定义常量都是： public static final…. 。现在好了，有了枚举，可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。
```java
public enum Color {  
  RED, GREEN, BLANK, YELLOW  
}
```

## 用法二：switch
JDK1.6之前的switch语句只支持int,char,enum类型，使用枚举，能让我们的代码可读性更强。
```java
enum Signal {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    Signal color = Signal.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = Signal.GREEN;  
            break;  
        case YELLOW:  
            color = Signal.RED;  
            break;  
        case GREEN:  
            color = Signal.YELLOW;  
            break;  
        }  
    }  
}  
```

## 用法三：向枚举中添加新方法
如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号。而且 Java 要求必须先定义 enum 实例。
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    // 普通方法  
    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    // get set 方法  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  
}  
```

## 用法四：覆盖枚举的方法
下面给出一个toString()方法覆盖的例子。
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //覆盖方法  
    @Override  
    public String toString() {  
        return this.index+"_"+this.name;  
    }  
}
```

## 用法五：实现接口
所有的枚举都继承自java.lang.Enum类。由于Java 不支持多继承，所以枚举对象不能再继承其他类。
```java
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}
```

## 用法六：使用接口组织枚举
```java
public interface Food {  
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}
```

## 用法七：使用构造函数协助描述枚举项

### 分析 
一般来说，我们经常使用的枚举项只有一个属性，即排序号，其默认值是从0、1、2... ...。但是除了排序号外，枚举还有一个（或多个）属性:枚举描述,它的含义是通过枚举的构造函数,声明每个枚举项(也就是枚举实例)必须具有的属性和行为,这是对枚举项的描述或补充,目的是使枚举项表述的意义更加清晰准确.

### 场景 
比如，可以通过枚举构造函数声明业务值，定义可选项，添加属性，看如下代码： 
```java
public class Client {
    public static void main(String[] args) {
        System.out.println(Season.Spring.getDesc());
        
    }
}

enum Season {
    Spring("春"), Summer("夏"), Autumn("秋"), Winter("冬");
    
    private String desc;
    Season(String _desc){
        desc = _desc;
    }
    
    //获得枚举值
    public String getDesc(){
        return desc;
    }
}
/*
运行输出: 春
*/
```
其枚举项是英文的,描述是英文的,这样使其描述更加准确.方便了多个协作者共同引用常量.若不考虑描述的使用(即访问getDesc方法),它与如下定义的描述很相似.
```java
interface Season{
    //春
    int Spring = 0;
    //夏
    int Summer =1
    .....
}
```
比较上面两段代码,很容易看出使用枚举项是一个很好的解决方案,非常简单,清晰.

可以通过枚举构造函数声明业务值,定义可选项,添加属性等.看如下代码:
```java
enum Role{ 
    Admin("管理员",new Lifetime(),new Scope()); 
    User("普通用户",new Lifetime(),new Scope()); 
 
    //中文描述 
    private String name; 
    //角色生命周期 
    private Lifetime lifeTime; 
    //权限范围 
    private Scope scope; 
 
    Role(String _name,Lifetime _lt,Scope _scope){ 
        name = _name; 
        lifeTime = _lifeTime; 
        scope = _scope; 
    } 
    /**name,lifeTime,scope的get方法较简单，不再赘述*/ 
}
```
这是一个角色定义类,描述了两个角色:管理员(Admin)和普通用户(User),同时它还通过构造函数对这两个角色进行了描述.:
1. name 表示的是该角色的中文名称
2. lifeTime 表示的是该角色的生命周期,也就是多长时间角色失效
3. scope 表示的是该角色的权限范围.

这样 一个描述可以使开发者对Admin和User两个常量有一个立体多维度的认知.有名称,生命期还有权限范围.而且还可以在程序中方便的获得这些属性.

建议在枚举定义中改为每个枚举项定义描述,特别是在大规模的项目开发中.大量的常量项定义使用枚举比在接口常量或者类常量中增加注释的方式友好简洁很多.

## 用法八：关于枚举集合的使用
java.util.EnumSet和java.util.EnumMap是两个枚举集合。EnumSet保证集合中的元素不重复；EnumMap中的key是enum类型，而value则可以是任意类型。关于这个两个集合的使用就不在这里赘述，可以参考JDK文档。

关于枚举的实现细节和原理请参考：

参考资料：《ThinkingInJava》第四版

**注意: 枚举类型对象之间的值比较，是可以使用==，直接来比较值，是否相等的，不是必须使用equals方法的**
见 Enum 源码
```java
public final boolean equals(Object other) {
    return this==other;
}
```

ref:
[http://blog.lichengwu.cn/java/2011/09/26/the-usage-of-enum-in-java/](http://blog.lichengwu.cn/java/2011/09/26/the-usage-of-enum-in-java/)
