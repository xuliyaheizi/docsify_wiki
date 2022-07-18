# Java IO/NIO/AIO

## 一、Java IO分类

### 1.1、从传输方式上

从传输方式或者说是运算方式角度来看，可以将IO类分为：

- 字节流
- 字符流

流：代表任何有能力产出数据的数据源对象或者是有能力接受数据的接收端对象。**本质**是数据传输，根据传输特性将流抽象为各种类，方便更直观的进行数据操作。**作用**是为数据源和目的地建立一个输送通道。

#### 字节流

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/java-io-category-1.png"/>

#### 字符流

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/java-io-category-2.png"/>

### 1.2、从数据操作方式上

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/java-io-category-3.png"/>

### 1.3、IO流的分类？

- 按照流的流向分，可以分为`输入流`和`输出流`；
- 按照操作单元划分，可以划分为`字节流`和`字符流`；
- 按照流的角色划分为`节点流`和`处理流`。
- InputStream/Reader: 所有的输入流的基类，前者是`字节输入流`，后者是`字符输入流`。
- OutputStream/Writer: 所有输出流的基类，前者是`字节输出流`，后者是`字符输出流`。

### 1.4、既然有了字节流,为什么还要有字符流?

问题本质想问：**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

回答：字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

### 1.5、字节流与字符流之间的选择

- 大多数情况下使用字节流会更好，因为字节流是字符流的包装，大多数时候IO操作都是直接操作磁盘文件，所有这些流在传输时都是以字节的方式进行的。（图片等都是按字节存储的）
- 如果对于操作需要通过IO在内存中频繁处理字符串的情况使用字符流会好些。因为字符流具备缓冲区，提高性能。

## 二、设计模式（装饰者模式）

### 2.1、装饰者模式

装饰者(Decorator)和具体组件(ConcreteComponent)都继承自组件(Component)，具体组件的方法实现不需要依赖于其它对象，而装饰者组合了一个组件，这样它可以装饰其它装饰者或者具体组件。所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件的方法实现不需要依赖于其它对象。

### 2.2、IO装饰者模式

以`InputStream`为例：

- InputStream是抽象组件。
- FileInputStream是InputStream的子类，属于具体组件，提供了字节流的输入操作。
- FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220612192603156.png"/>

```java
//实例化一个具有缓存功能的字节流对象
FileInputStream fileInputStream = new FileInputStream(path);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

## IO模型

UNIX系统下，IO模型一共有5种：**同步阻塞I/O**、**同步非阻塞I/O**、**I/O多路复用**、**信号驱动I/O和异步I/O**。

### BIO（同步阻塞IO模型）

应用程序发起 read 调用后，会一直阻塞，直到内核把数据拷贝到用户空间。

在客户端连接数量不高的情况下，是没问题的。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

在 BIO 模式中，服务器会为每个客户端请求建立一个线程，由该线程单独负责处理一个客户请求，这种模式虽然简单方便，但由于服务器为每个客户端的连接都采用一个线程去处理，使得资源占用非常大。因此，当连接数量达到上限时，如果再有用户请求连接，直接会导致资源瓶颈，严重的可能会直接导致服务器崩溃。

为避免该情况，大多都采用了线程池模型。也就是创建一个固定大小的线程池，如果有客户端请求，就从线程池中取一个空闲线程来处理，当客户端处理完操作之后，就会释放对线程的占用。因此这样就避免为每一个客户端都要创建线程带来的资源浪费，使得线程可以重用。但线程池也有它的弊端，如果连接大多是长连接，可能会导致在一段时间内，线程池中的线程都被占用，那么当再有客户端请求连接时，由于没有空闲线程来处理，就会导致客户端连接失败。

### NIO（IO多路复用模型）

支持面向缓冲的，基于通道的I/O操作方法。对于高负载、高并发的（网络）应用。NIO 采用非阻塞模式，基于 Reactor 模式的工作方式，I/O 调用不会被阻塞，它的实现过程是：会先对每个客户端注册感兴趣的事件，然后有一个线程专门去轮询每个客户端是否有事件发生，当有事件发生时，便顺序处理每个事件，当所有事件处理完之后，便再转去继续轮询。

#### 同步非阻塞IO模型

同步非阻塞 IO 模型中，应用程序会一直发起 read 调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是阻塞的，直到在内核把数据拷贝到用户空间。

相比于同步阻塞 IO 模型，同步非阻塞 IO 模型确实有了很大改进。通过轮询操作，避免了一直阻塞。

但是，这种 IO 模型同样存在问题：**应用程序不断进行 I/O 系统调用轮询数据是否已经准备好的过程是十分消耗 CPU 资源的。**

#### IO多路复用模型

IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间 -> 用户空间）还是阻塞的。**IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗。**Java 中的 NIO ，有一个非常重要的**选择器 ( Selector )** 的概念，也可以被称为 **多路复用器**。通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务。

NIO 的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上面，一个选择器线程可以同时处理成千上万个连接，系统不必创建大量的线程，也不必维护这些线程，从而大大减小了系统的开销。

#### AIO(异步IO模型)

  异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

## 网络编程系列之NIO

JavaNIO由三个核心部分组成：**Channels，Buffers，Selectors**

### Channel

Channel是一个通道，可以通过它完成读取和写入数据，通道和流的不同之处在于通道是双向的，流只是在一个方向流动（一个流必须是InputStream或OutputStream的子类），而且通道可以用于读，写或者同时用于读写。因此Channel是全双工的，可以比流更好的映射底层操作系统的API。

所有的数据都通过Buffer对象来处理，永远不会将字节直接写入通道中，而是将数据写入包含一个或多个字节的缓冲区。读取数据的时候也是先将数据从通道读入缓冲区，再从缓冲区获取数据。

- FileChannel 从文件中读写数据。 
- DatagramChannel 能通过 UDP 读写网络中的数据。 
- SocketChannel 能通过 TCP 读写网络中的数据。 
- ServerSocketChannel 可以监听新进来的 TCP 连接，像 Web 服务器那样。对每一个新进来的连接都会创建一个 SocketChannel。

#### FileChannel

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220513003125241.png"/>

**读取数据**

```java
//创建FileChannel
RandomAccessFile accessFile = new RandomAccessFile("text/01.txt", "rw");
FileChannel channel = accessFile.getChannel();

