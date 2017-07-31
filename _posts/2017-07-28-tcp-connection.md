---
layout: post
title: "计算机网络 —— tcp连接和断开"
subtitle: "三次握手，四次挥手详解"
date: 2017-07-28
author: "ChenJY"
header-img: "img/db.jpg"
catalog: true
tags: 
    - 网络
    - 面试
---

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px;" href="http://unsplash.com/@berry807?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Berry van der Velden"><span style="display:inline-block;padding:2px 3px;"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white;" viewBox="0 0 32 32"><title></title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px;">Berry van der Velden</span></a>

### 三次握手
![image](http://blog.chinaunix.net/attachment/201310/1/28263175_1380611229SPU6.png)
B 的 TCP 服务器进程先创建<b>传输控制块 TCB</b>，准备接受客户进程的连接请求，然后服务器进行就处于 <b>LISTEN</b> 状态，等待客户的连接请求。

A 的 TCP 客户进程也是首先创建传输控制块 TCB，然后向B发出连接请求报文段，这时首部中的<b>同步位 SYN = 1</b>，同时选择一个初始序号<b>seq = x</b>。TCP 规定，SYN 报文段（即 SYN = 1 的报文段）不能携带数据，但要消耗掉一个序号，此时，客户端进入 <b>SYN-SENT（同步已发送）</b>状态。

B 收到连接请求报文之后，若同意连接，则向 A 发送确认，在确认报文段中应把 <b>SYN 和 ACK</b> 位都置为1，确认序号是 <b>ack = x + 1</b>，同时也要为自己选择一个初始序号 <b>seq = y</b>。请注意，这个报文段也不能携带数据，但同样要消耗一个序号，这时，TCP 服务器进程进入 <b>SYN-REVD</b> 状态。

TCP 客户进程收到 B 的确认之后，还要向 B 给出确认。确认报文段的 <b>ACK 置为 1</b>，确认号 <b>ack = y + 1</b>，而自己的序号 <b>seq = x + 1</b>。TCP 的标准规定，ACK 报文段可以携带数据，但如果不携带数据则不消耗序号，在这种情况下，下一个数据报文段的序号仍然是 seq = x + 1。这时 TCP 连接已经建立，A 进入 <b>ESTABLISHED</b> 状态。当 B 收到 A 的确认之后，也进入 <b>ESTABLISHED</b> 状态。

上述称为三次握手

### 为什么要三次？
采用三次握手是为了<b>防止失效的连接请求报文段突然又传送到主机 B</b>，因而产生错误。失效的连接请求报文段是指：主机 A 发出的连接请求没有收到主机 B 的确认，于是经过一段时间后，主机 A 又重新向主机 B 发送连接请求，且建立成功，顺序完成数据传输。考虑这样一种特殊情况，主机 A 第一次发送的连接请求并没有丢失，而是因为网络节点导致延迟达到主机 B，主机 B 以为是主机 A 又发起的新连接，于是主机 B 同意连接，并向主机 A 发回确认，但是此时主机 A 根本不会理会，主机 B 就一直在等待主机 A 发送数据，导致主机 B 的资源浪费。

### 四次挥手
![image](http://blog.chinaunix.net/attachment/201310/1/28263175_13806112555gGu.png)
四次挥手过程稍微复杂一些，还是结合双方状态来阐述一下。
数据传输结束之后，通信双方都可以释放连接，现在 A 和 B 均处于释放报文阶段，并停止再发送数据，主动关闭 TCP 连接。A 的应用进程先向其 TCP 发出连接释放报文段，并停止再发送数据，主动关闭 TCP 连接。

A 把释放报文段首部的 <b>FIN 置为1</b>，其序号为 <b>seq = u</b>，他等于前面一传送过的数据的最后一个字节的序号加 1。这时 A 进入 <b>FIN-WAIT 1</b> 状态，等到 B 的确认。请注意，TCP 规定，FIN 报文段即使不携带数据，他也消耗掉一个序号。

B 收到连接释放报文段之后，即发出确认，确认号是 <b>ack = u + 1</b>，而这个报文段自己的序号是 v， 等于 B 前面已传送过的数据的最后一个字节的序号加 1。然后 B 就进入 <b>CLOSE-WAIT</b> 状态。TCP 服务器进程这时候通知高层应用进程，因而从 A 到 B 这个方向的连接就释放了，这时的 TCP 连接处于<b>半关闭状态（half-close）</b>，即 A 已经没有数据要发送了，但 B 若发送数据，A 仍需要接收。也就是说，从 B 到 A 这个方向的连接并未关闭，这个状态可能持续一段时间。

A收到来自B的确认后，就进入FIN-WAIT2状态 ，等待B发出的连接释放报文段。

若 B 已经没有要向 A 发送的数据了，其应用进程就通知 TCP 释放链接，这时 B 发出的连接释放报文段必须使 <b>FIN = 1</b>。现假定 B 的序号为 w（因为半关闭状态期间可能 B 还发送了一些数据）。B 还必须重复上次已发送过的确认号 <b>ack = u + 1</b>。这时 B 就进入了 <b>LAST-ACK（最后确认状态）状态</b>，等待 A 的确认。

A 在收到 B 的连接释放报文段之后，必须对此发出确认。在确认报文段中把 <b>ACK 置为 1</b>，确认号 <b>ack = w + 1</b>，而自己的序列号是 <b>seq = u + 1</b>（因为根据 TCP 标准，前面发送过的 FIN 报文段要消耗一个序列号）。然后进入到 <b>TIME-WAIT（时间等待）状态</b>。请注意，现在 TCP 连接还没有释放，必须经过时间等待计时器设置的时间 <b>2 MSL</b> 后，A 才进入 <b>CLOSED</b> 状态。时间 <b>MSL</b> 叫做<b>最长报文段寿命</b>，RPC 793 建议设置为 2 分钟，但实际工程中可根据实际情况调整。因此，从 A 进入到 <b>TIME-WAIT</b> 状态后，要经过 4 分钟才能进入 CLOSED 状态，才能开始建立下一个连接。当 A 撤销控制传输快 TCB 后，就结束了这次的 TCP 连接。

### 为什么A在TIME-WAIT之后要等待2MSL呢？
这里有两个理由：
1. 保证 A 发送的最后一个 ACK 报文能到达 B。这个 ACK 报文很有可能丢失，因而使得处在 LAST-ACK 状态的 B 收不到对已发送的 FIN+ACK 报文段的确认。B 会超时重传这个 FIN + ACK 报文段，而 A 就能在 2 MSL 时间内收到这个重传的 FIN + ACK 报文段。接着 A 重传一次确认，重新启动 2 MSL 计时器。最后 A B 均能进入正常的 CLOSED 状态。

2. 防止上一节提到的 “已失效的连接请求报文段” 出现在本链接中，A 在发送完最后一个 ACK 报文后，再经过时间 2 MSL，就可以使得本链接持续时间内所产生的所有报文段都从网络中消失。这样就可以使得下一个新连接中不会出现这种旧的连接请求报文段。

### 保活计时器
客户已主动与服务器建立 TCP 连接，但后来客户端出现故障，服务器收不到客户端的数据，因此服务器不能白白等下去，服务器每收到客户端的消息就重新设置保活计时器，通常是两小时，若两小时没再收到客户端发来的数据，服务器就发送探测报文段，以后每隔 75 分钟发送一次，若一连发送 10 个探测报文段仍然没有客户端的响应，服务器就认为客户端出现了故障，接着就关闭这个链接。

### TCP的有限状态机
![image](http://www.cnitblog.com/images/cnitblog_com/wildon/544465b00200001s.png)

### 参考资料
* 《计算机网络 第五版》 谢希仁著
