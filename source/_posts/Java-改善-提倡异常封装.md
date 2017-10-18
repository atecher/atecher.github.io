---
title: Java_改善_提倡异常封装
date: 2017-08-01 11:58:48
categories: Java
tags:
    - Java
    - 改善
comments: false
---

 JavaAPI提供的异常都是比较低级别的,低级别是指只有开发人员才能看懂的异常.而对于终端用户来说基本上就是天书,与业务无关,是纯计算机语言的描述.

异常封装的三方面的好处:
1. 提高系统的友好性
2. 提高性能的可维护性
3. 解决java异常机制自身的缺陷

<!-- more -->

## 提高系统的友好性.

打开一个文件,如果文件不存在,则会报FileNotFoundException异常,如果该方法的编写不做任何的处理,直接抛到上层,则会降低系统的友好性.

代码如下:
```java
public static void doStuff() throws Exception {
    InputStream is = new FileInputStream("无效文件.txt");
    /*文件操作*/
}
```
解决方法就是封装异常,可以把异常的读者分为两类:开发人员和用户.

开发人员查找问题需要打印出堆栈信息,而用户则需要了解具体的业务原因,比如:文件太大,不能同时编写文件等...

代码如下:
```java
public static void doStuff2() throws MyBussinessException{
    try {
        InputStream is = new FileInputStream("无效文件.txt");
    } catch (FileNotFoundException e) {
        //为方便开发和维护人员而设置的异常信息
        e.printStackTrace();
        //抛出业务异常
            throw new MyBussinessException(e);
        }
    /*文件操作*/
}
```

## 提高新能的可维护性

来看如下代码:
```java
public void doStuff(){
    try{
        //do something
    }catch(Exception e){
        e.printStackTrace();
    }
}
```
这种是很多程序员容易犯下的错误,抛出异常是吧..分类处理异常多麻烦,就写一个catch块来处理所有的异常吧,而且还信誓旦旦的说JVM会打印出出堆栈信息中的错误信息.

虽然没有错,但是堆栈信息只有开发人员能看懂,维护人员看到这段异常的时候基本上无法处理.因为需要深入到代码逻辑中去分析问题.

正确的做法是对异常进行分类处理,并进行封装输出,代码如下:
```java
public void doStuff2(){
    try{
        //do something
    }catch(FileNotFoundException e){
        log.info("文件问找到，使用默认配置文件……");
    }catch(SecurityException e){
        log.error("无权访问，可能原因是……");
        e.printStackTrace();
    }
}
```

## 解决java异常机制自身的缺陷

Java中的异常一次只能抛出一个,比如doStuff方法有两个逻辑代码片段,如果在第一个逻辑片段中抛出异常,第二个逻辑片段中就不再执行了.也就无法抛出第二个异常了,现在的问题的是如何一次抛出多个异常....

其实,使用自行封装的异常就可以解决问题了,代码如下:
```java
class MyException extends Exception {
    // 容纳所有的异常
    private List<Throwable> causes = new ArrayList<Throwable>();

    // 构造函数，传递一个异常列表
    public MyException(List<? extends Throwable> _causes) {
        causes.addAll(_causes);
    }

    // 读取所有的异常
    public List<Throwable> getExceptions() {
        return causes;
    }
}
```
如上MyExcepiton异常只是一个异常容器,可以容纳多个异常,但它本身并不代表任何异常含义,它所解决的是一次抛出多个异常的问题,具体调用如下:
```java
    public static void doStuff() throws MyException {
        List<Throwable> list = new ArrayList<Throwable>();
        // 第一个逻辑片段
        try {
            // Do Something
        } catch (Exception e) {

            list.add(e);
        }
        // 第二个逻辑片段
        try {
            // Do Something
        } catch (Exception e) {
            list.add(e);
        }

        if (list.size() > 0) {
            throw new MyException(list);
        }

    }
```
这样doStuff方法的调用者就可以一次获得多个异常了...也能够为用户提供完整的异常情况说明.

可能你会问,有这种情况的出现吗?怎么会要求一个方法抛出多个异常呢?

绝对可能出现,例如Web界面注册的时候.依次把User对象传递到逻辑层,Register方法需要对各个Field进行校验并注册,如果用户 填写的字段不只有一个错误,最好把所有的错误都一次性的提示给用户,而不是要求用户每次条件都进行修改.

一次性的对User对象进行校验,然后返回所有的异常.


ref:
[http://www.cnblogs.com/DreamDrive/p/5446819.html](http://www.cnblogs.com/DreamDrive/p/5446819.html)

