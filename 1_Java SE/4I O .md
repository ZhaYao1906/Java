# I/O

# 1. Java IO基础知识

Java的I/O(输入、输出)流是用于 **处理输入和输出数据的类库。**

**输入**是数据从外部源（**如键盘、文件、网络等）**流向程序。 **输出**是数据从程序流向外部目的地**（如屏幕、文件、网络等）**。

IO 流在 Java 中分为 **输入流** 和 **输出流** ，而根据数据的处理方式又分为 **字节流和字符流。**

## 1.1 4个抽象类基类

 `InputStream`/`Reader`: 所有的输入流的基类，前者是 字节输入流，后者是 字符输入流。

`OutputStream`/`Writer`: 所有输出流的基类，前者是 字节输出流，后者是 字符输出流。

### 为什么I/O操作要分为字节流和字符流操作？

- 字符流是由 Java 虚拟机将字节转换得到的，这个过程还算是比较耗时。
- 如果我们不知道编码类型就很容易出现**乱码问题**。

因此，I/O 流就干脆提供了**一个直接操作字符的接口，**方便我们平时对字符进行流操作。如果**音频文件、图片等媒体文件**用字节流比较好，如果**涉及到字符**的话使用字符流比较好。

## 1.2  字节缓冲流

IO 操作是很消耗性能的，缓冲流将数据加载至缓冲区，**一次性读取/写入多个字节，从而避免频繁的 IO 操作，** 提高流的传输效率。

字节缓冲流这里采用了 **装饰器模式** 来增强 `InputStream` 和`OutputStream`子类对象的功能。

**例：我们可以通过 `BufferedInputStream`（字节缓冲输入流）来增强 `FileInputStream` 的功能。**

```java
// 新建一个 BufferedInputStream 对象
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
```

由于字节缓冲流内部有缓冲区（字节数组），因此，字节缓冲流会先将读取到的字节存放在缓存区，大幅减少 IO 次数，提高读取效率。

## 1.3 打印流

`System.out` 实际是用于获取一个 `PrintStream` 对象，`print`方法实际调用的是 `PrintStream` 对象的 `write` 方法。

`PrintStream` 属于字节打印流，与之对应的是 `PrintWriter` （字符打印流）。`PrintStream` 是 `OutputStream` 的子类 `PrintWriter` 是 `Writer` 的子类。

## 1.4 随机访问流

支持随意跳转到文件的任意位置进行读写的 `RandomAccessFile` 。

```java
// openAndDelete 参数默认为 false 表示打开文件并且这个文件不会被删除
public RandomAccessFile(File file, String mode)
    throws FileNotFoundException {
    this(file, mode, false);
}
// 私有方法
private RandomAccessFile(File file, String mode, boolean openAndDelete)  throws FileNotFoundException{
  // 省略大部分代码
}
```

mode（读写模式），可由我们指定。

`r` : 只读模式。

`rw`: 读写模式。

`rws`: 相对于 `rw`，`rws` 同步更新对“文件的内容”或“元数据”的修改到外部存储设备。

`rwd` : 相对于 `rw`，`rwd` 同步更新对“文件的内容”的修改到外部存储设备。

文件内容指的是文件中实际保存的数据，元数据则是用来描述文件属性比如文件的大小信息、创建和修改时间。

# 2. Java IO设计模式

## 2.1 装饰器模式

**装饰器（Decorator）模式** 可以在不改变原有对象的情况下拓展其功能。

装饰器模式通过 **组合** 替代 **继承** 来扩展原始类的功能，在一些继承关系比较复杂的场景（IO 这一场景各种类的继承关系就比较复杂）**更加实用**。

对于字节流来说， `FilterInputStream` （对应输入流）和`FilterOutputStream`（对应输出流）**是装饰器模式的核心**，分别用于增强 `InputStream` 和`OutputStream`子类对象的功能。

常见的`BufferedInputStream`(字节缓冲输入流)、`DataInputStream` 等等都是`FilterInputStream` 的子类，`BufferedOutputStream`（字节缓冲输出流）、`DataOutputStream`等等都是`FilterOutputStream`的子类。

例：`BufferedInputStream` 构造函数