//创建Buffer  从FileChannel读取数据
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
//读取数据到buffer中   -1表示读取文件的末尾
int bytesRead = channel.read(byteBuffer);
while (bytesRead != -1) {
    System.out.println("读取了：" + bytesRead);
    byteBuffer.flip();
    while (byteBuffer.hasRemaining()) {
        System.out.println((char) byteBuffer.get());
    }
    byteBuffer.clear();
    bytesRead = channel.read(byteBuffer);
}
accessFile.close();
System.out.println("操作结束了");
```

**写入数据**

```java
//打开FileChannel
RandomAccessFile accessFile = new RandomAccessFile("text/001.txt", "rw");
FileChannel channel = accessFile.getChannel();

//创建buffer对象
ByteBuffer buffer = ByteBuffer.allocate(1024);

String newData = "data zhulin";
buffer.clear();
//将数据写入buffer
buffer.put(newData.getBytes());
buffer.flip();
while (buffer.hasRemaining()) {
channel.write(buffer);
}
//关闭channel
channel.close();
```

**数据传输**

如果两个通道中有一个是 FileChannel，那你可以直接将数据从一个 channel 传输到另外一个 channel。

```java
RandomAccessFile aFile = new RandomAccessFile("text/01.txt", "rw");
FileChannel fromChannel = aFile.getChannel();

RandomAccessFile bFile = new RandomAccessFile("text/02.txt", "rw");
FileChannel toChannel = bFile.getChannel();

long position = 0;
long size = fromChannel.size();
toChannel.transferFrom(fromChannel, position, size);

aFile.close();
bFile.close();
System.out.println("系统结束");
```

#### SocketChannel

1. `ScoketChannel`就是NIO对于非阻塞socket操作的支持的组件，其在socket上封装了一层，主要是支持了非阻塞的读写。同时改进了传统的单向流API，Channel同时支持读写。
2. socket通道类主要分为`DatagramChannel`、`SocketChannel`和`ServerSocketChannel`。它们在实例化时都会创建一个对等socket对象，要把一个socket通道置于非阻塞模式，依靠所有socket通道类的公有超级类：`SelectableChannel`。就绪选择是一种可以用来查询通道的机制，该查询可以判断通道是否准备好执行一个目标操作，如读或写，非阻塞I/O和可选择性是紧密相连的，也正是管理阻塞模式的API代码要在SelectableChannel超级类中定义的原因。
3. 设置或重新设置一个通道的阻塞模式是很简单的，只要调用configureBlocking()方法即可，传递参数值为true则设为阻塞模式，参数值为false值为非阻塞模式。isBlocking()方法可判断某个socket通道当前处于哪种模式。

##### ServerSocketChannel

`ServerSocketChannel`是一个基于通道的socket监听器。与java.net.ServerSocket执行相同的任务，不过它增加了通道语义，因此能够在非阻塞模式下运行。

由于ServerSocketChannel没有bind()方法，因此有必要取出对等的socket并使用它来绑定到一个端口以开始监听连接。

**ServerSocketChannel非阻塞的accept()方法**

```java
public class ServerSocketChannelDemo1 {
    public static final String GREETING = "hello java nio";

