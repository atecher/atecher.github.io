---
title: Java-NIO
date: 2017-10-09 00:00:00
categories: Java
tags:
    - Java
    - NIO
---

Java NIO(New IO)是一个可以替代标准Java IO API的IO API(从Java 1.4开始),Java NIO提供了与标准IO不同的IO工作方式。

**Java NIO: Channels and Buffers(通道和缓冲区)**

标准的IO基于字节流和字符流进行操作的,而NIO是基于通道(Channel)和缓冲区(Buffer)进行操作,数据总是从通道读取到缓冲区中,或者从缓冲区写入到通道中。

**Java NIO: Non-blocking IO(非阻塞IO)**

Java NIO可以让你非阻塞的使用IO,例如:当线程从通道读取数据到缓冲区时,线程还是可以进行其他事情。当数据被写入到缓冲区时,线程可以继续处理它。从缓冲区写入通道也类似。

**Java NIO: Selectors(选择器)**

Java NIO引入了选择器的概念,选择器用于监听多个通道的事件(比如:连接打开,数据到达)。因此,单个的线程可以监听多个数据通道。

<!-- more -->

## Java NIO 概述

Java NIO 由以下几个核心部分组成:

- Channels
- Buffers
- Selectors

虽然Java NIO 中除此之外还有很多类和组件,但在我看来,Channel,Buffer 和 Selector 构成了核心的API。其它组件,如Pipe和FileLock,只不过是与三个核心组件共同使用的工具类。因此,在概述中我将集中在这三个组件上。其它组件会在单独的章节中讲到。

### Channel 和 Buffer

基本上,所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中,也可以从Buffer 写到Channel中。这里有个图示:
{% asset_img overview-channels-buffers1.png %}

Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现:

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

正如你所看到的,这些通道涵盖了UDP 和 TCP 网络IO,以及文件IO。

与这些类一起的有一些有趣的接口,但为简单起见,我尽量在概述中不提到它们。本教程其它章节与它们相关的地方我会进行解释。

以下是Java NIO里关键的Buffer实现:

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型:byte, short, int, long, float, double 和 char。

Java NIO 还有个 MappedByteBuffer,用于表示内存映射文件, 我也不打算在概述中说明。

### Selector
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接(通道),但每个连接的流量都很低,使用Selector就会很方便。例如,在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示:
{% asset_img overview-selectors.png %}
要使用Selector,得向Selector注册Channel,然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回,线程就可以处理这些事件,事件的例子有如新连接进来,数据接收等。


## Channel

Java NIO的通道类似流,但又有些不同:

- 既可以从通道中读取数据,又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer,或者总是要从一个Buffer中写入。

正如上面所说,从通道读取数据到缓冲区,从缓冲区写入数据到通道。如下图所示:
{% asset_img overview-channels-buffers.png %}

### Channel的实现

这些是Java NIO中最重要的通道的实现:

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel
- FileChannel 从文件中读写数据。

DatagramChannel 能通过UDP读写网络中的数据。

SocketChannel 能通过TCP读写网络中的数据。

ServerSocketChannel可以监听新进来的TCP连接,像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

### 基本的 Channel 示例

