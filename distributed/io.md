### 1、基础概念
---
#### 1.1、缓冲区操作

缓冲区及操作是所有I/O的基础，进程执行I/O操作，归结起来就是向操作系统发出请求，让它要么把缓冲区里的数据排干（写），要么把缓冲区填满（读）。如下图
![](\\image\\455589f44a7d459c13a3782be5c8173734880bb1.jpg)

#### 1.2、内核空间、用户空间

上图简单描述了数据从磁盘到用户进程的内存区域移动的过程，其间涉及到了内核空间与用户空间。这两个空间有什么区别呢？ 

用户空间就是常规进程（如JVM）所在区域，用户空间是非特权区域，如不能直接访问硬件设备。内核空间是操作系统所在区域，那肯定是有特权啦，如能与设备控制器通讯，控制用户区域的进程运行状态。进程执行I/O操作时，它执行一个系统调用把控制权交由内核。 

### 2、java I/O基础
---
Java 的 I/O 操作类在包 java.io 下，大概有将近 80 个类，但是这些类大概可以分成四组，分别是：

- 基于字节操作的 I/O 接口：InputStream 和 OutputStream
- 基于字符操作的 I/O 接口：Writer 和 Reader
- 基于磁盘操作的 I/O 接口：File
- 基于网络操作的 I/O 接口：Socket

#### 2.1 字节与字符的转化接口
InputStreamReader 类是字节到字符的转化桥梁，InputStream到Reader的过程要指定编码字符集，否则将采用操作系统默认字符集，很可能会出现乱码问题。StreamDecoder 正是完成字节到字符的解码的实现类，FileReader是继承了InputStreamReader类

#### 2.2 磁盘 I/O 工作机制
下面介绍下如何从磁盘读取一段文本字符。如下图所示：
![](\\image\\image015.jpg)

当传入一个文件路径，将会根据这个路径创建一个 File 对象来标识这个文件，然后将会根据这个 File 对象创建真正读取文件的操作对象，这时将会真正创建一个关联真实存在的磁盘文件的文件描述符 FileDescriptor，通过这个对象可以直接控制这个磁盘文件。由于我们需要读取的是字符格式，所以需要 StreamDecoder 类将 byte 解码为 char 格式，至于如何从磁盘驱动器上读取一段数据，由操作系统帮我们完成。至于操作系统是如何将数据持久化到磁盘以及如何建立数据结构需要根据当前操作系统使用何种文件系统来回答，至于文件系统的相关细节可以参考另外的文章。

#### 2.3 Java Socket的工作机制
下图是典型的基于 Socket 的通信的场景：
![](\\image\\image017.jpg)

主机A的应用程序要能和主机B的应用程序通信，必须通过 Socket 建立连接，而建立 Socket 连接必须需要底层 TCP/IP 协议来建立 TCP 连接。建立 TCP 连接需要底层 IP 协议来寻址网络中的主机。我们知道网络层使用的 IP 协议可以帮助我们根据 IP 地址来找到目标主机，但是一台主机上可能运行着多个应用程序，如何才能与指定的应用程序通信就要通过 TCP 或 UPD 的地址也就是端口号来指定。这样就可以通过一个 Socket 实例唯一代表一个主机上的一个应用程序的通信链路了。

- 建立通信链路

当客户端要与服务端通信，客户端首先要创建一个 Socket 实例，操作系统将为这个 Socket 实例分配一个没有被使用的本地端口号，并创建一个包含本地和远程地址和端口号的套接字数据结构，这个数据结构将一直保存在系统中直到这个连接关闭。在创建 Socket 实例的构造函数正确返回之前，将要进行 TCP 的三次握手协议，TCP 握手协议完成后，Socket 实例对象将创建完成，否则将抛出 IOException 错误。
与之对应的服务端将创建一个 ServerSocket 实例，ServerSocket 创建比较简单只要指定的端口号没有被占用，一般实例创建都会成功，同时操作系统也会为 ServerSocket 实例创建一个底层数据结构，这个数据结构中包含指定监听的端口号和包含监听地址的通配符，通常情况下都是“*”即监听所有地址。之后当调用 accept() 方法时，将进入阻塞状态，等待客户端的请求。当一个新的请求到来时，将为这个连接创建一个新的套接字数据结构，该套接字数据的信息包含的地址和端口信息正是请求源地址和端口。这个新创建的数据结构将会关联到 ServerSocket 实例的一个未完成的连接数据结构列表中，注意这时服务端与之对应的 Socket 实例并没有完成创建，而要等到与客户端的三次握手完成后，这个服务端的 Socket 实例才会返回，并将这个 Socket 实例对应的数据结构从未完成列表中移到已完成列表中。所以 ServerSocket 所关联的列表中每个数据结构，都代表与一个客户端的建立的 TCP 连接。

