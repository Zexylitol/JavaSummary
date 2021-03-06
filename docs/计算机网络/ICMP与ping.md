# 0 前言

无论是在宿舍，还是在办公室，或者运维一个数据中心，我们常常会遇到**网络不通的问题**。那台机器明明就在那里，你甚至都可以通过机器的终端连上去看。它看着好好的，可是就是连不上去，究竟是哪里出了问题呢?

一般情况下，你会想到ping一下。那你知道ping是如何工作的吗?

# 1 ICMP协议的格式

<span style="color:red">ping是基于ICMP协议工作的。ICMP全称Internet Control Message Protocol，就是互联网控制报文协议</span>。

网络包在异常复杂的网络环境中传输时，常常会遇到各种各样的问题。当遇到问题的时候，总不能“死个不明不白”，要传出消息来，报告情况，这样才可以调整传输策略。

**ICMP报文是封装在IP包里面的。因为传输指令的时候，肯定需要源地址和目标地址**。它本身非常简单。因为作为侦查兵，要轻装上阵，不能携带大量的包袱。

<center><img src="https://ss.im5i.com/2021/09/29/l5Yrl.png" alt="l5Yrl.png" border="0" /></center>

ICMP报文有很多的类型，不同的类型有不同的代码。最常用的类型是主动请求为8，主动请求的应答为0。

## 1.1 查询报文类型

**常用的ping就是查询报文，是一种主动请求**，并且**获得主动应答**的ICMP协议。所以，ping 发的包也是符合ICMP协议格式的，只不过它在后面增加了自己的格式。

对ping的主动请求，进行网络抓包，称为ICMP ECHO REQUEST。同理主动请求的回复，称为ICMPECHO REPLY。比起原生的ICMP，这里面多了两个字段，一个是标识符。这个很好理解，你派出去两队侦查兵，一队是侦查战况的，一队是去查找水源的，要有个标识才能区分。另一个是序号，你派出去的侦查兵，都要编个号。如果派出去10个，回来10个，就说明前方战况不错;如果派出去10个，回来2个，说明情况可能不妙。

在选项数据中，ping还会存放发送请求的时间值，来**计算往返时间**，说明路程的长短。

## 1.2 差错报文类型

ICMP的差错报文类型：这种是异常情况发起的，来报告发生了不好的事情

举几个ICMP差错报文的例子：终点不可达为3，源抑制为4，超时为11，重定向为5。

**终点不可达**

具体的原因在代码中表示就是，网络不可达代码为0，主机不可达代码为1，协议不可达代码为2，端口不可达代码为3，需要进行分片但设置了不分片位代码为4。

具体的场景就像这样：

- 网络不可达：主公，找不到地方呀?
- 主机不可达：主公，找到地方没这个人呀?
- 协议不可达：主公，找到地方，找到人，口号没对上，人家天王盖地虎，我说12345!
- 端口不可达：主公，找到地方，找到人，对了口号，事儿没对上，我去送粮草人家说他们在等救兵。
- 需要进行分片但设置了不分片位：主公，走到一半，山路狭窄，想换小车，但是您的将令，严禁换小车，就没办法送到了。

**源站抑制**

也就是让源站放慢发送速度

**时间超时**

也就是超过网络包的生存时间还是没到。

**路由重定向**

也就是让下次发给另一个路由器。

**差错报文的结构相对复杂一些。除了前面还是IP，ICMP 的前8字节不变，后面则跟上出错的那个IP包的IP头和IP正文的前8个字节。**

# 2 ping：查询报文类型的使用

<center><img src="https://ss.im5i.com/2021/09/29/l5o42.png" alt="l5o42.png" border="0" /></center>

假定主机A的IP地址是192.168.1.1，主机B的IP地址是192.168.1.2，它们都在同一个子网。那当你在主机A上运行“ping 192.168.1.2”后，会发生什么呢?