    public static void main(String[] args) throws IOException, InterruptedException {
        int port = 8888;
        //buffer
        ByteBuffer buffer = ByteBuffer.wrap(GREETING.getBytes());
        //ServerSocketChannel
        ServerSocketChannel scc = ServerSocketChannel.open();
        scc.socket().bind(new InetSocketAddress(port));
        //设置非阻塞模式
        scc.configureBlocking(false);
        //一直监听连接传入
        while (true) {
            System.out.println("Waiting for connections");
            SocketChannel sc = scc.accept();
            if (sc == null) {
                System.out.println("null");
                Thread.sleep(2000);
            } else {
                System.out.println("Incoming connection from: " + sc.socket().getRemoteSocketAddress());
                buffer.rewind();
                sc.write(buffer);
                sc.close();
            }
        }
    }
}
```

##### SocketChannel

Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道，是一种面向流连接sockets套接字的可选择通道。

1. 对于已经存在的 socket 不能创建 SocketChannel 
2. SocketChannel 中提供的 open 接口创建的 Channel 并没有进行网络级联，需要使用 connect 接口连接到指定地址 
3. 未进行连接的 SocketChannle 执行 I/O 操作时，会抛出NotYetConnectedException 
4. SocketChannel 支持两种 I/O 模式：阻塞式和非阻塞式 
5. SocketChannel 支持异步关闭。如果 SocketChannel 在一个线程上 read 阻塞，另一个线程对该 SocketChannel 调用 shutdownInput，则读阻塞的线程将返回-1 表示没有读取任何数据；如果SocketChannel 在一个线程上 write 阻塞，另一个线程对该SocketChannel 调用 shutdownWrite，则写阻塞的线程将抛出AsynchronousCloseException 
6. SocketChannel 支持设定参数 
	- SO_SNDBUF 套接字发送缓冲区大小 
	- SO_RCVBUF 套接字接收缓冲区大小 
	- SO_KEEPALIVE 保活连接 
	- O_REUSEADDR 复用地址 
	- SO_LINGER 有数据传输时延缓关闭 Channel (只有在非阻塞模式下有用) 
	- TCP_NODELAY 禁用 Nagle 算法

```java
//创建SocketChannel
SocketChannel channel = SocketChannel.open(new InetSocketAddress("www.baidu.com", 80));
//两种创建SocketChannel的方式
SocketChannel channel1 = SocketChannel.open();
channel1.connect(new InetSocketAddress("www.baidu.com", 80));
//设置阻塞和非阻塞模式
channel.configureBlocking(false);

//读操作
ByteBuffer buffer = ByteBuffer.allocate(16);
channel.read(buffer);
channel.close();
System.out.println("read over");
```

##### DatagramChannel

正如 SocketChannel 对应 Socket，ServerSocketChannel 对应 ServerSocket，每一个 DatagramChannel 对象也有一个关联的 DatagramSocket 对象。正如SocketChannel 模拟连接导向的流协议（如 TCP/IP），DatagramChannel 则模拟包导向的无连接协议（如UDP/IP）。DatagramChannel 是无连接的，每个数据报（datagram）都是一个自包含的实体，拥有它自己的目的地址及不依赖其他数据报的数据负载。与面向流的的 socket 不同，DatagramChannel 可以发送单独的数据报给不同的目的地址。同样，DatagramChannel 对象也可以接收来自任意地址的数据包。 每个到达的数据报都含有关于它来自何处的信息（源地址）。

```java
	/**
     * 发送方法
     */
    @Test
    public void sendDatagram() throws IOException, InterruptedException {
        //打开Datagram
        DatagramChannel sendChannel = DatagramChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 9999);
        //发送
        while (true) {
            ByteBuffer buffer = ByteBuffer.wrap("发送zhulin信息".getBytes("UTF-8"));
            sendChannel.send(buffer, inetSocketAddress);
            System.out.println("已经完成发送");
            Thread.sleep(3000);
        }
    }

    /**
     * 接收方法
     */
    @Test
    public void receiveDatagram() throws IOException {
        //打开Datagram
        DatagramChannel receiveChannel = DatagramChannel.open();
        //接收地址
        InetSocketAddress receiveAddress = new InetSocketAddress(9999);
        //绑定
        receiveChannel.bind(receiveAddress);
        //buffer
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //接收
        while (true) {
            buffer.clear();
            //获取信息
            SocketAddress receive = receiveChannel.receive(buffer);
            buffer.flip();
            System.out.println(receive.toString());
            //重新编码
            System.out.println(Charset.forName("UTF-8").decode(buffer));
        }
    }
