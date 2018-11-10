---
title: Java-IO
date: 2017-10-09 00:00:00
categories: Java
tags:
    - Java
    - IO
---

Java IO 是一套Java用来读写数据(输入和输出)的API。大部分程序都要处理一些输入,并由输入产生一些输出。Java为此提供了java.io包。

如果你浏览下java.io包,会对其中各样的类选择感到迷惑。这些类的作用都是什么？对于某个任务该选择哪个类？怎样创建你自己的类做插件？这个手册的目的就是给你介绍这些类是如何组织的,以及怎样使用他们,因此你就不会疑惑需要时怎样选取合适的类,或者是否有一个满足你需求的类已经存在了。

<!-- more -->

## Java.io 包的范围
java.io 包并没有涵盖所有输入输出类型。例如,并不包含GUI或者网页上的输入输出,这些输入和输出在其它地方都涉及,比如Swing工程中的JFC (Java Foundation Classes) 类,或者J2EE里的Servlet和HTTP包。
Java.io 包主要涉及文件,网络数据流,内存缓冲等的输入输出。

## 更多的Java IO工具,提示等
这个手册也被称为” [Java How To’s and Utilities](http://tutorials.jenkov.com/java-howto/index.html) ”,包含一些Java IO的工具,例如替换流数据中的字符串,使用缓冲来反复处理流数据。

## 此Java IO 手册的范围
这个手册开始部分会给你一个Java IO API 工作的概览,以及你该怎样使用这些他们,接着会介绍包括所有Java IO API 的核心类。
这个手册不只是一个API的列表,这样的列表你可以从Sun公司的官方Java文档获得。事实上,每篇文档都是对一个类的简要介绍,设计它的目的以及一些实用的例子。换句话说,这些内容你在Sun公司的官方文档上是找不到的。

## Java IO 概述
在这一小节,我会试着给出Java IO(java.io)包下所有类的概述。更具体地说,我会根据类的用途对类进行分组。这个分组将会使你在未来的工作中,进行类的用途判定时,或者是为某个特定用途选择类时变得更加容易。

### 输入和输出 – 数据源和目标媒介
术语“输入”和“输出”有时候会有一点让人疑惑。一个应用程序的输入往往是另外一个应用程序的输出。那么OutputStream流到底是一个输出到目的地的流呢,还是一个产生输出的流？InputStream流到底会不会输出它的数据给读取数据的程序呢？就我个人而言,在第一天学习Java IO的时候我就感觉到了一丝疑惑。(校对注:输入流可以理解为向内存输入,输出流可以理解为从内存输出)

为了消除这个疑惑,我试着给输入和输出起一些不一样的别名,让它们从概念上与数据的来源和数据的流向相联系。

Java的IO包主要关注的是从原始数据源的读取以及输出原始数据到目标媒介。以下是最典型的数据源和目标媒介:

- 文件
- 管道
- 网络连接
- 内存缓存
- System.in, System.out, System.error(注:Java标准输入、输出、错误输出)

下面这张图描绘了一个程序从数据源读取数据,然后将数据输出到其他媒介的原理:

[![][image 1]][image 1]

### 流
在Java IO中,流是一个核心的概念。流从概念上来说是一个连续的数据流。你既可以从流中读取数据,也可以往流中写数据。流与数据源或者数据流向的媒介相关联。在Java IO中流既可以是字节流(以字节为单位进行读写),也可以是字符流(以字符为单位进行读写)。

### 类InputStream, OutputStream, Reader 和Writer
一个程序需要InputStream或者Reader从数据源读取数据,需要OutputStream或者Writer将数据写入到目标媒介中。以下的图说明了这一点:

[![][image 2]][image 2]

InputStream和Reader与数据源相关联,OutputStream和writer与目标媒介相关联。

Java IO中包含了许多InputStream、OutputStream、Reader、Writer的子类。这样设计的原因是让每一个类都负责不同的功能。这也就是为什么IO包中有这么多不同的类的缘故。各类用途汇总如下:

- 文件访问
- 网络访问
- 内存缓存访问
- 线程内部通信(管道)
- 缓冲
- 过滤
- 解析
- 读写文本 (Readers / Writers)
- 读写基本类型数据 (long, int etc.)
- 读写对象

当通读过Java IO类的源代码之后,我们很容易就能了解这些用途。这些用途或多或少让我们更加容易地理解,不同的类用于针对不同业务场景。

### Java IO类概述表
已经讨论了数据源、目标媒介、输入、输出和各类不同用途的Java IO类,接下来是一张通过输入、输出、基于字节或者字符、以及其他比如缓冲、解析之类的特定用途划分的大部分Java IO类的表格。

[![][image 3]][image 3]

## Java IO: 文件
在Java应用程序中,文件是一种常用的数据源或者存储数据的媒介。所以这一小节将会对Java中文件的使用做一个简短的概述。这篇文章不会对每一个技术细节都做出解释,而是会针对文件存取的方法提供给你一些必要的知识点。在之后的文章中,将会更加详细地描述这些方法或者类,包括方法示例等等。

### 通过Java IO读文件
如果你需要在不同端之间读取文件,你可以根据该文件是二进制文件还是文本文件来选择使用FileInputStream或者FileReader。这两个类允许你从文件开始到文件末尾一次读取一个字节或者字符,或者将读取到的字节写入到字节数组或者字符数组。你不必一次性读取整个文件,相反你可以按顺序地读取文件中的字节和字符。

如果你需要跳跃式地读取文件其中的某些部分,可以使用RandomAccessFile。

### 通过Java IO写文件
如果你需要在不同端之间进行文件的写入,你可以根据你要写入的数据是二进制型数据还是字符型数据选用FileOutputStream或者FileWriter。你可以一次写入一个字节或者字符到文件中,也可以直接写入一个字节数组或者字符数据。数据按照写入的顺序存储在文件当中。

### 通过Java IO随机存取文件
正如我所提到的,你可以通过RandomAccessFile对文件进行随机存取。

随机存取并不意味着你可以在真正随机的位置进行读写操作,它只是意味着你可以跳过文件中某些部分进行操作,并且支持同时读写,不要求特定的存取顺序。这使得RandomAccessFile可以覆盖一个文件的某些部分、或者追加内容到它的末尾、或者删除它的某些内容,当然它也可以从文件的任何位置开始读取文件

### 文件和目录信息的获取
有时候你可能需要读取文件的信息而不是文件的内容,举个例子,如果你需要知道文件的大小和文件的属性。对于目录来说也是一样的,比如你需要获取某个目录下的文件列表。通过File类可以获取文件和目录的信息。

## Java IO: 管道
Java IO中的管道为运行在同一个JVM中的两个线程提供了通信的能力。所以管道也可以作为数据源以及目标媒介。

你不能利用管道与不同的JVM中的线程通信(不同的进程)。在概念上,Java的管道不同于Unix/Linux系统中的管道。在Unix/Linux中,运行在不同地址空间的两个进程可以通过管道通信。在Java中,通信的双方应该是运行在同一进程中的不同线程。

### 通过Java IO创建管道
可以通过Java IO中的PipedOutputStream和PipedInputStream创建管道。一个PipedInputStream流应该和一个PipedOutputStream流相关联。一个线程通过PipedOutputStream写入的数据可以被另一个线程通过相关联的PipedInputStream读取出来。

### Java IO管道示例
这是一个如何将PipedInputStream和PipedOutputStream关联起来的简单例子:

[![][image 4]][image 4]

注:本例忽略了流的关闭。请在处理流的过程中,务必保证关闭流,或者使用jdk7引入的try-resources代替显示地调用close方法的方式。

你也可以使用两个管道共有的connect()方法使之相关联。PipedInputStream和PipedOutputStream都拥有一个可以互相关联的connect()方法。

### 管道和线程
请记得,当使用两个相关联的管道流时,务必将它们分配给不同的线程。read()方法和write()方法调用时会导致流阻塞,这意味着如果你尝试在一个线程中同时进行读和写,可能会导致线程死锁。

### 管道的替代
除了管道之外,一个JVM中不同线程之间还有许多通信的方式。实际上,线程在大多数情况下会传递完整的对象信息而非原始的字节数据。但是,如果你需要在线程之间传递字节数据,Java IO的管道是一个不错的选择。

### Java IO: 网络
Java中网络的内容或多或少的超出了Java IO的范畴。关于Java网络更多的是在我的[Java网络教程](http://ifeve.com/java-network/)中探讨。但是既然网络是一个常见的数据来源以及数据流目的地,并且因为你使用Java IO的API通过网络连接进行通信,所以本文将简要的涉及网络应用。

当两个进程之间建立了网络连接之后,他们通信的方式如同操作文件一样:利用InputStream读取数据,利用OutputStream写入数据。换句话来说,Java网络API用来在不同进程之间建立网络连接,而Java IO则用来在建立了连接之后的进程之间交换数据。

基本上意味着如果你有一份能够对文件进行写入某些数据的代码,那么这些数据也可以很容易地写入到网络连接中去。你所需要做的仅仅只是在代码中利用OutputStream替代FileOutputStream进行数据的写入。因为FileOutputStream是OutputStream的子类,所以这么做并没有什么问题.
实际上对于文件的读操作也类似,一个具有读取文件数据功能的组件,同样可以轻松读取网络连接中的数据。只需要保证读取数据的组件是基于InputStream而非FileInputStream即可。
这是一份简单的代码示例:
```java
public class MyClass {

    public static void main(String[] args) {
        InputStream inputStream = new FileInputStream("c:\\myfile.txt");
        process(inputStream);
    }

    public static void process(InputStream input) throws IOException {
        //do something with the InputStream
    }

}
```
在这个例子中,process()方法并不关心InputStream参数的输入流,是来自于文件还是网络(例子只展示了输入流来自文件的版本)。process()方法只会对InputStream进行操作。

## Java IO: 字节和字符数组

内容列表

- 从InputStream或者Reader中读入数组
- 从OutputStream或者Writer中写数组

在java中常用字节和字符数组在应用中临时存储数据。而这些数组又是通常的数据读取来源或者写入目的地。如果你需要在程序运行时需要大量读取文件里的内容,那么你也可以把一个文件加载到数组中。当然你可以通过直接指定索引来读取这些数组。但如果设计成为从InputStream或者Reader,而不是从数组中读取某些数据的话,你会用什么组件呢？

### 从 InputStream 或 Reader中读取数组
用ByteArrayInputStream或者CharArrayReader封装字节或者字符数组从数组中读取数据。通过这种方式字节和字符就可以以数组的形式读出了。
样例如下:
```java
byte[] bytes = new byte[1024];

//把数据写入字节数组...
InputStream input = new ByteArrayInputStream(bytes);

//读取第一个字节
int data = input.read();

while(data != -1) {
    //操作数据

    //读下一个字节
    data = input.read();
}
```
以同样的方式也可以用于读取字符数组,只要把字符数组封装在CharArrayReader上就行了。

### 通过 OutputStream 或者 Writer写数组

同样,也可以把数据写到ByteArrayOutputStream或者CharArrayWriter中。你只需要创建ByteArrayOutputStream或者CharArrayWriter,把数据写入,就像写其它的流一样。当所有的数据都写进去了以后,只要调用toByteArray()或者toCharArray,所有写入的数据就会以数组的形式返回。

样例如下:
```java
OutputStream output = new ByteArrayOutputStream();

output.write("This text is converted to bytes".toBytes("UTF-8"));

byte[] bytes = output.toByteArray();
```
写字符数组也和此例子类似。只要把字符数组封装在CharArrayWriter上就可以了。

## Java IO: System.in, System.out, System.err

System.in, System.out, System.err这3个流同样是常见的数据来源和数据流目的地。使用最多的可能是在控制台程序里利用System.out将输出打印到控制台上。

JVM启动的时候通过Java运行时初始化这3个流,所以你不需要初始化它们(尽管你可以在运行时替换掉它们)。

### System.in
System.in是一个典型的连接控制台程序和键盘输入的InputStream流。通常当数据通过命令行参数或者配置文件传递给命令行Java程序的时候,System.in并不是很常用。图形界面程序通过界面传递参数给程序,这是一块单独的Java IO输入机制。

### System.out
System.out是一个PrintStream流。System.out一般会把你写到其中的数据输出到控制台上。System.out通常仅用在类似命令行工具的控制台程序上。System.out也经常用于打印程序的调试信息(尽管它可能并不是获取程序调试信息的最佳方式)。

### System.err
System.err是一个PrintStream流。System.err与System.out的运行方式类似,但它更多的是用于打印错误文本。一些类似Eclipse的程序,为了让错误信息更加显眼,会将错误信息以红色文本的形式通过System.err输出到控制台上。

### System.out和System.err的简单例子:

这是一个System.out和System.err结合使用的简单示例:
```java
try {
    InputStream input = new FileInputStream("c:\\data\\...");
    System.out.println("File opened...");
} catch (IOException e) {
    System.err.println("File opening failed:");
    e.printStackTrace();
}
```

### 替换系统流
尽管System.in, System.out, System.err这3个流是java.lang.System类中的静态成员(译者注:这3个变量均为final static常量),并且已经预先在JVM启动的时候初始化完成,你依然可以更改它们。只需要把一个新的InputStream设置给System.in或者一个新的OutputStream设置给System.out或者System.err,之后的数据都将会在新的流中进行读取、写入。

可以使用System.setIn(), System.setOut(), System.setErr()方法设置新的系统流(译者注:这三个方法均为静态方法,内部调用了本地native方法重新设置系统流)。例子如下:
```java
OutputStream output = new FileOutputStream("c:\\data\\system.out.txt");
PrintStream printOut = new PrintStream(output);
System.setOut(printOut);
```
现在所有的System.out都将重定向到”c:\\data\\system.out.txt”文件中。请记住,务必在JVM关闭之前冲刷System.out(译者注:调用flush()),确保System.out把数据输出到了文件中。

## Java IO: 流

Java IO流是既可以从中读取,也可以写入到其中的数据流。正如这个系列教程之前提到过的,流通常会与数据源、数据流向目的地相关联,比如文件、网络等等。

流和数组不一样,不能通过索引读写数据。在流中,你也不能像数组那样前后移动读取数据,除非使用[RandomAccessFile](http://tutorials.jenkov.com/java-io/randomaccessfile.html) 处理文件。流仅仅只是一个连续的数据流。

某些类似[PushbackInputStream](http://tutorials.jenkov.com/java-io/pushbackinputstream.html) 流的实现允许你将数据重新推回到流中,以便重新读取。然而你只能把有限的数据推回流中,并且你不能像操作数组那样随意读取数据。流中的数据只能够顺序访问。

Java IO流通常是基于字节或者基于字符的。字节流通常以“stream”命名,比如InputStream和OutputStream。除了[DataInputStream](http://tutorials.jenkov.com/java-io/datainputstream.html) 和[DataOutputStream](http://tutorials.jenkov.com/java-io/dataoutputstream.html) 还能够读写int, long, float和double类型的值以外,其他流在一个操作时间内只能读取或者写入一个原始字节。

字符流通常以“Reader”或者“Writer”命名。字符流能够读写字符(比如Latin1或者Unicode字符)。可以浏览[Java Readers and Writers](http://tutorials.jenkov.com/java-io/readers-writers.html)获取更多关于字符流输入输出的信息。

### InputStream
java.io.InputStream类是所有Java IO输入流的基类。如果你正在开发一个从流中读取数据的组件,请尝试用InputStream替代任何它的子类(比如FileInputStream)进行开发。这么做能够让你的代码兼容任何类型而非某种确定类型的输入流。

然而仅仅依靠InputStream并不总是可行。如果你需要将读过的数据推回到流中,你必须使用PushbackInputStream,这意味着你的流变量只能是这个类型,否则在代码中就不能调用PushbackInputStream的unread()方法。

通常使用输入流中的read()方法读取数据。read()方法返回一个整数,代表了读取到的字节的内容(译者注:0 ~ 255)。当达到流末尾没有更多数据可以读取的时候,read()方法返回-1。

这是一个简单的示例:
```java
InputStream input = new FileInputStream("c:\\data\\input-file.txt");
int data = input.read(); 
while(data != -1){
        data = input.read();
}
```

### OutputStream
java.io.OutputStream是Java IO中所有输出流的基类。如果你正在开发一个能够将数据写入流中的组件,请尝试使用OutputStream替代它的所有子类。

这是一个简单的示例:
```java
OutputStream output = new FileOutputStream("c:\\data\\output-file.txt");
output.write("Hello World".getBytes());
output.close();
```

### 组合流
你可以将流整合起来以便实现更高级的输入和输出操作。比如,一次读取一个字节是很慢的,所以可以从磁盘中一次读取一大块数据,然后从读到的数据块中获取字节。为了实现缓冲,可以把InputStream包装到BufferedInputStream中。代码示例:

```java
InputStream input = new BufferedInputStream(new FileInputStream("c:\\data\\input-file.txt"));
```
缓冲同样可以应用到OutputStream中。你可以实现将大块数据批量地写入到磁盘(或者相应的流)中,这个功能由BufferedOutputStream实现。

缓冲只是通过流整合实现的其中一个效果。你可以把InputStream包装到PushbackInputStream中,之后可以将读取过的数据推回到流中重新读取,在解析过程中有时候这样做很方便。或者,你可以将两个InputStream整合成一个[SequenceInputStream](http://tutorials.jenkov.com/java-io/sequenceinputstream.html)。

将不同的流整合到一个链中,可以实现更多种高级操作。通过编写包装了标准流的类,可以实现你想要的效果和过滤器。

## Java IO: Input Parsing
Some of the classes in the Java IO API are designed to help you parse input. These classes are:

- [PusbackInputStream](http://tutorials.jenkov.com/java-io/pushbackinputstream.html)
- [PusbackReader](http://tutorials.jenkov.com/java-io/pushbackreader.html)
- [StreamTokenizer](http://tutorials.jenkov.com/java-io/streamtokenizer.html)
- [PushbackReader](http://tutorials.jenkov.com/java-io/pushbackreader.html)
- [LineNumberReader](http://tutorials.jenkov.com/java-io/linenumberreader.html)

It is not the purpose of this text to give you a complete course in parsing of data. The purpose was rather to give you above quick list of classes related to parsing of input data.

If you have to parse data you will often end up writing your own classes that use some of the classes in this list. I know I did when I wrote the parser for the Butterfly Container Script. I used the PushbackInputStream at the core of my parser, because sometimes I needed to read ahead a character or two, to determine what the character at hand meant.

I have a real life example that uses the PushbackReader in my article about [Replace Strings in Streams, Arrays, Files](http://tutorials.jenkov.com/java-howto/replace-strings-in-streams-arrays-files.html) tutorial. The example creates a TokenReplacingReader which can replace tokens of the format ${tokenName} in data read from an underlying Reader with values of your own choosing. The user of the TokenReplacingReader cannot see that this replacement takes place.


## Java IO: Reader And Writer

Java IO的Reader和Writer除了基于字符之外,其他方面都与InputStream和OutputStream非常类似。他们被用于读写文本。InputStream和OutputStream是基于字节的

### Reader
Reader类是Java IO中所有Reader的基类。子类包括BufferedReader,PushbackReader,InputStreamReader,StringReader和其他Reader。

这是一个简单的Java IO Reader的例子:
```java
Reader reader = new FileReader("c:\\data\\myfile.txt");
int data = reader.read();
while(data != -1){
    char dataChar = (char) data;
    data = reader.read();
}
```

请注意,InputStream的read()方法返回一个字节,意味着这个返回值的范围在0到255之间(当达到流末尾时,返回-1),Reader的read()方法返回一个字符,意味着这个返回值的范围在0到65535之间(当达到流末尾时,同样返回-1)。这并不意味着Reade只会从数据源中一次读取2个字节,Reader会根据文本的编码,一次读取一个或者多个字节。

你通常会使用Reader的子类,而不会直接使用Reader。Reader的子类包括InputStreamReader,CharArrayReader,FileReader等等。可以查看Java IO概述浏览完整的Reader表格。

### 整合Reader与InputStream
一个Reader可以和一个InputStream相结合。如果你有一个InputStream输入流,并且想从其中读取字符,可以把这个InputStream包装到InputStreamReader中。把InputStream传递到InputStreamReader的构造函数中:
```java
Reader reader = new InputStreamReader(inputStream);
```
在构造函数中可以指定解码方式。更多内容请参阅InputStreamReader。

### Writer
Writer类是Java IO中所有Writer的基类。子类包括BufferedWriter和PrintWriter等等。这是一个Java IO Writer的例子:
```java
Writer writer = new FileWriter("c:\\data\\file-output.txt"); 
writer.write("Hello World Writer"); 
writer.close();
```
同样,你最好使用Writer的子类,不需要直接使用Writer,因为子类的实现更加明确,更能表现你的意图。常用子类包括OutputStreamWriter,CharArrayWriter,FileWriter等。Writer的write(int c)方法,会将传入参数的低16位写入到Writer中,忽略高16位的数据。

### 整合Writer和OutputStream
与Reader和InputStream类似,一个Writer可以和一个OutputStream相结合。把OutputStream包装到OutputStreamWriter中,所有写入到OutputStreamWriter的字符都将会传递给OutputStream。这是一个OutputStreamWriter的例子:
```java
Writer writer = new OutputStreamWriter(outputStream);
```

### 整合Reader和Writer
和字节流一样,Reader和Writer可以相互结合实现更多更有趣的IO,工作原理和把Reader与InputStream或者Writer与OutputStream相结合类似。举个栗子,可以通过将Reader包装到BufferedReader、Writer包装到BufferedWriter中实现缓冲。以下是例子:
Reader reader = new BufferedReader(new FileReader(...));
Writer writer = new BufferedWriter(new FileWriter(...));

## Java IO: 并发IO

有时候你可能需要并发地处理输入和输出。换句话说,你可能有超过一个线程处理输入和产生输出。比如,你有一个程序需要处理磁盘上的大量文件,这个任务可以通过并发操作提高性能。又比如,你有一个web服务器或者聊天服务器,接收许多连接和请求,这些任务都可以通过并发获得性能的提升。


如果你需要并发处理IO,这里有几个问题可能需要注意一下:

在同一时刻不能有多个线程同时从InputStream或者Reader中读取数据,也不能同时往OutputStream或者Writer里写数据。你没有办法保证每个线程读取多少数据,以及多个线程写数据时的顺序。

如果线程之间能够保证操作的顺序,它们可以使用同一个stream、reader、writer。比如,你有一个线程判断当前的输入流来自哪种类型的请求,然后将流数据传递给其他合适的线程做后续处理。当有序存取流、reader、writer时,这种做法是可行的。请注意,在线程之间传递流数据的代码应当是同步的。

注意:在Java NIO中,你可以让一个线程读写多个“channel”。比如,你有很多网络连接处于开启状态,但是每个连接中都只有少量数据,类似于聊天服务器,可以让一个线程监视多个频道(连接)。Java NIO是另一个话题了,会后续教程中介绍。

## Java IO: 异常处理
流与Reader和Writer在结束使用的时候,需要正确地关闭它们。通过调用close()方法可以达到这一点。不过这需要一些思考。请看下边的代码:
```java
InputStream input = new FileInputStream("c:\\data\\input-text.txt");
int data = input.read();
while(data != -1) {
    //do something with data...  
    doSomethingWithData(data);
    data = input.read();
}
input.close();
```
第一眼看这段代码时,可能觉得没什么问题。可是如果在调用doSomethingWithData()方法时出现了异常,会发生什么呢？没错,这个InputStream对象就不会被关闭。

为了避免异常造成流无法被关闭,我们可以把代码重写成这样:
```java
InputStream input = null;
try{
    input = new FileInputStream("c:\\data\\input-text.txt");
    int data = input.read();
    while(data != -1) {
        //do something with data...
        doSomethingWithData(data);
        data = input.read();
}catch(IOException e){
    //do something with e... log, perhaps rethrow etc.
} finally {
    if(input != null)
        input.close();
}
```
注意到这里把InputStream的关闭代码放到了finally块中,无论在try-catch块中发生了什么,finally内的代码始终会被执行,所以这个InputStream总是会被关闭。

但是如果close()方法抛出了异常,告诉你流已经被关闭过了呢？为了解决这个难题,你也需要把close()方法写在try-catch内部,就像这样:
```java
finally {
    try{
        if(input != null)
            input.close();
    } catch(IOException e){
        //do something, or ignore.
    }
}
```

这段解决了InputStream(或者OutputStream)流关闭的问题的代码,确实是有一些不优雅,尽管能够正确处理异常。如果你的代码中重复地遍布了这段丑陋的异常处理代码,这不是很好的一个解决方案。如果一个匆忙的家伙贪图方便忽略了异常处理呢？

此外,想象一下某个异常最先从doSomethingWithData方法内抛出。第一个catch会捕获到异常,然后在finally里程序会尝试关闭InputStream。但是如果还有异常从close()方法内抛出呢？这两个异常中得哪个异常应当往调用栈上传播呢？

幸运的是,有一个办法能够解决这个问题。这个解决方案称作“异常处理模板”。创建一个正确关闭流的模板,能够在代码中做到一次编写,重复使用,既优雅又简单。详情参见[Java异常处理模板](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html)。

### Java7中IO的异常处理
从Java7开始,一种新的被称作“try-with-resource”的异常处理机制被引入进来。这种机制旨在解决针对InputStream和OutputStream这类在使用完毕之后需要关闭的资源的异常处理。可以浏览[Try with Resource in Java 7](http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html)获得更多信息。

## Java IO: InputStream

InputStream类是Java IO API中所有输入流的基类。InputStream子类包括FileInputStream,BufferedInputStream,PushbackInputStream等等。参考Java IO概述这一小节底部的表格,可以浏览完整的InputStream子类的列表。

### Java InputStream例子
InputStream用于读取基于字节的数据,一次读取一个字节,这是一个InputStream的例子:
```java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");
int data = inputstream.read();
while(data != -1) { 
    //do something with data...  
    doSomethingWithData(data);   
    data = inputstream.read();
}
inputstream.close();
```
这个例子创建了FileInputStream实例。FileInputStream是InputStream的子类,所以可以把FileInputStream实例赋值给InputStream变量。

注意:为了清晰,代码忽略了一些必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

从Java7开始,你可以使用“try-with-resource”结构确保InputStream在结束使用之后关闭,链接指向了一篇关于“try-with-resource”是如何工作的文章,这里只是一个简单的例子:
```java
try( InputStream inputstream = new FileInputStream("file.txt") ) {
    int data = inputstream.read();
    while(data != -1){
        System.out.print((char) data);
        data = inputstream.read();
    }

}
```

当执行线程退出try语句块的时候,InputStream变量会被关闭。

### read()
read()方法返回从InputStream流内读取到的一个字节内容(译者注:0~255),例子如下:
```java
int data = inputstream.read();
```
你可以把返回的int类型转化成char类型:
```java
char aChar = (char) data;
```
InputStream的子类可能会包含read()方法的替代方法。比如,DataInputStream允许你利用readBoolean(),readDouble()等方法读取Java基本类型变量int,long,float,double和boolean。

### 流末尾
如果read()方法返回-1,意味着程序已经读到了流的末尾,此时流内已经没有多余的数据可供读取了。-1是一个int类型,不是byte或者char类型,这是不一样的。

当达到流末尾时,你就可以关闭流了。

### read(byte[])
InputStream包含了2个从InputStream中读取数据并将数据存储到缓冲数组中的read()方法,他们分别是:

- int read(byte[])
- int read(byte, int offset, int length)

一次性读取一个字节数组的方式,比一次性读取一个字节的方式快的多,所以,尽可能使用这两个方法代替read()方法。

read(byte[])方法会尝试读取与给定字节数组容量一样大的字节数,返回值说明了已经读取过的字节数。如果InputStream内可读的数据不足以填满字节数组,那么数组剩余的部分将包含本次读取之前的数据。记得检查有多少数据实际被写入到了字节数组中。

read(byte, int offset, int length)方法同样将数据读取到字节数组中,不同的是,该方法从数组的offset位置开始,并且最多将length个字节写入到数组中。同样地,read(byte, int offset, int length)方法返回一个int变量,告诉你已经有多少字节已经被写入到字节数组中,所以请记得在读取数据前检查上一次调用read(byte, int offset, int length)的返回值。

这两个方法都会在读取到达到流末尾时返回-1。

这是一个使用InputStream的read(byte[])的例子:
```java
InputStream inputstream = new FileInputStream("c:\\data\\input-text.txt");
byte[] data = new byte[1024];
int bytesRead = inputstream.read(data);
while(bytesRead != -1) {
    doSomethingWithData(data, bytesRead);
    bytesRead = inputstream.read(data);
}
inputstream.close();
```

在代码中,首先创建了一个字节数组。然后声明一个叫做bytesRead的存储每次调用read(byte[])返回值的int变量,并且将第一次调用read(byte[])得到的返回值赋值给它。

在while循环内部,把字节数组和已读取字节数作为参数传递给doSomethingWithData方法然后执行调用。在循环的末尾,再次将数据写入到字节数组中。

你不需要想象出read(byte, int offset, int length)替代read(byte[])的场景,几乎可以在使用read(byte, int offset, int length)的任何地方使用read(byte[])。

### 输入流和数据源
一个输入流往往会和数据源联系起来,比如文件,网络连接,管道等,更多细节已经在Java IO概述文章中介绍过了。


## Java IO: OutputStream
OutputStream类是Java IO API中所有输出流的基类。子类包括BufferedOutputStream,FileOutputStream等等。参考Java IO概述这一小节底部的表格,可以浏览完整的子类的列表。


### 输出流和目标媒介
输出流往往和某些数据的目标媒介相关联,比如文件,网络连接,管道等。更多细节请参考Java IO概述。当写入到输出流的数据逐渐输出完毕时,目标媒介是所有数据的归属地。

### write(byte)
write(byte)方法用于把单个字节写入到输出流中。OutputStream的write(byte)方法将一个包含了待写入数据的int变量作为参数进行写入。只有int类型的第一个字节会被写入,其余位会被忽略。(译者注:写入低8位,忽略高24位)。

OutputStream的子类可能会包含write()方法的替代方法。比如,DataOutputStream允许你利用writeBoolean(),writeDouble()等方法将基本类型int,long,float,double,boolean等变量写入。

这是一个OutputStream的write()方法例子:
```java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");
while(hasMoreData()) {
    int data = getMoreData();
    output.write(data);
}
output.close();
```
这个例子首先创建了待写入的FileOutputStream。在进入while循环之后,循环的判断条件是hasMoreData()方法的返回值。hasMoreData()方法的实现不予展示,请把这个函数理解为:当有剩余可写数据时,返回true,否则返回false。

请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

### write(byte[])
OutputStream同样包含了将字节数据中全部或者部分数据写入到输出流中的方法,分别是write(byte[])和write(byte[], int offset, int length)。

write(byte[])把字节数组中所有数据写入到输出流中。

write(byte[], int offset, int length)把字节数据中从offset位置开始,length个字节的数据写入到输出流。

### flush()
OutputStream的flush()方法将所有写入到OutputStream的数据冲刷到相应的目标媒介中。比如,如果输出流是FileOutputStream,那么写入到其中的数据可能并没有真正写入到磁盘中。即使所有数据都写入到了FileOutputStream,这些数据还是有可能保留在内存的缓冲区中。通过调用flush()方法,可以把缓冲区内的数据刷新到磁盘(或者网络,以及其他任何形式的目标媒介)中。

### close()
当你结束数据写入时,需要关闭OutputStream。通过调用close()可以达到这一点。因为OutputStream的各种write()方法可能会抛出IO异常,所以你需要把调用close()的关闭操作方在finally块中执行。这是一个OutputStream调用close()的例子:
```java
OutputStream output = null;
try{
    output = new FileOutputStream("c:\\data\\output-text.txt");
    while(hasMoreData()) {
        int data = getMoreData();
        output.write(data);
    }
} finally {
    if(output != null) {
        output.close();
    }
}
```
这个例子在finally块中调用close()方法。虽然这种方式可以确保OutputStream关闭,但却不是一个完美的异常处理方案。我在Java IO异常处理这文章中更加详细地探讨了IO的异常处理。


## Java IO: FileInputStream

FileInputStream可以以字节流的形式读取文件内容。FileInputStream是InputStream的子类,这意味着你可以把FileInputStream当做InputStream使用(FileInputStream与InputStream的行为类似)。

这是一个FileInputStream的例子:
```java
InputStream input = new FileInputStream("c:\\data\\input-text.txt");
int data = input.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = input.read();
}
input.close();
```

请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

FileInputStream的read()方法返回读取到的包含一个字节内容的int变量(译者注:0~255)。如果read()方法返回-1,意味着程序已经读到了流的末尾,此时流内已经没有多余的数据可供读取了,你可以关闭流。-1是一个int类型,不是byte类型,这是不一样的。

FileInputStream也有其他的构造函数,允许你通过不同的方式读取文件。请参考[官方文档](http://docs.oracle.com/javase/7/docs/api/)查阅更多信息。

其中一个FileInputStream构造函数取一个File对象替代String对象作为参数。这里是一个使用该构造函数的例子:
```java
File file = new File("c:\\data\\input-text.txt");
InputStream input = new FileInputStream(file);
```
至于你该采用参数是String对象还是File对象的构造函数,取决于你当前是否已经拥有一个File对象,也取决于你是否要在打开FileOutputStream之前通过File对象执行某些检查(比如检查文件是否存在)。


## Java IO: FileOutputStream

FileOutputStream可以往文件里写入字节流,它是OutputStream的子类,所以你可以像使用OutputStream那样使用FileOutputStream。

这是一个FileOutputStream的例子:
```java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");
while(moreData) {
    int data = getMoreData();
    output.write(data);
}
output.close();
```

请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

FileOutputStream的write()方法取一个包含了待写入字节(译者注:低8位数据)的int变量作为参数进行写入。

FileOutputStream也有其他的构造函数,允许你通过不同的方式写入文件。请参考[官方文档](http://docs.oracle.com/javase/7/docs/api/)查阅更多信息。

### 文件内容的覆盖Override VS追加Appending

当你创建了一个指向已存在文件的FileOutputStream,你可以选择覆盖整个文件,或者在文件末尾追加内容。通过使用不同的构造函数可以实现不同的目的。

其中一个构造函数取文件名作为参数,会覆盖任何此文件名指向的文件。
```java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt");
```
另外一个构造函数取2个参数:文件名和一个布尔值,布尔值表明你是否需要覆盖文件。这是构造函数的例子:
```java
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", true); //appends to file
OutputStream output = new FileOutputStream("c:\\data\\output-text.txt", false); //overwrites file
```

### 写入字节数组
既然FileOutputStream是OutputStream的子类,所以你也可以往FileOutputStream中写入字节数组,而不需要每次都只写入一个字节。可以参考我的OutputStream教程查阅更多关于写入字节数组的信息。

### flush()
当你往FileOutputStream里写数据的时候,这些数据有可能会缓存在内存中。在之后的某个时间,比如,每次都只有X份数据可写,或者FileOutputStream关闭的时候,才会真正地写入磁盘。当FileOutputStream没被关闭,而你又想确保写入到FileOutputStream中的数据写入到磁盘中,可以调用flush()方法,该方法可以保证所有写入到FileOutputStream的数据全部写入到磁盘中。


## Java IO: RandomAccessFile

RandomAccessFile允许你来回读写文件,也可以替换文件中的某些部分。FileInputStream和FileOutputStream没有这样的功能。

### 创建一个RandomAccessFile
在使用RandomAccessFile之前,必须初始化它。这是例子:
```java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
```
请注意构造函数的第二个参数:“rw”,表明你以读写方式打开文件。请查阅Java文档获知你需要以何种方式构造RandomAccessFile。

### 在RandomAccessFile中来回读写
在RandomAccessFile的某个位置读写之前,必须把文件指针指向该位置。通过seek()方法可以达到这一目标。可以通过调用getFilePointer()获得当前文件指针的位置。例子如下:
```java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
file.seek(200);
long pointer = file.getFilePointer();
file.close();
```

### 读取RandomAccessFile
RandomAccessFile中的任何一个read()方法都可以读取RandomAccessFile的数据。例子如下:
```java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
int aByte = file.read();
file.close();
```
read()方法返回当前RandomAccessFile实例的文件指针指向的位置中包含的字节内容。Java文档中遗漏了一点:read()方法在读取完一个字节之后,会自动把指针移动到下一个可读字节。这意味着使用者在调用完read()方法之后不需要手动移动文件指针。

### 写入RandomAccessFile
RandomAccessFile中的任何一个write()方法都可以往RandomAccessFile中写入数据。例子如下:
```java
RandomAccessFile file = new RandomAccessFile("c:\\data\\file.txt", "rw");
file.write("Hello World".getBytes());
file.close();
```
与read()方法类似,write()方法在调用结束之后自动移动文件指针,所以你不需要频繁地把指针移动到下一个将要写入数据的位置。

### RandomAccessFile异常处理
为了本篇内容清晰,暂时忽略RandomAccessFile异常处理的内容。RandomAccessFile与其他流一样,在使用完毕之后必须关闭。想要了解更多信息,请参考Java IO异常处理。


## Java IO: File

Java IO API中的FIle类可以让你访问底层文件系统,通过File类,你可以做到以下几点:

- 检测文件是否存在
- 读取文件长度
- 重命名或移动文件
- 删除文件
- 检测某个路径是文件还是目录
- 读取目录中的文件列表

请注意:File只能访问文件以及文件系统的元数据。如果你想读写文件内容,需要使用FileInputStream、FileOutputStream或者RandomAccessFile。如果你正在使用Java NIO,并且想使用完整的NIO解决方案,你会使用到java.nio.FileChannel(否则你也可以使用File)。

实例化一个java.io.File对象
在使用File之前,必须拥有一个File对象,这是实例化的代码例子:
```java
File file = new File("c:\\data\\input-file.txt");
```
很简单,对吗？File类同样拥有多种不同实例化方式的构造函数。

### 检测文件是否存在
当你获得一个File对象之后,可以检测相应的文件是否存在。当文件不存在的时候,构造函数并不会执行失败。你已经准备好创建一个File了,对吧？

通过调用exists()方法,可以检测文件是否存在,代码如下:
```java
File file = new File("c:\\data\\input-file.txt");
boolean fileExists = file.exists();
```

### 文件长度
通过调用length()可以获得文件的字节长度,代码如下:
```java
File file = new File("c:\\data\\input-file.txt");
long length = file.length();
```

### 重命名或移动文件
通过调用File类中的renameTo()方法可以重命名(或者移动)文件,代码如下:
```java
File file = new File("c:\\data\\input-file.txt");
boolean success = file.renameTo(new File("c:\\data\\new-file.txt"));
```

### 删除文件
通过调用delete()方法可以删除文件,代码如下:
```java
File file = new File("c:\\data\\input-file.txt");
boolean success = file.delete();
```
delete()方法与rename()方法一样,返回布尔值表明是否成功删除文件,同样也会有相同的操作失败原因。

### 检测某个路径是文件还是目录
File对象既可以指向一个文件,也可以指向一个目录。可以通过调用isDirectory()方法,可以判断当前File对象指向的是文件还是目录。当方法返回值是true时,File指向的是目录,否则指向的是文件,代码如下:
```java
File file = new File("c:\\data");
boolean isDirectory = file.isDirectory();
```

### 读取目录中的文件列表
你可以通过调用list()或者listFiles()方法获取一个目录中的所有文件列表。list()方法返回当前File对象指向的目录中所有文件与子目录的字符串名称(译者注:不会返回子目录下的文件及其子目录名称)。listFiles()方法返回当前File对象指向的目录中所有文件与子目录相关联的File对象(译者注:与list()方法类似,不会返回子目录下的文件及其子目录)。代码如下:
```java
File file = new File("c:\\data");
String[] fileNames = file.list();
File[] files = file.listFiles();
```


## Java IO: PipedInputStream

PipedInputStream可以从管道中读取字节流数据,代码如下:
```java
InputStream input = new PipedInputStream(pipedOutputStream);
int data = input.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = input.read();
}
input.close();
```

请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

PipedInputStream的read()方法返回读取到的包含一个字节内容的int变量(译者注:0~255)。如果read()方法返回-1,意味着程序已经读到了流的末尾,此时流内已经没有多余的数据可供读取了,你可以关闭流。-1是一个int类型,不是byte类型,这是不一样的。


### Java IO管道
正如你所看到的例子那样,一个PipedInputStream需要与一个PipedOutputStream相关联,当这两种流联系起来时,就形成了一条管道。要想更多地了解Java IO中的管道,请参考Java IO管道。


## Java IO: PipedOutputStream

PipedOutputStream可以往管道里写入读取字节流数据,代码如下:
```java
OutputStream output = new PipedOutputStream(pipedInputStream);
while(moreData) {
    int data = getMoreData();
    output.write(data);
}
output.close();
```

请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

PipedOutputStream的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入。

### Java IO管道
一个PipedOutputStream总是需要与一个PipedInputStream相关联。当这两种流联系起来时,它们就形成了一条管道。要想更多地了解Java IO中的管道,请参考Java IO管道。


## Java IO: ByteArray和Filter

本小节会简要概括Java IO中字节数组与过滤器的输入输出流,主要涉及以下4个类型的流:ByteArrayInputStream,ByteArrayOutputStream,FilterInputStream,FilterOutputStream。请注意,为了清晰,这里忽略了必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

### ByteArrayInputStream
ByteArrayInputStream允许你从字节数组中读取字节流数据,代码如下:
```java
byte[] bytes = ... //get byte array from somewhere.
InputStream input = new ByteArrayInputStream(bytes);
int data = input.read();
while(data != -1) {
    //do something with data
    data = input.read();
}
input.close();
```
如果数据存储在数组中,ByteArrayInputStream可以很方便地读取数据。如果你有一个InputStream变量,又想从数组中读取数据呢？很简单,只需要把字节数组传递给ByteArrayInputStream的构造函数,在把这个ByteArrayInputStream赋值给InputStream变量就可以了(译者注:InputStream是所有字节输入流流的基类,Reader是所有字符输入流的基类,OutputStream与Writer同理)。

### ByteArrayOutputStream

ByteArrayOutputStream允许你以数组的形式获取写入到该输出流中的数据,代码如下:
```java
ByteArrayOutputStream output = new ByteArrayOutputStream();
//write data to output stream
byte[] bytes = output.toByteArray();
```

### FilterInputStream

FilterInputStream是实现自定义过滤输入流的基类,基本上它仅仅只是覆盖了InputStream中的所有方法。

就我自己而言,我没发现这个类明显的用途。除了构造函数取一个InputStream变量作为参数之外,我没看到FilterInputStream任何对InputStream新增或者修改的地方。如果你选择继承FilterInputStream实现自定义的类,同样也可以直接继承自InputStream从而避免额外的类层级结构。

### FilterOutputStream

内容同FilterInputStream,不再赘述。


## Java IO: 序列化与ObjectInputStream、ObjectOutputStream

本小节会简要概括Java IO中的序列化以及涉及到的流,主要包括ObjectInputStream和ObjectOutputStream。

### Serializable
如果你希望类能够序列化和反序列化,必须实现Serializable接口,就像所展示的ObjectInputStream和ObjectOutputStream例子一样。

对象序列化本身就是一个主题。Java IO系列教程主要关注流、reader和writer,所以我不会深入探讨对象序列化的细节。

### ObjectInputStream

ObjectInputStream能够让你从输入流中读取Java对象,而不需要每次读取一个字节。你可以把InputStream包装到ObjectInputStream中,然后就可以从中读取对象了。代码如下:
```java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
MyClass object = (MyClass) input.readObject(); //etc.
input.close();
```
在这个例子中,你读取的对象必须是MyClass的一个实例,并且必须事先通过ObjectOutputStream序列化到“object.data”文件中。(译者注:ObjectInputStream和ObjectOutputStream还有许多read和write方法,比如readInt、writeLong等等,详细信息请查看官方文档)

在你序列化和反序列化一个对象之前,该对象的类必须实现了java.io.Serializable接口。

### ObjectOutputStream

ObjectOutputStream能够让你把对象写入到输出流中,而不需要每次写入一个字节。你可以把OutputStream包装到ObjectOutputStream中,然后就可以把对象写入到该输出流中了。
```java
ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("object.data"));
MyClass object = new MyClass();  output.writeObject(object); //etc.
output.close();
```
例子中序列化的对象object现在可以从ObjectInputStream中读取了。

同样,在你序列化和反序列化一个对象之前,该对象的类必须实现了java.io.Serializable接口。


## Java IO: Reader和Writer

### Reader
Reader是Java IO中所有Reader的基类。Reader与InputStream类似,不同点在于,Reader基于字符而非基于字节。换句话说,Reader用于读取文本,而InputStream用于读取原始字节。


请记住,Java内部使用UTF8编码表示字符串。输入流中一个字节可能并不等同于一个UTF8字符。如果你从输入流中以字节为单位读取UTF8编码的文本,并且尝试将读取到的字节转换成字符,你可能会得不到预期的结果。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注:0~65535)。如果方法返回-1,表明Reader中已经没有剩余可读取字符,此时可以关闭Reader。-1是一个int类型,不是byte或者char类型,这是不一样的。

你通常会使用Reader的子类,而不会直接使用Reader。Reader的子类包括InputStreamReader,CharArrayReader,FileReader等等。可以查看Java IO概述浏览完整的Reader表格。

Reader通常与文件、字符数组、网络等数据源相关联,Java IO概述中同样说明了这一点。

###Writer

Writer是Java IO中所有Writer的基类。与Reader和InputStream的关系类似,Writer基于字符而非基于字节,Writer用于写入文本,OutputStream用于写入字节。

同样,你最好使用Writer的子类,不需要直接使用Writer,因为子类的实现更加明确,更能表现你的意图。常用子类包括OutputStreamWriter,CharArrayWriter,FileWriter等。

Writer的write(int c)方法,会将传入参数的低16位写入到Writer中,忽略高16位的数据。


## Java IO: InputStreamReader和OutputStreamWriter

本章节将简要介绍InputStreamReader和OutputStreamWriter。细心的读者可能会发现,在之前的文章中,IO中的类要么以Stream结尾,要么以Reader或者Writer结尾,那这两个同时以字节流和字符流的类名后缀结尾的类是什么用途呢？简单来说,这两个类把字节流转换成字符流,中间做了数据的转换,类似适配器模式的思想

### InputStreamReader

InputStreamReader会包含一个InputStream,从而可以将该输入字节流转换成字符流,代码例子:
```java
InputStream inputStream = new FileInputStream("c:\\data\\input.txt");
Reader reader = new InputStreamReader(inputStream);
int data = reader.read();
while(data != -1){
    char theChar = (char) data;
    data = reader.read();
}
reader.close();
```

注意:为了清晰,代码忽略了一些必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注:0~65535)。代码如下:
```java
int data = reader.read();
```
你可以把返回的int值转换成char变量,就像这样:
```java
char aChar = (char) data; //译者注:这里不会造成数据丢失,因为返回的int类型变量data只有低16位有数据,高16位没有数据
```
如果方法返回-1,表明Reader中已经没有剩余可读取字符,此时可以关闭Reader。-1是一个int类型,不是byte或者char类型,这是不一样的。

InputStreamReader同样拥有其他可选的构造函数,能够让你指定将底层字节流解释成何种编码的字符流。例子如下:
```java
InputStream inputStream = new FileInputStream("c:\\data\\input.txt");
Reader reader = new InputStreamReader(inputStream, "UTF-8");
```
注意构造函数的第二个参数,此时该InputStreamReader会将输入的字节流转换成UTF8字符流。

### OutputStreamWriter
OutputStreamWriter会包含一个OutputStream,从而可以将该输出字节流转换成字符流,代码如下:
```java
OutputStream outputStream = new FileOutputStream("c:\\data\\output.txt");
Writer writer = new OutputStreamWriter(outputStream);
writer.write("Hello World");
writer.close();
```
OutputStreamWriter同样拥有将输出字节流转换成指定编码的字符流的构造函数。


## Java IO: FileReader和FileWriter

本章节将简要介绍FileReader和FileWriter。与FileInputStream和FileOutputStream类似,FileReader与FileWriter用于处理文件内容。


FileReader
原文链接

FileReader能够以字符流的形式读取文件内容。除了读取的单位不同之外(译者注:FileReader读取字符,FileInputStream读取字节),FileReader与FileInputStream并无太大差异,也就是说,FileReader用于读取文本。根据不同的编码方案,一个字符可能会相当于一个或者多个字节。代码如下:
```java
Reader reader = new FileReader("c:\\data\\input-text.txt");
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```
注意:为了清晰,代码忽略了一些必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注:0~65535)。如果方法返回-1,表明FileReader中已经没有剩余可读取字符,此时可以关闭FileReader。-1是一个int类型,不是byte或者char类型,这是不一样的。

FileReader拥有其他可选的构造函数,能够让你使用不同的方式读取文件,更多内容请查看官方文档。

FileReader会假设你想使用你所使用的JVM的版本的默认编码处理字节流,但是这通常不是你想要的,你可以手动设置编码方案。

如果你想明确指定一种编码方案,利用InputStreamReader配合FileInputStream来替代FileReader(译者注:FileReader没有可以指定编码的构造函数)。InputStreamReader可以让你设置编码处理从底层文件中读取的字节。

### FileWriter

FileWriter能够把数据以字符流的形式写入文件。同样是处理文件,FileWriter处理字符,FileOutputStream处理字节。根据不同的编码方案,一个字符可能会相当于一个或者多个字节。代码如下:
```java
Writer writer = new FileWriter("c:\\data\\output.txt");
while(moreData) {
    String data = getMoreData();
    write.write(data);
}
writer.close();
```

处理文件都会碰到的一个问题是,当前写入的数据是覆盖原文件内容还是追加到文件末尾。当你创建一个FileWriter之后,你可以通过使用不同构造函数实现你的不同目的。

以下的构造函数取文件名作为参数,将会新写入的内容将会覆盖该文件:
```java
Writer writer = new FileWriter("c:\\data\\output.txt");
```
以下的构造函数取文件名和一个布尔变量作为参数,布尔值表明你是想追加还是覆盖该文件。例子如下:
```java
Writer writer = new FileWriter("c:\\data\\output.txt", true); //appends to file
Writer writer = new FileWriter("c:\\data\\output.txt", false); //overwrites file
```
同样,FileWriter不能指定编码,可以通过OutputStreamWriter配合FileOutputStream替代FileWriter。


## Java IO: 字符流的Buffered和Filter

本章节将简要介绍缓冲与过滤相关的reader和writer,主要涉及BufferedReader、BufferedWriter、FilterReader、FilterWriter。

### BufferedReader

BufferedReader能为字符输入流提供缓冲区,可以提高许多IO处理的速度。你可以一次读取一大块的数据,而不需要每次从网络或者磁盘中一次读取一个字节。特别是在访问大量磁盘数据时,缓冲通常会让IO快上许多。

BufferedReader和BufferedInputStream的主要区别在于,BufferedReader操作字符,而BufferedInputStream操作原始字节。只需要把Reader包装到BufferedReader中,就可以为Reader添加缓冲区(译者注:默认缓冲区大小为8192字节,即8KB)。代码如下:

```java
Reader input = new BufferedReader(new FileReader("c:\\data\\input-file.txt"));
```
你也可以通过传递构造函数的第二个参数,指定缓冲区大小,代码如下:

```java
Reader input = new BufferedReader(new FileReader("c:\\data\\input-file.txt"), 8 * 1024);
```
这个例子设置了8KB的缓冲区。最好把缓冲区大小设置成1024字节的整数倍,这样能更高效地利用内置缓冲区的磁盘。

除了能够为输入流提供缓冲区以外,其余方面BufferedReader基本与Reader类似。BufferedReader还有一个额外readLine()方法,可以方便地一次性读取一整行字符。

### BufferedWriter

与BufferedReader类似,BufferedWriter可以为输出流提供缓冲区。可以构造一个使用默认大小缓冲区的BufferedWriter(译者注:默认缓冲区大小8 * 1024B),代码如下:

```java
Writer writer = new BufferedWriter(new FileWriter("c:\\data\\output-file.txt"));
```
也可以手动设置缓冲区大小,代码如下:

```java
Writer writer = new BufferedWriter(new FileWriter("c:\\data\\output-file.txt"), 8 * 1024);
```
为了更好地使用内置缓冲区的磁盘,同样建议把缓冲区大小设置成1024的整数倍。除了能够为输出流提供缓冲区以外,其余方面BufferedWriter基本与Writer类似。类似地,BufferedWriter也提供了writeLine()方法,能够把一行字符写入到底层的字符输出流中。值得注意是,你需要手动flush()方法确保写入到此输出流的数据真正写入到磁盘或者网络中。

### FilterReader

与FilterInputStream类似,FilterReader是实现自定义过滤输入字符流的基类,基本上它仅仅只是简单覆盖了Reader中的所有方法。

就我自己而言,我没发现这个类明显的用途。除了构造函数取一个Reader变量作为参数之外,我没看到FilterReader任何对Reader新增或者修改的地方。如果你选择继承FilterReader实现自定义的类,同样也可以直接继承自Reader从而避免额外的类层级结构。

### FilterWriter

内容同FilterReader,不再赘述。


## Java IO: 字符流的Piped和CharArray

本章节将简要介绍管道与字符数组相关的reader和writer,主要涉及PipedReader、PipedWriter、CharArrayReader、CharArrayWriter。

### PipedReader

PipedReader能够从管道中读取字符流。与PipedInputStream类似,不同的是PipedReader读取的是字符而非字节。换句话说,PipedReader用于读取管道中的文本。代码如下:
```java
Reader reader = new PipedReader(pipedWriter);
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```
注意:为了清晰,代码忽略了一些必要的异常处理。想了解更多异常处理的信息,请参考Java IO异常处理。

read()方法返回一个包含了读取到的字符内容的int类型变量(译者注:0~65535)。如果方法返回-1,表明PipedReader中已经没有剩余可读取字符,此时可以关闭PipedReader。-1是一个int类型,不是byte或者char类型,这是不一样的。

正如你所看到的例子那样,一个PipedReader需要与一个PipedWriter相关联,当这两种流联系起来时,就形成了一条管道。要想更多地了解Java IO中的管道,请参考Java IO管道。

### PipedWriter

PipedWriter能够往管道中写入字符流。与PipedOutputStream类似,不同的是PipedWriter处理的是字符而非字节,PipedWriter用于写入文本数据。代码如下:
```java
PipedWriter writer = new PipedWriter(pipedReader);
while(moreData()) {
    int data = getMoreData();
    writer.write(data);
}
writer.close();
```
PipedWriter的write()方法取一个包含了待写入字节的int类型变量作为参数进行写入,同时也有采用字符串、字符数组作为参数的write()方法。

### CharArrayReader

CharArrayReader能够让你从字符数组中读取字符流。代码如下:
```java
char[] chars = ... //get char array from somewhere.
Reader reader = new CharArrayReader(chars);
int data = reader.read();
while(data != -1) {
    //do something with data
    data = reader.read();
}
reader.close();
```
如果数据的存储媒介是字符数组,CharArrayReader可以很方便的读取到你想要的数据。CharArrayReader会包含一个字符数组,然后将字符数组转换成字符流。(译者注:CharArrayReader有2个构造函数,一个是CharArrayReader(char[] buf),将整个字符数组创建成一个字符流。另外一个是CharArrayReader(char[] buf, int offset, int length),把buf从offset开始,length个字符创建成一个字符流。更多细节请参考Java官方文档)

### CharArrayWriter

CharArrayWriter能够把字符写入到字符输出流writer中,并且能够将写入的字符转换成字符数组。代码如下:

```java
CharArrayWriter writer = new CharArrayWriter();
//write characters to writer.
char[] chars = writer.toCharArray();
```
当你需要以字符数组的形式访问写入到writer中的字符流数据时,CharArrayWriter是个不错的选择。


## Java IO: 其他字节流

本小节会简要概括Java IO中的PushbackInputStream,SequenceInputStream,PrintStream, PushbackReader,LineNumberReader,StreamTokenizer,PrintWriter,StringReader,StringWriter。其中,最常用的是PrintStream,System.out和System.err都是PrintStream类型的变量,请查看Java IO: System.in, System.out, System.err浏览更多关于System.out和System.err的信息。

### PushbackInputStream

PushbackInputStream用于解析InputStream内的数据。有时候你需要提前知道接下来将要读取到的字节内容,才能判断用何种方式进行数据解析。PushBackInputStream允许你这么做,你可以把读取到的字节重新推回到InputStream中,以便再次通过read()读取。代码如下:
```java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"));
int data = input.read();
input.unread(data);
```
可以通过PushBackInputStream的构造函数设置推回缓冲区的大小,代码如下:
```java
PushbackInputStream input = new PushbackInputStream(new FileInputStream("c:\\data\\input.txt"), 8);
```
这个例子设置了8个字节的缓冲区,意味着你最多可以重新读取8个字节的数据。

### PushbackReader

PushbackReader与PushbackInputStream类似,唯一不同的是PushbackReader处理字符,PushbackInputStream处理字节。代码如下:
```java
PushbackReader reader = new PushbackReader(new FileReader("c:\\data\\input.txt"));
int data = reader.read();
reader.unread(data);
```

同样可以设置缓冲区大小,代码如下:
```java
PushbackReader reader = new PushbackReader(new FileReader("c:\\data\\input.txt"), 8);
```

### SequenceInputStream

SequenceInputStream把一个或者多个InputStream整合起来,形成一个逻辑连贯的输入流。当读取SequenceInputStream时,会先从第一个输入流中读取,完成之后再从第二个输入流读取,以此推类。代码如下:

```java
InputStream input1 = new FileInputStream("c:\\data\\file1.txt");
InputStream input2 = new FileInputStream("c:\\data\\file2.txt");
InputStream combined = new SequenceInputStream(input1, input2);
```
通过SequenceInputStream,例子中的2个InputStream使用起来就如同只有一个InputStream一样(译者注:SequenceInputStream的read()方法会在读取到当前流末尾时,关闭流,并把当前流指向逻辑链中的下一个流,最后返回新的当前流的read()值)。

### PrintStream

PrintStream允许你把格式化数据写入到底层OutputStream中。比如,写入格式化成文本的int,long以及其他原始数据类型到输出流中,而非它们的字节数据。代码如下:
```java
PrintStream output = new PrintStream(outputStream);
output.print(true);
output.print((int) 123);
output.print((float) 123.456);
output.printf(Locale.UK, "Text + data: %1$", 123);
output.close();
```
PrintStream包含2个强大的函数,分别是format()和printf()(这两个函数几乎做了一样的事情,但是C程序员会更熟悉printf())。

译者注:其中一个printf()函数实现如下:
```java
public PrintStream printf(String format, Object ... args) {
    return format(format, args);
}
```

### PrintWriter

与PrintStream类似,PrintWriter可以把格式化后的数据写入到底层writer中。由于内容相似,不再赘述。

值得一提的是,PrintWriter有更多种构造函数供使用者选择,除了可以输出到文件、Writer以外,还可以输出到OutputStream中(译者注:PrintStream只能把数据输出到文件和OutputStream)。


### LineNumberReader

LineNumberReader是记录了已读取数据行号的BufferedReader。默认情况下,行号从0开始,当LineNumberReader读取到行终止符时,行号会递增(译者注:换行\n,回车\r,或者换行回车\n\r都是行终止符)。

你可以通过getLineNumber()方法获取当前行号,通过setLineNumber()方法设置当前行数(译者注:setLineNumber()仅仅改变LineNumberReader内的记录行号的变量值,不会改变当前流的读取位置。流的读取依然是顺序进行,意味着你不能通过setLineNumber()实现流的跳跃读取)。代码如下:
```java
LineNumberReader reader = new LineNumberReader(new FileReader("c:\\data\\input.txt"));
int data = reader.read();
while(data != -1){
    char dataChar = (char) data;
    data = reader.read();
    int lineNumber = reader.getLineNumber();
}
```
如果解析的文本有错误,LineNumberReader可以很方便地定位问题。当你把错误报告给用户时,如果能够同时把出错的行号提供给用户,用户就能迅速发现并且解决问题。

### StreamTokenizer

StreamTokenizer(译者注:请注意不是StringTokenizer)可以把输入流(译者注:InputStream和Reader。通过InputStream构造StreamTokenizer的构造函数已经在JDK1.1版本过时,推荐将InputStream转化成Reader,再利用此Reader构造StringTokenizer)分解成一系列符号。比如,句子”Mary had a little lamb”的每个单词都是一个单独的符号。

当你解析文件或者计算机语言时,为了进一步的处理,需要将解析的数据分解成符号。通常这个过程也称作分词。

通过循环调用nextToken()可以遍历底层输入流的所有符号。在每次调用nextToken()之后,StreamTokenizer有一些变量可以帮助我们获取读取到的符号的类型和值。这些变量是:

ttype 读取到的符号的类型(字符,数字,或者行结尾符)

sval 如果读取到的符号是字符串类型,该变量的值就是读取到的字符串的值

nval 如果读取到的符号是数字类型,该变量的值就是读取到的数字的值

代码如下:
```java
StreamTokenizer tokenizer = new StreamTokenizer(new StringReader("Mary had 1 little lamb..."));
while(tokenizer.nextToken() != StreamTokenizer.TT_EOF){
    if(tokenizer.ttype == StreamTokenizer.TT_WORD) {
        System.out.println(tokenizer.sval);
    } else if(tokenizer.ttype == StreamTokenizer.TT_NUMBER) {
        System.out.println(tokenizer.nval);
    } else if(tokenizer.ttype == StreamTokenizer.TT_EOL) {
        System.out.println();
    }
}
```
译者注:TT_EOF表示流末尾,TT_EOL表示行末尾。

StreamTokenizer可以识别标示符,数字,引用的字符串,和多种注释类型。你也可以指定何种字符解释成空格、注释的开始以及结束等。在StreamTokenizer开始解析之前,所有的功能都可以进行配置。请查阅官方文档获取更多信息。

### StringReader

StringReader能够将原始字符串转换成Reader,代码如下:
```java
Reader reader = new StringReader("input string...");
int data = reader.read();
while(data != -1) {
    //do something with data...
    doSomethingWithData(data);
    data = reader.read();
}
reader.close();
```

### StringWriter

StringWriter能够以字符串的形式从Writer中获取写入到其中数据,代码如下:
```java
StringWriter writer = new StringWriter();
//write characters to writer.
String data = writer.toString();
StringBuffer dataBuffer = writer.getBuffer();
```
toString()方法能够获取StringWriter中的字符串数据。

getBuffer()方法能够获取StringWriter内部构造字符串时所使用的StringBuffer对象。

ref:
[http://tutorials.jenkov.com/java-io/index.html](http://tutorials.jenkov.com/java-io/index.html)

[image 1]: http://qn.atecher.com/无标题1.png
[image 2]: http://qn.atecher.com/无标题2.png
[image 3]: http://qn.atecher.com/QQ截图20141020174145.png
[image 4]: http://qn.atecher.com/1.png