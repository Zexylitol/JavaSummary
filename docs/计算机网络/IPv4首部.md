<table width="60%">
    <tr>
    	<td>0</td>
        <td>1</td>
        <td>2</td>
        <td>3</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
        <td>7</td>
        <td>8</td>
        <td>9</td>
        <td>10</td>
        <td>11</td>
        <td>12</td>
        <td>13</td>
        <td>14</td>
        <td>15</td>
        <td>16</td>
        <td>17</td>
        <td>18</td>
        <td>19</td>
        <td>20</td>
        <td>21</td>
        <td>22</td>
        <td>23</td>
        <td>24</td>
        <td>25</td>
        <td>26</td>
        <td>27</td>
        <td>28</td>
        <td>29</td>
        <td>30</td>
        <td>31</td>   
    </tr>
    <tr align="center">
    	<td colspan="4">Version<br/>版本</td>
        <td colspan="4">IHL<br/>首部长度</td>
        <td colspan="8">TOS<br/>区分服务</td>
        <td colspan="16">Total Length<br/>总长度</td>
    </tr>
    <tr align="center">
    	<td colspan="16">Identification<br/>标识</td>
        <td colspan="3">Flags<br/>标志</td>
        <td colspan="13">Fragment Offset<br/>片偏移</td>
    </tr>
    <tr align="center">
    	<td colspan="8">Time To Live<br/>生存时间</td>
        <td colspan="8">Protocol<br/>协议</td>
        <td colspan="16">Header Checksum<br/>首部校验和</td>
    </tr>
    <tr align="center">
    	<td colspan="32">Source Address<br/>源IP地址</td>
    </tr>
    <tr align="center">
    	<td colspan="32">Destination Address<br/>目标地址</td>
    </tr>
    <tr align="center">
    	<td colspan="16">Options<br/>可选字段</td>
        <td colspan="16">Padding<br/>填充</td>
    </tr>
    <tr align="center">
    	<td colspan="32">Data<br/>数据部分</td>
    </tr>
</table>




# 版本(Version)

- 由4比特构成，表示标识IP首部的版本号。IPv4的版本号即为4，IPv6的版本号为6

# 首部长度(IHL:Internet Header Length)

- 由4比特构成，表明IP首部大小，单位为4字节（32比特），对于没有可选项的IP包，首部长度则设置为“5”，也就是说，当没有可选项时，IP首部的长度为20字节（$4 \times  5=20$）

# 区分服务(TOS:Type Of Service)

- 由8比特构成，用来表明服务质量。每一位的具体含义见下表：

  - | 比特    | 含义       |
    | :------ | :--------- |
    | 0 1 2   | 优先度     |
    | 3       | 最低延迟   |
    | 4       | 最大吞吐   |
    | 5       | 最大可靠性 |
    | 6       | 最小代价   |
    | （3-6） | 最大安全   |
    | 7       | 未定义     |

# 总长度(Total Length)

- 由16比特构成，表示IP首部与数据部分合起来的总字节数，因此IP包的最大长度为$65535(2^{16})$字节
- 目前还不存在能够传输最大长度为65535字节的IP包的数据链路。不过，由于IP有分片处理，从IP的上一层的角度看，不论底层采取何种数据链路，都可以认为能够以IP的最大包长传输数据

# 标识(ID:Identification)

- 由16比特构成，用于分片重组。**同一个分片的标识值相同，不同分片的标识值不同**。通常，每发送一个IP包，它的值也逐渐递增。
- 此外，即使ID相同，如果目标地址、源地址或协议不同的话，也会被认为是不同的分片

# 标志(Flags)

- 由3比特构成，表示包被分片的相关信息。每一位的具体含义见下表：

  - | 比特 | 含义                                                         |
    | :--- | :----------------------------------------------------------- |
    | 0    | 未使用。现在必须是0                                          |
    | 1    | 指示是否进行分片<br/>0-可以分片<br/>1-不能分片               |
    | 2    | 包被分片的情况下，表示是否为最后一个包<br/>0-最后一个分片的包<br/>1-分片中段的包 |

# 片偏移(FO:Fragment Offset)

- 由13比特构成，用来标识被分片的每一个分段相对于原始数据的位置
- 第一个分片对应的值为0
- 最多可以表示$8192(2^{13})$个相对位置
- **单位为8字节**，因此最大可表示原始数据$8 \times 8192 = 65536$字节的位置

# 生存时间(TTL:Time of Live)

- 由8比特构成，**在实际中它是指可以中转多少个路由器的意思；每经过一个路由器，TTL会减少1，直到变为0则丢弃该包**
- 一个包的中转路由的次数不会超过$256(2^8)$次，由此可以避免IP包在网络内无限传递的问题

# 协议(Protocol)

- 由8比特构成，表示的是IP包传输层的上层协议编号

# 首部校验和(Header Checksum)

- 由16比特构成，也叫IP首部校验和
- 该字段只校验数据报的首部，不校验数据部分
- 主要用来确保IP数据报不被破坏

- 校验和的计算过程：
  - 首先要将该校验和的所有位置设置为0，然后以16比特为单位划分IP首部，并用1补数计算所有16位字的和，最后将所得这个和的1补数赋给首部校验和字段

# 源地址(Source Address)

- 由32比特构成，表示发送端IP地址

# 目标地址(Destination Address)

- 由32比特构成，表示接收端IP地址

# 可选项(Options)

- 长度可变，通常只在进行实验或诊断时使用，该字段包含如下几点信息：
  - 安全级别
  - 源路径
  - 路径记录
  - 时间戳

# 填充(Padding)

- 也称作填充物，通过向字段填充0，调整为32比特的整数倍



# 数据(Data)

- 存入数据，将IP上层协议的首部也作为数据进行处理