ping 命令执行的时候，源主机首先会构建一个ICMP 请求数据包，ICMP数据包内包含多个字段。最重要的是两个，**第一个是类型字段**，对于请求数据包而言该字段为8；**另外一个是顺序号**，主要用于区分连续ping的时候发出的多个数据包。每发出一个请求数据包，顺序号会自动加1。为了能够计算往返时间RTT，它会在报文的数据部分插入发送时间。

**然后，由ICMP协议将这个数据包连同地址192.168.1.2一起交给IP层。IP层将以192.168.1.2作为目的地址，本机IP地址作为源地址，加上一些其他控制信息，构建一个IP数据包**。

接下来，需要加入MAC头。如果在本节ARP映射表中查找出IP地址192.168.1.2所对应的MAC地址，则可以直接使用；如果没有，则需要发送ARP协议查询MAC地址，获得MAC地址后，由数据链路层构建一个数据帧，目的地址是IP层传过来的MAC地址，源地址则是本机的 MAC地址；还要附加上一些控制信息，依据以太网的介质访问规则，将它们传送出去。

主机B收到这个数据帧后，先检查它的目的MAC地址，并和本机的MAC地址对比，如符合，则接收，否则就丢弃。接收后检查该数据帧，将IP数据包从帧中提取出米，交给本机的IP层。同样，IP层检查后，将有用的信息提取后交给ICMP协议。

主机B 会构建一个ICMP应答包，应答数据包的类型字段为0，顺序号为接收到的请求数据包中的顺序号，然后再发送出去给主机A。

在规定的时候间内，源主机如果没有接到ICMP的应答包，则说明目标主机不可达；如果接收到了ICMP应答包，则说明目标主机可达。此时，源主机会检查，用当前时刻减去该数据包最初从源主机上发出的时刻，就是ICMP数据包的时间延迟。

当然这只是最简单的，同一个局域网里面的情况。**如果跨网段的话，还会涉及网关的转发、路由器的转发等等**。但是对于ICMP的头来讲，是没什么影响的。会影响的是根据目标IP地址，选择路由的下一跳，还有每经过一个路由器到达一个新的局域网，需要换MAC头里面的MAC地址。

# 3 Traceroute：差错报文类型的使用

**Traceroute的第一个作用就是故意设置特殊的TTL，来追踪去往目的地时沿途经过的路由器**。Traceroute 的参数指向某个目的IP地址，它会发送一个UDP的数据包。将TTL设置成1，也就是说一旦遇到一个路由器或者一个关卡，就表示它“牺牲”了。

如果中间的路由器不止一个，当然碰到第一个就“牺牲”。于是，**返回一个ICMP包，也就是网络差错包，类型是时间超时**。那大军前行就带一顿饭，试一试走多远会被饿死，然后找个哨探回来报告，那我就知道大军只带一顿饭能走多远了。

接下来，将TTL设置为2。第一关过了，第二关就“牺牲”了，那我就知道第二关有多远。如此反复，直到到达目的主机。**这样，Traceroute就拿到了所有的路由器IP**。当然，**有的路由器压根不会回这个ICMP。这也是Traceroute 一个公网的地址，看不到中间路由的原因**。

怎么知道UDP有没有到达目的主机呢? Traceroute程序会发送一份UDP数据报给目的主机但它会**选择一个不可能的值作为UDP端口号**(大于30000)。当该数据报到达时，将使目的主机的UDP模块产生一份“端口不可达”错误ICMP 报文。如果数据报没有到达，则可能是超时。

**Traceroute还有一个作用是故意设置不分片，从而确定路径的MTU**。要做的工作首先是发送分组，并设置“不分片”标志。发送的第一个分组的长度正好与出口MTU相等。如果中间遇到窄的关口会被卡住，会发送ICMP网络差错包，类型为“需要进行分片但设置了不分片位”。其实，这是人家故意的好吧，每次收到ICMP“不能分片”差错时就减小分组的长度，直到到达目标主机。

