下面是一个使用FileChannel读取数据到Buffer中的示例:
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
    System.out.println("Read " + bytesRead);
    buf.flip();

    while(buf.hasRemaining()){
        System.out.print((char) buf.get());
    }

    buf.clear();
    bytesRead = inChannel.read(buf);
}
aFile.close();
```
注意 buf.flip() 的调用,首先读取数据到Buffer,然后反转Buffer,接着再从Buffer中读取数据。下一节会深入讲解Buffer的更多细节。


## Buffer

Java NIO中的Buffer用于和NIO通道进行交互。如你所知,数据是从通道读入缓冲区,从缓冲区写入到通道中的。

缓冲区本质上是一块可以写入数据,然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象,并提供了一组方法,用来方便的访问该块内存。

### Buffer的基本用法

使用Buffer读写数据一般遵循以下四个步骤:

1. 写入数据到Buffer
2. 调用flip()方法
3. 从Buffer中读取数据
4. 调用clear()方法或者compact()方法

当向buffer写入数据时,buffer会记录下写了多少数据。一旦要读取数据,需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下,可以读取之前写入到buffer的所有数据。

一旦读完了所有的数据,就需要清空缓冲区,让它可以再次被写入。有两种方式能清空缓冲区:调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处,新写入的数据将放到缓冲区未读数据的后面。

下面是一个使用Buffer的例子:
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

### Buffer的capacity,position和limit

缓冲区本质上是一块可以写入数据,然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象,并提供了一组方法,用来方便的访问该块内存。

为了理解Buffer的工作原理,需要熟悉它的三个属性:

- capacity
- position
- limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式,capacity的含义总是一样的。

这里有一个关于capacity,position和limit在读写模式中的说明,详细的解释在插图后面。
{% asset_img buffers-modes.png %}

#### capacity

作为一个内存块,Buffer有一个固定的大小值,也叫“capacity”.你只能往里写capacity个byte、long,char等类型。一旦Buffer满了,需要将其清空(通过读数据或者清除数据)才能继续写数据往里写数据。

#### position

当你写数据到Buffer中时,position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后, position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时,也是从某个特定位置读。当将Buffer从写模式切换到读模式,position会被重置为0. 当从Buffer的position处读取数据时,position向前移动到下一个可读的位置。

#### limit

在写模式下,Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下,limit等于Buffer的capacity。

当切换Buffer到读模式时, limit表示你最多能读到多少数据。因此,当切换Buffer到读模式时,limit会被设置成写模式下的position值。换句话说,你能读到之前写入的所有数据(limit被设置成已写数据的数量,这个值在写模式下就是position)


### Buffer的类型

Java NIO 有以下Buffer类型

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

如你所见,这些Buffer类型代表了不同的数据类型。换句话说,就是可以通过char,short,int,long,float 或 double类型来操作缓冲区中的字节。

MappedByteBuffer 有些特别,在涉及它的专门章节中再讲。

### Buffer的分配

要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配48字节capacity的ByteBuffer的例子。

```java
ByteBuffer buf = ByteBuffer.allocate(48);
```
这是分配一个可存储1024个字符的CharBuffer:
```java
CharBuffer buf = CharBuffer.allocate(1024);
```

### 向Buffer中写数据

写数据到Buffer有两种方式:

- 从Channel写到Buffer。
- 通过Buffer的put()方法写到Buffer里。

从Channel写到Buffer的例子

```java
int bytesRead = inChannel.read(buf); //read into buffer.
```
通过put方法写Buffer的例子:
```java
buf.put(127);
```
put方法有很多版本,允许你以不同的方式把数据写入到Buffer中。例如, 写到一个指定的位置,或者把一个字节数组写入到Buffer。 更多Buffer实现的细节参考JavaDoc。

### flip()方法

flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0,并将limit设置成之前position的值。

换句话说,position现在用于标记读的位置,limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

### 从Buffer中读取数据

从Buffer中读取数据有两种方式:

- 从Buffer读取数据到Channel。
- 使用get()方法从Buffer中读取数据。

从Buffer读取数据到Channel的例子:
```java
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```
使用get()方法从Buffer中读取数据的例子
```java
byte aByte = buf.get();
```
get方法有很多版本,允许你以不同的方式从Buffer中读取数据。例如,从指定position读取,或者从Buffer中读取数据到字节数组。更多Buffer实现的细节参考JavaDoc。

### rewind()方法
Buffer.rewind()将position设回0,所以你可以重读Buffer中的所有数据。limit保持不变,仍然表示能从Buffer中读取多少个元素(byte、char等)。

### clear()与compact()方法

一旦读完Buffer中的数据,需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法,position将被设回0,limit被设置成 capacity的值。换句话说,Buffer 被清空了。Buffer中的数据并未清除,只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据,调用clear()方法,数据将“被遗忘”,意味着不再有任何标记会告诉你哪些数据被读过,哪些还没有。

如果Buffer中仍有未读的数据,且后续还需要这些数据,但是此时想要先先写些数据,那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样,设置成capacity。现在Buffer准备好写数据了,但是不会覆盖未读的数据。

### mark()与reset()方法

通过调用Buffer.mark()方法,可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如:
```java
buffer.mark();
//call buffer.get() a couple of times, e.g. during parsing.
buffer.reset();  //set position back to mark.
```

### equals()与compareTo()方法

可以使用equals()和compareTo()方法比较两个Buffer。

#### equals()

当满足下列条件时,表示两个Buffer相等:

1. 有相同的类型(byte、char、int等)。
2. Buffer中剩余的byte、char等的个数相等。
3. Buffer中所有剩余的byte、char等都相同。

如你所见,equals只是比较Buffer的一部分,不是每一个在它里面的元素都比较。实际上,它只比较Buffer中的剩余元素。

#### compareTo()方法

compareTo()方法比较两个Buffer的剩余元素(byte、char等), 如果满足下列条件,则认为一个Buffer“小于”另一个Buffer:

1. 第一个不相等的元素小于另一个Buffer中对应的元素 。
2. 所有元素都相等,但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

(译注:剩余元素是从 position到limit之间的元素)


## Scatter/Gather

Java NIO开始支持scatter/gather,scatter/gather用于描述从Channel(译者注:Channel在中文经常翻译为通道)中读取或者写入到Channel的操作。
分散(scatter)从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此,Channel将从Channel中读取的数据“分散(scatter)”到多个Buffer中。
聚集(gather)写入Channel是指在写操作时将多个buffer的数据写入同一个Channel,因此,Channel 将多个Buffer中的数据“聚集(gather)”后发送到Channel。

scatter / gather经常用于需要将传输的数据分开处理的场合,例如传输一个由消息头和消息体组成的消息,你可能会将消息体和消息头分散到不同的buffer中,这样你可以方便的处理消息头和消息体。

### Scattering Reads
Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述:
{% asset_img scatter.png %}

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```
注意buffer首先被插入到数组,然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer,当一个buffer被写满后,channel紧接着向另一个buffer中写。

Scattering Reads在移动下一个buffer前,必须填满当前的buffer,这也意味着它不适用于动态消息(译者注:消息大小不固定)。换句话说,如果存在消息头和消息体,消息头必须完成填充(例如 128byte),Scattering Reads才能正常工作。

### Gathering Writes

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述:
{% asset_img gather.png %}
代码示例如下:
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```
buffers数组是write()方法的入参,write()方法会按照buffer在数组中的顺序,将数据写入到channel,注意只有position和limit之间的数据才会被写入。因此,如果一个buffer的容量为128byte,但是仅仅包含58byte的数据,那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反,Gathering Writes能较好的处理动态消息。


## 通道之间的数据传输

在Java NIO中,如果两个通道中有一个是FileChannel,那你可以直接将数据从一个channel(译者注:channel中文常译作通道)传输到另外一个channel。

### transferFrom()

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中(译者注:这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中)。下面是一个简单的例子:
```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(position, count, fromChannel);
```
方法的输入参数position表示从position处开始向目标文件写入数据,count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节,则所传输的字节数要小于请求的字节数。
此外要注意,在SoketChannel的实现中,SocketChannel只会传输此刻准备好的数据(可能不足count字节)。因此,SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

### transferTo()

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子:
```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```
是不是发现这个例子和前面那个例子特别相似？除了调用方法的FileChannel对象不一样外,其他的都一样。
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。


## Selector

Selector(选择器)是Java NIO中能够检测一到多个NIO通道,并能够知晓通道是否为诸如读写事件做好准备的组件。这样,一个单独的线程可以管理多个channel,从而管理多个网络连接。

### 为什么使用Selector?

仅用单个线程来处理多个Channels的好处是,只需要更少的线程来处理通道。事实上,可以只用一个线程处理所有的通道。对于操作系统来说,线程之间上下文切换的开销很大,而且每个线程都要占用系统的一些资源(如内存)。因此,使用的线程越少越好。

但是,需要记住,现代的操作系统和CPU在多任务方面表现的越来越好,所以多线程的开销随着时间的推移,变得越来越小了。实际上,如果一个CPU有多个内核,不使用多任务可能是在浪费CPU能力。不管怎么说,关于那种设计的讨论应该放在另一篇不同的文章中。在这里,只要知道使用Selector能够处理多个通道就足够了。

下面是单线程使用一个Selector处理3个channel的示例图:
{% asset_img overview-selectors.png %}

### Selector的创建

通过调用Selector.open()方法创建一个Selector,如下:
```java
Selector selector = Selector.open();
```

### 向Selector注册通道

为了将Channel和Selector配合使用,必须将channel注册到selector上。通过SelectableChannel.register()方法来实现,如下:
```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,
	Selectionkey.OP_READ);
