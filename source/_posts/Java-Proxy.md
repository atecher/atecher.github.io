---
title: Java-Proxy
date: 2017-09-22 00:00:00
categories: Java
tags:
    - Java
---

Proxy,也就是“代理”了。意思就是,你不用去做,别人代替你去处理. 
它在程序开发中起到了非常重要的作用,比如传说中的 AOP(面向切面编程),就是针对代理的一种应用。此外,在设计模式中,还有一个“代理模式”。

<!-- more -->

## 概念

### 什么是代理:

代理分为静态代理和动态代理
静态代理是在编译时就将接口、实现类、代理类一股脑儿全部手动完成,但如果我们需要很多的代理,每一个都这么手动的去创建实属浪费时间,而且会有大量的重复代码,此时我们就可以采用动态代理
动态代理可以在程序运行期间根据需要动态的创建代理类及其实例,来完成具体的功能,主要用的是JAVA的反射机制。

其实方法直接调用就可以完成功能,为什么还要加个代理呢？
**原因是采用代理模式可以有效的将具体的实现与调用方进行解耦,通过面向接口进行编码完全将具体的实现隐藏在内部。**

Proxy代理模式是一种结构型设计模式,主要解决的问题是:在直接访问对象时带来的问题

代理是一种常用的设计模式,其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息,过滤消息并转发消息,以及进行消息被委托类执行后的后续处理。

更通俗的说,代理解决的问题当两个类需要通信时,引入第三方代理类,将两个类的关系解耦,让我们只了解代理类即可,而且代理的出现还可以让我们完成与另一个类之间的关系的统一管理,但是切记,代理类和委托类要实现相同的接口,因为代理真正调用的还是委托类的方法。

按照代理的创建时期,代理类可以分为两种:

- 静态:由程序员创建代理类或特定工具自动生成源代码再对其编译。在程序运行前代理类的.class文件就已经存在了。
- 动态:在程序运行时运用反射机制动态创建而成。

## 静态代理
实代理的一般模式就是静态代理的实现模式:首先创建一个接口(JDK代理都是面向接口的),然后创建具体实现类来实现这个接口,在创建一个代理类同样实现这个接口,不同指出在于,具体实现类的方法中需要将接口中定义的方法的业务逻辑功能实现,而代理类中的方法只要调用具体类中的对应方法即可,这样我们在需要使用接口中的某个方法的功能时直接调用代理类的方法即可,将具体的实现类隐藏在底层。

第一步:定义总接口Iuser.java
```java
public interface Iuser {
    void eat(String s);
}
```

第二步:创建具体实现类UserImpl.java (被代理人)
```java
public class UserImpl implements Iuser {
　　@Override
　　public void eat(String s) {
　　　　System.out.println("我要吃"+s);
　　}
}
```

第三步:创建代理类UserProxy.java (代理人)
```java
public class UserProxy implements Iuser {
　　private Iuser user = new UserImpl();
　　@Override
　　public void eat(String s) {
　　　　System.out.println("静态代理前置内容");
　　　　user.eat(s);
　　　　System.out.println("静态代理后置内容");
　　}
}
```

第四步:创建测试类ProxyTest.java
```java
public class ProxyTest {
　　public static void main(String[] args) {    
　　　　UserProxy proxy = new UserProxy();
　　　　proxy.eat("苹果");
　　}
}
```

运行结果:
```java
静态代理前置内容
我要吃苹果
静态代理后置内容
```
综上的代码和输出结果可以看出,静态代理的实现方法还是很简单的。都需要实现总接口,代理人里面持有被代理人的对象。代理人可以根据情况的不同,添加一些操作。

## 静态代理类优缺点

- 优点 代理使客户端不需要知道实现类是什么,怎么做的,而客户端只需知道代理即可(解耦合)。
- 缺点:

    1. 代理类和委托类实现了相同的接口,代理类通过委托类实现了相同的方法。这样就出现了大量的代码重复。如果接口增加一个方法,除了所有实现类需要实现这个方法外,所有代理类也需要实现此方法。增加了代码维护的复杂度。
    2. 代理对象只服务于一种类型的对象,如果要服务多类型的对象。势必要为每一种对象都进行代理,静态代理在程序规模稍大时就无法胜任了

