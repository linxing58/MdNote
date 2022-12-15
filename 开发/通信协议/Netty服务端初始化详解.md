# Netty服务端初始化详解
「我正在参与掘金会员专属活动-源码共读第一期，[点击参与](https://juejin.cn/post/7169502488557518878 "https://juejin.cn/post/7169502488557518878")」

这是**源码共读活动**的第二篇文章, 在上一章节中我们分析了 backlog 的作用, 接下来我们看一看**张师傅**为我们准备的`Netty`启动类都进行了哪些配置吧

往期文章:

*   [Netty源码分析(一) backlog 参数](https://juejin.cn/post/7172450784041762830 "https://juejin.cn/post/7172450784041762830")

没有拉取代码的小伙伴可以通过`git`输入以下命令拉一下代码, 让我们保持代码的同步

```bash
git clone https://github.com/arthur-zhang/netty-study.git

```

首先, 我们可以看到这个项目的目录结构很简单, 只有三个类, 通过名称可以知道, 分别是两个 Headler 类和一个 启动类 `MyServer`, 本篇文章也是主要针对`MyServer`来进行讲解的

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30c3e432ea1c4d8993736914e0154a9a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

下图是`MyServer`类的全部代码, 可以看到实际上没有多少,一图装得下, 接下来我们一起逐行代码去学习

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c02eb2185a694a3ea7672134b7fafa9b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

ServerBootstrap serverBootstrap = new ServerBootstrap();
--------------------------------------------------------

`Bootstrap`的意思是引导, 在`Netty`应用中, 通常也是由`Bootstrap`开始的, 他的作用就是对`Netty`进行配置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b0467ca86e04451ae11a965cfb8ee9a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

翻译工具为\`uTools\`插件 词典

本次主要对`Netty`中的`ServerBootstrap`进行讲解, `ServerBootstrap`是服务端引导类, 他的继承关系如下所示

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd7a5dfa28f64c2a8f2cb4d7544ace1d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> 在`Netty`中, `AbstractBootstrap`的实现类主要有两种, 分别是服务端`ServeBootstrap`和客户端`Bootstrap`, 对`Bootstrap`感兴趣的小伙伴可以评论区留言
> 
> ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a46e287f81bf48cd9fbff280a2f1b0ac~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

serverBootstrap.channel() 方法;
-----------------------------

`serverBootstrap.channel()`方法是用来设置`Netty`对应的通道的

在执行该方法的时候可以看到, 他实际上是调用了`ServeBootstrap`类的抽象父类`AbstractBootstrap`

在`channel`方法中实际上是初始化了一个`ReflectiveChannelFactory`工厂类, 同时将该工厂对象保存在`AbstractBootstrap`抽象类的`channelFactory`属性中, 后续可以调用生成`channel`对象, 目前只是保存

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/951c3311abb546e0871c6c23a5608384~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

下面我们看一下在`ReflectiveChannelFactory`工厂类的初始化和后续调用生成`channel`对象的方法

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/788507af519042d0b52b35b66fcbbd69~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

NioServerSocketChannel.class
----------------------------

`NioServerSocketChannel`是`Netty`官方封装的, 用来代替或包装 JDK 原生的`SocketChannel`对象, 他的继承关系图如下所示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fef85bbe4d98447ca8bada0d3e4d788a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> 过多的就不讲了, 在讲下去就该这个类的源码分析, 感兴趣的小伙伴可以评论区说出来

option()和 childOption() 方法
--------------------------

在`Netty`中`option()`方法主要是设置`ServerChannel`的一些选项, 而`childOption()`方法是用来设置`ServerChannel`的`子Channel`的选项

> 注: 如果是**客户端**, 因为是`Bootstrap`, 只会有`option()`, 没有`childOption()`, 所以设置的是**客户端Channel**的选项 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0f843beb094431daad07e6b47ff9c78~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

所以, `childOption()`方法是写在`ServerBootstrap`类中, 而不是继承于`AbstractBootstrap`抽象类的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c51b3797b4134dcaa081986cd1bdcd1e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

NioChannelOption
----------------

`NioChannelOption`类是继承于`ChannelOption`同时新增了几个方法, 那么我们主要讲一下`ChannelOption`里面的常量信息

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c0f0c9d00574936bfb449b33cd61f2b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> @SuppressJava6Requirement 注解的作用: 清除 java6 的警告, 现在我们大多使用的都是 java8 以上了, 这个注解可以无视掉

### 1、ChannelOption.SO\_BACKLOG

**SO\_BACKLOG**参数用来初始化服务端可连接队列。

服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接，多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，**backlog** 参数指定了队列的大小。

同时在设置**backlog**参数的时候, 也要根据需求来设置, 避免因为队列设置的太小导致消息溢出

> 在推一手上一篇文章 [Netty源码分析(一) backlog 参数](https://juejin.cn/post/7172450784041762830 "https://juejin.cn/post/7172450784041762830")

### 2、ChannelOption.SO\_REUSEADDR

**SO\_REUSEADDR**参数表示允许重复使用本地地址和端口。

比如，某个服务器进程占用了TCP的80端口进行监听，此时再次监听该端口就会返回错误，使用该参数就可以解决问题，该参数允许共用该端口，这个在服务器程序中比较常使用。

### 3、ChannelOption.SO\_KEEPALIVE

**SO\_KEEPALIVE** 默认为 false, 启用该功能时, TCP 会主动探测空闲连接的有效性, 可以将此功能视为TCP的心跳机制，需要注意的是：默认的心跳间隔是7200s即2小时。Netty默认关闭该功能

### 4、ChannelOption.SO\_SNDBUF和ChannelOption.SO\_RCVBUF

**SO\_SNDBUF** 和 **SO\_RCVBUF**这两个参数用于操作发送缓冲区大小和接受缓冲区大小。

接收缓冲区用于保存网络协议站内收到的数据，直到应用程序读取成功，发送缓冲区用于保存发送数据，直到发送成功。

### 5、ChannelOption.SO\_LINGER

Linux内核默认的处理方式是当用户调用close（）方法的时候，函数返回，在可能的情况下，尽量发送数据，不一定保证会发送剩余的数据，造成了数据的不确定性，使用**SO\_LINGER**可以阻塞close()的调用时间，直到数据完全发送.

### 6、ChannelOption.TCP\_NODELAY

**TCP\_NODELAY**参数的使用与`Nagle`算法有关。

该参数的作用就是禁止使用Nagle算法，使用于小数据即时传输。和TCP\_NODELAY相对应的是TCP\_CORK，该选项是需要等到发送的数据量最大的时候，一次性发送数据，适用于文件传输。

> Nagle算法是将小的数据包组装为更大的帧然后进行发送，而不是输入一次发送一次，因此在数据包不足的时候会等待其他数据的到来，组装成大的数据包进行发送，虽然该算法有效提高了网络的有效负载，但是却造成了延时。

handler() 和 childHandler()
--------------------------

本来是到 `serverBootstrap.heandler()`方法了, 但是我一想下面还有一个`serverBootstrap.childHandler()`方法, 正好一起讲了, 顺便对比一下两个**headler**方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f77b4bdf42cc499291db966e6cb1f2b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

同时大家应该看到了, 在实现`childHeadler()`方法的时候有创建一个类并实现了相应的方法, 他的作用就是初始化了一下**Channel**并把日志级别更改为 `info`

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd570a9baec54e3cb768bc5890727c0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 在 ServerBootstrap 中

`handler()`方法是针对`bossGroup`线程组起作用  
`childHandler()`方法是针对`workerGroup`线程组起作用

### 在 Bootstrap 中

只有`handler()`方法, 因为客户端只需要一个事件线程组

NioEventLoopGroup 和 serverBootstrap.group()
-------------------------------------------

**NioEventLoopGroup**是一个可处理I/O操作的多线程事件循环。Netty提供了多种 EventLoopGroup 的实现用于不同类型的传输。

`serverBootstrap.group()`方法就是分配 bossGroup 和 workGroup 两个线程组的

### bossGroup 和 workGroup

`bossGroup`是负责处理客户端和服务端建立连接注册的 selector

`workGroup`是负责处理客户端读事件的 selector 逻辑

全部代码
----

```java
public static void main(String[] args) throws InterruptedException {

    
    ServerBootstrap serverBootstrap = new ServerBootstrap();

    
    serverBootstrap.channel(NioServerSocketChannel.class);
    
    serverBootstrap.option(NioChannelOption.SO_BACKLOG, 511);
    
    
    serverBootstrap.childOption(NioChannelOption.TCP_NODELAY, true);
    
    serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));

    
    NioEventLoopGroup bossGroup = new NioEventLoopGroup(0, new DefaultThreadFactory("boss"));
    
    NioEventLoopGroup workGroup = new NioEventLoopGroup(0, new DefaultThreadFactory("worker"));

    try {
        
        serverBootstrap.group(bossGroup, workGroup);
        final MyEchoServerHandler serverHandler = new MyEchoServerHandler();
        serverBootstrap.childHandler(new ChannelInitializer<NioSocketChannel>() {
            @Override
            protected void initChannel(NioSocketChannel ch) {
                ChannelPipeline pipeline = ch.pipeline();
                pipeline.addLast(new ServerIdleCheckHandler());
                pipeline.addLast(new LoggingHandler(LogLevel.INFO));
                pipeline.addLast(serverHandler);
            }
        });

        
        ChannelFuture f = serverBootstrap.bind(8888).sync();
        
        f.channel().closeFuture().sync();
    } finally {
        
        bossGroup.shutdownGracefully();
        workGroup.shutdownGracefully();
    }
}

```

查看类的继承关系图
---------

双击选中想要查看的类 ==> 右键 ==> Diagrams ==> ShowDiagramPopup

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acca99e92aa5465eb802439ca84aa598~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

本篇文章我们对**Netty**的启动类进行了逐行分析, 也让我对`Netty`的大概配置有了更全面的了解, 每天进步一丢丢, 加油

> 本文内容到此结束了
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> 我是 **宁轩** , 我们下次再见