- 数据传输

传输数据是我们建立连接的主要目的，如何通过 Socket 传输数据，下面将详细介绍。
当连接已经建立成功，服务端和客户端都会拥有一个 Socket 实例，每个 Socket 实例都有一个 InputStream 和 OutputStream，正是通过这两个对象来交换数据。同时我们也知道网络 I/O 都是以字节流传输的。当 Socket 对象创建时，操作系统将会为 InputStream 和 OutputStream 分别分配一定大小的缓冲区，数据的写入和读取都是通过这个缓存区完成的。写入端将数据写到 OutputStream 对应的 SendQ 队列中，当队列填满时，数据将被发送到另一端 InputStream 的 RecvQ 队列中，如果这时 RecvQ 已经满了，那么 OutputStream 的 write 方法将会阻塞直到 RecvQ 队列有足够的空间容纳 SendQ 发送的数据。值得特别注意的是，这个缓存区的大小以及写入端的速度和读取端的速度非常影响这个连接的数据传输效率，由于可能会发生阻塞，所以网络 I/O 与磁盘 I/O 在数据的写入和读取还要有一个协调的过程，如果两边同时传送数据时可能会产生死锁，在后面 NIO 部分将介绍避免这种情况。

###3、Java I/O模型
---
####3.1、同步 vs. 异步
> 同步I/O　每个请求必须逐个地被处理，一个请求的处理会导致整个流程的暂时等待，这些事件无法并发地执行。用户线程发起I/O请求后需要等待或者轮询内核I/O操作完成后才能继续执行。

> 异步I/O　多个请求可以并发地执行，一个请求或者任务的执行不会导致整个流程的暂时等待。用户线程发起I/O请求后仍然继续执行，当内核I/O操作完成后会通知用户线程，或者调用用户线程注册的回调函数。

####3.2、阻塞 vs. 非阻塞
> 阻塞　某个请求发出后，由于该请求操作需要的条件不满足，请求操作一直阻塞，不会返回，直到条件满足。

> 非阻塞　请求发出后，若该请求需要的条件不满足，则立即返回一个标志信息告知条件不满足，而不会一直等待。一般需要通过循环判断请求条件是否满足来获取请求结果。

需要注意的是，阻塞并不等价于同步，而非阻塞并非等价于异步。事实上这两组概念描述的是I/O模型中的两个不同维度。

同步和异步着重点在于多个任务执行过程中，后发起的任务是否必须等先发起的任务完成之后再进行。而不管先发起的任务请求是阻塞等待完成，还是立即返回通过循环等待请求成功。

而阻塞和非阻塞重点在于请求的方法是否立即返回（或者说是否在条件不满足时被阻塞）。

### 4、Unix五种I/O模型
---
说起I/O模型，网络上有一个错误的概念，异步非阻塞/阻塞模型，其实异步根本就没有阻不阻塞之说，异步模型就是异步模型。让我们来看一看Richard Stevens在其UNIX网络编程卷1中提出的5个I/O模型吧。

- 阻塞式I/O
- 非阻塞式I/O
- I/O 多路复用（select和poll） 
- 信号驱动 I/O（SIGIO）
- 异步 I/O（Posix.1的aio_系列函数）

####4.1、阻塞I/O
如上文所述，阻塞I/O下请求无法立即完成则保持阻塞。阻塞I/O分为如下两个阶段。

- 阶段1：等待数据就绪。网络 I/O 的情况就是等待远端数据陆续抵达；磁盘I/O的情况就是等待磁盘数据从磁盘上读取到内核态内存中。
- 阶段2：数据拷贝。出于系统安全，用户态的程序没有权限直接读取内核态内存，因此内核负责把内核态内存中的数据拷贝一份到用户态内存中。

####4.2、非阻塞I/O
非阻塞I/O请求包含如下三个阶段