举例说明:代理可以对实现类进行统一的管理,如在调用具体实现类之前,需要打印日志等信息,这样我们只需要添加一个代理类,在代理类中添加打印日志的功能,然后调用实现类,这样就避免了修改具体实现类。满足我们所说的开闭原则。但是如果想让每个实现类都添加打印日志的功能的话,就需要添加多个代理类,以及代理类中各个方法都需要添加打印日志功能(如上的代理方法中删除,修改,以及查询都需要添加上打印日志的功能)
即静态代理类只能为特定的接口(Service)服务。如想要为多个接口服务则需要建立很多个代理类。

## 动态代理
动态代理的思维模式与之前的一般模式是一样的,也是面向接口进行编码,创建代理类将具体类隐藏解耦,不同之处在于代理类的创建时机不同,动态代理需要在运行时因需实时创建。

第一步:定义总接口Iuser.java
```java
public interface Iuser {
　　void eat(String s);
}
```

第二步:创建具体实现类UserImpl.java (被代理人)
```java
public class UserImpl implements Iuser {
　　@Override
　　public void eat(String s) {
　　　　System.out.println("我要吃"+s);
　　}
}
```

第三步:创建实现InvocationHandler接口的代理类 (代理人)
```java
//动态代理类只能代理接口(不支持抽象类),代理类都需要实现InvocationHandler类,实现invoke方法。该invoke方法就是调用被代理接口的所有方法时需要调用的,该invoke方法返回的值是被代理接口的一个实现类  
public class DynamicProxy implements InvocationHandler {
　　private Object object;//用于接收具体实现类的实例对象
　　//使用带参数的构造器来传递具体实现类的对象
　　public DynamicProxy(Object obj){
　　　　this.object = obj;
　　}
　　@Override
　　public Object invoke(Object proxy, Method method, Object[] args)throws Throwable {
　　　　System.out.println("前置内容");
　　　　method.invoke(object, args);
　　　　System.out.println("后置内容");
　　　　return null;
　　}
}
```

第四步:创建测试类ProxyTest.java
```java
public class ProxyTest {
　　public static void main(String[] args) {
　　　　Iuser user = new UserImpl();
　　　　InvocationHandler h = new DynamicProxy(user);
　　　　Iuser proxy = (Iuser) Proxy.newProxyInstance(Iuser.class.getClassLoader(), new Class[]{Iuser.class}, h);
　　　　proxy.eat("苹果");
　　}
}
```

运行结果为:
```
动态代理前置内容
我要吃苹果
动态代理后置内容
```

第五步:重构

要注意的是,Proxy.newProxyInstance() 方法的参数实在是让人难以忍受

- 参数1:ClassLoader
- 参数2:该实现类的所有接口
- 参数3:动态代理对象

调用完了还要来一个强制类型转换一下。

一定要想办法封装一下,避免再次发生到处都是 Proxy.newProxyInstance()
```java
public class DynamicProxy implements InvocationHandler {

    ...

    @SuppressWarnings("unchecked")
    public <T> T getProxy() {
        return (T) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            this
        );
    }

    ...
}
```

我在 DynamicProxy 里添加了一个 getProxy() 方法,无需传入任何参数,将刚才所说的那些参数,放在这个方法中,并且该方法返回一个泛型类型,就不会强制类型转换了。方法头上加那个 @SuppressWarnings("unchecked") 注解表示忽略编译时的警告(因为 Proxy.newProxyInstance() 方法返回的是一个 Object,这里我强制转换为 T 了,这是向下转型,IDE 中就会有警告,编译时也会出现提示,很烦)。

