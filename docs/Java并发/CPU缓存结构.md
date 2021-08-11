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

MESI协议

MESI中每个缓存行都有四个状态，分别是E（exclusive）、M（modified）、S（shared）、I（invalid）。

M：代表该缓存行中的内容被修改了，并且该缓存行只被缓存在该CPU中。这个状态的缓存行中的数据和内存中的不一样，在未来的某个时刻它会被写入到内存中（当其他CPU要读取该缓存行的内容时。或者其他CPU要修改该缓存对应的内存中的内容时（个人理解CPU要修改该内存时先要读取到缓存中再进行修改），这样的话和读取缓存中的内容其实是一个道理）。

E：E代表该缓存行对应内存中的内容只被该CPU缓存，其他CPU没有缓存该缓存对应内存行中的内容。这个状态的缓存行中的内容和内存中的内容一致。该缓存可以在任何其他CPU读取该缓存对应内存中的内容时变成S状态。或者本地处理器写该缓存就会变成M状态。

S:该状态意味着数据不止存在本地CPU缓存中，还存在别的CPU的缓存中。这个状态的数据和内存中的数据是一致的。当有一个CPU修改该缓存行对应的内存的内容时会使该缓存行变成 I 状态。

I：代表该缓存行中的内容时无效的。

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



















