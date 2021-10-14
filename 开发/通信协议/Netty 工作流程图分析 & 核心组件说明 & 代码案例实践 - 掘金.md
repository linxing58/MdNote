# Netty | 工作流程图分析 & 核心组件说明 & 代码案例实践 - 掘金
本文已参与「[掘力星计划](https://juejin.cn/post/7012210233804079141/ "https&#x3A;//juejin.cn/post/7012210233804079141/")」，赢取创作大礼包，挑战创作激励金。

> 前文：[你的第一款 Netty 应用程序](https://juejin.cn/post/7017253518960492575 "https&#x3A;//juejin.cn/post/7017253518960492575")
>
> 前一篇文章写了第一款 Netty 入门的应用程序，本文主要就是从上文的代码结合本文的流程图进一步分析 Netty 的工作流程和核心组件。
>
> 最后再进一步举一个实例来让大家进一步理解。
>
> 希望能够让你有所收获！！🚀

## 一、Netty 工作流程

我们先来看看 Netty 的工作原理图，简单说一下工作流程，然后通过这张图来一一分析 Netty 的核心组件。

### 1.1、Server 工作流程图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20e0e02df7af4b989761599a20d0e823~tplv-k3u1fbpfcp-watermark.awebp)

### 1.2、Server 工作流程分析：

1.  server 端启动时绑定本地某个端口，初始化`NioServerSocketChannel`.
2.  将自己`NioServerSocketChannel`注册到某个`BossNioEventLoopGroup`的 selector 上。

    -   server 端包含 1 个`Boss NioEventLoopGroup`和 1 个`Worker NioEventLoopGroup`，
    -   `Boss NioEventLoopGroup`专门负责接收客户端的连接，`Worker NioEventLoopGroup`专门负责网络的读写
    -   NioEventLoopGroup 相当于 1 个事件循环组，这个组里包含多个事件循环 NioEventLoop，每个 NioEventLoop 包含 1 个 selector 和 1 个事件循环线程。
3.  `BossNioEventLoopGroup`循环执行的任务：

    1、轮询 accept 事件；

    2、处理 accept 事件，将生成的 NioSocketChannel 注册到某一个`WorkNioEventLoopGroup`的 Selector 上。

    3、处理任务队列中的任务，runAllTasks。任务队列中的任务包括用户调用`eventloop.execute 或 schedule`执行的任务，或者其它线程提交到该`eventloop`的任务。
4.  `WorkNioEventLoopGroup`循环执行的任务：

    -   轮询`read 和 Write`事件
    -   处理 IO 事件，在 NioSocketChannel 可读、可写事件发生时，回调（触发）ChannelHandler 进行处理。
    -   处理任务队列的任务，即 `runAllTasks`

### 1.3、Client 工作流程图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7dad6fd930b441db81b6d2b59ff7849f~tplv-k3u1fbpfcp-watermark.awebp)

流程就不重复概述啦😁

## 二、核心模块组件

Netty 的核心组件大致是以下几个：

1.  Channel 接口
2.  EventLoopGroup 接口
3.  ChannelFuture 接口
4.  ChannelHandler 接口
5.  ChannelPipeline 接口
6.  ChannelHandlerContext 接口
7.  SimpleChannelInboundHandler 抽象类
8.  Bootstrap、ServerBootstrap 类
9.  ChannelFuture 接口
10. ChannelOption 类

### 2.1、Channel 接口

我们平常用到基本的 I/O 操作（bind()、connect()、read() 和 write()），其本质都依赖于底层网络传输所提供的原语，在 Java 中就是`Socket`类。

Netty 的 Channel 接 口所提供的 API，大大地降低了直接使用`Socket` 类的复杂性。另外`Channel` 提供异步的网络 `I/O` 操作 (如建立连接，读写，绑定端口)，异步调用意味着任何 `I/O` 调用都将立即返回，并且不保证在调用结束时所请求的 `I/O` 操作已完成。

在调用结束后立即返回一个 `ChannelFuture` 实例，通过注册监听器到 `ChannelFuture` 上，支持 在`I/O` 操作成功、失败或取消时立马回调通知调用方。

此外，Channel 也是拥有许多预定义的、专门化实现的广泛类层次结构的根，比如：

-   `LocalServerChannel`：用于本地传输的 ServerChannel ，允许 VM 通信。
-   `EmbeddedChannel`：以嵌入式方式使用的 Channel 实现的基类。
-   `NioSocketChannel`：异步的客户端 TCP 、Socket 连接。
-   `NioServerSocketChannel`：异步的服务器端 TCP、Socket 连接。
-   `NioDatagramChannel`： 异步的 UDP 连接。
-   `NioSctpChannel`：异步的客户端 Sctp 连接, 它使用非阻塞模式并允许将 SctpMessage 读 / 写到底层 SctpChannel。
-   `NioSctpServerChannel`：异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。

### 2.2、EventLoopGroup 接口

`EventLoop` 定义了 Netty 的核心抽象，用于处理连接的生命周期中所发生的事件。

Netty 通过触发事件将 Selector 从应用程序中抽象出来，消除了所有本来将需要手动编写 的派发代码。在内部，将会为每个 Channel 分配一个 EventLoop，用以处理所有事件，包括：

-   注册事件；
-   将事件派发给 ChannelHandler；
-   安排进一步的动作。

不过在这里我们不深究它，针对 Channel、EventLoop、Thread 以及 EventLoopGroup 之间的关系做一个简单说明。

-   一个`EventLoopGroup` 包含一个或者多个 `EventLoop`；
-   每个 `EventLoop` 维护着一个 `Selector` 实例，所以一个 EventLoop 在它的生命周期内只和一个 `Thread` 绑定；
-   因此所有由 `EventLoop` 处理的 I/O 事件都将在它专有的 `Thread` 上被处理，实际上消除了对于同步的需要；
-   一个 `Channel` 在它的生命周期内只注册于一个 `EventLoop`；
-   一个 `EventLoop` 可能会被分配给一个或多个`Channel`。
-   通常一个服务端口即一个 `ServerSocketChannel` 对应一个 `Selector` 和一个 `EventLoop` 线程。`BossEventLoop` 负责接收客户端的连接并将 `SocketChannel` 交给 `WorkerEventLoopGroup` 来进行 IO 处理，就如上文中的流程图一样。

### 2.3、ChannelFuture 接口

Netty 中所有的 I/O 操作都是异步的。因为一个操作可能不会 立即返回，所以我们需要一种用于在之后的某个时间点确定其结果的方法。具体的实现就是通过 `Future` 和 `ChannelFutures`，其 `addListener()`方法注册了一个 `ChannelFutureListener`，以便在某个操作完成时（无论是否成功）自动触发注册的监听事件。

常见的方法有

-   `Channel channel()`，返回当前正在进行 `IO` 操作的通道
-   `ChannelFuture sync()`，等待异步操作执行完毕

### 2.4、ChannelHandler 接口

从之前的入门程序中，我们可以看到`ChannelHandler`在 Netty 中的重要性，它充当了所有处理入站和出站数据的应用程序逻辑的容器。 我们的业务逻辑也大都写在实现的字类中，另外`ChannelHandler` 的方法是由事件自动触发的，并不需要我们自己派发。

`ChannelHandler`的实现类或者实现子接口有很多。平时我们就是去继承或子接口，然后重写里面的方法。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cbc5fafa6cb4104950705ee07c67d94~tplv-k3u1fbpfcp-watermark.awebp)

最常见的几种 Handler:

-   `ChannelInboundHandler` : 接收入站事件和数据
-   `ChannelOutboundHandler`：用于处理出站事件和数据。

常见的适配器：

-   `ChannelInboundHandlerAdapter`：用于处理入站 IO 事件

    `ChannelInboundHandler`实现的抽象基类，它提供了所有方法的实现。 这个实现只是将操作转发到`ChannelPipeline`的下一个`ChannelHandler` 。 子类可以覆盖方法实现来改变这一
-   `ChannelOutboundHandlerAdapter`： 用于处理出站 IO 事件

我们经常需要自定义一个 `Handler` 类去继承 `ChannelInboundHandlerAdapter`，然后通过重写相应方法实现业务逻辑，我们来看看有哪些方法可以重写：

```java
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {
	
    public void channelRegistered(ChannelHandlerContext ctx) ;
	
    public void channelUnregistered(ChannelHandlerContext ctx);
	
    public void channelActive(ChannelHandlerContext ctx);
	
    public void channelInactive(ChannelHandlerContext ctx) ;
    
    public void channelRead(ChannelHandlerContext ctx, Object msg) ;
	
    public void channelReadComplete(ChannelHandlerContext ctx) ;

    public void userEventTriggered(ChannelHandlerContext ctx, Object evt);
	
    public void channelWritabilityChanged(ChannelHandlerContext ctx);
	
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
}
```

### 2.5、ChannelPipeline 接口

`ChannelPipeline` 提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站和出站事件流的 API。当 Channel 被创建时，它会被自动地分配到它专属的 `ChannelPipeline`。他们的组成关系如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cee5fe804934379972ec6da66d223d8~tplv-k3u1fbpfcp-watermark.awebp)