```

### Buffer

Java NIO 中的 Buffer 用于和 NIO 通道进行交互。数据是从通道读入缓冲区，从缓冲区写入到通道中的。

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成 NIO Buffer 对象，并提供了一组方法，用来方便的访问该块内存。缓冲区实际上是一个容器对象，更直接的说，其实就是一个数组，在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的； 在写入数据时，它也是写入到缓冲区中的；任何时候访问 NIO 中的数据，都是将它放到缓冲区中。而在面向流 I/O系统中，所有数据都是直接写入或者直接将数据读取到 Stream 对象中。

Buffer用法

```java
//打开FileChannel
RandomAccessFile aFile = new RandomAccessFile("text/01.txt", "rw");
FileChannel channel = aFile.getChannel();
//buffer 定义缓冲区大小
ByteBuffer buffer = ByteBuffer.allocate(1024);
//将channel数据写入buffer缓冲区冲
int read = channel.read(buffer);

while (read != -1) {
//转换read模式 flip()方法将 Buffer 从写模式切换到读模式
buffer.flip();
while (buffer.hasRemaining()) {
System.out.println((char) buffer.get());
}
//调用 clear()或 compact()方法。clear()方法会清空整个缓冲 区。compact()方法只会清除已经读过的数据。
buffer.clear();
//buffer.compact()
read = channel.read(buffer);
}
aFile.close();
```

#### Buffer 的 capacity、position 和 limit

position 和 limit 的含义取决于 Buffer 处在读模式还是写模式。不管 Buffer 处在什么模式，capacity 的含义总是一样的。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220514225853831.png"/>

**capacity**

作为一个内存块，Buffer 有一个固定的大小值，也叫capacity”.你只能往里写capacity 个 byte、long，char 等类型。一旦 Buffer 满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

**position**

- 写数据到 Buffer 中时，position 表示写入数据的当前位置，position 的初始值为0。当一个 byte、long 等数据写到 Buffer 后， position 会向下移动到下一个可插入数据的 Buffer 单元。position 最大可为 capacity – 1（因为 position 的初始值为 0）。
- 读数据到 Buffer 中时，position 表示读入数据的当前位置，如 position=2 时表示已开始读入了 3 个 byte，或从第 3 个 byte 开始读取。通过 ByteBuffer.flip()切换到读模式时 position 会被重置为 0，当 Buffer 从 position 读入数据后，position 会下移到下一个可读入的数据 Buffer 单元。

**limit**

- 写数据时，limit 表示可对 Buffer 最多写入多少个数据。写模式下，limit 等于Buffer 的 capacity。
- 读数据时，limit 表示 Buffer 里有多少可读数据（not null 的数据），因此能读到之前写入的所有数据（limit 被设置成已写数据的数量，这个值在写模式下就是position）。

#### clear()与 compact()方法

一旦读完 Buffer 中的数据，需要让 Buffer 准备好再次被写入。可以通过 **clear**()或**compact**()方法来完成。如果调用的是 clear()方法，position 将被设回 0，limit 被设置成 capacity 的值。换句话说，Buffer 被清空了。**Buffer 中的数据并未清除**，只是这些标记告诉我们可以从哪里开始往 Buffer 里写数据。如果 Buffer 中有一些未读的数据，调用 clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果 Buffer 中仍有未读的数据，且后续还需要这些数据，但是此时想要先写些数据，那么使用 **compact**()方法。compact()方法将所有未读的数据拷贝到 Buffer 起始处。然后将 **position** 设到最后一个未读元素正后面。limit 属性依然像 clear()方法一样，设置成 capacity。现在Buffer 准备好写数据了，**但是不会覆盖未读的数据**。

#### 缓冲区分片

在 NIO 中，除了可以分配或者包装一个缓冲区对象外，还可以根据现有的缓冲区对象来创建一个子缓冲区，即在现有缓冲区上切出一片来作为一个新的缓冲区，但现有的缓冲区与创建的子缓冲区在底层数组层面上是数据共享的，也就是说，子缓冲区相当于是现有缓冲区的一个视图窗口。调用 slice()方法可以创建一个子缓冲区。 

#### 只读缓冲区

通过调用缓冲区的 `asReadOnlyBuffer()`方法，将任何常规缓冲区转换为只读缓冲区，这个方法返回一个与原缓冲区**完全相同**的缓冲区，并与原缓冲区共享数据，只不过它是只读的。如果原缓冲区的内容发生了变化，只读缓冲区的内容也随之发生变化。

#### 直接缓冲区

直接缓冲区是为**加快** I/O 速度，使用一种特殊方式为其分配内存的缓冲区，JDK 文档中的描述为：给定一个直接字节缓冲区，Java 虚拟机将尽最大努力直接对它执行本机I/O 操作。也就是说，它会在每一次调用底层操作系统的本机 I/O 操作之前(或之后)，尝试避免将缓冲区的内容拷贝到一个中间缓冲区中 或者从一个中间缓冲区中拷贝数据。要分配直接缓冲区，需要调用 `allocateDirect()`方法，而不是 `allocate()`方法，使用方式与普通缓冲区并无区别。

#### 内存映射文件 I/O

内存映射文件 I/O 是一种**读和写**文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快的多。内存映射文件 I/O 是通过使文件中的数据出现为 内存数组的内容来完成的，这其初听起来似乎不过就是将整个文件读到内存中，但是事实上并不是这样。一般来说，只有文件中实际读取或者写入的部分才会映射到内存中。

```java
RandomAccessFile aFile = new RandomAccessFile("text/01.txt", "rw");
FileChannel channel = aFile.getChannel();
MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);

