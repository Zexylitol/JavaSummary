# 1 如何查看IP地址

在Windows上是`ipconfig`，在`Linux`上是`ifconfig`和`ip addr`

```shell
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 inet 127.0.0.1/8 scope host lo
 valid_lft forever preferred_lft forever
 inet6 ::1/128 scope host 
 valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
 link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
 inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
 valid_lft forever preferred_lft forever
 inet6 fe80::f816:3eff:fec7:7975/64 scope link 
 valid_lft forever preferred_lft forever
```

这个命令显示了这台机器上所有的网卡。大部分的网卡都会有一个IP地址，当然，这不是必须的。

`inet 10.100.122.2/24`是IPv4地址，总共32位；`inet6  fe80::f816:3eff:fec7:7975/64`是IPv6地址，总共64位

# 2 IPv4

<span style="color:red">IP地址是一个网卡在网络世界的通讯地址，相当于我们现实世界的门牌号码。</span>

## 2.1 分类

<center><img src="https://ss.im5i.com/2021/09/30/lTR14.png" alt="lTR14.png" border="0" /></center>

在网络地址中，至少在当时设计的时候，对于A、B、C类主要分两部分，前面一部分是**网络号**，后面一部分是**主机号**。这很好理解，大家都是六单元1001号，我是小区A的六单元1001号，而你是小区B的六单元1001号。

下面这个表格，详细地展示了A、B、C三类地址所能包含的**主机的数量**。

| 类别 |         IP地址范围          | 最大主机数 |        私有IP地址范围         |
| :--: | :-------------------------: | :--------: | :---------------------------: |
|  A   |  0.0.0.0 - 127.255.255.255  |  16777214  |   10.0.0.0 - 10.255.255.255   |
|  B   | 128.0.0.0 - 191.255.255.255 |   65534    |  172.16.0.0 - 172.31.255.255  |
|  C   | 192.0.0.0 - 223.255.255.255 |    254     | 192.168.0.0 - 192.168.255.255 |

这里面有个尴尬的事情，就是**C类地址能包含的最大主机数量实在太少**了，只有254个。当时设计的时候恐怕没想到，现在估计一个网吧都不够用吧。**而B类地址能包含的最大主机数量又太多**了。6万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。

## 2.2 无类型域间选路（CIDR）

**于是有了一个折中的方式叫作无类型域间选路，简称CIDR**。这种方式打破了原来设计的几类地址的做法，**将32位的IP地址一分为二，前面是网络号，后面是主机号**。`10.100.122.2/24`这个IP地址中有一个斜杠，斜杠后面有个数字24。这种地址表示形式，就是CIDR。**后面24的意思是，32位中，前24位是网络号，后8位是主机号**。

伴随着CIDR存在的，一个是**广播地址**，`10.100.122.255`。如果发送这个地址，所有`10.100.122`网络里面的机器都可以收到。另一个是**子网掩码**，`255.255.255.0`。**将子网掩码和IP地址按位计算AND，就可得到网络号**。

## 2.3 公有IP地址和私有IP地址

平时我们看到的数据中心里，办公室、家里或学校的IP地址，**一般都是私有IP地址段**。因为这些地址允许组织内部的IT人员自己管理、自己分配，而且可以重复。因此，你学校的某个私有IP地址段和我学校的可以是一样的。

**公有IP地址有个组织统一分配，你需要去买**。如果你搭建一个网站，给你学校的人使用，让你们学校的IT人员给你一个IP地址就行。但是假如你要做一个类似网易163这样的网站就需要有公有IP地址，**这样全世界的人才能访问。**

表格中的`192.168.0.x`是最常用的私有IP地址。你家里有Wi-Fi，对应就会有一个IP地址。一般你家里地上网设备不会超过256个，所以`/24`基本就够了。有时候我们也能见到`/16`的CIDR，这两种是最常见的，也是最容易理解的。不需要将十进制转换为二进制32位，就能明显看出`192.168.0`是网络号，后面是主机号。而整个网络里面的第一个地址`192.168.0.1`，往往就是你这个**私有网络的出口地址**。**例如，你家里的电脑连接Wi-Fi，Wi-Fi路由器的地址就是192.168.0.1，而192.168.0.255就是广播地址**。一旦发送这个地址，`192.168.0`网络里面的所有机器都能收到。