一个 **Channel** 包含了一个 `ChannelPipeline`，而 _ChannelPipeline_ 中又维护了一个由`ChannelHandlerContext` 组成的双向链表，并且每个**ChanneHandlerContext**中又关联着一个`ChannelHandler`。

`ChannelHandler` 安装到 `ChannelPipeline` 中的过程：

1.  一个`ChannelInitializer`的实现被注册到了`ServerBootstrap`中 ；
2.  当 `ChannelInitializer.initChannel()`方法被调用时，**ChannelInitializer**将在 `ChannelPipeline` 中安装一组自定义的 `ChannelHandler`；
3.  `ChannelInitializer` 将它自己从 `ChannelPipeline` 中移除。

**从一个客户端应用程序 的角度来看，如果事件的运动方向是从客户端到服务器端，那么我们称这些事件为出站的，反之 则称为入站的。服务端反之。** 

如果一个消息或者任何其他的入站事件被读取，那么它会从 `ChannelPipeline` 的头部 开始流动，并被传递给第一个 `ChannelInboundHandler`。次此 handler 处理完后，数据将会被传递给链中的下一个 `ChannelInboundHandler`。最终，数据将会到达 `ChannelPipeline` 的尾端，至此，所有处理就结束了。

出站事件会从尾端往前传递到最前一个出站的 handler。出站和入站两种类型的 handler 互不干扰。