```
与Selector一起使用时,Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用,因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注意register()方法的第二个参数。这是一个“interest集合”,意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件:

1. Connect
2. Accept
3. Read
4. Write

通道触发了一个事件意思是该事件已经就绪。所以,某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

这四种事件用SelectionKey的四个常量来表示:

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对不止一种事件感兴趣,那么可以用“位或”操作符将常量连接起来,如下:

int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
在下面还会继续提到interest集合。

### SelectionKey

在上一小节中,当向Selector注册Channel时,register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性:

- interest集合
- ready集合
- Channel
- Selector
- 附加的对象(可选)

下面我会描述这些属性。

#### interest集合

就像向Selector注册通道一节中所描述的,interest集合是你所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合,像这样:
```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```
可以看到,用“位与”操作interest 集合和给定的SelectionKey常量,可以确定某个确定的事件是否在interest 集合中。

#### ready集合

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后,你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合:
```java
int readySet = selectionKey.readyOps();
```
可以用像检测interest集合那样的方法,来检测channel中什么事件或操作已经就绪。但是,也可以使用以下四个方法,它们都会返回一个布尔类型:
```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

#### Channel + Selector

从SelectionKey访问Channel和Selector很简单。如下:
```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

#### 附加的对象

可以将一个对象或者更多信息附着到SelectionKey上,这样就能方便的识别某个给定的通道。例如,可以附加 与通道一起使用的Buffer,或是包含聚集数据的某个对象。使用方法如下:
```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```
还可以在用register()方法向Selector注册Channel的时候附加对象。如:
```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

### 通过Selector选择通道

一旦向Selector注册了一或多个通道,就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件(如连接、接受、读或写)已经准备就绪的那些通道。换句话说,如果你对“读就绪”的通道感兴趣,select()方法会返回读事件已经就绪的那些通道。

下面是select()方法:

- int select()
- int select(long timeout)
- int selectNow()

select()阻塞到至少有一个通道在你注册的事件上就绪了。

select(long timeout)和select()一样,除了最长会阻塞timeout毫秒(参数)。

selectNow()不会阻塞,不管什么通道就绪都立刻返回(译者注:此方法执行非阻塞的选择操作。如果自从前一次选择操作后,没有通道变成可选择的,则此方法直接返回零。)。

select()方法返回的int值表示有多少通道已经就绪。亦即,自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法,因为有一个通道变成就绪状态,返回了1,若再次调用select()方法,如果另一个通道就绪了,它会再次返回1。如果对第一个就绪的channel没有做任何操作,现在就有两个就绪的通道,但在每次select()方法调用之间,只有一个通道就绪了。

#### selectedKeys()

一旦调用了select()方法,并且返回值表明有一个或更多个通道就绪了,然后可以通过调用selector的selectedKeys()方法,访问“已选择键集(selected key set)”中的就绪通道。如下所示:
```java
Set selectedKeys = selector.selectedKeys();
```
当像Selector注册Channel时,Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下:
```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```
这个循环遍历已选择键集中的每个键,并检测各个键所对应的通道的就绪事件。

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时,Selector会再次将其放入已选择键集中。

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型,如ServerSocketChannel或SocketChannel等。

### wakeUp()

某个线程调用select()方法后阻塞了,即使没有通道已经就绪,也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

如果有其它线程调用了wakeup()方法,但当前没有线程阻塞在select()方法上,下个调用select()方法的线程会立即“醒来(wake up)”。

### close()

用完Selector后调用其close()方法会关闭该Selector,且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。

完整的示例

这里有一个完整的示例,打开一个Selector,注册一个通道注册到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件(接受,连接,读,写)是否就绪。
```java
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
  }
}
```


## FileChannel


Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel无法设置为非阻塞模式,它总是运行在阻塞模式下。

### 打开FileChannel