但是也不总都是这样的情况。因此，其他情况往往就会很难理解，还容易出错。

我们来看`16.158.165.91/22`这个CIDR。求一下这个网络的第一个地址、子网掩码和广播地址。

第一个地址：`16.158.<101001><00>.1`即`16.158.164.1`

子网掩码：`255.255.<111111><00>.0`即`255.255.252.0`

广播地址：`16.158.<101001><11>.255`即`16.158.167.255`

这五类地址中，还有一类D类是组播地址。使用这一类地址，属于某个组的机器都能收到。这有点类似在公司里面大家都加入了一个邮件组。发送邮件，加入这个组的都能收到。组播地址在后面讲述VXLAN协议的时候会提到。

# 3 解析ip addr输出

```shell
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 inet 127.0.0.1/8 scope host lo
 valid_lft forever preferred_lft forever
 inet6 ::1/128 scope host 
 valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
 link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
 inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
 valid_lft forever preferred_lft forever
 inet6 fe80::f816:3eff:fec7:7975/64 scope link 
 valid_lft forever preferred_lft forever
```



## 3.1 ip地址

在IP地址的后面有个**scope**，对于eth0这张网卡来讲，是**global**，说明这张网卡是**可以对外的**，可以接收来自各个地方的包。对于lo来讲，是**host**，说明这张网卡**仅仅可以供本机相互通信**。

**lo 全称是loopback，又称环回接口，往往会被分配到127.0.0.1这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现**。

## 3.2 MAC地址

在IP地址的上一行是`link/ether fa:16:3e:c7:79:75 brd f:f:f:ff:ffiff`，这个被称为MAC地址，是一个**网卡的物理地址**，用十六进制，6个byte表示。

一个网络包要从一个地方传到另一个地方，除了要有确定的地址，还需要有定位功能。而**有门牌号码属性的IP地址，才是有远程定位功能的**。

MAC地址号称全局唯一，MAC地址更像是身份证，是一个唯一的标识。它的唯一性设计是为了组网的时候，不同的网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。

MAC地址是有一定定位功能的，只不过范围非常有限，局限在一个子网里面。例如，从`192.168.0.2/24`访问
`192.168.0.3/24`是可以用MAC地址的。一旦跨子网，即从`192.168.0.2/24`到`192.168.1.2/24`，MAC地址就不行了，需要IP地址起作用了。

## 3.3 网络设备的状态标识

`<BROADCAST,MULTICAST,UP,LOWER_UP>`叫做net_device flags，网络设备的状态标识

**UP**表示网卡处于启动的状态

**BROADCAST**表示这个网卡有广播地址，可以发送广播包

**MULTICAST**表示网卡可以发送多播包

**LOWER _UP**表示L1是启动的，也即网线插着呢

**MTU1500**是指什么意思呢?是哪一层的概念呢？最大传输单元MTU为1500，这是以太网的默认值。

**qdisc pfifo_fast**是什么意思呢？qdisc 全称是queueing discipline，中文叫排队规则。**内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的qdisc(排队规则）把数据包加入队列**。

最简单的qdisc是pfifo，它不对进入的数据包做任何的处理，数据包采用先入先出的方式通过队列。pfifo_fast稍微复杂一些，它的队列包括三个波段(band)。在每个波段里面，使用先进先出规则。三个波段(band)的优先级也不相同。band 0 的优先级最高，band 2的最低。如果band 0里面有数据包，系统就不会处理band 1里面的数据包，band 1和band 2之间也是一样。数据包是按照服务类型(Type of Service，TOS)被分配多三个波段(band)里面的。TOS是IP头里面的一个字段，代表了当前的包是高优先级的，还是低优先级的。

# 4 net-tools和iproute2

net-tools起源于BSD，自2001年起，Linux社区已经对其停止维护，而iproute2旨在取代net-tools，并提供了一些新功能。一些Linux发行版已经停止支持net-tools，只支持iproute2。

net-tools通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。

net-tools中工具的名字比较杂乱，而iproute2则相对整齐和直观，基本是ip命令加后面的子命令。

虽然取代意图很明显，但是这么多年过去了，net-tool依然还在被广泛使用，最好还是两套命令都掌握吧。



# 小结

- IP是地址，有定位功能；MAC是身份证，无定位功能

















