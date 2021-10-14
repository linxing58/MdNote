# Netty实战之使用Netty解析交通部JT808协议 - 掘金
## 1. 写此文的目的

使用 Netty 也有一段时间了，对 Netty 也有个大概的了解。回想起刚刚使用 Netty 的时候踩了很多坑，很多 Netty 的组件也不会使用，或者说用得不够好，不能称之为 "最佳实践"。此文的目的便是带领大家使用 Netty 构建出一个完整的项目，将自己在实际开发经验中整理出的一些最佳实践分享出来，当然这些最佳实践不一定就是真正的最佳实践，只是自己在开发中整理的，或者参考其他优秀的代码一起整理出的，大家如果有什么不同意见或者更好的实践，欢迎大家在评论区分享，大家一起学习一起进步！

## 2. 项目准备

先奉上完整版代码 [zpsw/jt808-netty](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fzpsw%2Fjt808-netty "https&#x3A;//github.com/zpsw/jt808-netty")

开发环境: IDEA+JDK1.8+Maven

使用框架: Netty + Spring Boot + Spring Data JPA

其他工具: lombok(没用过的同学建议了解一下，很方便)

## 3. 开发过程

    3.1.认识JT808协议
    3.2.构建编/解码器
    3.3.构建业务Handler
    3.4.Channel的高效管理方式
    3.5.一些改进
    复制代码

### 3.1 认识 JT808 协议

下面简单介绍一下 JT808 协议的格式说明，完全版在[JT808 协议技术规范. pdf](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fzpsw%2Fjt808-netty%2Fblob%2Fmaster%2FJT808%25E5%258D%258F%25E8%25AE%25AE%25E6%258A%2580%25E6%259C%25AF%25E8%25A7%2584%25E8%258C%2583.pdf "https&#x3A;//github.com/zpsw/jt808-netty/blob/master/JT808%E5%8D%8F%E8%AE%AE%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83.pdf")

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/17/16ac47c0aab70310~tplv-t2oaga2asx-watermark.awebp)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/17/16ac47c45e2dd960~tplv-t2oaga2asx-watermark.awebp)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/17/16ac47cbd885a24f~tplv-t2oaga2asx-watermark.awebp)

其中消息体属性中我们先只关注消息体长度，不关注其他，分包情况先不考虑。

根据消息头和消息体我们可以抽象出一个最基本的数据结构

    @Data
    public class DataPacket {

        protected Header header = new Header(); //消息头
        protected ByteBuf byteBuf; //消息流

        @Data
        public static class Header {
            private short msgId;// 消息ID 2字节
            private short msgBodyProps;//消息体属性 2字节
            private String terminalPhone; // 终端手机号 6字节
            private short flowId;// 流水号 2字节

            //获取包体长度
            public short getMsgBodyLength() {
                return (short) (msgBodyProps & 0x3ff);
            }

            //获取加密类型 3bits
            public byte getEncryptionType() {
                return (byte) ((msgBodyProps & 0x1c00) >> 10);
            }

            //是否分包
            public boolean hasSubPackage() {
                return ((msgBodyProps & 0x2000) >> 13) == 1;
            }
        }
    }
    复制代码

我们可以先将 Header 解析出来，然后由子类自己解析包体

     public void parse() {
            try{
                this.parseHead();
                //验证包体长度
                if (this.header.getMsgBodyLength() != this.byteBuf.readableBytes()) {
                    throw new RuntimeException("包体长度有误");
                }
                this.parseBody();//由子类重写
            }finally {
                ReferenceCountUtil.safeRelease(this.byteBuf);//注意释放
            }
        }

        protected void parseHead() {
            header.setMsgId(byteBuf.readShort());
            header.setMsgBodyProps(byteBuf.readShort());
            header.setTerminalPhone(BCD.BCDtoString(readBytes(6)));
            header.setFlowId(byteBuf.readShort());
        }
        protected void parseBody() {

        }
    复制代码

其中 readByte(int length) 方法是对 ByteBuf.readBytes(byte\[] dst) 的一个简单封装

    public byte[] readBytes(int length) {
            byte[] bytes = new byte[length];
            this.byteBuf.readBytes(bytes);
            return bytes;
    }
    复制代码

因为没有在 Netty 官方的 Api 中找到类似的方法，所以自己定义了一个

另外定义一个方法用于响应重写。

