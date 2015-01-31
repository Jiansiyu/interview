笔试面试知识整理
=============

### 目录

* [网络](#网络)
    
    * [TCP](#tcp)
        
        * [三次握手与四次挥手](#三次握手与四次挥手)
        * [SYN攻击](#SYN攻击)
* [数据结构与算法](#数据结构与算法)
* [体系结构与操作系统](#体系结构与操作系统)
* [编译原理](#编译原理)
* [数据库](#数据库)


网络
----

### TCP

#### 三次握手与四次挥手

* 所谓三次握手(Three-way Handshake)，是指建立一个 TCP 连接时，需要客户端和服务器总共发送3个包。

    三次握手的目的是连接服务器指定端口，建立 TCP 连接，并同步连接双方的序列号和确认号，交换 TCP 窗口大小信息。在 socket 编程中，客户端执行 connect() 时。将触发三次握手。
    
    
    * 第一次握手(SYN=1, seq=x):
    
        客户端发送一个 TCP 的 SYN 标志位置1的包，指明客户端打算连接的服务器的端口，以及初始序号 X,保存在包头的序列号(Sequence Number)字段里。

        发送完毕后，客户端进入 `SYN_SEND` 状态。
        
    * 第二次握手(SYN=1, ACK=1, seq=y, ACKnum=x+1):
    
        服务器发回确认包(ACK)应答。即 SYN 标志位和 ACK 标志位均为1。服务器端选择自己 ISN 序列号，放到 Seq 域里，同时将确认序号(Acknowledgement Number)设置为客户的 ISN 加1，即X+1。 
        发送完毕后，服务器端进入 `SYN_RCVD` 状态。

    * 第三次握手(ACK=1，ACKnum=y+1)
        
        客户端再次发送确认包(ACK)，SYN 标志位为0，ACK 标志位为1，并且把服务器发来 ACK 的序号字段+1，放在确定字段中发送给对方，并且在数据段放写ISN的+1
        
        发送完毕后，客户端进入 `ESTABLISHED` 状态，当服务器端接收到这个包时，也进入 `ESTABLISHED` 状态，TCP 握手结束。

* TCP的连接的拆除需要发送四个包，因此称为四次挥手(Four-way handshake)。客户端或服务器均可主动发起挥手动作，在 socket 编程中，任何一方执行 close() 操作即可产生挥手操作。

    * 第一次挥手(FIN=1，seq=x)
        
        假设客户端想要关闭连接，客户端发送一个 FIN 标志位置为1的包，表示自己已经没有数据可以发送了，但是仍然可以接受数据。
        
        发送完毕后，客户端进入 `FIN_WAIT_1` 状态。
        
    * 第二次挥手(ACK=1，ACKnum=x+1)
    
        服务器端确认客户端的 FIN 包，发送一个确认包，表明自己接受到了客户端关闭连接的请求，但还没有准备好关闭连接。
        
        发送完毕后，服务器端进入 `CLOSE_WAIT` 状态，客户端接收到这个确认包之后，进入 `FIN_WAIT_2` 状态，等待服务器端关闭连接。
        
    * 第三次挥手(FIN=1，seq=y)

        服务器端准备好关闭连接时，向客户端发送结束连接请求，FIN 置为1。
        
        发送完毕后，服务器端进入 `LAST_ACK` 状态，等待来自客户端的最后一个ACK。
        
    * 第四次挥手(ACK=1，ACKnum=y+1)
    
        客户端接收到来自服务器端的关闭请求，发送一个确认包，并进入 `TIME_WAIT `状态，等待可能出现的要求重传的 ACK 包。
        
        服务器端接收到这个确认包之后，关闭连接，进入 `CLOSED` 状态。
        
        客户端等待了某个固定时间之后，没有收到服务器端的 ACK ，认为服务器端已经正常关闭连接，于是自己也关闭连接，进入 `CLOSED` 状态。
        

#### SYN攻击

* 什么是 SYN 攻击（SYN Flood）？

    在三次握手过程中，服务器发送 SYN-ACK 之后，收到客户端的 ACK 之前的 TCP 连接称为半连接(half-open connect)。此时服务器处于 SYC_RCVD 状态。当收到 ACK 后，服务器才能转入 ESTABLISHED 状态.

    SYN 攻击指的是，攻击客户端在短时间内伪造大量不存在的IP地址，向服务器不断地发送SYN包，服务器回复确认包，并等待客户的确认。由于源地址是不存在的，服务器需要不断的重发直至超时，这些伪造的SYN包将长时间占用未连接队列，正常的SYN请求被丢弃，导致目标系统运行缓慢，严重者会引起网络堵塞甚至系统瘫痪。
    
    SYN 攻击是一种典型的 DoS/DDoS 攻击。
    
* 如何检测 SYN 攻击？

     检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。在 Linux/Unix 上可以使用系统自带的 netstats 攻击来检测 SYN 攻击。
     
* 如何防御 SYN 攻击？

    SYN攻击不能完全被阻止，除非将TCP协议重新设计。我们所做的是尽可能的减轻SYN攻击的危害，常见的防御 SYN 攻击的方法有如下几种：
    
    * 缩短超时（SYN Timeout）时间
    * 增加最大半连接数
    * 过滤网关防护
    * SYN cookies技术
     

     
参考: 

* 计算机网络：自顶向下方法
* http://www.cnblogs.com/hnrainll/archive/2011/10/14/2212415.html
* http://www.cnblogs.com/rootq/articles/1377355.html
* http://blog.csdn.net/whuslei/article/details/6667471
* 百度百科：SYN攻击

数据结构和算法
------------

体系结构和操作系统
---------------

编译原理
-------

数据库
-----