mappedByteBuffer.put(0, (byte) 97);
mappedByteBuffer.put(1023, (byte) 122);
while (mappedByteBuffer.hasRemaining()) {
System.out.println(mappedByteBuffer.get());
}
aFile.close();
```

### Selector

Selector 一般称 为选择器 ，也可以翻译为 多路复用器 。它是 Java NIO 核心组件中的一个，用于检查一个或多个 NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个 channels,也就是可以管理多个网络链接。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220517201535405.png"/>

  使用 Selector 的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

#### 可选择通道(SelectableChannel)

1. 不是所有的 Channel 都可以被 Selector 复用的。比方说，FileChannel 就不能被选择器复用。判断一个 Channel 能被 Selector 复用，有一个前提：判断他是否继承了一个抽象类 SelectableChannel。如果继承了 SelectableChannel，则可以被复用，否则不能。
2. SelectableChannel 类提供了实现通道的可选择性所需要的公共方法。它是所有支持就绪检查的通道类的父类。所有 socket 通道，都继承了 SelectableChannel 类都是可选择的，包括从管道(Pipe)对象的中获得的通道。而 FileChannel 类，没有继承SelectableChannel，因此是不是可选通道。
3. 一个通道可以被注册到多个选择器上，但对每个选择器而言只能被注册一次。通道和选择器之间的关系，使用注册的方式完成。SelectableChannel 可以被注册到Selector 对象上，在注册的时候，需要指定通道的哪些操作，是 Selector 感兴趣的。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220517201851267.png"/>

#### Channel 注册到 Selector

1. 使用 Channel.register（Selector sel，int ops）方法，将一个通道注册到一个选择器时。第一个参数，指定通道要注册的选择器。第二个参数指定选择器需要查询的通道操作。
2. 选择器查询的不是通道的操作，而是通道的某个操作的一种就绪状态。什么是操作的就绪状态？一旦通道具备完成某个操作的条件，表示该通道的某个操作已经就绪，就可以被 Selector 查询到，程序可以对通道进行对应的操作。比方说，某个SocketChannel 通道可以连接到一个服务器，则处于“连接就绪”(OP_CONNECT)。再比方说，一个 ServerSocketChannel 服务器通道准备好接收新进入的连接，则处于“接收就绪”（OP_ACCEPT）状态。还比方说，一个有数据可读的通道，可以说是“读就绪”(OP_READ)。一个等待写数据的通道可以说是“写就绪”(OP_WRITE)。

#### 选择键(SelectionKey)

1. Channel 注册到后，并且一旦通道处于某种就绪的状态，就可以被选择器查询到。这个工作，使用选择器 Selector 的 select（）方法完成。select 方法的作用，对感兴趣的通道操作，进行就绪状态的查询。
2. Selector 可以不断的查询 Channel 中发生的操作的就绪状态。并且挑选感兴趣的操作就绪状态。一旦通道有操作的就绪状态达成，并且是 Selector 感兴趣的操作，就会被 Selector 选中，放入选择键集合中。
3. 一个选择键，首先是包含了注册在 Selector 的通道操作的类型，比方说SelectionKey.OP_READ。也包含了特定的通道与特定的选择器之间的注册关系。开发应用程序是，选择键是编程的关键。NIO 的编程，就是根据对应的选择键，进行不同的业务逻辑处理。
4. 选择键的概念，和事件的概念比较相似。一个选择键类似监听器模式里边的一个事件。由于 Selector 不是事件触发的模式，而是主动去查询的模式，所以不叫事件Event，而是叫 SelectionKey 选择键。

  与 Selector 一起使用时，Channel 必须处于非阻塞模式下，否则将抛出异常IllegalBlockingModeException。这意味着，FileChannel 不能与 Selector 一起使用，因为 FileChannel 不能切换到非阻塞模式，而套接字相关的所有的通道都可以。

  一个通道，并没有一定要支持所有的四种操作。比如服务器通道ServerSocketChannel 支持 Accept 接受操作，而 SocketChannel 客户端通道则不支持。可以通过通道上的 validOps()方法，来获取特定通道下所有支持的操作集合。

#### NIO 编程步骤

- 第一步：创建 Selector 选择器 
- 第二步：创建 ServerSocketChannel 通道，并绑定监听端口 
- 第三步：设置 Channel 通道是非阻塞模式 
- 第四步：把 Channel 注册到 Socketor 选择器上，监听连接事件 
- 第五步：调用 Selector 的 select 方法（循环调用），监测通道的就绪状况 
- 第六步：调用 selectKeys 方法获取就绪 channel 集合 
- 第七步：遍历就绪 channel 集合，判断就绪事件类型，实现具体的业务操作 
- 第八步：根据业务，决定是否需要再次注册监听事件，重复执行第三步操作

#### 服务端代码

```java
public static void serverDemo() throws IOException {
        //1.获取服务端通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //2.切换非阻塞模式
        serverSocketChannel.configureBlocking(false);

        //4.绑定端口号
        InetSocketAddress inetSocketAddress = new InetSocketAddress(8000);
        serverSocketChannel.bind(inetSocketAddress);
        //5.获取selector选择器
        Selector selector = Selector.open();
        //6.通道注册到选择器，进行监听
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        //7.选择器进行轮询，后续操作
        while (selector.select() > 0) {
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                //获取就绪操作
                SelectionKey key = iterator.next();
                //判断什么操作
                if (key.isAcceptable()) {
                    //获取连接
                    SocketChannel accept = serverSocketChannel.accept();
                    //切换非阻塞模式
                    accept.configureBlocking(false);
                    //注册
                    accept.register(selector, SelectionKey.OP_READ);
                    System.out.println("连接状态-------");
                } else if (key.isReadable()) {
                    //读状态
                    System.out.println("读取状态------");
                    SocketChannel channel = (SocketChannel) key.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    //读取数据
                    int length = 0;
                    while ((length = channel.read(byteBuffer)) > 0) {
                        byteBuffer.flip();
                        System.out.println(new String(byteBuffer.array(), 0, length));
                        byteBuffer.clear();
                    }
                }
            }
            iterator.remove();
        }
    }