响应重写:

     public ByteBuf toByteBufMsg() {
            ByteBuf bb = ByteBufAllocator.DEFAULT.heapBuffer();
            bb.writeInt(0);//先占4字节用来写msgId和msgBodyProps
            bb.writeBytes(BCD.toBcdBytes(StringUtils.leftPad(this.header.getTerminalPhone(), 12, "0")));
            bb.writeShort(this.header.getFlowId());
            return bb;
    }
    **
    "最佳实践":尽量使用内存池分配ByteBuf，效率相比非池化Unpooled.buffer()高很多，但是得注意释放，否则会内存泄漏
    在ChannelPipeLine中我们可以使用ctx.alloc()或者channel.alloc()获取Netty默认内存分配器，
    其他地方不一定要建立独有的内存分配器，可以通过ByteBufAllocator.DEFAULT获取，跟前面获取的是同一个(不特别配置的话)。
    **
    复制代码

这里当我们将响应转化为 ByteBuf 写出去的时候，此时并不知道消息体的具体长度，所有此时我们先占住位置，回头再来写。

所有的消息都继承自 DataPacket，我们挑出一个字段相对较多的 -》 位置上报消息

然后我们建立位置上报消息的数据结构，先看位置消息的格式

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/17/16ac49328c33ce40~tplv-t2oaga2asx-watermark.awebp)

建立结构如下:

    @Data
    public class LocationMsg extends DataPacket {

        private int alarm; //告警信息 4字节
        private int statusField;//状态 4字节
        private float latitude;//纬度 4字节
        private float longitude;//经度 4字节
        private short elevation;//海拔高度 2字节
        private short speed; //速度 2字节
        private short direction; //方向 2字节
        private String time; //时间 6字节BCD

        public LocationMsg(ByteBuf byteBuf) {
            super(byteBuf);
        }
        
        @Override
        public void parseBody() {
            ByteBuf bb = this.byteBuf;
            this.setAlarm(bb.readInt());
            this.setStatusField(bb.readInt());
            this.setLatitude(bb.readUnsignedInt() * 1.0F / 1000000);
            this.setLongitude(bb.readUnsignedInt() * 1.0F / 1000000);
            this.setElevation(bb.readShort());
            this.setSpeed(bb.readShort());
            this.setDirection(bb.readShort());
            this.setTime(BCD.toBcdTimeString(readBytes(6)));
        }
    }
    复制代码

所有的消息如果没有自己的应答的话，需要默认应答，默认应答格式如下

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/17/16ac4a5bc30ac6b1~tplv-t2oaga2asx-watermark.awebp)

    @Data
    public class CommonResp extends DataPacket {

        private short replyFlowId; //应答流水号 2字节
        private short replyId; //应答 ID  2字节
        private byte result;    //结果 1字节

        public CommonResp() {
            this.getHeader().setMsgId(JT808Const.SERVER_RESP_COMMON);
        }

        @Override
        public ByteBuf toByteBufMsg() {
            ByteBuf bb = super.toByteBufMsg();
            bb.writeShort(replyFlowId);
            bb.writeShort(replyId);
            bb.writeByte(result);
            return bb;
        }
    }

    复制代码

### 3.2 构建编 / 解码器

#### 解码器

前面协议可以看到，标识位为 0x7e，所以我们第一个解码器可以用 Netty 自带的 DelimiterBasedFrameDecoder, 其中的 delimiters 自然就是 0x7e 了。（Netty 有很多自带的编解码器，建议先确认 Netty 自带的不能满足需求，再自己自定义）

经过 DelimiterBasedFrameDecoder 帮我们截断之后，信息就到了我们自己的解码器中了，我们的目的是将 ByteBuf 转化为我们前面定义的数据结构。 定义解码器

    public class JT808Decoder extends ByteToMessageDecoder {
        protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        
        }
    }
    复制代码

第一步：转义还原, 转义规则如下

0x7d 0x01 -> 0x7d

0x7d 0x02 -> 0x7e

    public ByteBuf revert(byte[] raw) {
            int len = raw.length;
            ByteBuf buf = ByteBufAllocator.DEFAULT.heapBuffer(len);//DataPacket parse方法回收
            for (int i = 0; i < len; i++) {
                if (raw[i] == 0x7d && raw[i + 1] == 0x01) {
                    buf.writeByte(0x7d);
                    i++;
                } else if (raw[i] == 0x7d && raw[i + 1] == 0x02) {
                    buf.writeByte(0x7e);
                    i++;
                } else {
                    buf.writeByte(raw[i]);
                }
            }
            return buf;
        }
    复制代码

