# 名词

## Clock Cycle Time
主频的概念大家接触的比较多，而CPU的Clock Cycle Time(**时钟周期时间**)，**等于主频的倒数，是CPU能够识别的最小时间单位**，比如说4G主频的CPU的Clock Cycle Time就是0.25 ns，作为对比，我们墙上挂钟的Cycle Time是1s

例如，运行一条加法指令一般需要一个时钟周期时间

## CPl
有的指令需要更多的时钟周期时间，所以引出了CPI ( Cycles Per Instruction )指令平均时钟周期数

## IPC

IPC ( lnstruction Per Clock Cycle ）即CPI的倒数，表示每个时钟周期能够运行的指令数

## CPU执行时间

程序的CPU执行时间，即user + system时间，可以用下面的公式来表示

```java
程序 CPU 执行时间 = 指令数 * CPI * Clock Cycle Time
```

# CPU缓存结构原理

## CPU缓存

<center><img src="https://ss.im5i.com/2021/08/11/PxCSn.png" alt="PxCSn.png" border="0" /></center>

查看CPU缓存：`lscpu`

```shell
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             4
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 158
Model name:            Intel(R) Core(TM) i5-9300H CPU @ 2.40GHz
Stepping:              10
CPU MHz:               2400.002
CPU max MHz:           0.0000
CPU min MHz:           0.0000
BogoMIPS:              4800.00
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              8192K
NUMA node0 CPU(s):     0-3
```

速度比较：

| 从CPU到 | 大约需要的时钟周期 |
| :------ | :----------------- |
| 寄存器  | 1cycle             |
| L1      | 3~4cycle           |
| L2      | 10~20cycle         |
| L3      | 40~45cycle         |
| 内存    | 120~240cycle       |

查看 cpu 缓存行：`cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size`

```shell
64
```

> 缓存是分段（line）的，一个段对应一块存储空间，称之为**缓存行**，它是CPU缓存中可分配的最小存储单元，大小32字节、64字节、128字节不等，这与CPU架构有关，通常来说是64字节。当CPU看到一条读取内存的指令时，它会把内存地址传递给一级数据缓存，一级数据缓存会检查它是否有这个内存地址对应的缓存段，如果没有就把整个缓存段从内存（或更高一级的缓存）中加载进来。注意，这里说的是一次加载整个缓存段

cpu拿到的内存地址格式是这样的：

```
[高位组标记][低位索引][偏移量]
```

<center><img src="https://ss.im5i.com/2021/08/11/PxNw2.png" alt="PxNw2.png" border="0" /></center>

## CPU缓存读

读取数据流程如下

- 根据低位，计算在缓存中的索引
- 判断是否有效
  - 0去内存读取新数据更新缓存行
  - 1再对比高位组标记是否一致
    - 一致，根据偏移量返回缓存数据
    - 不一致，去内存读取新数据更新缓存行

## CPU缓存一致性

缓存一致性协议有多种，但是日常处理的大多数计算机设备都属于**"嗅探（snooping）"协议**，它的基本思想是：

- 所有内存的传输都发生在一条共享的总线上，而所有的处理器都能看到这条总线：缓存本身是独立的，但是内存是共享资源，所有的内存访问都要经过仲裁（同一个指令周期中，只有一个CPU缓存可以读写内存）
- CPU缓存不仅仅在做内存传输的时候才与总线打交道，而是不停在嗅探总线上发生的数据交换，跟踪其他缓存在做什么。所以<span style="color:red">当一个缓存代表它所属的处理器去读写内存时，其它处理器都会得到通知，它们以此来使自己的缓存保持同步。只要某个处理器一写内存，其它处理器马上知道这块内存在它们的缓存段中已失效</span>。

MESI协议是当前最主流的缓存一致性协议，在MESI协议中，每个缓存行有4个状态，可用2个bit表示，它们分别是：

MESI中每个缓存行都有四个状态，分别是E（exclusive）、M（modified）、S（shared）、I（invalid）。

<center><img src="https://ss.im5i.com/2021/08/30/fbp0R.png" alt="fbp0R.png" border="0" /></center>

1. E、S、M状态的缓存行都可以满足CPU的读请求
2. E状态的缓存行，有写请求，会将状态改为M，这时并不触发向主存的写
3. E状态的缓存行，必须监听该缓存行的读操作，如果有，要变为S状态

<center><img src="https://ss.im5i.com/2021/08/11/PxJCQ.png" alt="PxJCQ.png" border="0" /></center>

4. M状态的缓存行，必须监听该缓存行的读操作，如果有，先将其它缓存(S状态）中该缓存行变成l状态（即6.的流程），写入主存，自己变为S状态
5. S状态的缓存行，有写请求，走4.的流程
6. S状态的缓存行，必须监听该缓存行的失效操作，如果有，自己变为Ⅰ状态
7. l状态的缓存行，有读请求，必须从主存读取

<center><img src="https://ss.im5i.com/2021/08/11/Px3aJ.png" alt="Px3aJ.png" border="0" /></center>

# Reference

- [黑马程序员Java并发编程](https://www.bilibili.com/video/BV16J411h7Rd)

- [MESI（缓存一致性协议）](https://blog.csdn.net/xiaowenmu1/article/details/89705740)

- [Java中volatile关键字实现原理](https://www.cnblogs.com/xrq730/p/7048693.html)

