```java
public BufferedInputStream(InputStream in) {
    this(in, DEFAULT_BUFFER_SIZE);
}

public BufferedInputStream(InputStream in, int size) {
    super(in);
    if (size <= 0) {
        throw new IllegalArgumentException("Buffer size <= 0");
    }
    buf = new byte[size];
}
```

---

`BufferedInputStream` 的构造函数其中的一个参数就是 `InputStream` 。

为什么这么用？

`InputStream`的子类实在太多，如果为每一个子类继承一个对应的缓冲输入流，继承关系也太复杂了。

第二个优点就是，对原始类可以嵌套使用多个装饰器。

---

## 2.2 适配器模式

主要用于接口互不兼容的类的协调工作。

适配器模式中存在**被适配的对象**或者类称为 **适配者(Adapter)** ，**作用于适配者的对象**或者类称为**适配器(Adapter)** 。适配器分为对象适配器和类适配器。类适配器使用继承关系来实现，对象适配器使用组合关系来实现。
**IO 流中的字符流和字节流的接口不同，它们之间可以协调工作就是基于适配器模式来做的，**更准确点来说是**对象适配器（组合关系）**。通过适配器，我们可以将字节流对象适配成一个字符流对象，这样我们可以直接通过字节流对象来读取或者写入字符数据。

`InputStreamReader` 和 `OutputStreamWriter` 就是两个适配器(Adapter)， 同时，它们两个也是**字节流和字符流之间的桥梁。**`InputStream` 和 `OutputStream` 的子类是被适配者， `InputStream-Reader` 和 `OutputStreamWriter`是适配器。

```java
// InputStreamReader 是适配器，FileInputStream 是被适配的类
InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
// BufferedReader 增强 InputStreamReader 的功能（装饰器模式）
BufferedReader bufferedReader = new BufferedReader(isr);
```

### 适配器模式和装饰器模式有什么区别呢？

**装饰器模式 更侧重于动态地增强原始类的功能，** 装饰器类需要跟原始类继承相同的抽象类或者实现相同的接口。
**适配器模式 更侧重于让接口不兼容而不能交互的类可以一起工作，** 当我们调用适配器对应的方法时，适配器内部会调用适配者类或者和适配类相关的类的方法，这个过程透明的。

适配器和适配者两者不需要继承相同的抽象类或者实现相同的接口。

## 2.3 工厂模式

工厂模式用于创建对象，NIO 中大量用到了工厂模式，比如 `Files` 类的 `newInputStream` 方法用于创建`InputStream` 对象（静态工厂）、 `Paths` 类的`get` 方法创建 `Path` 对象（静态工厂）。

```java
InputStream is = Files.newInputStream(Paths.get(generatorLogoPath))
```

## 2.4 观察者模式

NIO 中的文件目录监听服务使用到了观察者模式。

NIO 中的文件目录监听服务基于 `WatchService` 接口和 `Watchable` 接口。`WatchService` 属于观察者，`Watchable` 属于被观察者。

**从计算机结构的视角来看的话， I/O 描述了计算机系统与外部设备之间通信的过程。**

# 3. Java IO 模型

**从应用程序的视角来看的话，我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。**

当应用程序发起 I/O 调用后，会经历两个步骤：

1. 内核等待 I/O 设备准备好数据。
2. 内核将数据从内核空间拷贝到用户空间。

## 常见的IO模型？

IO模型一共有5种：**同步阻塞 I/O**、**同步非阻塞 I/O、I/O 多路复用**、**信号驱动 I/O** 和**异步 I/O**。

## 3.1 BIO （Blocking I/O）

同步阻塞IO模型,应用程序发起read调用后,会一直阻塞,直到内核把数据拷贝到用户空间.

![image.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/image.png)

适用于**连接数少且负载较轻的应用程序**，例如传统的服务器应用或学习阶段的小型项目。

特点

- **阻塞模式**：每个请求都由一个**独立的线程**处理，I/O 操作会阻塞线程，直到数据准备好或操作完成。
- **资源开销大**：对于每个连接，都需要一个独立的线程，当连接数增加时，会占用大量的系统资源（如线程、内存）