好了,这下子使用 DynamicProxy 就简单了吧:
```java
public static void main(String[] args) {
    DynamicProxy dynamicProxy = new DynamicProxy(new UserImpl());
    Iuser proxy = dynamicProxy.getProxy();
    proxy.eat("苹果");
}
```

## 代理没有接口的类

用了这个 DynamicProxy 以后,我觉得它还是非常爽的,爽的地方是,接口变了,这个动态代理类不用动。而静态代理就不一样了,接口变了,实现类还要动,代理类也要动。但我也发现动态代理并不是“万灵丹”,它也有搞不定的时候,比如说,我要代理一个没有任何接口的类,它就没有勇武之地了

能否代理没有接口的类呢？
那就是 CGLib 这个类库。虽然它看起来不太起眼,但 Spring、Hibernate 这样牛逼的开源框架都用到了它。它就是一个在运行期间动态生成字节码的工具,也就是动态生成代理类了。说起来好高深,实际用起来一点都不难。我再写一个 CGLibProxy 吧:
```java
public class CGLibProxy implements MethodInterceptor {

    public <T> T getProxy(Class<T> cls) {
        return (T) Enhancer.create(cls, this);
    }

    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        before();
        Object result = proxy.invokeSuper(obj, args);
        after();
        return result;
    }

    ...
}
```

需要实现 CGLib 给我们提供的 MethodInterceptor 实现类,并填充 intercept() 方法。方法中最后一个 MethodProxy 类型的参数 proxy,值得注意！CGLib 给我们提供的是方法级别的代理,也可以理解为对方法的拦截(这不就是传说中的“方法拦截器”吗？)。我们直接调用 proxy 的 invokeSuper() 方法,将被代理的对象 obj 以及方法参数 args 传入其中即可。

与 DynamicProxy 类似,我在 CGlibProxy 中也添加了一个泛型的 getProxy() 方法,便于我们可以快速地获取自动生成的代理对象。还是用一个 main() 方法来描述吧:
```java
public static void main(String[] args) {
    CGLibProxy cgLibProxy = new CGLibProxy();
    UserImpl proxy = cgLibProxy.getProxy(UserImpl.class);
    proxy.eat("苹果");
}
```
与 JDK 动态代理不同的是,这里不需要任何的接口信息,对谁都可以生成动态代理对象

重构:
不想总是去 new 这个 CGLibProxy 对象,最好 new 一次,可以使用“单例模式”
```java
public class CGLibProxy implements MethodInterceptor {

    private static CGLibProxy instance = new CGLibProxy();
    
    //private 的构造方法,就是为了限制外界不能再去 new 它了
    private CGLibProxy() {
    

    public static CGLibProxy getInstance() {
        return instance;
    }

    ...
}

public static void main(String[] args) {
    UserImpl userImpl = CGLibProxy.getInstance().getProxy(UserImpl.class);
    userImpl.eat("苹果");
}
```

## 动态代理的实现过程

1. 首先我要说的就是接口，为什么JDK的动态代理是基于接口实现的呢？
　　因为通过使用接口指向实现类的实例的多态实现方式，可以有效的将具体的实现与调用之间解耦，便于后期修改与维护。
再具体的说就是我们在代理类中创建一个私有成员变量（private修饰），使用接口来指向实现类的对象（纯种的多态体现，向上转型的体现），然后在该代理类中的方法中使用这个创建的实例来调用实现类中的相应方法来完成业务逻辑功能。
这么说起来，我之前说的“将具体实现类完全隐藏”就不怎么正确了，可以改成，将具体实现类的细节向调用方完全隐藏（调用方调用的是代理类中的方法，而不是实现类中的方法）。
　　这就是面向接口编程，利用java的多态特性，实现程序代码的解耦。
2. 创建代理类的过程
　　如果你了解静态代理，那么你会发现动态代理的实现其实与静态代理类似，都需要创建代理类，但是不同之处也很明显，创建方式不同！
　　不同之处体现在静态代理我们知根知底，我们知道要对哪个接口、哪个实现类来创建代理类，所以我们在编译前就直接实现与实现类相同的接口，直接在实现的方法中调用实现类中的相应（同名）方法即可；而动态代理不同，我们不知道它什么时候创建，也不知道要创建针对哪个接口、实现类的代理类（因为它是在运行时因需实时创建的）。
　　虽然二者创建时机不同，创建方式也不相同，但是原理是相同的，不同之处仅仅是：静态代理可以直接编码创建，而动态代理是利用反射机制来抽象出代理类的创建过程。