```

#### 客户端代码

```java
public static void clientDemo() throws IOException {
        //1.获取通道，绑定主机和端口号
        SocketChannel socketChannel = SocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 8000);
        socketChannel.connect(inetSocketAddress);
        //2.切换到非阻塞模式
        socketChannel.configureBlocking(false);
        //3.创建buffer
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //4.写入buffer数据
        System.out.println("客户端输入数据：");
        buffer.put("祝林".getBytes());
        //5.模式切换
        buffer.flip();
        //6.写入通道
        socketChannel.write(buffer);
        //7.清空，关闭
    
        buffer.clear();
        socketChannel.close();
    }
```

### Pipe

Java NIO 管道是 2 个线程之间的单向数据连接。Pipe 有一个 source 通道和一个 sink通道。数据会被写到 sink 通道，从 source 通道读取。

<img src="https://knowledgeimagebed.oss-cn-hangzhou.aliyuncs.com/img/image-20220520152958439.png"/>

**创建管道**

通过 Pipe.open()方法打开管道。 

```java
Pipe pipe = Pipe.open();
```

**写入管道**

要向管道写数据，需要访问 sink 通道。： 

```java
Pipe.SinkChannel sinkChannel = pipe.sink();

通过调用 SinkChannel 的 write()方法，将数据写入 SinkChannel： 

String newData = "New String to write to file..." + System.currentTimeMillis(); 

ByteBuffer buf = ByteBuffer.allocate(48); 

buf.clear(); 

buf.put(newData.getBytes()); 

buf.flip(); 

while(buf.hasRemaining()) { 

sinkChannel.write(buf); 

}
```

**从管道读取数据**

从读取管道的数据，需要访问 source 通道，像这样： 

```java
Pipe.SourceChannel sourceChannel = pipe.source(); 
```

调用 source 通道的 read()方法来读取数据： 

```java
ByteBuffer buf = ByteBuffer.allocate(48); 