### 2.6、ChannelHandlerContext 接口

作用就是使`ChannelHandler`能够与其`ChannelPipeline`和其他处理程序交互。因为 ChannelHandlerContext 保存`channel`相关的所有上下文信息，同时关联一个 `ChannelHandler` 对象， 另外，`ChannelHandlerContext` 可以通知`ChannelPipeline`的下一个`ChannelHandler`以及动态修改它所属的`ChannelPipeline` 。

### 2.7、SimpleChannelInboundHandler 抽象类

我们常常能够遇到应用程序会利用一个 `ChannelHandler` 来接收解码消息，并在这个 Handler 中实现业务逻辑，要写一个这样的 `ChannelHandler` ，我们只需要扩展抽象类 `SimpleChannelInboundHandler<T>` 即可, 其中 T 类型是我们要处理的消息的 Java 类型。

在`SimpleChannelInboundHandler` 中最重要的方法就是`void channelRead0(ChannelHandlerContext ctx, T msg)`，

我们自己实现了这个方法之后，接收到的消息就已经被解码完的消息啦。

举个例子：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e770bd905d4442d6b96cb0ddbea7eae5~tplv-k3u1fbpfcp-watermark.awebp)

### 2.8、Bootstrap、ServerBootstrap 类

`Bootstrap` 意思是引导，一个 `Netty` 应用通常由一个 `Bootstrap` 开始，主要作用是配置整个 `Netty` 程序，串联各个组件。

| 类别                 | Bootstrap    | ServerBootstrap |
| ------------------ | ------------ | --------------- |
| 引导                 | 用于引导客户端      | 用于引导服务端         |
| 在网络编程中作用           | 用于连接到远程主机和端口 | 用于绑定到一个本地端口     |
| EventLoopGroup 的数目 | 1            | 2               |

我想大家对于最后一点可能会存有疑惑，为什么一个是 1 一个是 2 呢？

因为服务器需要两组不同的 `Channel`。

第一组将只包含一个 `ServerChannel`，代表服务 器自身的已绑定到某个本地端口的正在监听的套接字。

而第二组将包含所有已创建的用来处理传入客户端连接（对于每个服务器已经接受的连接都有一个）的 `Channel`。

这一点可以上文中的流程图。

### 2.9、ChannelFuture 接口

异步 Channel I/O 操作的结果。 Netty 中的所有 I/O 操作都是异步的。 这意味着任何 I/O 调用将立即返回，但不保证在调用结束时请求的 I/O 操作已完成。 相反，您将返回一个 ChannelFuture 实例，该实例为您提供有关 I/O 操作的结果或状态的信息。 ChannelFuture 要么未完成，要么已完成。 当 I/O 操作开始时，会创建一个新的未来对象。 新的未来最初是未完成的——它既没有成功，也没有失败，也没有取消，因为 I/O 操作还没有完成。 如果 I/O 操作成功完成、失败或取消，则使用更具体的信息（例如失败原因）将未来标记为已完成。 请注意，即使失败和取消也属于完成状态。

`Netty` 中所有的 `IO` 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 `Future` 和 `ChannelFutures`，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

常见的方法有

-   `Channel channel()`，返回当前正在进行 `IO` 操作的通道
-   `ChannelFuture sync()`，等待异步操作执行完毕

### 2.10、ChannelOption 类

1.  `Netty` 在创建 `Channel` 实例后，一般都需要设置 `ChannelOption` 参数。
2.  `ChannelOption` 参数如下：
    -   `ChannelOption.SO_KEEPALIVE` ：一直保持连接状态
    -   `ChannelOption.SO_BACKLOG`：对应 TCP/IP 协议 listen 函数中的 backlog 参数，用来初始化服务器可连接队列大小。服务端处理客户端连接请求是顺序处理内，所 N 博求放在队刚中等待处理，backilog 参数指定端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理, backlog 参数指定了队列的大小。