**BIO作为通信工具在默认情况下一次只能处理一个请求，（单线程情况下）。虽然基础BIO是单请求的，但可以通过多线程模型实现多请求处理。具体还要看线程数。**

## 3.2 NIO（New I/O、Non-blocking I/O）

适用于**高并发、高负载的服务器应用，** 例如即时通讯、游戏服务器等。Java中的NIO可以看作 **I/O多路复用模型**

**同步非阻塞**

![image.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/image%201.png)

缺点：**应用程序不断进行 I/O 系统调用轮询数据是否已经准备好的过程是十分消耗 CPU 资源的。**

### **I/O多路复用**

核心思想：**让单个线程去监视多个连接**，一旦某个连接就绪（触发了读/写事件的时候），就会**通知对应的程序去主动获取这个就绪的连接。**

在对系统资源消耗比较小的情况下，去提升服务端的连接处理数量。

![image.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/image%202.png)

特点

- **非阻塞模式**：使用选择器（Selector）和通道（Channel）进行多路复用，**单个线程可以处理多个 I/O 事件。**
- **提高性能**：通过减少线程数，降低上下文切换的开销，提高系统的吞吐量。
- **复杂性增加**：编程模型较 BIO 复杂，需要处理选择器和通道，以及非阻塞 I/O 的状态管理。

Java 中的 NIO ，有一个非常重要的**选择器 ( Selector )** 的概念，也可以被称为 **多路复用器**。(通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务。)

客户端请求到服务端以后，此时客户端在传输数据的过程中，为了避免Server端在read客户端数据的时候阻塞，**服务端会先把请求注册到一个Selector中，服务端此时不需要阻塞，只需要通过selector.select()方法去阻塞轮询复路器上就绪的channel就可以了。**

I/O多路复用相比同步非阻塞,用select调用,准备就绪后才会发起read调用 (数据从内核空间 → 用户空间)

## 3.3 AIO (**Asynchronous 异步 I/O**)

![image.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/image%203.png)

 适用于高并发、需要高吞吐量和低延迟的应用，例如高性能的 Web 服务器、大规模的消息处理系统等。

特点

- **异步非阻塞模式**：采用异步 I/O 操作，**I/O 操作完成后会通知应用程序（通过回调）。**
- **减少阻塞**：大大减少了阻塞等待的时间，适用于高延迟的 I/O 操作。

## Java NIO 总结

NIO弥补了同步阻塞I/O的不足，它在标准的Java代码中提供了非阻塞、面向缓冲、基于通道的I/O，可以使少量的线程来处理多个连接，大大提高了I/O的效率和并发。

### NIO核心组件

- **Buffer（缓冲区）**

NIO 读写数据都是通过缓冲区进行操作的。读操作的时候将 Channel 中的数据填充到 Buffer 中，而写操作时将 Buffer 中的数据写入到 Channel 中。

- **Channel（通道）**

Channel 是一个双向的、可读可写的数据传输通道，NIO 通过 Channel 来实现数据的输入输出。通道是一个抽象的概念，它可以代表文件、套接字或者其他数据源之间的连接。

- **Selector（选择器）**

允许一个线程处理多个 Channel，基于事件驱动的 I/O 多路复用模型。所有的 Channel 都可以注册到 Selector 上，由 Selector 来分配线程来处理事件。

![image.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/image%204.png)

## Java在等待IO时会占用CPU时间吗

当一个线程因等待I/O操作而阻塞时，该线程**不会占用CPU时间**。（这是由操作系统的线程调度机制和Java的I/O模型共同决定的）

BIO：线程会被操作系统挂起，释放 CPU 资源，直到 I/O 操作完成。

NIO：若未设置超时，selector.select() 会阻塞线程，此时不占用 CPU。

AIO：零 CPU 占用，等待期间，无任何线程被阻塞，完全由操作系统内核管理 I/O 操作。

# 4 零拷贝

零拷贝（Zero-copy）是一种优化技术，旨在**减少数据在内存中的拷贝次数**，从而提升I/O操作的性能。

### **DMA （Direct Memory Access） 直接内存访问技术**

在进行I/O设备和内存的数据传输的时候，**数据搬运的工作全部交给DMA控制器**，而CPU不再参与任何与数据搬运相关的事情，这样CPU就可以处理别的事务了。

