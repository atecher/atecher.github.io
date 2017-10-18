---
title: Java_规避_警惕浅拷贝
date: 2017-07-31 11:58:48
categories: Java
tags:
    - Java
    - 规避
comments: false
---


<!-- more -->

## 数组的浅拷贝

有这样一个例子，第一个箱子里面与赤橙黄绿青蓝紫7色气球，现在希望第二个箱子也放入7个气球，其中最后一个气球改为蓝色，也就是赤橙黄绿青蓝蓝七个气球。
```java
import org.apache.commons.lang3.builder.ToStringBuilder;
public class Client{
    public static void main(String[] args){
        //气球的数量
        int ballonNum = 7;
        //第一个箱子
        Ballon[] box1 = new Ballon[ballonNum];
        //初始化第一个箱子
        for(int i = 0; i < ballonNum; i++){
            box1[i] = new Ballon(Color.values()[i],i);
        }

        //第二个箱子的小球是拷贝的第一个箱子里的
        Ballon[] box2 = Arrays.copyOf(box1,box1.length);
        //修改最后一个气球的颜色
        box2[6].setColor(Color.Blue);
        //打印出第一个箱子中的气球颜色
        for(Ballon b:box1){
            System.out.println(b);
        }
    }
}

//气球的颜色
enum Color{
    Red,Orange,Yellow,Green,Indigo,Blue,Violet;
}

//气球
class Ballon{
    private int id; //编号
    private Color color; //颜色

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public Color getColor() {
        return color;
    }
    public void setColor(Color color) {
        this.color = color;
    }
    public Ballon(Color _color,int _id){
        color = _color;
        id = _id;
    }

    /*id、color的getter/setter方法省略*/
    //apache-common包下的ToStringBuilder重写toString方法
    public String toString(){
        return new ToStringBuilder(this).append("编号",id).append("颜色",color).toString();
    }
}
```

第二个箱子的最后一个气球毫无疑问是被修改了蓝色，不过是通过拷贝第一个箱子的气球实现的，那么会对第一个箱子的气球颜色有影响吗？输出结果：
Balloon@b2fd8f[编号=0,颜色=Red]
Balloon@a20892[编号=1,颜色=Orange]
Balloon@158b649[编号=2,颜色=Yellow]
Balloon@1037c71[编号=3,颜色=Green]
Balloon@1546e25[编号=4,颜色=Indigo]
Balloon@8a0d5d[编号=5,颜色=Blue]
Balloon@a470b8[编号=6,颜色=Blue]
最后一个气球竟然被修改了。这是为何？

这是典型的浅拷贝（Shallow Clone）问题，通过copyOf()方法产生的数组是一个浅拷贝引用地址。需要说明的是数组的clone()方法也是与此相同，同样是浅拷贝，而且集合的clone()方法也是浅拷贝。这就需要大家多留心了。
问题找到了，解决办法也很简单，遍历box1的每个元素，重新生成一个气球（Ballon）对象，并放置到box2数组中。
很多地方使用集合（如List）进行业务处理时，比如发觉需要拷贝集合中的元素，可集合没有提供任何拷贝方法，所以干脆使用 List.toArray方法转换成数组，然后通过Arrays.copyOf拷贝，然后转换成集合，简单便捷！但是，非常遗憾，这里我们又撞到浅拷贝的 枪口上了！！！！


## 对象的浅拷贝

我们知道一个类实现了Cloneable接口就表示它具备了被拷贝的能力，如果再覆写clone()方法就会完全具备拷贝能力。拷贝是在内存中进行的，所以在性能方面比直接通过new生成对象要快很多，特别是在大对象的生成上，这会使性能的提升非常显著。但是对象拷贝也有一个比较容易忽略的问题：浅拷贝（Shadow Clone，也叫做影子拷贝）存在对象属性拷贝不彻底的问题。我们来看这样一段代码：
```java
public class Client {
    public static void main(String[] args) {
         //定义父亲
         Person f = new Person("父亲");
         //定义大儿子
         Person s1 = new Person("大儿子",f);
         //小儿子的信息是通过大儿子拷贝过来的
         Person s2 = s1.clone();
         s2.setName("小儿子");
         System.out.println(s1.getName() +" 的父亲是 " + s1.getFather().getName());
         System.out.println(s2.getName() +" 的父亲是 " + s2.getFather().getName());
    }
}

class Person implements Cloneable{
    //姓名
    private String name;
    //父亲
    private Person father;

    public Person(String _name){
         name = _name;
    }
    public Person(String _name,Person _parent){
         name = _name;
         father = _parent;
    }
    /*name和parent的getter/setter方法省略*/

    //拷贝的实现
    @Override
    public Person clone(){
         Person p = null;
         try {
           p = (Person) super.clone();
         } catch (CloneNotSupportedException e) {
           e.printStackTrace();
         }
         return p;
   }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Person getFather() {
        return father;
    }
    public void setFather(Person father) {
        this.father = father;
    }
}
```
程序中，我们描述了这样一个场景：一个父亲，有两个儿子，大小儿子同根同种，所以小儿子对象就通过拷贝大儿子对象来生成，运行输出的结果如下：
```
大儿子 的父亲是 父亲
小儿子 的父亲是 父亲
```
这很正确，没有问题。突然有一天，父亲心血来潮想让大儿子去认个干爹，也就是大儿子的父亲名称需要重新设置一下，代码如下：
```
public static void main(String[] args) {
     //定义父亲
     Person f = new Person("父亲");
     //定义大儿子
     Person s1 = new Person("大儿子",f);
     //小儿子的信息是通过大儿子拷贝过来的
     Person s2 = s1.clone();
     s2.setName("小儿子");
     //认干爹
     s1.getFather().setName("干爹");
     System.out.println(s1.getName() +" 的父亲是 " + s1.getFather().getName());
     System.out.println(s2.getName() +" 的父亲是 " + s2.getFather().getName());
}
```
上面仅仅修改了加粗字体部分，大儿子重新设置了父亲名称，我们期望的输出是：将大儿子父亲的名称修改为干爹，小儿子的父亲名称保持不变。下面来检查一下结果是否如此：
```
大儿子 的父亲是 干爹
小儿子 的父亲是 干爹
```