## 三、应用实例

【案例】：

写一个服务端，两个或多个客户端，客户端可以相互通信。

### 3.1、服务端 Handler

`ChannelHandler`的实现类或者实现子接口有很多。平时我们就是去继承或子接口，然后重写里面的方法。

在这里我们就是继承了 SimpleChannelInboundHandler&lt;T> ，这里面许多方法都是大都只要我们重写一下业务逻辑，触发大都是在事件发生时自动调用的，无需我们手动调用。

```java
package com.crush.atguigu.group_chat;

import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.util.concurrent.GlobalEventExecutor;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;


public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");


    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        
        
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 加入聊天" + sdf.format(new java.util.Date()) + " \n");
        channelGroup.add(channel);

    }

    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        Channel channel = ctx.channel();
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + " 离开了\n");
        System.out.println("channelGroup size" + channelGroup.size());

    }

    
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 上线了~");
    }

    
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {

        System.out.println(ctx.channel().remoteAddress() + " 离线了~");
    }

    
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {

        
        Channel channel = ctx.channel();
        

        channelGroup.forEach(ch -> {
            if (channel != ch) { 
                ch.writeAndFlush("[客户]" + channel.remoteAddress() + " 发送了消息" + msg + "\n");
            } else {
                ch.writeAndFlush("[自己]发送了消息" + msg + "\n");
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        
        ctx.close();
    }
}
```

### 3.2、服务端 Server 启动

```java
package com.crush.atguigu.group_chat;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;


public class GroupChatServer {

    
    private int port;

    public GroupChatServer(int port) {
        this.port = port;
    }

    
    public void run() throws Exception {

        
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();

            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            
                            ChannelPipeline pipeline = ch.pipeline();
                            
                            pipeline.addLast("decoder", new StringDecoder());
                            
                            pipeline.addLast("encoder", new StringEncoder());
                            
                            pipeline.addLast(new GroupChatServerHandler());
                        }
                    });

            System.out.println("netty 服务器启动");

            ChannelFuture channelFuture = b.bind(port).sync();

            
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new GroupChatServer(7000).run();
    }
}
```

### 3.3、客户端 Handler

```java
package com.crush.atguigu.group_chat;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class GroupChatClientHandler extends SimpleChannelInboundHandler<String> {
    
    
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg.trim());
    }
}
```

### 3.4、客户端 Server

```java
package com.crush.atguigu.group_chat;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

import java.util.Scanner;



public class GroupChatClient {
    
    private final String host;
    private final int port;

    public GroupChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            
            Bootstrap bootstrap = new Bootstrap()
                    .group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {

                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {

                            
                            ChannelPipeline pipeline = ch.pipeline();
                            
                            pipeline.addLast("decoder", new StringDecoder());
                            pipeline.addLast("encoder", new StringEncoder());
                            
                            pipeline.addLast(new GroupChatClientHandler());
                        }
                    });

            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            
            Channel channel = channelFuture.channel();
            System.out.println("-------" + channel.localAddress() + "--------");
            
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNextLine()) {
                String msg = scanner.nextLine();
                
                channel.writeAndFlush(msg + "\r\n");
            }
        } finally {
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new GroupChatClient("127.0.0.1", 7000).run();
    }
}
```

多个客户端，cv 一下即可。

### 3.5、测试：

测试流程是先启动 服务端 Server，再启动客户端 。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb775bbabd30459b88971cd06ca6e4cc~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/714bf30d3f5a406ba9e232ba7f4a3f8f~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35364a6137b24fcf97b22fe6e4875e91~tplv-k3u1fbpfcp-watermark.awebp)

## 四、自言自语

这篇文章应该算是个存稿了，之前忙其他事情去了😂。

> 今天的文章就到这里了。
>
> 你好，我是博主`宁在春`：[主页](https://juejin.cn/user/2859142558267559 "https&#x3A;//juejin.cn/user/2859142558267559")
>
> 如若在文章中遇到疑惑，请留言或私信，或者加主页联系方式，都会尽快回复。
>
> 如若发现文章中存在问题，望你能够指正，不胜感谢。
>
> 如果觉得对你有所帮助的话，请点个赞再走吧！

欢迎大家一起在讨论区讨论哦，增加增加自己的幸运，蹭一蹭掘金官方发送的周边哦！！！

评论越走心越有可能拿奖哦！！！

详情👉[掘力星计划](https://juejin.cn/post/7012210233804079141 "https&#x3A;//juejin.cn/post/7012210233804079141") 
 [https://juejin.cn/post/7017602386747195429](https://juejin.cn/post/7017602386747195429)