- socket设置为 NONBLOCK（非阻塞）就是告诉内核，当所请求的I/O操作无法完成时，不要将线程睡眠，而是返回一个错误码(EWOULDBLOCK) ，这样请求就不会阻塞。
- I/O操作函数将不断的测试数据是否已经准备好，如果没有准备好，继续测试，直到数据准备好为止。整个I/O 请求的过程中，虽然用户线程每次发起I/O请求后可以立即返回，但是为了等到数据，仍需要不断地轮询、重复请求，消耗了大量的 CPU 的资源。
- 数据准备好了，从内核拷贝到用户空间。

一般很少直接使用这种模型，而是在其他I/O模型中使用非阻塞I/O 这一特性。这种方式对单个I/O 请求意义不大，但给I/O多路复用提供了条件。

####4.3、I/O多路复用（异步阻塞 I/O）
I/O多路复用会用到select或者poll函数，这两个函数也会使线程阻塞，但是和阻塞I/O所不同的是，这两个函数可以同时阻塞多个I/O操作。而且可以同时对多个读操作，多个写操作的I/O函数进行检测，直到有数据可读或可写时，才真正调用I/O操作函数。

从流程上来看，使用select函数进行I/O请求和同步阻塞模型没有太大的区别，甚至还多了添加监视Channel，以及调用select函数的额外操作，增加了额外工作。但是，使用 select以后最大的优势是用户可以在一个线程内同时处理多个Channel的I/O请求。用户可以注册多个Channel，然后不断地调用select读取被激活的Channel，即可达到在同一个线程内同时处理多个I/O请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

调用select/poll该方法由一个用户态线程负责轮询多个Channel，直到某个阶段1的数据就绪，再通知实际的用户线程执行阶段2的拷贝。 通过一个专职的用户态线程执行非阻塞I/O轮询，模拟实现了阶段一的异步化。

####4.4、信号驱动I/O（SIGIO）
首先我们允许socket进行信号驱动I/O，并安装一个信号处理函数，线程继续运行并不阻塞。当数据准备好时，线程会收到一个SIGIO 信号，可以在信号处理函数中调用I/O操作函数处理数据。

####4.5、异步I/O
调用aio_read 函数，告诉内核描述字，缓冲区指针，缓冲区大小，文件偏移以及通知的方式，然后立即返回。当内核将数据拷贝到缓冲区后，再通知应用程序。所以异步I/O模式下，阶段1和阶段2全部由内核完成，完成不需要用户线程的参与。

> 几种I/O模型对比

除异步I/O外，其它四种模型的阶段2基本相同，都是从内核态拷贝数据到用户态。区别在于阶段1不同。前四种都属于同步I/O。

###5、Java四种I/O模型
---
上一章所述Unix中的五种I/O模型，除信号驱动I/O外，Java对其它四种I/O模型都有所支持。其中Java最早提供的blocking I/O即是阻塞I/O，而NIO即是非阻塞I/O，同时通过NIO实现的Reactor模式即是I/O复用模型的实现，通过AIO实现的Proactor模式即是异步I/O模型的实现。

####5.1、从IO到NIO
*面向流 vs. 面向缓冲*
Java IO是面向流的，每次从流（InputStream/OutputStream）中读一个或多个字节，直到读取完所有字节，它们没有被缓存在任何地方。另外，它不能前后移动流中的数据，如需前后移动处理，需要先将其缓存至一个缓冲区。

Java NIO面向缓冲，数据会被读取到一个缓冲区，需要时可以在缓冲区中前后移动处理，这增加了处理过程的灵活性。但与此同时在处理缓冲区前需要检查该缓冲区中是否包含有所需要处理的数据，并需要确保更多数据读入缓冲区时，不会覆盖缓冲区内尚未处理的数据。

*阻塞 vs. 非阻塞*
Java IO的各种流是阻塞的。当某个线程调用read()或write()方法时，该线程被阻塞，直到有数据被读取到或者数据完全写入。阻塞期间该线程无法处理任何其它事情。

Java NIO为非阻塞模式。读写请求并不会阻塞当前线程，在数据可读/写前当前线程可以继续做其它事情，所以一个单独的线程可以管理多个输入和输出通道。

*选择器（Selector）*
Java NIO的选择器允许一个单独的线程同时监视多个通道，可以注册多个通道到同一个选择器上，然后使用一个单独的线程来“选择”已经就绪的通道。这种“选择”机制为一个单独线程管理多个通道提供了可能。