int bytesRead = sourceChannel.read(buf); 
```

read()方法返回的 int 值会告诉我们多少字节被读进了缓冲区。

### FileLock

文件锁在 OS 中很常见，如果多个程序同时访问、修改同一个文件，很容易因为文件数据不同步而出现问题。给文件加一个锁，同一时间，只能有一个程序修改此文件，或者程序都只能读此文件，这就解决了同步问题。文件锁是**进程级别**的，不是**线程级别**的。文件锁可以解决多个进程并发访问、修改同一个文件的问题，但**不能解决多线程并发访问、修改同一文件的问题**。使用文件锁时，**同一进程内的多个线程，可以同时访问、修改此文件。**

文件锁是当前程序所属的 JVM 实例持有的，一旦获取到文件锁（对文件加锁），要调用 release()，或者关闭对应的 FileChannel 对象，或者当前 JVM 退出，才会释放这个锁。一旦某个进程（比如说 JVM 实例）对某个文件加锁，则在释放这个锁之前，此进程不能再对此文件加锁，就是说 JVM 实例在同一文件上的文件锁是不重叠的（进程级别不能重复在同一文件上获取锁）。

- 排它锁：又叫独占锁。对文件加排它锁后，该进程可以对此文件进行读写，该进程独占此文件，其他进程不能读写此文件，直到该进程释放文件锁。 
- 共享锁：某个进程对文件加共享锁，其他进程也可以访问此文件，但这些进程都只能读此文件，不能写。线程是安全的。只要还有一个进程持有共享锁，此文件就只能读， 不能写。 

#### 获取文件锁的方法

**有 4 种获取文件锁的方法：** 

- lock() //对整个文件加锁，默认为排它锁。 

- lock(long position, long size, booean shared) //自定义加锁方式。前 2 个参数指定要加锁的部分（可以只对此文件的部分内容加锁），第三个参数值指定是否是共享锁。 

- tryLock() //对整个文件加锁，默认为排它锁。 

- tryLock(long position, long size, booean shared)    //自定义加锁方式。 


如果指定为共享锁，则其它进程可读此文件，所有进程均不能写此文件，如果某进程试图对此文件进行写操作，会抛出异常

#### lock与tryLock的区别

- lock 是阻塞式的，如果未获取到文件锁，会一直阻塞当前线程，直到获取文件锁 
- tryLock 和 lock 的作用相同，只不过 tryLock 是非阻塞式的，tryLock 是尝试获取文件锁，获取成功就返回锁对象，否则返回 null，不会阻塞当前线程。

```java
public static void main(String[] args) throws IOException {
        String input = "zhulin";
        ByteBuffer buffer = ByteBuffer.wrap(input.getBytes());
        String filePath = "text/01.txt";
        Path path = Paths.get(filePath);
        FileChannel channel = FileChannel.open(path, StandardOpenOption.WRITE, StandardOpenOption.APPEND);
        channel.position(channel.size() - 1);
        //加锁  false为排它锁  true为共享锁
        FileLock lock = channel.lock(0L, Long.MAX_VALUE, true);
        //是否是一个共享锁
        System.out.println("是否是一个共享锁:" + lock.isShared());
        channel.write(buffer);
        channel.close();
        //读文件
        readFile(filePath);
    }

    private static void readFile(String path) throws IOException {
        FileReader filereader = new FileReader(path);
        //缓冲区
        BufferedReader bufferedreader = new BufferedReader(filereader);
        String tr = bufferedreader.readLine();
        System.out.println("读取内容: ");
        while (tr != null) {
            System.out.println(" " + tr);
            tr = bufferedreader.readLine();
        }
        filereader.close();
        bufferedreader.close();
    }
```

## 聊天室

### 服务端

```java
public class ChatServer {

    /**
     * 服务器启动
     */
    public void startServer() throws IOException {
        //1.创建Selector选择器
        Selector selector = Selector.open();
        //2.创建ServerSocketChannel通道
        ServerSocketChannel socketChannel = ServerSocketChannel.open();
        //3.为channel通道绑定端口
        socketChannel.bind(new InetSocketAddress(8000));
        //设置非阻塞模式
        socketChannel.configureBlocking(false);
        //4.循环，等待有新地连接接入
        socketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务器已经启动成功了");
        //循环
        for (; ; ) {
            //获取channel通道数量
            int readChannels = selector.select();
            if (readChannels == 0) {
                continue;
            }
            //获取可用的channel
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            //遍历集合
            Iterator<SelectionKey> selectionKeyIterator = selectionKeys.iterator();
            while (selectionKeyIterator.hasNext()) {
                SelectionKey selectionKey = selectionKeyIterator.next();
                //移除set集合当前selectionKey
                selectionKeyIterator.remove();

                if (selectionKey.isAcceptable()) {
                    //TODO:接收状态
                    acceptOperator(socketChannel, selector);
                }

                if (selectionKey.isReadable()) {
                    //TODO:可读状态
                    readOperator(selector, selectionKey);
                }
            }
        }
    }