怎么回事，小儿子的父亲也成了“干爹”?两个儿子都没有，岂不是要气死“父亲”了！出现这个问题的原因就在于clone方法，我们知道所有类都继承自Object，Object提供了一个对象拷贝的默认方法，即上面代码中的super.clone方法，但是该方法是有缺陷的，它提供的是一种浅拷贝方式，也就是说它并不会把对象的所有属性全部拷贝一份，而是有选择性的拷贝，它的拷贝规则如下：

- 基本类型
如果变量是基本类型，则拷贝其值，比如int、float等。
- 对象
如果变量是一个实例对象，则拷贝地址引用，也就是说此时新拷贝出的对象与原有对象共享该实例变量，不受访问权限的限制。这在Java中是很疯狂的，因为它突破了访问权限的定义：一个private修饰的变量，竟然可以被两个不同的实例对象访问，这让Java的访问权限体系情何以堪！
- String字符串
这个比较特殊，拷贝的也是一个地址，是个引用，但是在修改时，它会从字符串池（String Pool）中重新生成新的字符串，原有的字符串对象保持不变，在此处我们可以认为String是一个基本类型。

明白了这三个规则，上面的例子就很清晰了，小儿子对象是通过拷贝大儿子产生的，其父亲都是同一个人，也就是同一个对象，大儿子修改了父亲名称，小儿子也就跟着修改了—于是，父亲的两个儿子都没了！其实要更正也很简单，clone方法的代码如下：
```java
public Person clone(){
     Person p = null;
     try {
        p = (Person) super.clone();
        p.setFather(new Person(p.getFather().getName()));
     } catch (CloneNotSupportedException e) {
        e.printStackTrace();
     }
     return p;
}
```
然后再运行，小儿子的父亲就不会是“干爹”了。如此就实现了对象的深拷贝（Deep Clone），保证拷贝出来的对象自成一体，不受“母体”的影响，和new生成的对象没有任何区别。
注意　浅拷贝只是Java提供的一种简单拷贝机制，不便于直接使用。


## 推荐使用序列化实现对象的拷贝

上一个建议说了对象的浅拷贝问题，实现Cloneable接口就具备了拷贝能力，那我们来思考这样一个问题：如果一个项目中有大量的对象是通过拷贝生成的，那我们该如何处理？每个类都写一个clone方法，并且还要深拷贝？想想看这是何等巨大的工作量呀，是否有更好的方法呢？

其实，可以通过序列化方式来处理，在内存中通过字节流的拷贝来实现，也就是把母对象写到一个字节流中，再从字节流中将其读出来，这样就可以重建一个新对象了，该新对象与母对象之间不存在引用共享的问题，也就相当于深拷贝了一个新对象，代码如下：
```java
public class CloneUtils {
     // 拷贝一个对象
     @SuppressWarnings("unchecked")
     public static <T extends Serializable> T clone(T obj) {
          // 拷贝产生的对象
          T clonedObj = null;
          try {
            // 读取对象字节数据
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(obj);
            oos.close();
            // 分配内存空间，写入原始对象，生成新对象
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);
            //返回新对象，并做类型转换
            clonedObj = (T)ois.readObject();
            ois.close();
         } catch (Exception e) {
            e.printStackTrace();
         }
         return clonedObj;
     }
}
```
此工具类要求被拷贝的对象必须实现Serializable接口，否则是没办法拷贝的（当然，使用反射那是另外一种技巧），上一个建议中的例子只要稍微修改一下即可实现深拷贝，代码如下：
```java
class Person implements Serializable{
     private static final long serialVersionUID = 1611293231L;
     /*删除掉clone方法，其他代码保持不变*/
}
```

然后我们就可以通过CloneUtils工具进行对象的深拷贝了。用此方法进行对象拷贝时需要注意两点：

- 对象的内部属性都是可序列化的
如果有内部属性不可序列化，则会抛出序列化异常，这会让调试者很纳闷：生成一个对象怎么会出现序列化异常呢？从这一点来考虑，也需要把CloneUtils工具的异常进行细化处理。
- 注意方法和属性的特殊修饰符
比如final、static变量的序列化问题会被引入到对象拷贝中来，这点需要特别注意，同时transient变量（瞬态变量，不进行序列化的变量）也会影响到拷贝的效果。

当然，采用序列化方式拷贝时还有一个更简单的办法，即使用Apache下的commons工具包中的SerializationUtils类，直接使用更加简洁方便。

ref:
[http://www.cnblogs.com/DreamDrive/p/5422216.html](http://www.cnblogs.com/DreamDrive/p/5422216.html)
[http://www.cnblogs.com/DreamDrive/p/5430479.html](http://www.cnblogs.com/DreamDrive/p/5430479.html)
[http://www.cnblogs.com/DreamDrive/p/5430981.html](http://www.cnblogs.com/DreamDrive/p/5430981.html)