*零拷贝*
Java NIO中提供的FileChannel拥有transferTo和transferFrom两个方法，可直接把FileChannel中的数据拷贝到另外一个Channel，或者直接把另外一个Channel中的数据拷贝到FileChannel。该接口常被用于高效的网络/文件者数据传输和大文件拷贝。在操作系统支持的情况下，通过该方法传输数据并不需要将源数据从内核态拷贝到用户态，再从用户态拷贝到目标通道的内核态，同时也避免了两次用户态和内核态间的上下文切换，也即使用了“零拷贝”，所以其性能一般高于Java IO中提供的方法。

使用FileChannel的零拷贝将本地文件内容传输到网络的示例代码如下所示。
```
public class NIOClient {

  public static void main(String[] args) throws IOException, InterruptedException {
    SocketChannel socketChannel = SocketChannel.open();
    InetSocketAddress address = new InetSocketAddress(1234);
    socketChannel.connect(address);

    RandomAccessFile file = new RandomAccessFile(
        NIOClient.class.getClassLoader().getResource("test.txt").getFile(), "rw");
    FileChannel channel = file.getChannel();
    channel.transferTo(0, channel.size(), socketChannel);
    channel.close();
    file.close();
    socketChannel.close();
  }
}
```

####5.2、Why NIO?
开始讲NIO之前，了解为什么会有NIO，相比传统流I/O的优势在哪，它可以用来做什么等等的问题，还是很有必要的。
传统流I/O是基于字节的，所有I/O都被视为单个字节的移动；而NIO是基于块的，大家可能猜到了，NIO的性能肯定优于流I/O。没错！其性能的提高 要得益于其使用的结构更接近操作系统执行I/O的方式：通道和缓冲器。我们可以把它想象成一个煤矿，通道是一个包含煤层（数据）的矿藏，而缓冲器则是派送 到矿藏的卡车。卡车载满煤炭而归，我们再从卡车上获得煤炭。也就是说，我们并没有直接和通道交互；我们只是和缓冲器交互，并把缓冲器派送到通道。通道要么 从缓冲器获得数据，要么向缓冲器发送数据。（这段比喻出自Java编程思想）
NIO的主要应用在高性能、高容量服务端应用程序，典型的有Apache Mina就是基于它的。

####5.3、NIO 的工作方式
BIO 带来的挑战，BIO 即阻塞 I/O，不管是磁盘 I/O 还是网络 I/O，数据在写入 OutputStream 或者从 InputStream 读取时都有可能会阻塞。一旦有线程阻塞将会失去 CPU 的使用权，这在当前的大规模访问量和有性能要求情况下是不能接受的。虽然当前的网络 I/O 有一些解决办法，如一个客户端一个处理线程，出现阻塞时只是一个线程阻塞而不会影响其它线程工作，还有为了减少系统线程的开销，采用线程池的办法来减少线程创建和回收的成本，但是有一些使用场景仍然是无法解决的。如当前一些需要大量 HTTP 长连接的情况，像淘宝现在使用的 Web 旺旺项目，服务端需要同时保持几百万的 HTTP 连接，但是并不是每时每刻这些连接都在传输数据，这种情况下不可能同时创建这么多线程来保持连接。即使线程的数量不是问题，仍然有一些问题还是无法避免的。如这种情况，我们想给某些客户端更高的服务优先级，很难通过设计线程的优先级来完成，另外一种情况是，我们需要让每个客户端的请求在服务端可能需要访问一些竞争资源，由于这些客户端是在不同线程中，因此需要同步，而往往要实现这些同步操作要远远比用单线程复杂很多。以上这些情况都说明，我们需要另外一种新的 I/O 操作方式。
![](\\image\\image019.jpg)
上图中有两个关键类：Channel 和 Selector，它们是 NIO 中两个核心概念。我们还用前面的城市交通工具来继续比喻 NIO 的工作方式，这里的 Channel 要比 Socket 更加具体，它可以比作为某种具体的交通工具，如汽车或是高铁等，而 Selector 可以比作为一个车站的车辆运行调度系统，它将负责监控每辆车的当前运行状态：是已经出战还是在路上等等，也就是它可以轮询每个 Channel 的状态。这里还有一个 Buffer 类，它也比 Stream 更加具体化，我们可以将它比作为车上的座位，Channel 是汽车的话就是汽车上的座位，高铁上就是高铁上的座位，它始终是一个具体的概念，与 Stream 不同。Stream 只能代表是一个座位，至于是什么座位由你自己去想象，也就是你在去上车之前并不知道，这个车上是否还有没有座位了，也不知道上的是什么车，因为你并不能选择，这些信息都已经被封装在了运输工具（Socket）里面了，对你是透明的。NIO 引入了 Channel、Buffer 和 Selector 就是想把这些信息具体化，让程序员有机会控制它们，如：当我们调用 write() 往 SendQ 写数据时，当一次写的数据超过 SendQ 长度是需要按照 SendQ 的长度进行分割，这个过程中需要有将用户空间数据和内核地址空间进行切换，而这个切换不是你可以控制的。而在 Buffer 中我们可以控制 Buffer 的 capacity，并且是否扩容以及如何扩容都可以控制。