**CPU将不再参与【将数据从磁盘控制器缓冲区搬运到内核空间】的工作，这部分工作全程由DMA完成**。但是CPU在这个过程中也是必不可少的，因为传输什么数据，从哪里传输到哪里，都需要CPU来告诉DMA控制器。

## 4.1 传统I/O数据传输过程

![屏幕截图 2024-09-13 120909.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/dec6385e-ec3e-41e8-821f-bc5f51e92049.png)

在传统的I/O操作中，数据从文件或网络传输到应用程序通常需要以下步骤：

Step1、从磁盘读取数据到内核缓冲区：数据首先从磁盘读取到内核空间的缓冲区。

Step2、从**内核缓冲区**拷贝到**用户缓冲区**：数据从内核缓冲区拷贝到用户空间的缓冲区，供应用程序使用。

Step3、从**用户缓冲区**拷贝到**内核缓冲区**：如果数据需要发送到网络，数据会从用户缓冲区再次拷贝到内核空间的Socket缓冲区。

Step4、从**内核缓冲区**发送到**网络**：数据从Socket缓冲区发送到网络。

## 4.2 零拷贝

零拷贝技术的核心思想是**避免数据在内核空间和用户空间之间的拷贝，直接在内核空间中完成数据的传输。**

### mmap 内存映射

mmap是一种将文件直接映射到用户空间内存的技术。通过mmap，应用程序可以直接访问文件数据，而不需要将数据从内核缓冲区拷贝到用户缓冲区。

![屏幕截图 2024-09-13 121159.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-09-13_121159.png)

### sendFile

sendfile是一种系统调用，允许数据直接从文件描述符传输到Socket描述符，完全避免了数据在用户空间和内核空间之间的拷贝。

![屏幕截图 2024-09-13 121441.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-09-13_121441.png)

### 零拷贝

![屏幕截图 2025-02-22 121806.png](I%20O%2049c58a6d84544771a59ad8e25bd3b077/6fc7f51e-62f6-4d15-af12-58023c6b8331.png)

零拷贝是指计算机执行 IO 操作时，CPU 不需要将数据从一个存储区域复制到另一个存储区域，从而可以减少上下文切换以及 CPU 的拷贝时间。

无论是传统的 I/O 方式，还是引入了零拷贝之后，2 次 DMA(Direct Memory Access) 拷贝是都少不了的。因为两次 DMA 都是依赖硬件完成的。**零拷贝主要是减少了 CPU 拷贝及上下文的切换**

# **5. 对 select，poll，epoll 的详细解析（I/O多路复用机制）**

select，poll，epoll  都是 **IO 多路复用的机制** IO 多路复用的本质是通过一种机制，**让单个进程可以监视多个描述符，**当发现某个描述符就绪之后，能够**通知程序进行相应的读写操作。**

## **5.1 select**

**函数参数**

**int maxfdp1：** 待测试的文件描述字个数

fd_set *readset , fd_set *writeset , fd_set *exceptset：fd_set 是一个集合，里面存放的是文件描述符，三个参数分别表示让内核测试读、写和异常条件的文件描述符集合。若对某一条件不感兴趣，可以将其设置为空指针

const struct timeval *timeout :告诉内核，等待所指定文件描述符集合中的任意一个就绪，一共可以花费多少时间

**返回值**  返回值类型为 int ，若有就绪描述符，则返回其数目，若超时则返回0，若出错则返回-1。

**运行机制** 当我们调用 select 函数时，内核会根据 IO 状态对 fd_set 的内容进行修改，从而通知执行 select 函数的进程哪一个文件或者 Socket 是可读的。

**优点** 用户可以在一个线程内同时处理多个 socket 的 IO 请求。用户可以注册多个 socket，然后调用 select 函数读取被激活的 socket，从而实现在同一个线程内同时处理多个 IO 请求，在这点上select 函数与同步阻塞模型不同，因为在同步阻塞模型中需要通过多线程才能达到这个目的。

**缺点** 调用 select 函数时，**需要把 fd_set 集合从用户态拷贝到内核态，当 fd_set 集合很大时，这个开销将会非常巨大。**