在使用FileChannel之前,必须先打开它。但是,我们无法直接打开一个FileChannel,需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。下面是通过RandomAccessFile打开FileChannel的示例:
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```

### 从FileChannel读取数据

调用多个read()方法之一从FileChannel中读取数据。如:
```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```
首先,分配一个Buffer。从FileChannel中读取的数据将被读到Buffer中。

然后,调用FileChannel.read()方法。该方法将数据从FileChannel读取到Buffer中。read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1,表示到了文件末尾。

### 向FileChannel写数据

使用FileChannel.write()方法向FileChannel写数据,该方法的参数是一个Buffer。如:
```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
	channel.write(buf);
}
```
注意FileChannel.write()是在while循环中调用的。因为无法保证write()方法一次能向FileChannel写入多少字节,因此需要重复调用write()方法,直到Buffer中已经没有尚未写入通道的字节。

### 关闭FileChannel

用完FileChannel后必须将其关闭。如:
```java
channel.close();
```

### FileChannel的position方法

有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取FileChannel的当前位置。

也可以通过调用position(long pos)方法设置FileChannel的当前位置。

这里有两个例子:
```java
long pos = channel.position();
channel.position(pos +123);
```
如果将位置设置在文件结束符之后,然后试图从文件通道中读取数据,读方法将返回-1 —— 文件结束标志。

如果将位置设置在文件结束符之后,然后向通道中写数据,文件将撑大到当前位置并写入数据。这可能导致“文件空洞”,磁盘上物理文件中写入的数据间有空隙。

### FileChannel的size方法

FileChannel实例的size()方法将返回该实例所关联文件的大小。如:
```java
long fileSize = channel.size();
```

### FileChannel的truncate方法

可以使用FileChannel.truncate()方法截取一个文件。截取文件时,文件将中指定长度后面的部分将被删除。如:
```java
channel.truncate(1024);
```
这个例子截取文件的前1024个字节。

### FileChannel的force方法

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑,操作系统会将数据缓存在内存中,所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点,需要调用force()方法。

force()方法有一个boolean类型的参数,指明是否同时将文件元数据(权限信息等)写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上:
```java
channel.force(true);
```


## SocketChannel

Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel:

1. 打开一个SocketChannel并连接到互联网上的某台服务器。
2. 一个新连接到达ServerSocketChannel时,会创建一个SocketChannel。

### 打开 SocketChannel

下面是SocketChannel的打开方式:
```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

### 关闭 SocketChannel

当用完SocketChannel之后调用SocketChannel.close()关闭SocketChannel:
```java
socketChannel.close();
```

### 从 SocketChannel 读取数据

要从SocketChannel中读取数据,调用一个read()的方法之一。以下是例子:
```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);
```
首先,分配一个Buffer。从SocketChannel读取到的数据将会放到这个Buffer中。

然后,调用SocketChannel.read()。该方法将数据从SocketChannel 读到Buffer中。read()方法返回的int值表示读了多少字节进Buffer里。如果返回的是-1,表示已经读到了流的末尾(连接关闭了)。

### 写入 SocketChannel

写数据到SocketChannel用的是SocketChannel.write()方法,该方法以一个Buffer作为参数。示例如下:
```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```
注意SocketChannel.write()方法的调用是在一个while循环中的。Write()方法无法保证能写多少字节到SocketChannel。所以,我们重复调用write()直到Buffer没有要写的字节为止。

### 非阻塞模式

可以设置 SocketChannel 为非阻塞模式(non-blocking mode).设置之后,就可以在异步模式下调用connect(), read() 和write()了。

#### connect()

如果SocketChannel在非阻塞模式下,此时调用connect(),该方法可能在连接建立之前就返回了。为了确定连接是否建立,可以调用finishConnect()的方法。像这样:
```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}
```

#### write()

非阻塞模式下,write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。前面已经有例子了,这里就不赘述了。

#### read()

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值,它会告诉你读取了多少字节。

### 非阻塞模式与选择器

非阻塞模式与选择器搭配会工作的更好,通过将一或多个SocketChannel注册到Selector,可以询问选择器哪个通道已经准备好了读取,写入等。Selector与SocketChannel的搭配使用会在后面详讲。


## ServerSocketChannel

Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道, 就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。

这里有个例子:
```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    //do something with socketChannel...
}
```