###6、阻塞I/O下的服务器实现
---
####6.1、单线程逐个处理所有请求
使用阻塞I/O的服务器，一般使用循环，逐个接受连接请求并读取数据，然后处理下一个请求。
```
public class IOServer {

  private static final Logger LOGGER = LoggerFactory.getLogger(IOServer.class);

  public static void main(String[] args) {
    ServerSocket serverSocket = null;
    try {
      serverSocket = new ServerSocket();
      serverSocket.bind(new InetSocketAddress(2345));
    } catch (IOException ex) {
      LOGGER.error("Listen failed", ex);
      return;
    }
    try{
      while(true) {
        Socket socket = serverSocket.accept();
        InputStream inputstream = socket.getInputStream();
        LOGGER.info("Received message {}", IOUtils.toString(new InputStreamReader(inputstream)));
      }
    } catch(IOException ex) {
      try {
        serverSocket.close();
      } catch (IOException e) {
      }
      LOGGER.error("Read message failed", ex);
    }
  }
}
```

####6.2、为每个请求创建一个线程
上例使用单线程逐个处理所有请求，同一时间只能处理一个请求，等待I/O的过程浪费大量CPU资源，同时无法充分使用多CPU的优势。下面是使用多线程对阻塞I/O模型的改进。一个连接建立成功后，创建一个单独的线程处理其I/O操作。
```
public class IOServerMultiThread {

  private static final Logger LOGGER = LoggerFactory.getLogger(IOServerMultiThread.class);

  public static void main(String[] args) {
    ServerSocket serverSocket = null;
    try {
      serverSocket = new ServerSocket();
      serverSocket.bind(new InetSocketAddress(2345));
    } catch (IOException ex) {
      LOGGER.error("Listen failed", ex);
      return;
    }
    try{
      while(true) {
        Socket socket = serverSocket.accept();
        new Thread( () -> {
          try{
            InputStream inputstream = socket.getInputStream();
            LOGGER.info("Received message {}", IOUtils.toString(new InputStreamReader(inputstream)));
          } catch (IOException ex) {
            LOGGER.error("Read message failed", ex);
          }
        }).start();
      }
    } catch(IOException ex) {
      try {
        serverSocket.close();
      } catch (IOException e) {
      }
      LOGGER.error("Accept connection failed", ex);
    }
  }
}
```

####6.3、使用线程池处理请求
为了防止连接请求过多，导致服务器创建的线程数过多，造成过多线程上下文切换的开销。可以通过线程池来限制创建的线程数，如下所示。
```
public class IOServerThreadPool {

  private static final Logger LOGGER = LoggerFactory.getLogger(IOServerThreadPool.class);

  public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    ServerSocket serverSocket = null;
    try {
      serverSocket = new ServerSocket();
      serverSocket.bind(new InetSocketAddress(2345));
    } catch (IOException ex) {
      LOGGER.error("Listen failed", ex);
      return;
    }
    try{
      while(true) {
        Socket socket = serverSocket.accept();
        executorService.submit(() -> {
          try{
            InputStream inputstream = socket.getInputStream();
            LOGGER.info("Received message {}", IOUtils.toString(new InputStreamReader(inputstream)));
          } catch (IOException ex) {
            LOGGER.error("Read message failed", ex);
          }
        });
      }
    } catch(IOException ex) {
      try {
        serverSocket.close();
      } catch (IOException e) {
      }
      LOGGER.error("Accept connection failed", ex);
    }
  }
}
```

####6.4、精典Reactor模式
精典的Reactor模式示意图如下所示。

