---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2020-02-13T03:38:51+08:00'
description: 详解linux中TCP连接与断开与其中的状态
lastmod: '2020-04-13T03:38:57+08:00'
showToc: true
tags: [Linux, Network]
title: TCP连接和各状态浅析
---

# TCP连接和各状态浅析

> TCP协议中三次握手连接和四次挥手断开和其中的状态变化

[TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)(Transmission Control Protocol)协议在[OSI模型](https://en.wikipedia.org/wiki/OSI_model)中属于传输层的协议(第4层)也是最常用的互联网协议，与[UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol)相比它是一种可靠的传输协议，建立和断开连接中一共有11种可能发生的状态(states)。

> Linux系统中可以用`ss -nat`命令查看所有TCP连接情况如果是旧的系统没有此命令可以使用`netstat`代替只是输出格式不一致，统计当前所有TCP状态可以加管道实现
> `ss -nat | awk 'NR>1 { d[$1]++ } END { for (i in d) print i, d[i] }'`

## 建立连接(三次握手）

TCP是面向连接的所以传输双方在传输前要建立一条连接后才能通信，建立连接后TCP协议提供全双⼯(就是可以同时发送和接收不影响)的通信服务。建立连接的过程我们给它取了个生动的名字叫“三次握手”，因为从发起到建立连接一共有三个步骤也就是双方会发三个包，先看下面的图

![3-way-handshake](../images/3-way-handshake.jpg)

1. 首先客户端发送`SYN`(Synchronous)包到服务端告诉服务器开始建立连接，包里面会包含一个随机数(`j`)序列号(Sequence Number)，此时客户端进入`SYC-SENT`状态。当然最初服务器是处于`LISTEN`状态监听某一个端口，本文图片以绿底黑框的都代表TCP状态。
2. 服务端收到`SYN`发送应答`SYN-ACK(Acknowledgement)`携带确认号码(Acknowledge Number)为收到的序列号+1也就是`j+1`，加上一个自己的`SYC`序列号随机数(`k`)，服务端把状态置为`SYN-RECEIVED`
3. 客户端收到`SYN-ACK`包并发送确认号码`k+1`，此时客户端的状态为`ESTABLISHED`，服务器收到`ACK`后状态也变为`ESTABLISHED`

以下是wireshake抓包后TCP连接的三次握手，这里序列号码`j`和`k`都为`0`。

![wireshake-3-handshake](../images/wireshark-3-way-handshake.png)

```
    TCP A (port 65525)                                     TCP B (port 443)

                                                           LISTEN
       (Start)
  1.  SYN-SENT    --> <SEQ=0><CTL=SYN>                 --> SYN-RECEIVE

  2.  ESTABLISHED <-- <SEQ=0><ACK=1><CTL=SYN,ACK>      <-- SYN-RECEIVE

  3.  ESTABLISHED --> <SEQ=1><ACK=1><CTL=ACK>          --< ESTABLISHED

```

## 断开连接(四次挥手)

断开连接称为“挥手”一共有四步每一方向占两步，由于TCP是全双工的所以关闭连接需要双方都完成关闭(close)才算结束，其中会涉及状态也就比握手多。客户端和服务端都可以发起挥手动作，先发送`FIN`包的动作叫主动关闭(active close)然后另一方回应`ACK`包叫执行被动关闭(passive colse)，双方各完成一次主动和被动关闭一共四次。

![4-way-handshake](../images/4-way-handshake.jpg)

1. 客户端数据传输完毕发起断开连接(主动关闭)，发送`FIN`包携带一个序列号(`x`)到服务端并更新自己的状态为`FIN-WAIT-1`
2. 服务端收到`FIN`包返回`ACK`包(被动关闭)，应答ACK确认序号为`x+1`，服务端此时的状态为`CLOSE-WAIT`。另一边客户端收到`ACK`更新状态为`FIN-WAIT-2`
3. 服务端准备执行断开连接(主动关闭)，此时服务端的状态还是`CLOSE-WAIT`，发送`FIN`包携带序列号(`y`)到客户端更新服务端状态为`LAST-ACK`，客户端收到`FIN`包后状态变为`TIME-WAIT`。
4. 客户端收到`FIN`应答`ACK`(被动关闭)携带确认序号为`y+1`，服务端收到`ACK`立即关闭连接状态变为`CLOSED`
5. 因为客户端最后不知道服务端有没有收到`ACK`包，所以默认等待两倍的[MSL](https://en.wikipedia.org/wiki/Maximum_segment_lifetime)(Maximum Segment Lifetime)，Linux上MSL一般为60s，等待120s使服务器如果没有收到`ACK`也能重传`FIN`包，最后把自己的状态置为`CLOSED`在此之前的MSL时间中一直是`TIME-WAIT`状态

看一个现实中的例子ssh连接退出

![wireshake-4-handshake](../images/wireshark-4-way-handshake.png)

```
    TCP A (port 49612)                                     TCP B (port 22)

  1.  ESTABLISHED                                          ESTABLISHED

  2.  (Close)
      FIN-WAIT-1  --> <SEQ=277><ACK=401><CTL=FIN,ACK>  --> CLOSE-WAIT

  3.  FIN-WAIT-2  <-- <SEQ=401><ACK=278><CTL=ACK>      <-- CLOSE-WAIT

  4.                                                       (Close)
      TIME-WAIT   <-- <SEQ=401><ACK=278><CTL=FIN,ACK>  <-- LAST-ACK

  5.  TIME-WAIT   --> <SEQ=278><ACK=402><CTL=ACK>      --< CLOSED

  6.  (2 MSL)
      CLOSED
```

## TCP状态迁移图

TCP一共有11种状态，虚线箭头表示客户端状态转换轨迹，实线表示服务端，但这里的客户端和服务端不是绝对的可以互换。

![TCP state transition diagram](../images/TCP-state-transition-diagram.gif)

### CLOSING状态

仔细看你会发现有一个`CLOSING`状态有些陌生，因为以上介绍的三次握手和四次挥手都是正常且顺序连接或断开中TCP状态，现实应用中比较复杂上面还有一个状态没有涉及到就是`CLOSING`

从上面状态转换图中可看到`CLOSING`状态发生的情况是在首先发起挥手的一方(比如说客户端)正常如上图在`FIN-WAIT-1`后会进入`FIN-WAIT-2`，可进入`FIN-WAIT-2`是有条件的那就是收到第一次发`FIN`后服务端返回的`ACK`，可此时客户端没有收到`ACK`反而收到了服务端的`FIN`包，按上面讲的这就乱序了，这种特殊情况就是`CLOSING`状态，也就是说进入`CLOSING`有下面三个条件

- 主动关闭发送`FIN`包发起后(此时是`FIN-WAIT-1`)
- 没有收到另一方的`ACK`包等待中
- 收到另一方发送过来的 `FIN`包

### 状态表

现在对所有状态做个总结

| TCP connection state |                                          Description                                           |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| LISTEN               | SOCKET监听中等待远程连接                                                                           |
| SYN-SENT             | 发送连接请求SYN包后等待SYN-ACK包，第一次握手                                                          |
| SYN-RECEIVED         | 连接收到SYN包到并返回SYN-ACK包等待远端ACK，第二次握手                                                  |
| ESTABLISHED          | 连接已经建立，双方可以传输数据了,第三次握手                                                            |
| FIN-WAIT-1           | 发起主动关闭发送FIN包后等待ACK包,第一次挥手，另一方一般很快返回很少见到                                     |
| CLOSE-WAIT           | 被动关闭收到对方FIN包，返回发送ACK包，在没有发起主动关闭之前一直使这个状态，第二次挥手                         |
| FIN-WAIT-2           | 收到发起主动关闭返回的ACK包，等待对方发送FIN包，现在只能接收数据不能发送数据，也叫半关闭状态，持续时间短，第二次挥手 |
| LAST-ACK             | 进行一次被动关闭后，发起主动关闭等待ACK，第三次挥手                                                      |
| TIME-WAIT            | 主动关闭执行后，收到FIN包(第三次挥手)，并返回ACK(第四次挥手) 等待两倍的MSL之后进入CLOSED                     |
| CLOSING             | 见上面                                                                                          |
| CLOSED              | 连接关闭没有连接                                                                                  |



## Reference

1. [www.ibm.com](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.1.0/com.ibm.zos.v2r1.halu101/constatus.htm)
2. [community.apigee.com](https://community.apigee.com/articles/7970/tcp-states-explained.html)
3. [blog.confirm.ch](https://blog.confirm.ch/tcp-connection-states/)
4. [blog.csdn.net](https://blog.csdn.net/dog250/article/details/52070680)