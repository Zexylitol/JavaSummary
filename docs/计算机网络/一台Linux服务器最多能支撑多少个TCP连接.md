TCP连接四元组是**源IP地址、源端口、目的IP地址和目的端口**。任意一个元素发生了改变，那么就代表的是一条完全不同的连接了。拿Nginx举例，它的端口是固定使用80。另外IP也是固定的，这样目的IP地址、目的端口都是固定的。剩下源IP地址、源端口是可变的。所以理论上Nginx上最多可以建立$2^{32}（ip数）\times 2^{16}（port数）$个连接。这是两百多万亿的一个大数字！！

## 端口号

端口号是 16 位的，范围是 1~65535。Linux 对可使用的端口范围是有具体限制的，具体可以用如下命令查看：

```shell
[root]# cat /proc/sys/net/ipv4/ip_local_port_range 
1024 65000
```

## 文件描述符

进程每打开一个文件（linux下一切皆文件，包括socket），都会消耗一定的内存资源。如果有不怀好心的人启动一个进程来无限的创建和打开新的文件，会让服务器崩溃。**所以Linux系统出于安全角度的考虑，在多个位置都限制了可打开的文件描述符的数量**，包括系统级、用户级、进程级：

- 系统级：当前系统可打开的最大数量，通过fs.file-max参数可修改
- 用户级：指定用户可打开的最大数量，修改/etc/security/limits.conf
- 进程级：单个进程可打开的最大数量，通过fs.nr_open参数可修改

## 内存

**TCP接收缓存区大小**是可以配置的，通过`sysctl`命令可以查看：

```shell
$ sysctl -a | grep rmem
net.ipv4.tcp_rmem = 4096 87380 8388608
net.core.rmem_default = 212992
net.core.rmem_max = 8388608
```

**其中tcp_rmem中的第一个值是TCP连接所需分配的最少字节数**。该值默认是4K，最大的话8MB之多。也就是说有数据发送的时候Linux服务器至少为对应的socket再分配4K内存，甚至可能更大

**TCP分配发送缓存区的大小**受参数net.ipv4.tcp_wmem配置影响：

```shell
$ sysctl -a | grep wmem
net.ipv4.tcp_wmem = 4096 65536 8388608
net.core.wmem_default = 212992
net.core.wmem_max = 8388608
```

**在net.ipv4.tcp_wmem中的第一个值是发送缓存区的最小值**，默认也是4K。当然了如果数据很大的话，该缓存区实际分配的也会比默认值大



# Reference

- [一台Linux服务器最多能支撑多少个TCP连接？](https://blog.csdn.net/sqlquan/article/details/111561959)
- [一台Linux服务器最多能支撑多少个TCP连接？](https://blog.csdn.net/qq_16059847/article/details/116102880)