第二步: 校验

        byte pkgCheckSum = escape.getByte(escape.writerIndex() - 1);
        escape.writerIndex(escape.writerIndex() - 1);//排除校验码
        byte calCheckSum = JT808Util.XorSumBytes(escape);
        if (pkgCheckSum != calCheckSum) {
            log.warn("校验码错误,pkgCheckSum:{},calCheckSum:{}", pkgCheckSum, calCheckSum);
            ReferenceCountUtil.safeRelease(escape);//一定不要漏了释放
            return null;
        }
    复制代码

第三步: 解码

     public DataPacket parse(ByteBuf bb) {
            DataPacket packet = null;
            short msgId = bb.getShort(bb.readerIndex());
            switch (msgId) {
                case TERNIMAL_MSG_HEARTBEAT:
                    packet = new HeartBeatMsg(bb);
                    break;
                case TERNIMAL_MSG_LOCATION:
                    packet = new LocationMsg(bb);
                    break;
                case TERNIMAL_MSG_REGISTER:
                    packet = new RegisterMsg(bb);
                    break;
                case TERNIMAL_MSG_AUTH:
                    packet = new AuthMsg(bb);
                    break;
                case TERNIMAL_MSG_LOGOUT:
                    packet = new LogOutMsg(bb);
                    break;
                default:
                    packet = new DataPacket(bb);
                    break;
            }
            packet.parse();
            return packet;
        }
    复制代码

switch 里我们尽量将收到频率高的放在前面，避免过多的 if 判断

然后我们将消息 out.add(msg) 就可以让消息到我们的业务 Handler 中了。

#### 编码器

编码器需要讲我们的 DataPacket 转化为 ByteBuf，然后再转义发送出去。 定义编码器

```

public class JT808Encoder extends MessageToByteEncoder<DataPacket> {
    protected void encode(ChannelHandlerContext ctx, DataPacket msg, ByteBuf out) throws Exception {
    
    }
}
复制代码
```

第一步：转换

    ByteBuf bb = msg.toByteBufMsg();
    复制代码

还记得我们 DataPacket 转换 header 时占用了 4 个字节等到后面覆盖吗

            bb.markWriterIndex();//标记一下，先到前面去写覆盖的，然后回到标记写校验码
            short bodyLen = (short) (bb.readableBytes() - 12);
            short bodyProps = createDefaultMsgBodyProperty(bodyLen);
            //覆盖占用的4字节
            bb.writerIndex(0);
            bb.writeShort(msg.getHeader().getMsgId());
            bb.writeShort(bodyProps);
            bb.resetWriterIndex();
            bb.writeByte(JT808Util.XorSumBytes(bb));
    复制代码

第二步：转义

     public ByteBuf escape(ByteBuf raw) {
            int len = raw.readableBytes();
            ByteBuf buf = ByteBufAllocator.DEFAULT.directBuffer(len + 12);//假设最多有12个需要转义
            buf.writeByte(JT808Const.PKG_DELIMITER);
            while (len > 0) {
                byte b = raw.readByte();
                if (b == 0x7e) {
                    buf.writeByte(0x7d);
                    buf.writeByte(0x02);
                } else if (b == 0x7d) {
                    buf.writeByte(0x7d);
                    buf.writeByte(0x01);
                } else {
                    buf.writeByte(b);
                }
                len--;
            }
            ReferenceCountUtil.safeRelease(raw);
            buf.writeByte(JT808Const.PKG_DELIMITER);
            return buf;
        }
    **
    "最佳实践":我们这里返回ByteBuf是写出去的，所以采用directBuffer效率更高
    **
    复制代码

转义完成，就直接发送出去了，当然不能忘了释放。

            ByteBuf escape = escape(bb);
            out.writeBytes(escape);
            ReferenceCountUtil.safeRelease(escape);
    复制代码

### 3.3 构建业务 Handler

解码器中我们返回的是 DataPacket 对象，所以编写 Handler 此时我们有两种选择：