### 打开 ServerSocketChannel

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如:
```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

### 关闭 ServerSocketChannel

通过调用ServerSocketChannel.close() 方法来关闭ServerSocketChannel. 如:
```java
serverSocketChannel.close();
```

### 监听新进来的连接

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 accept()方法. 如下面的例子:
```java
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    //do something with socketChannel...
}
```
当然,也可以在while循环中使用除了true以外的其它退出准则。

### 非阻塞模式

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下,accept() 方法会立刻返回,如果还没有新进来的连接,返回的将是null。 因此,需要检查返回的SocketChannel是否是null.如:
```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    if(socketChannel != null){
        //do something with socketChannel...
    }
}
```


## 非阻塞式服务器

即使你知道Java NIO 非阻塞的工作特性(如Selector,Channel,Buffer等组件),但是想要设计一个非阻塞的服务器仍然是一件很困难的事。非阻塞式服务器相较于阻塞式来说要多上许多挑战。本文将会讨论非阻塞式服务器的主要几个难题,并针对这些难题给出一些可能的解决方案。

本文的设计思路想法都是基于Java NIO的。但是我相信如果某些语言中也有像Selector之类的组件的话,文中的想法也能用于该语言。据我所知,类似的组件底层操作系统会提供,所以对你来说也可以根据其中的思想运用在其他语言上。

### 非阻塞式服务器– GitHub 仓库
我已经创建了一些简单的这些思想的概念验证呈现在这篇教程中,并且为了让你可以看到,我把源码放到了github资源库上了。这里是GitHub资源库地址:
[https://github.com/jjenkov/java-nio-server](https://github.com/jjenkov/java-nio-server)

### 非阻塞式IO管道(Pipelines)
一个非阻塞式IO管道是由各个处理非阻塞式IO组件组成的链。其中包括读/写IO。下图就是一个简单的非阻塞式IO管道组成:
{% asset_img non-blocking-server-1.png %}
一个组件使用 Selector 监控 Channel 什么时候有可读数据。然后这个组件读取输入并且根据输入生成相应的输出。最后输出将会再次写入到一个Channel中。

一个非阻塞式IO管道不需要将读数据和写数据都包含,有一些管道可能只会读数据,另一些可能只会写数据。

上图仅显示了一个单一的组件。一个非阻塞式IO管道可能拥有超过一个以上的组件去处理输入数据。一个非阻塞式管道的长度是由他的所要完成的任务决定。

一个非阻塞IO管道可能同时读取多个Channel里的数据。举个例子:从多个SocketChannel管道读取数据。

其实上图的控制流程还是太简单了。这里是组件从Selector开始从Channel中读取数据,而不是Channel将数据推送给Selector进入组件中,即便上图画的就是这样。

### 非阻塞式vs. 阻塞式管道
非阻塞和阻塞IO管道两者之间最大的区别在于他们如何从底层Channel(Socket或者file)读取数据。

IO管道通常从流中读取数据(来自socket或者file)并且将这些数据拆分为一系列连贯的消息。这和使用tokenizer(这里估计是解析器之类的意思)将数据流解析为token(这里应该是数据包的意思)类似。相反你只是将数据流分解为更大的消息体。我将拆分数据流成消息这一组件称为“消息读取器”(Message Reader)下面是Message Reader拆分流为消息的示意图:
{% asset_img non-blocking-server-2.png %}
一个阻塞IO管道可以使用类似InputStream的接口每次一个字节地从底层Channel读取数据,并且这个接口阻塞直到有数据可以读取。这就是阻塞式Message Reader的实现过程。

使用阻塞式IO接口简化了Message Reader的实现。阻塞式Message Reader从不用处理在流没有数据可读的情况,或者它只读取流中的部分数据并且对于消息的恢复也要延迟处理的情况。

同样,阻塞式Message Writer(一个将数据写入流中组件)也从不用处理只有部分数据被写入和写入消息要延迟恢复的情况。

#### 阻塞式IO管道的缺陷

虽然阻塞式Message Reader容易实现,但是也有一个不幸的缺点:每一个要分解成消息的流都需要一个独立的线程。必须要这样做的理由是每一个流的IO接口会阻塞,直到它有数据读取。这就意味着一个单独的线程是无法尝试从一个没有数据的流中读取数据转去读另一个流。一旦一个线程尝试从一个流中读取数据,那么这个线程将会阻塞直到有数据可以读取。

如果IO管道是必须要处理大量并发链接服务器的一部分的话,那么服务器就需要为每一个链接维护一个线程。对于任何时间都只有几百条并发链接的服务器这确实不是什么问题。但是如果服务器拥有百万级别的并发链接量,这种设计方式就没有良好收放。每个线程都会占用栈32bit-64bit的内存。所以一百万个线程占用的内存将会达到1TB！不过在此之前服务器将会把所有的内存用以处理传经来的消息(例如:分配给消息处理期间使用对象的内存)

为了将线程数量降下来,许多服务器使用了服务器维持线程池(例如:常用线程为100)的设计,从而一次一个地从入站链接(inbound connections)地读取。入站链接保存在一个队列中,线程按照进入队列的顺序处理入站链接。这一设计如下图所示:(译者注:Tomcat就是这样的)
{% asset_img non-blocking-server-3.png %}
然而,这一设计需要入站链接合理地发送数据。如果入站链接长时间不活跃,那么大量的不活跃链接实际上就造成了线程池中所有线程阻塞。这意味着服务器响应变慢甚至是没有反应。

一些服务器尝试通过弹性控制线程池的核心线程数量这一设计减轻这一问题。例如,如果线程池线程不足时,线程池可能开启更多的线程处理请求。这一方案意味着需要大量的长时链接才能使服务器不响应。但是记住,对于并发线程数任然是有一个上限的。因此,这一方案仍然无法很好地解决一百万个长时链接。

### 基础非阻塞式IO管道设计
一个非阻塞式IO管道可以使用一个单独的线程向多个流读取数据。这需要流可以被切换到非阻塞模式。在非阻塞模式下,当你读取流信息时可能会返回0个字节或更多字节的信息。如果流中没有数据可读就返回0字节,如果流中有数据可读就返回1+字节。

为了避免检查没有可读数据的流我们可以使用 Java NIO Selector. 一个或多个SelectableChannel 实例可以同时被一个Selector注册。**当你调用Selector的select()或者 selectNow() 方法它只会返回有数据读取的SelectableChannel的实例**. 下图是该设计的示意图:
{% asset_img non-blocking-server-4.png %}

### 读取部分消息
当我们从一个SelectableChannel读取一个数据包时,我们不知道这个数据包相比于源文件是否有丢失或者重复数据(原文是:When we read a block of data from a SelectableChannel we do not know if that data block contains less or more than a message)。一个数据包可能的情况有:缺失数据(比原有消息的数据少)、与原有一致、比原来的消息的数据更多(例如:是原来的1.5或者2.5倍)。数据包可能出现的情况如下图所示:
{% asset_img non-blocking-server-5.png %}
在处理类似上面这样部分信息时,有两个问题:

- 判断你是否能在数据包中获取完整的消息。
- 在其余消息到达之前如何处理已到达的部分消息。

判断消息的完整性需要消息读取器(Message Reader)在数据包中寻找是否存在至少一个完整消息体的数据。如果一个数据包包含一个或多个完整消息体,这些消息就能够被发送到管道进行处理。寻找完整消息体这一处理可能会重复多次,因此这一操作应该尽可能的快。

判断消息完整性和存储部分消息都是消息读取器(Message Reader)的责任。为了避免混合来自不同Channel的消息,我们将对每一个Channel使用一个Message Reader。设计如下图所示:
{% asset_img non-blocking-server-6.png %}
在从Selector得到可从中读取数据的Channel实例之后, 与该Channel相关联的Message Reader读取数据并尝试将他们分解为消息。这样读出的任何完整消息可以被传到读取通道(read pipeline)任何需要处理这些消息的组件中。

一个Message Reader一定满足特定的协议。Message Reader需要知道它尝试读取的消息的消息格式。如果我们的服务器可以通过协议来复用,那它需要有能够插入Message Reader实现的功能 – 可能通过接收一个Message Reader工厂作为配置参数。

### 存储部分消息
现在我们已经确定Message Reader有责任存储部分消息,直到收到完整的消息,我们需要弄清楚这些部分消息的存储应该如何实现。

有两个设计因素我们要考虑:

- 我们想尽可能少地复制消息数据。复制越多,性能越低。
- 我们希望将完整的消息存储在连续的字节序列中,使解析消息更容易。

#### 每个Message Reader的缓冲区

很显然部分消息需要存储某些缓冲区中。简单的实现方式可以是每一个Message Reader内部简单地有一个缓冲区。但是这个缓冲区应该多大？它要大到足够储存最大允许储存消息。因此,如果最大允许储存消息是1MB,那么Message Reader内部缓冲区将至少需要1MB。

当我们的链接达到百万数量级,每个链接都使用1MB并没有什么作用。1,000,000 * 1MB仍然是1TB的内存！那如果最大的消息是16MB甚至是128MB呢？
 
#### 大小可调的缓冲区

另一个选择是在Message Reader内部实现一个大小可调的缓冲区。大小可调的缓冲区开始的时候很小,如果它获取的消息过大,那缓冲区会扩大。这样每一条链接就不一定需要如1MB的缓冲区。每条链接的缓冲区只要需要足够储存下一条消息的内存就行了。

有几个可实现可调大小缓冲区的方法。它们都各自有自己的优缺点,所以接下来的部分我将逐个讨论。

**通过复制调整大小**

实现可调大小缓冲区的第一种方式是从一个大小(例如:4KB)的缓冲区开始。如果4KB的缓冲区装不下一个消息,则会分配一个更大的缓冲区(如:8KB),并将大小为4KB的缓冲区数据复制到这个更大的缓冲区中去。

通过复制实现大小可调缓冲区的优点在于消息的所有数据被保存在一个连续的字节数组中,这就使得消息的解析更加容易。它的缺点就是在复制更大消息的时候会导致大量的数据。

为了减少消息的复制,你可以分析流进你系统的消息的大小,并找出尽量减少复制量的缓冲区的大小。例如,你可能看到大多数消息都小于4KB,这是因为它们都仅包含很小的 request/responses。这意味着缓冲区的初始值应该设为4KB。

然后你可能有一个消息大于4KB,这通常是因为它里面包含一个文件。你可能注意到大多数流进系统的文件都是小于128KB的。这样第二个缓冲区的大小设置为128KB就较为合理。

最后你可能会发现一旦消息超过128KB之后,消息的大小就没有什么固定的模式,因此缓冲区最终的大小可能就是最大消息的大小。

根据流经系统的消息大小,上面三种缓冲区大小可以减少数据的复制。小于4KB的消息将不会复制。对于一百万个并发链接其结果是:1,000,000 * 4KB = 4GB,对于目前大多数服务器还是有可能的。介于4KB – 128KB的消息将只会复制一次,并且只有4KB的数据复制进128KB的缓冲区中。介于128KB至最大消息大小的消息将会复制两次。第一次复制4KB,第二次复制128KB,所以最大的消息总共复制了132KB。假设没有那么多超过128KB大小的消息那还是可以接受的。

一旦消息处理完毕,那么分配的内存将会被清空。这样在同一链接接收到的下一条消息将会再次从最小缓冲区大小开始算。这样做的必要性是确保了不同连接间内存的有效共享。所有的连接很有可能在同一时间并不需要打的缓冲区。

我有一篇介绍如何实现这样支持可调整大小的数组的内存缓冲区的完整文章:

Resizable Arrays(http://tutorials.jenkov.com/java-performance/resizable-array.html)

文章包含一个GitHub仓库连接,其中的代码演示了是如何实现的。

**通过追加调整大小**

调整缓冲区大小的另一种方法是使缓冲区由多个数组组成。当你需要调整缓冲区大小时,你只需要另一个字节数组并将数据写进去就行了。

这里有两种方法扩张一个缓冲区。一个方法是分配单独的字节数组,并将这些数组保存在一个列表中。另一个方法是分配较大的共享字节数组的片段,然后保留分配给缓冲区的片段的列表。就个人而言,我觉得片段的方式会好些,但是差别不大。

通过追加单独的数组或片段来扩展缓冲区的优点在于写入过程中不需要复制数据。所有的数据可以直接从socket (Channel)复制到一个数组或片段中。

以这种方式扩展缓冲区的缺点是在于数据不是存储在单独且连续的数组中。这将使得消息的解析更困难,因为解析器需要同时查找每个单独数组的结尾处和所有数组的结尾处。由于你需要在写入的数据中查找消息的结尾,所以该模型并不容易使用。

### TLV编码消息
一些协议消息格式是使用TLV格式(类型(Type)、长度(Length)、值(Value))编码。这意味着当消息到达时,消息的总长度被存储在消息的开头。这一方式你可以立即知道应该对整个消息分配多大的内存。

TLV编码使得内存管理变得更加容易。你可以立即知道要分配多大的内存给这个消息。只有部分在结束时使用的缓冲区才会使得内存浪费。

TLV编码的一个缺点是你要在消息的所有数据到达之前就分配好这个消息需要的所有内存。一些慢连接可能因此分配完你所有可用内存,从而使得你的服务器无法响应。

此问题的解决方法是使用包含多个TLV字段的消息格式。因此,服务器是为每个字段分配内存而不是为整个消息分配内存,并且是字段到达之后再分配内存。然而,一个大消息中的一个大字段在你的内存管理有同样的影响。

另外一个方案就是对于还未到达的信息设置超时时间,例如10-15秒。当恰好有许多大消息到达服务器时,这个方案能够使得你的服务器可以恢复,但是仍然会造成服务器一段时间无法响应。另外,恶意的DoS(Denial of Service拒绝服务)攻击仍然可以分配完你服务器的所有内存。

TLV编码存在许多不同的形式。实际使用的字节数、自定字段的类型和长度都依赖于每一个TLV编码。TLV编码首先放置字段的长度、然后是类型、然后是值(一个LTV编码)。 虽然字段的顺序不同,但它仍然是TLV的一种。

TLV编码使内存管理更容易这一事实,其实是HTTP 1.1是如此可怕的协议的原因之一。 这是他们试图在HTTP 2.0中修复数据的问题之一,数据在LTV编码帧中传输。 这也是为什么我们使用TLV编码的VStack.co project 设计了我们自己的网络协议。

### 写部分数据
在非阻塞IO管道中写数据仍然是一个挑战。当你调用一个处于非阻塞式Channel对象的write(ByteBuffer)方法时,ByteBuffer写入多少数据是无法保证的。write(ByteBuffer)方法会返回写入的字节数,因此可以跟踪写入的字节数。这就是挑战:跟踪部分写入的消息,以便最终可以发送一条消息的所有字节。

为了管理部分消息写入Channel,我们将创建一个消息写入器(Message Writer)。就像Message Reader一样,每一个要写入消息的Channel我们都需要一个Message Writer。在每个Message Writer中,我们跟踪正在写入的消息的字节数。

如果达到的消息量超过Message Writer可直接写入Channel的消息量,消息就需要在Message Writer排队。然后Message Writer尽快地将消息写入到Channel中。

下图是部分消息如何写入的设计图:
{% asset_img non-blocking-server-8.png %}

为了使Message Writer能够尽快发送数据,Message Writer需要能够不时被调用,这样就能发送更多的消息。

如果你又大量的连接那你将需要大量的Message Writer实例。检查Message Writer实例(如:一百万个)看写任何数据时是否缓慢。 首先,许多Message Writer实例都没有任何消息要发送,我们并不想检查那些Message Writer实例。其次,并不是所有的Channel实例都可以准备好写入数据。 我们不想浪费时间尝试将数据写入无法接受任何数据的Channel。

为了检查Channel是否准备好进行写入,您可以使用Selector注册Channel。然而我们并不想将所有的Channel实例注册到Selector中去。想象一下,如果你有1,000,000个连接且其中大多是空闲的,并且所有的连接已经与Selector注册。然后当你调用select()时,这些Channel实例的大部分将被写入就绪(它们大都是空闲的,记得吗？)然后你必须检查所有这些连接的Message Writer,以查看他们是否有任何数据要写入。

为了避免检查所有消息的Message Writer实例和所有不可能被写入任何信息的Channel实例,我们使用这两步的方法:

- 当一个消息被写入Message Writer,Message Writer向Selector注册其相关Channel(如果尚未注册)。
- 当你的服务器有时间时,它检查Selector以查看哪些注册的Channel实例已准备好进行写入。 对于每个写就绪Channel,请求其关联的Message Writer将数据写入Channel。 如果Message Writer将其所有消息写入其Channel,则Channel将再次从Selector注册。

这两个小步骤确保了有消息写入的Channel实际上已经被Selector注册了。

### 汇总
正如你所见,一个非阻塞式服务器需要时不时检查输入的消息来判断是否有任何的新的完整的消息发送过来。服务器可能会在一个或多个完整消息发来之前就检查了多次。检查一次是不够的。

同样,一个非阻塞式服务器需要时不时检查是否有任何数据需要写入。如果有,服务器需要检查是否有任何相应的连接准备好将该数据写入它们。只有在第一次排队消息时才检查是不够的,因为消息可能被部分写入。

所有这些非阻塞服务器最终都需要定期执行的三个“管道”(pipelines)::

读取管道(The read pipeline),用于检查是否有新数据从开放连接进来的。
处理管道(The process pipeline),用于所有任何完整消息。
写入管道(The write pipeline),用于检查是否可以将任何传出的消息写入任何打开的连接。
这三条管道在循环中重复执行。你可能可以稍微优化执行。例如,如果没有排队的消息可以跳过写入管道。 或者,如果我们没有收到新的,完整的消息,也许您可以跳过流程管道。

以下是说明完整服务器循环的图:

{% asset_img non-blocking-server-9.png %}

如果仍然发现这有点复杂,请记住查看GitHub资料库:https://github.com/jjenkov/java-nio-server

也许看到正在执行的代码可能会帮助你了解如何实现这一点。

### 服务器线程模型
GitHub资源库里面的非阻塞式服务器实现使用了两个线程的线程模式。第一个线程用来接收来自ServerSocketChannel的传入连接。第二个线程处理接受的连接,意思是读取消息,处理消息并将响应写回连接。这两个线程模型的图解如下:
{% asset_img non-blocking-server-10.png %}
上一节中说到的服务器循环处理是由处理线程(Processor Thread)执行。


## Java NIO DatagramChannel

Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议,所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

### 打开 DatagramChannel

下面是 DatagramChannel 的打开方式:
```java
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```
这个例子打开的 DatagramChannel可以在UDP端口9999上接收数据包。

### 接收数据

通过receive()方法从DatagramChannel接收数据,如:
```java
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```
receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据,多出的数据将被丢弃。

### 发送数据

通过send()方法从DatagramChannel发送数据,如:
```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```
这个例子发送一串字符到"jenkov.com"服务器的UDP端口80。 因为服务端并没有监控这个端口,所以什么也不会发生。也不会通知你发出的数据包是否已收到,因为UDP在数据传送方面没有任何保证。

### 连接到特定的地址

可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的,连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ,让其只能从特定地址收发数据。

这里有个例子:
```java
channel.connect(new InetSocketAddress("jenkov.com", 80));
```
当连接后,也可以使用read()和write()方法,就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子:
```java
int bytesRead = channel.read(buf);
int bytesWritten = channel.write(but);
```


## Pipe

Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道,从source通道读取。

这里是Pipe原理的图示:
{% asset_img pipe.bmp %}

### 创建管道

通过Pipe.open()方法打开管道。例如:
```java
Pipe pipe = Pipe.open();
```

### 向管道写数据

要向管道写数据,需要访问sink通道。像这样:
```java
Pipe.SinkChannel sinkChannel = pipe.sink();
```
通过调用SinkChannel的write()方法,将数据写入SinkChannel,像这样:
```java
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

