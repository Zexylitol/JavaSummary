# 1. Netty是什么

- Netty是由JBOSS提供的一个Java开源框架
- 异步事件驱动框架，用于快速开发高性能服务端和客户端
- 封装了JDK底层BIO和NIO模型，提供高度可用的API
- 自带编解码器解决拆包粘包问题，用户只用关心业务逻辑
- 精心设计的reactor线程模型支持高并发海量连接
- 自带各种协议栈让你处理任何一种通用协议都几乎不用亲自动手
- 设计优雅，提供阻塞和非阻塞的Socket；提供灵活可拓展的事件模型；提供高度可定制的线程模型
- 具备更高的性能和更大的吞吐量，使用零拷贝技术最小化不必要的内存复制，减少资源的消耗
- 提供安全传输特性

# 2. 线程模型

不同的线程模式，对程序的性能有很大影响，目前存在的线程模型有：

- 传统阻塞I/O服务模型
- Reactor模型
  - 单Reactor 单线程
  - 单Reactor 多线程
  - 主从Reactor 多线程

## 2.1 传统阻塞I/O服务模型

采用阻塞IO模式获取输入的数据，每个连接都需要独立的线程完成数据的输入，业务处理和数据返回工作。

<center><img src="https://ss.im5i.com/2021/09/23/lyR0W.jpg" alt="lyR0W.jpg" border="0" /></center>



存在问题：

1. 当并发数很大，就会创建大量的线程，占用很大系统资源

2. 连接创建后，如果当前线程暂时没有数据可读，该线程会阻塞在`read`操作，造成线程资源浪费

## 2.2 Reactor模型

Reactor模式，通过一个或多个输入同时传递给服务处理器的模式，服务器端程序处理传入的多个请求并将它们同步分派到相应的处理线程，因此Reactor模式也叫Dispatcher模式。**Reactor模式使用IO复用监听事件，收到事件后，分发给某个线程(进程)**，这点就是网络服务器高并发处理关键。

### 2.2.1 单Reactor 单线程

<center><img src="https://ss.im5i.com/2021/09/23/ly3x5.jpg" alt="ly3x5.jpg" border="0" /></center>

存在问题：

1. 性能问题：只有一个线程，无法完全发挥多核CPU的性能。Handler在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈

2. 可靠性问题：线程意外终止或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障

### 2.2.2 单Reactor 多线程

<center><img src="https://ss.im5i.com/2021/09/23/ly7cP.jpg" alt="ly7cP.jpg" border="0" /></center>

存在问题：

- 多线程数据共享和访问比较复杂，reactor处理所有的事件的监听和响应，在单线程运行，在高并发场景容易出现性能瓶颈

### 2.2.3 主从Reactor 多线程

<center><img src="https://ss.im5i.com/2021/09/23/lyMoD.jpg" alt="lyMoD.jpg" border="0" /></center>

存在问题：

- 这种模式的缺点是编程复杂度较高。但是由于其优点明显，在许多项目中被广泛使用，包括Nginx、Memcached、Netty等。这种模式也被叫做服务器的<span style="color:red">1+M+N线程模式</span>，即使用该模式开发的服务器包含<span style="color:red">一个(或多个，1只是表示相对较少）连接建立线程 + M个IO线程 + N个业务处理线程</span>。这是业界成熟的服务器程序设计模式

# 3. Netty线程模型

Netty的设计主要基于主从 Reactor 多线程模式，并做了一定的改进。

<center><img src="https://ss.im5i.com/2021/09/23/lyOOj.jpg" alt="lyOOj.jpg" border="0" /></center>

# 4. Netty基本组件

<center><img src="https://ss.im5i.com/2021/09/23/lB1fS.png" alt="lB1fS.png" border="0" /></center>

<center><img src="https://ss.im5i.com/2021/09/23/lBFXL.jpg" alt="lBFXL.jpg" border="0" /></center>