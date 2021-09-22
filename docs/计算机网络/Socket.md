我们知道两个进程如果需要进行通讯最基本的一个前提是能够唯一的标示一个进程，在本地进程通讯中我们可以使用PID来唯一标示一个进程，但PID只在本地唯一，网络中的两个进程PID冲突几率很大，这时候我们需要另辟它径了，我们知道**IP层的ip地址可以唯一标示主机，而TCP层协议和端口号可以唯一标示主机的一个进程，这样可以利用ip地址＋协议＋端口号唯一标示网络中的一个进程**。

能够唯一标示网络中的进程后，它们就可以利用socket进行通信了，什么是socket呢？我们经常把socket翻译为套接字，<span style="color:red">socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信</span>。

<center><img src="https://ss.im5i.com/2021/09/22/lf0Ss.png" alt="lf0Ss.png" border="0" /></center>

socket起源于UNIX，在Unix一切皆文件的思想下，socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。

tcp服务端和tcp客户端使用socket通信的过程如下。从图中可以看到，socket连接可以保持长连接。

<center><img src="https://ss.im5i.com/2021/09/22/lfDuQ.png" alt="lfDuQ.png" border="0" /></center>

# Reference

- [tcp、http和socket的区别](https://www.cnblogs.com/leftJS/p/11082910.html)
- [简单理解socket及TCP/IP、HTTP、socket的区别](https://blog.csdn.net/u012359618/article/details/53353505/)