一种是定义一个 Handler 接收 DataPacket 然后判断具体类型，如下图

    @Component
    @ChannelHandler.Sharable
    public class JT808ServerHandler extends SimpleChannelInboundHandler<DataPacket> {

        @Override
        protected void channelRead0(ChannelHandlerContext ctx, DataPacket msg) throws Exception {
            log.debug(msg.toString());
            if (msg instanceof AuthMsg || msg instanceof HeartBeatMsg || msg instanceof LocationMsg || msg instanceof LogOutMsg) {
                CommonResp resp = CommonResp.success(msg, getFlowId(ctx));
                ctx.writeAndFlush(resp);
            } else if (msg instanceof RegisterMsg) {
                RegisterResp resp = RegisterResp.success(msg, getFlowId(ctx));
                ctx.writeAndFlush(resp);
            }
        }

    }
    复制代码

另一种是每个 DataPacket 的子类型都定义一个 Handler，如下图

    public class LocationMsgHandler extends SimpleChannelInboundHandler<LocationMsg> 
    public class HeartBeatMsgHandler extends SimpleChannelInboundHandler<HeartBeatMsg> 
    public class RegisterMsgHandler extends SimpleChannelInboundHandler<LogOutMsg> 
    复制代码

这里我选择第二种，一个原因是因为代码风格好，另一个原因后面会具体说明。

这里列举一个 LocationMsgHandler 的详细代码，将位置保存到数据库然后回复设备

```

@Slf4j
@Component
@ChannelHandler.Sharable
public class LocationMsgHandler extends BaseHandler<LocationMsg> {

    @Autowired
    private LocationRepository locationRespository;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, LocationMsg msg) throws Exception {
        log.debug(msg.toString());
        locationRespository.save(LocationEntity.parseFromLocationMsg(msg));
        CommonResp resp = CommonResp.success(msg, getSerialNumber(ctx.channel()));
        write(ctx, resp);
    }
}
复制代码
```

BaseHandler 继承 SimpleChannelInboundHandler ，里面定义了一些通用的方法，例如 getSerialNumber() 获取应答的流水号

        private static final AttributeKey<Short> SERIAL_NUMBER = AttributeKey.newInstance("serialNumber");

        public short getSerialNumber(Channel channel){
            Attribute<Short> flowIdAttr = channel.attr(SERIAL_NUMBER);
            Short flowId = flowIdAttr.get();
            if (flowId == null) {
                flowId = 0;
            } else {
                flowId++;
            }
            flowIdAttr.set(flowId);
            return flowId;
        }

    复制代码

我们将流水号存入 Channel 内部，方便维护。

### 3.4.Channel 的高效管理方式

假设现在出现了一个需求，我们需要找到一个特定的连接发送一条消息，在我们这个项目里，特定指的是根据 header 中的手机号找到连接并发送消息。我们可以自己维护一个 Map 用来存放所有 Channel，但是这样就浪费了 Netty 自带的 DefaultChannelGroup 提供的一系列方法了。所以我们改进一下，定义一个 ChannelManager，内部采用 DefaultChannelGroup 维护 Channel，自己维护手机号 ->ChannelId 的映射关系。

    @Component
    public class ChannelManager {

        private static final AttributeKey<String> TERMINAL_PHONE = AttributeKey.newInstance("terminalPhone");
        
        private ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

        private Map<String, ChannelId> channelIdMap = new ConcurrentHashMap<>();

        private ChannelFutureListener remover = future ->
                channelIdMap.remove(future.channel().attr(TERMINAL_PHONE).get());


        public boolean add(String terminalPhone, Channel channel) {
            boolean added = channelGroup.add(channel);
            if (added) {
                channel.attr(TERMINAL_PHONE).set(terminalPhone);
                channel.closeFuture().addListener(remover);
                channelIdMap.put(terminalPhone, channel.id());
            }
            return added;
        }

        public boolean remove(String terminalPhone) {
            return channelGroup.remove(channelIdMap.remove(terminalPhone));
        }

        public Channel get(String terminalPhone) {
            return channelGroup.find(channelIdMap.get(terminalPhone));
        }

        public ChannelGroup getChannelGroup() {
            return channelGroup;
        }
    }

    复制代码

我们定义了一个 ChannelFutureListener，当 channel 关闭时，会执行这个回调，帮助我们维护自己的 channelIdMap 不至于太过臃肿，提升效率，DefaultChannelGroup 中也是如此，所以不必担心 Channel 都不存在了 还占用着内存这种情况。另外我们可以将 DefaultChannelGroup 提供出去，以便某些时候进行广播。