## **5.2 poll**

poll 的机制与 select 几乎相同，**会对管理的描述符进行轮询操作，并根据描述符的状态进行相应的处理。**

poll 将用户传入的数组拷贝到内核空间，然后查询每个描述符对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有描述符后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时。

### **poll 与 select 的区别**

**select 函数中，内核对 fd_set 集合的大小做出了限制，大小不可变为1024；**

**而 poll 函数中，并没有最大文件描述符数量的限制（基于链表存储）。**

## **5.3 epoll**

epoll 是基于**事件驱动**的 IO 方式，与 select 相比，epoll 并没有描述符个数限制。

epoll 使用一个文件描述符管理多个描述符，它将文件描述符的事件放入内核的一个事件表中，从而在用户空间和内核空间的复制操作只用实行一次即可。

**特点：** epoll 是 poll 的增强版，在获取事件时，epoll 无需遍历整个被监听的描述符集，而是**只需遍历被内核 IO 事件异步唤醒而加入 Ready 队列的描述符集合即可**。因此，epoll 能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率。

**优点：** 

1 没有最大并发连接的限制

2 不采取轮询的方式，效率高，只会处理活跃的连接，与连接总数无关

### **两种模式**

**epoll 提供了两种模式，一种是水平触发，一种是边缘触发。** 边缘触发与水平触发相比较，可以使用户空间程序可能缓存 IO 状态，并减少 epoll_wait 的调用，从而提高应用程序的效率。

**1、水平触发（LT）**：默认工作模式，当 epoll_wait 检测到某描述符事件就绪并通知应用程序时，应用程序可以不立即处理该事件；等到下次调用 epoll_wait 时，会再次通知此事件

**2、边缘触发（ET）**：当 epoll_wait 检测到某描述符事件就绪并通知应用程序时，应用程序必须立即处理该事件。如果不处理，下次调用 epoll_wait 时，不会再次通知此事件。

总结：水平触发是指epoll_wait检测到事件不立即处理**（留一手）**，边缘触发是指会立即处理。

### **select,、poll、epoll之间的对比**

- **IO 效率：** select 只知道有 IO 事件发生，却不知道是哪几个流，只能**采取轮询所有流的方式**，故其具有 O(n) 的无差别轮询复杂度，处理的流越多，无差别轮询时间就越长；poll 与 select 并无区别，它的时间复杂度也是 **O(n)；**
    
epoll 会将哪个流发生了怎样的 IO 事件通知我们（当描述符就绪时，系统注册的回调函数会被调用，将就绪描述符放到 readyList 里面），它是**事件驱动**的，其时间复杂度为 **O(1)**
    
- **底层实现：select 的底层实现为数组，poll 的底层实现为链表，而 epoll 的底层实现为红黑树**
    
红黑树是个高效的数据结构，增删改一般时间复杂度是O(logn)，通过对红黑树的管理，不需要像select/poll在每次操作时都传入整个Socket集合，减少了内核和用户空间大量的数据拷贝和内存分配。
    
- **最大连接数：** select 的最大连接数为 1024 或 2048，而 poll 和 epoll 是无上限的
- **对描述符的拷贝：** select和poll:在使用的时候，首先需要把关注的**Socket集合**通过**select/poll系统调用从用户态拷贝到内核态。然后内核检测事件，当有网络事件产生时，内核需要遍历进程关注的Socket集合，找到对应的Socket,并设置其状态为可读/可写，** 然后把整个Socket集合从内核态拷贝至用户态，当客户端越多，也就是Socket集合越大，Socket集合遍历和拷贝带来的开销很大。不如epoll的红黑树+事件驱动。

（基于前4点，导致性能差距）性能：epoll 在绝大多数情况下性能远超 select 和 poll，但**在连接数少并且连接都十分活跃的情况下，** select 和 poll 的性能可能比 epoll 好，因为 epoll 的通知机制需要很多函数回调。

### **总结**

select、poll都是基于轮询的方式去获取就绪连接的，而epoll是基于事件驱动的方式去获取就绪连接。从性能的角度来看，基于事件驱动的方式是要优于轮询的方式。