### 从管道读取数据

从读取管道的数据,需要访问source通道,像这样:
```java
Pipe.SourceChannel sourceChannel = pipe.source();
```
调用source通道的read()方法来读取数据,像这样:
```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```
read()方法返回的int值会告诉我们多少字节被读进了缓冲区。


## Java NIO与IO

当学习了Java NIO和IO的API后,一个问题马上涌入脑海:

我应该何时使用IO,何时使用NIO呢？在本文中,我会尽量清晰地解析Java NIO和IO的差异、它们的使用场景,以及它们如何影响您的代码设计。

### Java NIO和IO的主要区别
下表总结了Java NIO和IO之间的主要差别,我会更详细地描述表中每部分的差异。

|IO|NIO|
|:---|:---|
|面向流|面向缓冲|
|阻塞IO|非阻塞IO|
|无|选择器|

#### 面向流与面向缓冲
Java NIO和IO之间第一个最大的区别是,IO是面向流的,NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节,直至读取所有字节,它们没有被缓存在任何地方。此外,它不能前后移动流中的数据。如果需要前后移动从流中读取的数据,需要先将它缓存到一个缓冲区。 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区,需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是,还需要检查是否该缓冲区中包含所有您需要处理的数据。而且,需确保当更多的数据读入缓冲区时,不要覆盖缓冲区里尚未处理的数据。