## 分析

1. 静态代理需要实现与实现类相同的接口，而动态代理需要实现的是固定的Java提供的内置接口（一种专门提供来创建动态代理的接口）InvocationHandler接口，因为java在接口中提供了一个可以被自动调用的方法invoke，这个之后再说。
2. 先看代码
```java
private Object object;
public UserProxy(Object obj){
    this.object = obj;
}
```
这几行代码与静态代理之中在代理类中定义的接口指向具体实现类的实例的代码异曲同工，通过这个构造器可以创建代理类的实例，创建的同时还能将具体实现类的实例与之绑定（object指的就是实现类的实例，这个实例需要在测试类中创建并作为参数来创建代理类的实例），实现了静态代理类中private Iuser user = new UserImpl();一行代码的作用相近，这里为什么不是相同，而是相近呢，主要就是因为静态代理的那句代码中包含的实现类的实例的创建，而动态代理中实现类的创建需要在测试类中完成，所以此处是相近。
3. invoke(Object proxy, Method method, Object[] args)方法，该方法是InvocationHandler接口中定义的唯一方法，该方法在调用指定的具体方法时会自动调用。其参数为：代理实例、调用的方法、方法的参数列表
　　在这个方法中我们定义了几乎和静态代理相同的内容，仅仅是在方法的调用上不同，不同的原因与之前分析的一样（创建时机的不同，创建的方式的不同，即反射），Method类是反射机制中一个重要的类，用于封装方法，该类中有一个方法那就是invoke(Object object,Object…args)方法，其参数分别表示：所调用方法所属的类的对象和方法的参数列表，这里的参数列表正是从测试类中传递到代理类中的invoke方法三个参数中最后一个参数（调用方法的参数列表）中，在传递到method的invoke方法中的第二个参数中的（此处有点啰嗦）。
4. 测试类中的异同
　　静态代理中我们测试类中直接创建代理类的对象，使用代理类的对象来调用其方法即可，若是别的接口（这里指的是别的调用方）要调用Iuser的方法，也可以使用此法
动态代理中要复杂的多，首先我们要将之前提到的实现类的实例创建（补充完整），然后利用这个实例作为参数，调用代理来的带参构造器来创建“代理类实例对象”，这里加引号的原因是因为它并不是真正的代理类的实例对象，而是创建真正代理类实例的一个参数，这个实现了InvocationHandler接口的类严格意义上来说并不是代理类，我们可以将其看作是创建代理类的必备中间环节，这是一个调用处理器，也就是处理方法调用的一个类，不是真正意义上的代理类，可以这么说：创建一个方法调用处理器实例。
　　下面才是真正的代理类实例的创建，之前创建的”代理类实例对象“仅仅是一个参数
　　　　Iuser proxy = (Iuser) Proxy.newProxyInstance(Iuser.class.getClassLoader(), new Class[]{Iuser.class}, h);
　　这里使用了动态代理所依赖的第二个重要类Proxy，此处使用了其静态方法来创建一个代理实例，其参数分别是：类加载器（可为父类的类加载器）、接口数组、方法调用处理器实例
　　这里同样使用了多态，使用接口指向代理类的实例，最后会用该实例来进行具体方法的调用即可。

## 动态代理优点

动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强

ref:
[http://www.daidingkang.cc/2017/07/18/Java%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/](http://www.daidingkang.cc/2017/07/18/Java%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/)
[https://my.oschina.net/huangyong/blog/159788](https://my.oschina.net/huangyong/blog/159788)
