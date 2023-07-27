# WebRTC 入门教程（二）| WebRTC信令控制与STUN/TURN服务器搭建 
> 作者：李超，音视频技术专家。本文首发于 [RTC 开发者社区](https://rtcdeveloper.com/t/topic/13742)，欢迎在社区留言与作者交流。

本文将向大家介绍两个方面的知识：

*   WebRTC信令控制
*   STUN/TURN服务器的搭建

在[前面的文章中](https://webrtc.org.cn/webrtc-tutorial-1-setup-signaling/)已经向大家介绍了如何构建信令服务器。但构建的信令服务器是如何工作的？那些消息需要信令服务器控制和中转？这些此前并没有做详细的说明，而本文将对这些问题做详细的讨论。

另一方面，在真实的网络中，WebRTC是如何进行NAT穿越的呢？如果穿越不成功，我们又该如何保证用户服务的呢？这些知识也将在本文中给出答案。

信令
--

WebRTC 信令控制的架构图如下所示：

[![](https://webrtc.org.cn/wp-content/uploads/2019/04/2db6189b1b416d0a4b8ddad740780fc484eaa5d3.png)
](https://webrtc.org.cn/wp-content/uploads/2019/04/2db6189b1b416d0a4b8ddad740780fc484eaa5d3.png)

信令服务器用于交换三种类型的信息：

*   会话控制消息：初始化/关闭，各种业务逻辑消息以及错误报告。
*   网络相关：外部可以识别的IP地址和端口。
*   媒体能力：客户端能控制的编解码器、分辩率，以及它想与谁通讯。

下面我们就来详细讨论一下这三类消息：

### 会话控制消息

会话控制消息比较简单，像房间的创建与销毁、加入房间、离开房间、开启音频/关闭音频、开启视频/关闭视频等等这些都是会话控制消息。

对于一个真正商业的WebRTC信令服务器，还有许多的会话控制消息。像获取房间人数、静音/取消静音、切换主讲人、视频轮询、白板中的画笔、各种图型等等。但相对来说都是一引起比较简单的消息。

在我们之前的例子中，服务端只处理了一个会话消息 **create or join**，即房间的创建与加入消息。代码如下：

socket.on('create or join', function(room)  {

var clientsInRoom = io.sockets.adapter.rooms\[room\];

var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length  :  0;

logger.debug('Client ID ' \+ socket.id \+ ' created room ' \+ room);

socket.emit('created', room, socket.id);

}  else  if  (numClients === 1)  {

io.sockets.in(room).emit('join', room);

socket.emit('joined', room, socket.id);

io.sockets.in(room).emit('ready');

}  else  { // max two clients

socket.emit('full', room);

... socket.on('create or join', function(room) { var clientsInRoom = io.sockets.adapter.rooms\[room\]; var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length : 0; if (numClients === 0) { socket.join(room); logger.debug('Client ID ' + socket.id + ' created room ' + room); socket.emit('created', room, socket.id); } else if (numClients === 1) { io.sockets.in(room).emit('join', room); socket.join(room); socket.emit('joined', room, socket.id); io.sockets.in(room).emit('ready'); } else { // max two clients socket.emit('full', room); } }); ...

```
...

socket.on('create or join', function(room) {

    var clientsInRoom = io.sockets.adapter.rooms\[room\];
    var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length : 0;

    if (numClients === 0) {
      socket.join(room);
      logger.debug('Client ID ' + socket.id + ' created room ' + room);
      socket.emit('created', room, socket.id);

    } else if (numClients === 1) {
      io.sockets.in(room).emit('join', room);
      socket.join(room);
      socket.emit('joined', room, socket.id);
      io.sockets.in(room).emit('ready');
    } else { // max two clients
      socket.emit('full', room);
    }
});
  
...
```

该代码的逻辑非常简单，当收到 **create or join** 消息后，判断房间里当前人数，如果房间里的人数为 0，说明是第一个人进来，此时，需要向连接的客户端发送 **created** 消息；如果房间里的人数为 1，说明是第二个人进来，需要向客户端发送 **joined**消息；否则发送 **full** 消息，说明房间已满，因为目前一个房间最多只允许有两个人。

### 网络信息消息

网络信息消息用于两个客户端之间交换网络信息。在WebRTC中使用 ICE 机制建立网络连接。

在WebRTC的每一端，当创建好 RTCPeerConnection 对象，且调用了setLocalDescription 方法后，就开始收集 **ICE候选者** 了。

在WebRTC中有三种类型的**候选者**，它们分别是：

*   主机候选者
*   反射候选者
*   中继候选者

**主机候选者**，表示的是本地局域网内的 IP 地址及端口。它是三个候选者中优先级最高的，也就是说在 WebRTC 底层，首先会偿试本地局域网内建立连接。

**反射候选者**，表示的是获取 NAT 内主机的外网IP地址和端口。其优先级低于 **主机候选者**。也就是说当WebRTC偿试本地连接不通时，会偿试通过**反射候选者**获得的 IP地址和端口进行连接。

其结构如下图所示：

[![](https://webrtc.org.cn/wp-content/uploads/2019/04/be430b75a7c0b76e1f1685daf656520754a99f94.png)
](https://webrtc.org.cn/wp-content/uploads/2019/04/be430b75a7c0b76e1f1685daf656520754a99f94.png)

在上面这幅图中可以看到，WebRTC通过 STUN server 获得自己的外网IP和端口，然后通过信令服务器与远端的WebRTC交换网络信息。之后双方就可以偿试建立 P2P 连接了。

以上就是我们通常所说的 P2P NAT 穿越。在WebRTC内部会探测用户的 NAT 类型，最终采用不同的方法进行 NAT 穿越。不过，如果双方都是 **对称NAT** 类型，是无法进行 P2P NAT 穿越的，此时只能使用中继了。

**中继候选者**，表示的是中继服务器的IP地址与端口，即通过服务器中转媒体数据。当WebRTC客户端通信双方无法穿越 P2P NAT 时，为了保证双方可以正常通讯，此时只能通过服务器中转来保证服务质量了。

所以 **中继候选者**的优先级是最低的，只有上述两种候选者都无法进行连接时，才会使用它。

在 WebRTC 信令服务器端，收到网络消息信令，即 **message** 消息时，不做任何处理，直接转发。代码如下：

socket.on('message', function(message)  {

socket.broadcast.emit('message', message);

socket.on('message', function(message) { socket.broadcast.emit('message', message); });

```
socket.on('message', function(message) {
     socket.broadcast.emit('message', message);
});
```

客户端接收到 **message** 消息后，会做进一步判断。如果消息类型为 **candidate**，即 网络消息信令时，会生成 RTCIceCandidate 对象，并将其添加到 RTCPeerConnection 对象中，从而使 WebRTC 在底层自动建立连接。 其代码如下：

socket.on('message', function(message)  {

}  else  if  (message.type === 'candidate')  {

var candidate = new  RTCIceCandidate({

sdpMLineIndex: message.label,

candidate: message.candidate

pc.addIceCandidate(candidate);

socket.on('message', function(message) { ... } else if (message.type === 'candidate') { var candidate = new RTCIceCandidate({ sdpMLineIndex: message.label, candidate: message.candidate }); pc.addIceCandidate(candidate); } else if (...) { ... } });

```
socket.on('message', function(message) {
  ...
  } else if (message.type === 'candidate') {
    var candidate = new RTCIceCandidate({
      sdpMLineIndex: message.label,
      candidate: message.candidate
    });
    pc.addIceCandidate(candidate);
  } else if (...) {
    ...
  }
});
```

### 交换媒体能力消息

在WebRTC中，媒体能力最终通过 SDP 呈现。在传输媒体数据之前，首先要进行媒体能力协商，看双方都支持那些编码方式，支持哪些分辨率等。协商的方法是通过信令服务器交换媒体能力信息。

[![](https://webrtc.org.cn/wp-content/uploads/2019/04/a13b245a9b7f304a64990ab3d6b78bc5c3fd5976-1024x412.png)
](https://webrtc.org.cn/wp-content/uploads/2019/04/a13b245a9b7f304a64990ab3d6b78bc5c3fd5976.png)

WebRTC 媒体协商的过种如上图所示。

*   第一步，Amy 调用 createOffer 方法创建 offer 消息。offer 消息中的内容是 Amy 的 SDP 信息。
*   第二步，Amy 调用 setLocalDescription 方法，将本端的 SDP 信息保存起来。
*   第三步，Amy 将 offer 消息通过信令服务器传给 Bob。
*   第四步，Bob 收到 offer 消息后，调用 setRemoteDescription 方法将其存储起来。
*   第五步，Bob 调用 createAnswer 方法创建 answer 消息， 同样，answer 消息中的内容是 Bob 的 SDP 信息。
*   第六步，Bob 调用 setLocalDescription 方法，将本端的 SDP 信息保存起来。
*   第七步，Bob 将 anwser 消息通过信令服务器传给 Amy。
*   第八步，Amy 收到 anwser 消息后，调用 setRemoteDescription 方法，将其保存起来。

通过以上步骤就完成了通信双方媒体能力的交换。

上以就是信令服务器应该处理的所有消息，这些消息组成了信令服务器最基本的信令，每一个都必不可少，否则的话双方就无法进行最终的通信了。

在WebRTC 通讯时，光有信令是远远不够的。因为 WebRTC真正要传输的是媒体数据，信令只不过是其中的一部分。在WebRTC中他会尽可能的通过P2P进行数据的传输，但在 P2P穿越不成功时怎么办呢？

那就需要通过媒体中继服务器进行媒体数据的转发，下面我们就来看一下如何搭建媒体中继服务器吧。

搭建 STUN/TURN
------------

在公网搭建一套 STUN/TURN 服务并不难。首先要有一台云主机，云主机的获我就不做介绍了，大家去某个云厂商购买就好了。

目前比较流行的 STUN/TURN 服务器是 [coturn](https://github.com/coturn/coturn)，使用它搭建 STUN/TURN 服务非常的方便。

下面我们就来看一下它的基本步骤：

*   获取 coturn 源码
    
    git clone https://github.com/coturn/coturn.git
    
    git clone https://github.com/coturn/coturn.git
    
    ```
    git clone https://github.com/coturn/coturn.git
    ```
    
*   编译安装

./configure --prefix=/usr/local/coturn

sudo make -j 4  && make install

cd coturn ./configure --prefix=/usr/local/coturn sudo make -j 4 && make install

```
cd coturn
  ./configure --prefix=/usr/local/coturn
  sudo make -j 4 && make install
```

*   配置 coturn

网上有很多关于 coturn 的配置文章，搞的很复杂。大多数人都是从网上拷贝转发的，其中有很多错误。其实只要使用 coturn 的默认设置就可以了，我这里整理了一份，如下：

listening-port=3478  #指定侦听的端口

external-ip=39.105.185.198 #指定云主机的公网IP地址

user=aaaaaa:bbbbbb #访问 stun/turn服务的用户名和密码

realm=stun.xxx.cn  #域名，这个一定要设置

listening-port=3478 #指定侦听的端口 external-ip=39.105.185.198 #指定云主机的公网IP地址 user=aaaaaa:bbbbbb #访问 stun/turn服务的用户名和密码 realm=stun.xxx.cn #域名，这个一定要设置

```
listening-port=3478        #指定侦听的端口
  external-ip=39.105.185.198 #指定云主机的公网IP地址
  user=aaaaaa:bbbbbb         #访问 stun/turn服务的用户名和密码
  realm=stun.xxx.cn          #域名，这个一定要设置
```

所以，只需将上面 4 行配置项写入到 /usr/local/coturn/etc/turnserver.conf 配置文件中，你的 stun/turn 服务就配置好了。

*   启动 stun/turn 服务

turnserver -c ../etc/turnserver.conf

cd /usr/local/coturn/bin turnserver -c ../etc/turnserver.conf

```
cd /usr/local/coturn/bin
  turnserver -c ../etc/turnserver.conf
```

*   测试 stun/turn 服务

打开 [trickle-ice](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/) ，按里面的要求输入 stun/turn 地址、用户和密码后就可以探测stun/turn服务是否正常了。

以我们的配置为例，输入的信息分别是：

1.  **STUN or TURN URI** 的值为： **turn:stun.xxx.cn**
2.  用户名为： **aaaaaa**
3.  密码为： **bbbbbb**

测试的结果如下图所示：

[![](https://webrtc.org.cn/wp-content/uploads/2019/04/5f894015af35721d174219c64a128ba1dabe626e_2_1380x298-1024x221.png)
](https://webrtc.org.cn/wp-content/uploads/2019/04/5f894015af35721d174219c64a128ba1dabe626e_2_1380x298.png)

从上图我们可以看到该服务提供了 stun(srflx)和turn（relay）两种服务。

STUN/TURN布署好后，我们就可以使用它进行多媒体数据的传输了，再也不怕因为 NAT 和防火墙的原因导致双方无法通信的问题了。

小结
--

本文首先向大家详细介绍了 WebRTC 三种类型信令消息的控制与交换。然后给出了 STUN/TURN 服务器的布署、配置以及如何进行测试。

这里需要特别强调的是，STUN/TURN的布署虽然非常简单，但像 WebRTC 一样，其背后的原理确很复杂。由于篇幅的原因，我这里并没有向大家做详细的介绍，感兴趣的同学可以将其做为了一切入点进行深入的研究。