### 3.5. 一些改进

1\. 我们的 LocationMsgHandler 中出现了数据库操作

            locationRespository.save(LocationEntity.parseFromLocationMsg(msg));
    复制代码

然而在 Netty 中，默认情况下 Handler 由 Reactor 线程驱动，一旦阻塞就会大大降低并发能力，所以我们定义一个专门的 EventExecutorGroup(不认识的话可以先理解为线程池)，用来驱动耗时的 Handler，只要在初始化 Channel 时指定即可。前面所说的每个 DataPacket 子类型定义一个 Handler 的另一个好处就体现在这里，我们可以让那些耗时的 Handler 用专门的业务线程池去驱动，而不耗时的 Handler 由默认的 Reactor 线程驱动，增加了灵活性。

            pipeline.addLast(heartBeatMsgHandler);
            pipeline.addLast(businessGroup,locationMsgHandler);//因为locationMsgHandler中涉及到数据库操作，所以放入businessGroup
            pipeline.addLast(authMsgHandler);
            pipeline.addLast(registerMsgHandler);
            pipeline.addLast(logOutMsgHandler);
    复制代码

另外如解码器 parse() 中的 switch 里的 case 顺序一样，我们这里也可以利用增加 Handler 的顺序，节省一些 if 判断。

2\. 接上面的，现在我们 LocationMsgHandler 由 businessGroup 驱动了，然而写响应的时候还是会移交给 Reactor 线程，所以为了减少一些判断提升略微的性能，我们可以将 write(ctx, resp); 改为

    workerGroup.execute(() -> write(ctx, resp));
    复制代码

其中的 workerGroup 正是启动引导中的，我们借助 Spring 把它单独定义成了 bean，用的时候直接注解引入即可

    serverBootstrap.group(bossGroup, workerGroup)
    复制代码

3\. 借助 Spring 的力量我们可以将几乎所有的组件定义成单例，提升了略微的性能，除了编码器和解码器，因为他们有一些属性需要维护，不能定义为单例。

## 结束语

一直看到这里的朋友感谢你们的耐心，这是我第一次写文章，有错误的地方还请多多包涵。

另外将完全版代码奉上 [zpsw/jt808-netty](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fzpsw%2Fjt808-netty "https&#x3A;//github.com/zpsw/jt808-netty")

这也是个人开源的第一个项目，如果对你有帮助，给个 Star 将不胜感激。

附上一些其他的 Netty 最佳实践 ([转自 best practice in netty](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspccold%2Fsailfish%2Fblob%2Fmaster%2Fbest-practice-in-netty.md "https&#x3A;//github.com/spccold/sailfish/blob/master/best-practice-in-netty.md"))：

-   writeAndFlush 不要一直调用， 是否可以通过调用 write，并且在适当的时间 flush，因为每次系统 flush 都是一次系统调用，如果可以的话 write 的调用次数也应该减少，因为它会经过整个 pipeline([github.com/netty/netty…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fnetty%2Fnetty%2Fissues%2F1759 "https&#x3A;//github.com/netty/netty/issues/1759"))
-   如果你不是很关注 write 的结果，可以使用 channel.voidPromise(), 可以减少对象的创建
-   一直写对于处理能力较弱的接受者来说，可能会引起 OutMemoryError，关注 channel.isWritable() 和 channelhandler 中的 cahnnelWritabilityChanged() 将会很有帮助，channel.bytesBeforeUnwritable 和 channel.bytesBeforeWritable() 同样值得关注
-   关注 write_buffer_high_water_mark 和 write_buffer_low_water_mark 的配置， 例如 high:32kb(default 64kb), low:8kb(default 32kb)
-   可以通过 channelpipeline 触发 custome events (pipeline.fireUserEventTriggered(MyCustomEvent)), 可以在 DuplexChannelHandler 中处理相应的事件

还有一些英文的就不贴过来了

另外给新手安利一个网络调试工具[NetAssist 网络调试助手](https://link.juejin.cn/?target=http%3A%2F%2Fwww.cmsoft.cn%2Fgetware.php%3Fid%3D205%26id2%3D361 "http&#x3A;//www.cmsoft.cn/getware.php?id=205&id2=361")

再见 
 [https://juejin.cn/post/6844903846125240327](https://juejin.cn/post/6844903846125240327)