在Reactor模式中，包含如下角色

Reactor 将I/O事件发派给对应的Handler
Acceptor 处理客户端连接请求
Handlers 执行非阻塞读/写

####6.5、多工作线程Reactor模式
经典Reactor模式中，尽管一个线程可同时监控多个请求（Channel），但是所有读/写请求以及对新连接请求的处理都在同一个线程中处理，无想充分利用多CPU的优势，同时读/写操作也会阻塞对新连接请求的处理。因此可以引入多线程，并行处理多个读/写操作，如下图所示。

####6.6、多Reactor
Netty中使用的Reactor模式，引入了多Reactor，也即一个主Reactor负责监控所有的连接请求，多个子Reactor负责监控并处理读/写请求，减轻了主Reactor的压力，降低了主Reactor压力太大而造成的延迟。
并且每个子Reactor分别属于一个独立的线程，每个成功连接后的Channel的所有操作由同一个线程处理。这样保证了同一请求的所有状态和上下文在同一个线程中，避免了不必要的上下文切换，同时也方便了监控请求响应状态。





### 9、I/O 调优
下面就磁盘 I/O 和网络 I/O 的一些常用的优化技巧进行总结如下：
#### 9.1、磁盘 I/O 优化
- 性能检测

我们的应用程序通常都需要访问磁盘读取数据，而磁盘 I/O 通常都很耗时，我们要判断 I/O 是否是一个瓶颈，我们有一些参数指标可以参考：
如我们可以压力测试应用程序看系统的 I/O wait 指标是否正常，例如测试机器有 4 个 CPU，那么理想的 I/O wait 参数不应该超过 25%，如果超过 25% 的话，I/O 很可能成为应用程序的性能瓶颈。Linux 操作系统下可以通过 iostat 命令查看。
通常我们在判断 I/O 性能时还会看另外一个参数就是 IOPS，我们应用程序需要最低的 IOPS 是多少，而我们的磁盘的 IOPS 能不能达到我们的要求。每个磁盘的 IOPS 通常是在一个范围内，这和存储在磁盘的数据块的大小和访问方式也有关。但是主要是由磁盘的转速决定的，磁盘的转速越高磁盘的 IOPS 也越高。
现在为了提高磁盘 I/O 的性能，通常采用一种叫 RAID 的技术，就是将不同的磁盘组合起来来提高 I/O 性能，目前有多种 RAID 技术，每种 RAID 技术对 I/O 性能提升会有不同，可以用一个 RAID 因子来代表，磁盘的读写吞吐量可以通过 iostat 命令来获取，于是我们可以计算出一个理论的 IOPS 值，计算公式如下所以：
( 磁盘数 * 每块磁盘的 IOPS)/( 磁盘读的吞吐量 +RAID 因子 * 磁盘写的吞吐量 )=IOPS
这个公式的详细信息请查阅参考资料 Understanding Disk I/O。

- 提升 I/O 性能

提升磁盘 I/O 性能通常的方法有：
1.增加缓存，减少磁盘访问次数
2.优化磁盘的管理系统，设计最优的磁盘访问策略，以及磁盘的寻址策略，这里是在底层操作系统层面考虑的。
3.设计合理的磁盘存储数据块，以及访问这些数据块的策略，这里是在应用层面考虑的。如我们可以给存放的数据设计索引，通过寻址索引来加快和减少磁盘的访问，还有可以采用异步和非阻塞的方式加快磁盘的访问效率。
4.应用合理的 RAID 策略提升磁盘 IO，每种 RAID 的区别我们可以用下表所示：

- 表 2.RAID 策略
磁盘阵列	说明
RAID 0	 数据被平均写到多个磁盘阵列中，写数据和读数据都是并行的，所以磁盘的 IOPS 可以提高一倍。
RAID 1	 RAID 1 的主要作用是能够提高数据的安全性，它将一份数据分别复制到多个磁盘阵列中。并不能提升 IOPS 但是相同的数据有多个备份。通常用于对数据安全性较高的场合中。
RAID 5	 这中设计方式是前两种的折中方式，它将数据平均写到所有磁盘阵列总数减一的磁盘中，往另外一个磁盘中写入这份数据的奇偶校验信息。如果其中一个磁盘损坏，可以通过其它磁盘的数据和这个数据的奇偶校验信息来恢复这份数据。
RAID 0+1	 如名字一样，就是根据数据的备份情况进行分组，一份数据同时写到多个备份磁盘分组中，同时多个分组也会并行读写。