    /**
     * 可读状态
     * @param selector
     * @param selectionKey
     */
    private void readOperator(Selector selector, SelectionKey selectionKey) throws IOException {
        //从selectionKey获取到已经就绪的通道
        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
        //创建buffer
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //循环读取客户端消息
        int readLength = socketChannel.read(buffer);
        String message = "";
        if (readLength > 0) {
            //切换读模式
            buffer.flip();
            //读取内容
            message += Charset.forName("UTF-8").decode(buffer);
        }
        //channel注册到selector选择器
        socketChannel.register(selector, SelectionKey.OP_READ);
        //把客户端发送的消息，广播到其他客户端
        if (message.length() > 0) {
            //广播给其他客户端
            System.out.println(message);
            castOrtherClient(message, selector, socketChannel);
        }
    }

    /**
     * 广播给其他客户端
     * @param message
     * @param selector
     * @param channel
     */
    private void castOrtherClient(String message, Selector selector, SocketChannel channel) throws IOException {
        //获取所有已经接入的客户端
        Set<SelectionKey> keys = selector.keys();
        //循环所有广播消息
        for (SelectionKey key : keys) {
            //获取每个channel
            Channel tarChannel = key.channel();
            //不需要给自己发送
            if (tarChannel instanceof SocketChannel && tarChannel != channel) {
                ((SocketChannel) tarChannel).write(Charset.forName("UTF-8").encode(message));
            }
        }
    }

    /**
     * 接收状态
     * @param socketChannel
     * @param selector
     */
    private void acceptOperator(ServerSocketChannel socketChannel, Selector selector) throws IOException {
        //接入状态  创建socketChannel
        SocketChannel accept = socketChannel.accept();
        //socketChannel设置非阻塞模式
        accept.configureBlocking(false);
        //把channel注册到选择器  监听可读状态
        accept.register(selector, SelectionKey.OP_READ);
        //客户端回复消息
        accept.write(Charset.forName("UTF-8").encode("欢迎您进入聊天室，请注意隐私"));
    }
}
```

### 客户端

```java
public class ChatClient {

    /**
     * 启动客户端
     */
    public void startClient(String name) throws IOException {
        //连接服务器端
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8000));

        //接收服务端响应数据
        Selector selector = Selector.open();
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);
        //创建线程
        new Thread(new ClientThread(selector)).start();

        //向服务端发送消息
        Scanner sc = new Scanner(System.in);
        while (sc.hasNextLine()) {
            String msg = name + "：" + sc.nextLine();
            if (msg.length() > 0) {
                socketChannel.write(Charset.forName("UTF-8").encode(msg));
            }
        }
    }
}
```

### 客户端线程

```java
public class ClientThread implements Runnable {

    private Selector selector;

    public ClientThread(Selector selector) {
        this.selector = selector;
    }

    @Override
    public void run() {
        try {
            for (; ; ) {
                //获取channel通道数量
                int readChannels = selector.select();

                if (readChannels == 0) {
                    continue;
                }
                //获取可用的channel
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                //便利集合
                Iterator<SelectionKey> selectionKeyIterator = selectionKeys.iterator();
                while (selectionKeyIterator.hasNext()) {
                    SelectionKey selectionKey = selectionKeyIterator.next();
                    //移除set集合当前seletionKey
                    selectionKeyIterator.remove();

                    //可读状态
                    if (selectionKey.isReadable()) {
                        //TODO:可读状态
                        readOperator(selector, selectionKey);
                    }
                }
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 可读状态
     * @param selector
     * @param selectionKey
     */
    private void readOperator(Selector selector, SelectionKey selectionKey) throws IOException {
        //从selectionKey获取到已经就绪的通道
        SocketChannel channel = (SocketChannel) selectionKey.channel();
        //创建buffer
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //循环读取客户端消息
        int readLength = channel.read(buffer);
        String message = "";
        if (readLength > 0) {
            //切换读模式
            buffer.flip();
            //读取内容
            message += Charset.forName("UTF-8").decode(buffer);
        }
        //channel注册到selector选择器
        channel.register(selector, SelectionKey.OP_READ);
        if (message.length() > 0) {
            //广播给其他客户端
            System.out.println(message);
        }
    }
}
```