#### 阻塞与非阻塞IO
Java IO的各种流是阻塞的。这意味着,当一个线程调用read() 或 write()时,该线程被阻塞,直到有一些数据被读取,或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式,使一个线程从某通道发送请求读取数据,但是它仅能得到目前可用的数据,如果目前没有数据可用时,就什么都不会获取。而不是保持线程阻塞,所以直至数据变的可以读取之前,该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道,但不需要等待它完全写入,这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作,所以一个单独的线程现在可以管理多个输入和输出通道(channel)。

#### 选择器(Selectors)
Java NIO的选择器允许一个单独的线程来监视多个输入通道,你可以注册多个通道使用一个选择器,然后使用一个单独的线程来“选择”通道:这些通道里已经有可以处理的输入,或者选择已准备写入的通道。这种选择机制,使得一个单独的线程很容易来管理多个通道。

### NIO和IO如何影响应用程序的设计
无论您选择IO或NIO工具箱,可能会影响您应用程序设计的以下几个方面:

1. 对NIO或IO类的API调用。
2. 数据处理。
3. 用来处理数据的线程数。

### API调用
当然,使用NIO的API调用时看起来与使用IO时有所不同,但这并不意外,因为并不是仅从一个InputStream逐字节读取,而是数据必须先读入缓冲区再处理。