#### 9.2、网络 I/O 优化
网络 I/O 优化通常有一些基本处理原则：
1.一个是减少网络交互的次数：要减少网络交互的次数通常我们在需要网络交互的两端会设置缓存，比如 Oracle 的 JDBC 驱动程序，就提供了对查询的 SQL 结果的缓存，在客户端和数据库端都有，可以有效的减少对数据库的访问。关于 Oracle JDBC 的内存管理可以参考《 Oracle JDBC 内存管理》。除了设置缓存还有一个办法是，合并访问请求：如在查询数据库时，我们要查 10 个 id，我可以每次查一个 id，也可以一次查 10 个 id。再比如在访问一个页面时通过会有多个 js 或 css 的文件，我们可以将多个 js 文件合并在一个 HTTP 链接中，每个文件用逗号隔开，然后发送到后端 Web 服务器根据这个 URL 链接，再拆分出各个文件，然后打包再一并发回给前端浏览器。这些都是常用的减少网络 I/O 的办法。
2.减少网络传输数据量的大小：减少网络数据量的办法通常是将数据压缩后再传输，如 HTTP 请求中，通常 Web 服务器将请求的 Web 页面 gzip 压缩后在传输给浏览器。还有就是通过设计简单的协议，尽量通过读取协议头来获取有用的价值信息。比如在代理程序设计时，有 4 层代理和 7 层代理都是来尽量避免要读取整个通信数据来取得需要的信息。
3.尽量减少编码：通常在网络 I/O 中数据传输都是以字节形式的，也就是通常要序列化。但是我们发送要传输的数据都是字符形式的，从字符到字节必须编码。但是这个编码过程是比较耗时的，所以在要经过网络 I/O 传输时，尽量直接以字节形式发送。也就是尽量提前将字符转化为字节，或者减少字符到字节的转化过程。
4.根据应用场景设计合适的交互方式：所谓的交互场景主要包括同步与异步阻塞与非阻塞方式，下面将详细介绍。

- 同步与异步
所谓同步就是一个任务的完成需要依赖另外一个任务时，只有等待被依赖的任务完成后，依赖的任务才能算完成，这是一种可靠的任务序列。要么成功都成功，失败都失败，两个任务的状态可以保持一致。而异步是不需要等待被依赖的任务完成，只是通知被依赖的任务要完成什么工作，依赖的任务也立即执行，只要自己完成了整个任务就算完成了。至于被依赖的任务最终是否真正完成，依赖它的任务无法确定，所以它是不可靠的任务序列。我们可以用打电话和发短信来很好的比喻同步与异步操作。
在设计到 IO 处理时通常都会遇到一个是同步还是异步的处理方式的选择问题。因为同步与异步的 I/O 处理方式对调用者的影响很大，在数据库产品中都会遇到这个问题。因为 I/O 操作通常是一个非常耗时的操作，在一个任务序列中 I/O 通常都是性能瓶颈。但是同步与异步的处理方式对程序的可靠性影响非常大，同步能够保证程序的可靠性，而异步可以提升程序的性能，必须在可靠性和性能之间做个平衡，没有完美的解决办法。

- 阻塞与非阻塞
阻塞与非阻塞主要是从 CPU 的消耗上来说的，阻塞就是 CPU 停下来等待一个慢的操作完成 CPU 才接着完成其它的事。非阻塞就是在这个慢的操作在执行时 CPU 去干其它别的事，等这个慢的操作完成时，CPU 再接着完成后续的操作。虽然表面上看非阻塞的方式可以明显的提高 CPU 的利用率，但是也带了另外一种后果就是系统的线程切换增加。增加的 CPU 使用时间能不能补偿系统的切换成本需要好好评估。

- 两种的方式的组合
组合的方式可以由四种，分别是：同步阻塞、同步非阻塞、异步阻塞、异步非阻塞，这四种方式都对 I/O 性能有影响。下面给出分析，并有一些常用的设计用例参考。

表 3. 四种组合方式
组合方式	性能分析
同步阻塞	 最常用的一种用法，使用也是最简单的，但是 I/O 性能一般很差，CPU 大部分在空闲状态。
同步非阻塞	 提升 I/O 性能的常用手段，就是将 I/O 的阻塞改成非阻塞方式，尤其在网络 I/O 是长连接，同时传输数据也不是很多的情况下，提升性能非常有效。
这种方式通常能提升 I/O 性能，但是会增加 CPU 消耗，要考虑增加的 I/O 性能能不能补偿 CPU 的消耗，也就是系统的瓶颈是在 I/O 还是在 CPU 上。
异步阻塞	 这种方式在分布式数据库中经常用到，例如在网一个分布式数据库中写一条记录，通常会有一份是同步阻塞的记录，而还有两至三份是备份记录会写到其它机器上，这些备份记录通常都是采用异步阻塞的方式写 I/O。
异步阻塞对网络 I/O 能够提升效率，尤其像上面这种同时写多份相同数据的情况。
异步非阻塞	 这种组合方式用起来比较复杂，只有在一些非常复杂的分布式情况下使用，像集群之间的消息同步机制一般用这种 I/O 组合方式。如 Cassandra 的 Gossip 通信机制就是采用异步非阻塞的方式。
它适合同时要传多份相同的数据到集群中不同的机器，同时数据的传输量虽然不大，但是却非常频繁。这种网络 I/O 用这个方式性能能达到最高。

虽然异步和非阻塞能够提升 I/O 的性能，但是也会带来一些额外的性能成本，例如会增加线程数量从而增加 CPU 的消耗，同时也会导致程序设计的复杂度上升。如果设计的不合理的话反而会导致性能下降。在实际设计时要根据应用场景综合评估一下。

下面举一些异步和阻塞的操作实例：

在 Cassandra 中要查询数据通常会往多个数据节点发送查询命令，但是要检查每个节点返回数据的完整性，所以需要一个异步查询同步结果的应用场景，部分代码如下：
清单 3.异步查询同步结果
```
 class AsyncResult implements IAsyncResult{ 
    private byte[] result_; 
    private AtomicBoolean done_ = new AtomicBoolean(false); 
    private Lock lock_ = new ReentrantLock(); 
    private Condition condition_; 
    private long startTime_; 
    public AsyncResult(){        
        condition_ = lock_.newCondition();// 创建一个锁
        startTime_ = System.currentTimeMillis(); 
    }    
 /*** 检查需要的数据是否已经返回，如果没有返回阻塞 */ 
 public byte[] get(){ 
        lock_.lock(); 
        try{ 
            if (!done_.get()){condition_.await();} 
        }catch (InterruptedException ex){ 
            throw new AssertionError(ex); 
        }finally{lock_.unlock();} 
        return result_; 
 } 
 /*** 检查需要的数据是否已经返回 */ 
    public boolean isDone(){return done_.get();} 
 /*** 检查在指定的时间内需要的数据是否已经返回，如果没有返回抛出超时异常 */ 
    public byte[] get(long timeout, TimeUnit tu) throws TimeoutException{ 
        lock_.lock(); 
        try{            boolean bVal = true; 
            try{ 
                if ( !done_.get() ){ 
           long overall_timeout = timeout - (System.currentTimeMillis() - startTime_); 
                    if(overall_timeout > 0)// 设置等待超时的时间
                        bVal = condition_.await(overall_timeout, TimeUnit.MILLISECONDS); 
                    else bVal = false; 
                } 
            }catch (InterruptedException ex){ 
                throw new AssertionError(ex); 
            } 
            if ( !bVal && !done_.get() ){// 抛出超时异常
                throw new TimeoutException("Operation timed out."); 
            } 
        }finally{lock_.unlock();      } 
        return result_; 
 } 
 /*** 该函数拱另外一个线程设置要返回的数据，并唤醒在阻塞的线程 */ 
    public void result(Message response){        
        try{ 
            lock_.lock(); 
            if ( !done_.get() ){                
                result_ = response.getMessageBody();// 设置返回的数据
                done_.set(true); 
                condition_.signal();// 唤醒阻塞的线程
            } 
        }finally{lock_.unlock();}        
    }    
}
```

### 参考资料：

- [深入分析 Java I/O 的工作机制](http://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)
- [《Jetty 的工作原理和与 Tomcat 的比较》：这里介绍了 Jetty 是如何使用 NIO 技术处理 HTTP 连接请求的，以及与 Tomcat 处理有何不同之处。](http://www.ibm.com/developerworks/cn/java/j-lo-jetty/)
- [](http://www.jasongj.com/java/nio_reactor/) 