### 数据处理
使用纯粹的NIO设计相较IO设计,数据处理也受到影响。

在IO设计中,我们从InputStream或 Reader逐字节读取数据。假设你正在处理一基于行的文本数据流,例如:
```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```
该文本行的流可以这样处理:
```java
InputStream input = ... ; // get the InputStream from the client socket
BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```
请注意处理状态由程序执行多久决定。换句话说,一旦reader.readLine()方法返回,你就知道肯定文本行就已读完, readline()阻塞直到整行读完,这就是原因。你也知道此行包含名称；同样,第二个readline()调用返回的时候,你知道这行包含年龄等。 正如你可以看到,该处理程序仅在有新数据读入时运行,并知道每步的数据是什么。一旦正在运行的线程已处理过读入的某些数据,该线程不会再回退数据(大多如此)。下图也说明了这条原则:
{% asset_img nio-vs-io-1.png %}
(Java IO: 从一个阻塞的流中读数据) 而一个NIO的实现会有所不同,下面是一个简单的例子:
```java
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);
```
注意第二行,从通道读取字节到ByteBuffer。当这个方法调用返回时,你不知道你所需的所有数据是否在缓冲区内。你所知道的是,该缓冲区包含一些字节,这使得处理有点困难。
假设第一次 read(buffer)调用后,读入缓冲区的数据只有半行,例如,“Name:An”,你能处理数据吗？显然不能,需要等待,直到整行数据读入缓存,在此之前,对数据的任何处理毫无意义。

所以,你怎么知道是否该缓冲区包含足够的数据可以处理呢？好了,你不知道。发现的方法只能查看缓冲区中的数据。其结果是,在你知道所有数据都在缓冲区里之前,你必须检查几次缓冲区的数据。这不仅效率低下,而且可以使程序设计方案杂乱不堪。例如:
```java
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```
bufferFull()方法必须跟踪有多少数据读入缓冲区,并返回真或假,这取决于缓冲区是否已满。换句话说,如果缓冲区准备好被处理,那么表示缓冲区满了。

bufferFull()方法扫描缓冲区,但必须保持在bufferFull()方法被调用之前状态相同。如果没有,下一个读入缓冲区的数据可能无法读到正确的位置。这是不可能的,但却是需要注意的又一问题。

如果缓冲区已满,它可以被处理。如果它不满,并且在你的实际案例中有意义,你或许能处理其中的部分数据。但是许多情况下并非如此。下图展示了“缓冲区数据循环就绪”:
{% asset_img nio-vs-io-2.png %}
Java NIO:从一个通道里读数据,直到所有的数据都读到缓冲区里.

### 总结

NIO可让您只使用一个(或几个)单线程管理多个通道(网络连接或文件),但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。

如果需要管理同时打开的成千上万个连接,这些连接每次只是发送少量的数据,例如聊天服务器,实现NIO的服务器可能是一个优势。同样,如果你需要维持许多打开的连接到其他计算机上,如P2P网络中,使用一个单独的线程来管理你所有出站连接,可能是一个优势。一个线程多个连接的设计方案如下图所示:
{% asset_img nio-vs-io-3.png %}
Java NIO: 单线程管理多个连接

如果你有少量的连接使用非常高的带宽,一次发送大量的数据,也许典型的IO服务器实现可能非常契合。下图说明了一个典型的IO服务器设计:
{% asset_img nio-vs-io-4.png %}
Java IO: 一个典型的IO服务器设计- 一个连接通过一个线程处理.

未完待续...

ref:
[http://ifeve.com/java-nio-all/](http://ifeve.com/java-nio-all/)
