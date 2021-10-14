# java解析JT808协议_hylexus的博客-CSDN博客_jt808协议解析
本篇文章将介绍 JT808 协议的解析思路。  
**另请大神绕路，不喜勿喷！**  
先写个大致的思路, 有疑问可以联系本人，联系方式：

-   emial: hylexus@163.com

## 1.1 数据类型

| 数据类型     | 描述及要求              |
| -------- | ------------------ |
| BYTE     | 无符号单字节整形（字节， 8 位）  |
| WORD     | 无符号双字节整形（字， 16 位）  |
| DWORD    | 无符号四字节整形（双字， 32 位） |
| BYTE\[n] | n 字节               |
| BCD\[n]  | 8421 码， n 字节       |
| STRING   | GBK 编码，若无数据，置空     |

## 1.2 消息结构

| 标识位         | 消息头    | 消息体 | 校验码   | 标识位         |
| ----------- | ------ | --- | ----- | ----------- |
| 1byte(0x7e) | 16byte |     | 1byte | 1byte(0x7e) |

## 1.3 消息头

````null
消息ID(0-1)   消息体属性(2-3)  终端手机号(4-9)  消息流水号(10-11)    消息包封装项(12-15)

byte[0-1]   消息ID word(16)
byte[2-3]   消息体属性 word(16)
        bit[0-9]    消息体长度
        bit[10-12]  数据加密方式
                        此三位都为 0，表示消息体不加密
                        第 10 位为 1，表示消息体经过 RSA 算法加密
                        其它保留
        bit[13]     分包
                        1：消息体卫长消息，进行分包发送处理，具体分包信息由消息包封装项决定
                        0：则消息头中无消息包封装项字段
        bit[14-15]  保留
byte[4-9]   终端手机号或设备ID bcd[6]
        根据安装后终端自身的手机号转换
        手机号不足12 位，则在前面补 0
byte[10-11]     消息流水号 word(16)
        按发送顺序从 0 开始循环累加
byte[12-15]     消息包封装项
        byte[0-1]   消息包总数(word(16))
                        该消息分包后得总包数
        byte[2-3]   包序号(word(16))
                        从 1 开始
        如果消息体属性中相关标识位确定消息分包处理,则该项有内容
        否则无该项```

整个消息体结构中最复杂的就是消息头了。

2.1 消息体实体类
----------

以下是对整个消息体抽象出来的一个java实体类。

```null
import java.nio.channels.Channel;

public class PackageData {

    /**
     * 16byte 消息头
     */
    protected MsgHeader msgHeader;

    
    protected byte[] msgBodyBytes;

    /**
     * 校验码 1byte
     */
    protected int checkSum;

    
    protected Channel channel;

    public MsgHeader getMsgHeader() {
        return msgHeader;
    }

    

    
    public static class MsgHeader {
        
        protected int msgId;

        
        
        protected int msgBodyPropsField;
        
        protected int msgBodyLength;
        
        protected int encryptionType;
        
        protected boolean hasSubPackage;
        
        protected String reservedBit;
        

        
        protected String terminalPhone;
        
        protected int flowId;

        
        
        protected int packageInfoField;
        
        protected long totalSubPackage;
        
        protected long subPackageSeq;
        

        
    }

}
````

## 2.2 字节数组到消息体实体类的转换

### 2.2.1 消息转换器

````null
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import cn.hylexus.jt808.util.BCD8421Operater;
import cn.hylexus.jt808.util.BitOperator;
import cn.hylexus.jt808.vo.PackageData;
import cn.hylexus.jt808.vo.PackageData.MsgHeader;

public class MsgDecoder {

    private static final Logger log = LoggerFactory.getLogger(MsgDecoder.class);

    private BitOperator bitOperator;
    private BCD8421Operater bcd8421Operater;

    public MsgDecoder() {
        this.bitOperator = new BitOperator();
        this.bcd8421Operater = new BCD8421Operater();
    }

    
    public PackageData queueElement2PackageData(byte[] data) {
        PackageData ret = new PackageData();

        
        MsgHeader msgHeader = this.parseMsgHeaderFromBytes(data);
        ret.setMsgHeader(msgHeader);

        int msgBodyByteStartIndex = 12;
        
        
        if (msgHeader.isHasSubPackage()) {
            msgBodyByteStartIndex = 16;
        }

        byte[] tmp = new byte[msgHeader.getMsgBodyLength()];
        System.arraycopy(data, msgBodyByteStartIndex, tmp, 0, tmp.length);
        ret.setMsgBodyBytes(tmp);

        
        
        
        int checkSumInPkg = data[data.length - 1];
        int calculatedCheckSum = this.bitOperator.getCheckSum4JT808(data, 0, data.length - 1);
        ret.setCheckSum(checkSumInPkg);
        if (checkSumInPkg != calculatedCheckSum) {
            log.warn("检验码不一致,msgid:{},pkg:{},calculated:{}", msgHeader.getMsgId(), checkSumInPkg, calculatedCheckSum);
        }
        return ret;
    }

    private MsgHeader parseMsgHeaderFromBytes(byte[] data) {
        MsgHeader msgHeader = new MsgHeader();

        
        
        
        
        msgHeader.setMsgId(this.parseIntFromBytes(data, 0, 2));

        
        
        
        int msgBodyProps = this.parseIntFromBytes(data, 2, 2);
        msgHeader.setMsgBodyPropsField(msgBodyProps);
        
        msgHeader.setMsgBodyLength(msgBodyProps & 0x1ff);
        
        msgHeader.setEncryptionType((msgBodyProps & 0xe00) >> 10);
        
        msgHeader.setHasSubPackage(((msgBodyProps & 0x2000) >> 13) == 1);
        
        msgHeader.setReservedBit(((msgBodyProps & 0xc000) >> 14) + "");
        

        
        
        
        
        msgHeader.setTerminalPhone(this.parseBcdStringFromBytes(data, 4, 6));

        
        
        
        
        msgHeader.setFlowId(this.parseIntFromBytes(data, 10, 2));

        
        
        if (msgHeader.isHasSubPackage()) {
            
            msgHeader.setPackageInfoField(this.parseIntFromBytes(data, 12, 4));
            
            
            
            
            msgHeader.setTotalSubPackage(this.parseIntFromBytes(data, 12, 2));

            
            
            
            
            msgHeader.setSubPackageSeq(this.parseIntFromBytes(data, 12, 2));
        }
        return msgHeader;
    }

    protected String parseStringFromBytes(byte[] data, int startIndex, int lenth) {
        return this.parseStringFromBytes(data, startIndex, lenth, null);
    }

    private String parseStringFromBytes(byte[] data, int startIndex, int lenth, String defaultVal) {
        try {
            byte[] tmp = new byte[lenth];
            System.arraycopy(data, startIndex, tmp, 0, lenth);
            return new String(tmp, "UTF-8");
        } catch (Exception e) {
            log.error("解析字符串出错:{}", e.getMessage());
            e.printStackTrace();
            return defaultVal;
        }
    }

    private String parseBcdStringFromBytes(byte[] data, int startIndex, int lenth) {
        return this.parseBcdStringFromBytes(data, startIndex, lenth, null);
    }

    private String parseBcdStringFromBytes(byte[] data, int startIndex, int lenth, String defaultVal) {
        try {
            byte[] tmp = new byte[lenth];
            System.arraycopy(data, startIndex, tmp, 0, lenth);
            return this.bcd8421Operater.bcd2String(tmp);
        } catch (Exception e) {
            log.error("解析BCD(8421码)出错:{}", e.getMessage());
            e.printStackTrace();
            return defaultVal;
        }
    }

    private int parseIntFromBytes(byte[] data, int startIndex, int length) {
        return this.parseIntFromBytes(data, startIndex, length, 0);
    }

    private int parseIntFromBytes(byte[] data, int startIndex, int length, int defaultVal) {
        try {
            
            final int len = length > 4 ? 4 : length;
            byte[] tmp = new byte[len];
            System.arraycopy(data, startIndex, tmp, 0, len);
            return bitOperator.byteToInteger(tmp);
        } catch (Exception e) {
            log.error("解析整数出错:{}", e.getMessage());
            e.printStackTrace();
            return defaultVal;
        }
    }
}```

### 2.2.2 用到的工具类

#### 2.2.2.1 BCD操作工具类

```null
package cn.hylexus.jt808.util;

public class BCD8421Operater {

    /**
     * BCD字节数组===>String
     * 
     * @param bytes
     * @return 十进制字符串
     */
    public String bcd2String(byte[] bytes) {
        StringBuilder temp = new StringBuilder(bytes.length * 2);
        for (int i = 0; i < bytes.length; i++) {
            
            temp.append((bytes[i] & 0xf0) >>> 4);
            
            temp.append(bytes[i] & 0x0f);
        }
        return temp.toString().substring(0, 1).equalsIgnoreCase("0") ? temp.toString().substring(1) : temp.toString();
    }

    /**
     * 字符串==>BCD字节数组
     * 
     * @param str
     * @return BCD字节数组
     */
    public byte[] string2Bcd(String str) {
        
        if ((str.length() & 0x1) == 1) {
            str = "0" + str;
        }

        byte ret[] = new byte[str.length() / 2];
        byte bs[] = str.getBytes();
        for (int i = 0; i < ret.length; i++) {

            byte high = ascII2Bcd(bs[2 * i]);
            byte low = ascII2Bcd(bs[2 * i + 1]);

            
            ret[i] = (byte) ((high << 4) | low);
        }
        return ret;
    }

    private byte ascII2Bcd(byte asc) {
        if ((asc >= '0') && (asc <= '9'))
            return (byte) (asc - '0');
        else if ((asc >= 'A') && (asc <= 'F'))
            return (byte) (asc - 'A' + 10);
        else if ((asc >= 'a') && (asc <= 'f'))
            return (byte) (asc - 'a' + 10);
        else
            return (byte) (asc - 48);
    }
}
````

#### 2.2.2.2 位操作工具类

````null
package cn.hylexus.jt808.util;

import java.util.Arrays;
import java.util.List;

public class BitOperator {

    /**
     * 把一个整形该为byte
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte integerTo1Byte(int value) {
        return (byte) (value & 0xFF);
    }

    /**
     * 把一个整形该为1位的byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] integerTo1Bytes(int value) {
        byte[] result = new byte[1];
        result[0] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * 把一个整形改为2位的byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] integerTo2Bytes(int value) {
        byte[] result = new byte[2];
        result[0] = (byte) ((value >>> 8) & 0xFF);
        result[1] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * 把一个整形改为3位的byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] integerTo3Bytes(int value) {
        byte[] result = new byte[3];
        result[0] = (byte) ((value >>> 16) & 0xFF);
        result[1] = (byte) ((value >>> 8) & 0xFF);
        result[2] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * 把一个整形改为4位的byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] integerTo4Bytes(int value){
        byte[] result = new byte[4];
        result[0] = (byte) ((value >>> 24) & 0xFF);
        result[1] = (byte) ((value >>> 16) & 0xFF);
        result[2] = (byte) ((value >>> 8) & 0xFF);
        result[3] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * 把byte[]转化位整形,通常为指令用
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public int byteToInteger(byte[] value) {
        int result;
        if (value.length == 1) {
            result = oneByteToInteger(value[0]);
        } else if (value.length == 2) {
            result = twoBytesToInteger(value);
        } else if (value.length == 3) {
            result = threeBytesToInteger(value);
        } else if (value.length == 4) {
            result = fourBytesToInteger(value);
        } else {
            result = fourBytesToInteger(value);
        }
        return result;
    }

    /**
     * 把一个byte转化位整形,通常为指令用
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public int oneByteToInteger(byte value) {
        return (int) value & 0xFF;
    }

    /**
     * 把一个2位的数组转化位整形
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public int twoBytesToInteger(byte[] value) {
        
        
        
        int temp0 = value[0] & 0xFF;
        int temp1 = value[1] & 0xFF;
        return ((temp0 << 8) + temp1);
    }

    /**
     * 把一个3位的数组转化位整形
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public int threeBytesToInteger(byte[] value) {
        int temp0 = value[0] & 0xFF;
        int temp1 = value[1] & 0xFF;
        int temp2 = value[2] & 0xFF;
        return ((temp0 << 16) + (temp1 << 8) + temp2);
    }

    /**
     * 把一个4位的数组转化位整形,通常为指令用
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public int fourBytesToInteger(byte[] value) {
        
        
        
        int temp0 = value[0] & 0xFF;
        int temp1 = value[1] & 0xFF;
        int temp2 = value[2] & 0xFF;
        int temp3 = value[3] & 0xFF;
        return ((temp0 << 24) + (temp1 << 16) + (temp2 << 8) + temp3);
    }

    /**
     * 把一个4位的数组转化位整形
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public long fourBytesToLong(byte[] value) throws Exception {
        
        
        
        int temp0 = value[0] & 0xFF;
        int temp1 = value[1] & 0xFF;
        int temp2 = value[2] & 0xFF;
        int temp3 = value[3] & 0xFF;
        return (((long) temp0 << 24) + (temp1 << 16) + (temp2 << 8) + temp3);
    }

    /**
     * 把一个数组转化长整形
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public long bytes2Long(byte[] value) {
        long result = 0;
        int len = value.length;
        int temp;
        for (int i = 0; i < len; i++) {
            temp = (len - 1 - i) * 8;
            if (temp == 0) {
                result += (value[i] & 0x0ff);
            } else {
                result += (value[i] & 0x0ff) << temp;
            }
        }
        return result;
    }

    /**
     * 把一个长整形改为byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] longToBytes(long value){
        return longToBytes(value, 8);
    }

    /**
     * 把一个长整形改为byte数组
     * 
     * @param value
     * @return
     * @throws Exception
     */
    public byte[] longToBytes(long value, int len) {
        byte[] result = new byte[len];
        int temp;
        for (int i = 0; i < len; i++) {
            temp = (len - 1 - i) * 8;
            if (temp == 0) {
                result[i] += (value & 0x0ff);
            } else {
                result[i] += (value >>> temp) & 0x0ff;
            }
        }
        return result;
    }

    /**
     * 得到一个消息ID
     * 
     * @return
     * @throws Exception
     */
    public byte[] generateTransactionID() throws Exception {
        byte[] id = new byte[16];
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 0, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 2, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 4, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 6, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 8, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 10, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 12, 2);
        System.arraycopy(integerTo2Bytes((int) (Math.random() * 65536)), 0, id, 14, 2);
        return id;
    }

    /**
     * 把IP拆分位int数组
     * 
     * @param ip
     * @return
     * @throws Exception
     */
    public int[] getIntIPValue(String ip) throws Exception {
        String[] sip = ip.split("[.]");
        
        
        
        int[] intIP = { Integer.parseInt(sip[0]), Integer.parseInt(sip[1]), Integer.parseInt(sip[2]),
                Integer.parseInt(sip[3]) };
        return intIP;
    }

    /**
     * 把byte类型IP地址转化位字符串
     * 
     * @param address
     * @return
     * @throws Exception
     */
    public String getStringIPValue(byte[] address) throws Exception {
        int first = this.oneByteToInteger(address[0]);
        int second = this.oneByteToInteger(address[1]);
        int third = this.oneByteToInteger(address[2]);
        int fourth = this.oneByteToInteger(address[3]);

        return first + "." + second + "." + third + "." + fourth;
    }

    /**
     * 合并字节数组
     * 
     * @param first
     * @param rest
     * @return
     */
    public byte[] concatAll(byte[] first, byte[]... rest) {
        int totalLength = first.length;
        for (byte[] array : rest) {
            if (array != null) {
                totalLength += array.length;
            }
        }
        byte[] result = Arrays.copyOf(first, totalLength);
        int offset = first.length;
        for (byte[] array : rest) {
            if (array != null) {
                System.arraycopy(array, 0, result, offset, array.length);
                offset += array.length;
            }
        }
        return result;
    }

    /**
     * 合并字节数组
     * 
     * @param rest
     * @return
     */
    public byte[] concatAll(List<byte[]> rest) {
        int totalLength = 0;
        for (byte[] array : rest) {
            if (array != null) {
                totalLength += array.length;
            }
        }
        byte[] result = new byte[totalLength];
        int offset = 0;
        for (byte[] array : rest) {
            if (array != null) {
                System.arraycopy(array, 0, result, offset, array.length);
                offset += array.length;
            }
        }
        return result;
    }

    public float byte2Float(byte[] bs) {
        return Float.intBitsToFloat(
                (((bs[3] & 0xFF) << 24) + ((bs[2] & 0xFF) << 16) + ((bs[1] & 0xFF) << 8) + (bs[0] & 0xFF)));
    }

    public float byteBE2Float(byte[] bytes) {
        int l;
        l = bytes[0];
        l &= 0xff;
        l |= ((long) bytes[1] << 8);
        l &= 0xffff;
        l |= ((long) bytes[2] << 16);
        l &= 0xffffff;
        l |= ((long) bytes[3] << 24);
        return Float.intBitsToFloat(l);
    }

    public int getCheckSum4JT808(byte[] bs, int start, int end) {
        if (start < 0 || end > bs.length)
            throw new ArrayIndexOutOfBoundsException("getCheckSum4JT808 error : index out of bounds(start=" + start
                    + ",end=" + end + ",bytes length=" + bs.length + ")");
        int cs = 0;
        for (int i = start; i < end; i++) {
            cs ^= bs[i];
        }
        return cs;
    }

    public int getBitRange(int number, int start, int end) {
        if (start < 0)
            throw new IndexOutOfBoundsException("min index is 0,but start = " + start);
        if (end >= Integer.SIZE)
            throw new IndexOutOfBoundsException("max index is " + (Integer.SIZE - 1) + ",but end = " + end);

        return (number << Integer.SIZE - (end + 1)) >>> Integer.SIZE - (end - start + 1);
    }

    public int getBitAt(int number, int index) {
        if (index < 0)
            throw new IndexOutOfBoundsException("min index is 0,but " + index);
        if (index >= Integer.SIZE)
            throw new IndexOutOfBoundsException("max index is " + (Integer.SIZE - 1) + ",but " + index);

        return ((1 << index) & number) >> index;
    }

    public int getBitAtS(int number, int index) {
        String s = Integer.toBinaryString(number);
        return Integer.parseInt(s.charAt(index) + "");
    }

    @Deprecated
    public int getBitRangeS(int number, int start, int end) {
        String s = Integer.toBinaryString(number);
        StringBuilder sb = new StringBuilder(s);
        while (sb.length() < Integer.SIZE) {
            sb.insert(0, "0");
        }
        String tmp = sb.reverse().substring(start, end + 1);
        sb = new StringBuilder(tmp);
        return Integer.parseInt(sb.reverse().toString(), 2);
    }
}```

2.3 和netty结合
------------

### 2.3.1 netty处理器链

```null
import java.util.concurrent.TimeUnit;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import cn.kkbc.tpms.tcp.service.TCPServerHandler;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.timeout.IdleStateHandler;
import io.netty.util.concurrent.Future;

public class TCPServer2 {

    private Logger log = LoggerFactory.getLogger(getClass());
    private volatile boolean isRunning = false;

    private EventLoopGroup bossGroup = null;
    private EventLoopGroup workerGroup = null;
    private int port;

    public TCPServer2() {
    }

    public TCPServer2(int port) {
        this();
        this.port = port;
    }

    private void bind() throws Exception {
        this.bossGroup = new NioEventLoopGroup();
        this.workerGroup = new NioEventLoopGroup();
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class) 
                .childHandler(new ChannelInitializer<SocketChannel>() { 
                    @Override
                    public void initChannel(SocketChannel ch) throws Exception {
                        
                        ch.pipeline().addLast("idleStateHandler",
                                new IdleStateHandler(15, 0, 0, TimeUnit.MINUTES));
                        
                        
                        ch.pipeline().addLast(
                                new DelimiterBasedFrameDecoder(1024, Unpooled.copiedBuffer(new byte[] { 0x7e }),
                                        Unpooled.copiedBuffer(new byte[] { 0x7e, 0x7e })));
                        ch.pipeline().addLast(new TCPServerHandler());
                    }
                }).option(ChannelOption.SO_BACKLOG, 128) 
                .childOption(ChannelOption.SO_KEEPALIVE, true);

        this.log.info("TCP服务启动完毕,port={}", this.port);
        ChannelFuture channelFuture = serverBootstrap.bind(port).sync();

        channelFuture.channel().closeFuture().sync();
    }

    public synchronized void startServer() {
        if (this.isRunning) {
            throw new IllegalStateException(this.getName() + " is already started .");
        }
        this.isRunning = true;

        new Thread(() -> {
            try {
                this.bind();
            } catch (Exception e) {
                this.log.info("TCP服务启动出错:{}", e.getMessage());
                e.printStackTrace();
            }
        }, this.getName()).start();
    }

    public synchronized void stopServer() {
        if (!this.isRunning) {
            throw new IllegalStateException(this.getName() + " is not yet started .");
        }
        this.isRunning = false;

        try {
            Future<?> future = this.workerGroup.shutdownGracefully().await();
            if (!future.isSuccess()) {
                log.error("workerGroup 无法正常停止:{}", future.cause());
            }

            future = this.bossGroup.shutdownGracefully().await();
            if (!future.isSuccess()) {
                log.error("bossGroup 无法正常停止:{}", future.cause());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        this.log.info("TCP服务已经停止...");
    }

    private String getName() {
        return "TCP-Server";
    }

    public static void main(String[] args) throws Exception {
        TCPServer2 server = new TCPServer2(20048);
        server.startServer();

        
        
    }

}```

### 2.3.2 netty针对于JT808的消息处理器

```null
package cn.hylexus.jt808.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import cn.hylexus.jt808.server.SessionManager;
import cn.hylexus.jt808.service.codec.MsgDecoder;
import cn.hylexus.jt808.vo.PackageData;
import cn.hylexus.jt808.vo.Session;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.handler.timeout.IdleState;
import io.netty.handler.timeout.IdleStateEvent;
import io.netty.util.ReferenceCountUtil;

public class TCPServerHandler extends ChannelInboundHandlerAdapter { 

    private final Logger logger = LoggerFactory.getLogger(getClass());

    
    private final SessionManager sessionManager;
    private MsgDecoder decoder = new MsgDecoder();

    public TCPServerHandler() {
        this.sessionManager = SessionManager.getInstance();
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws InterruptedException { 
        try {
            ByteBuf buf = (ByteBuf) msg;
            if (buf.readableBytes() <= 0) {
                
                return;
            }

            byte[] bs = new byte[buf.readableBytes()];
            buf.readBytes(bs);

            PackageData jt808Msg = this.decoder.queueElement2PackageData(bs);
            
            this.processClientMsg(jt808Msg);
        } finally {
            release(msg);
        }
    }

    private void processClientMsg(PackageData jt808Msg) {
        
        if (jt808Msg.getMsgHeader().getMsgId() == 0x900) {
            
        } else if (jt808Msg.getMsgHeader().getMsgId() == 0x9001) {
            
        }
        
        
        
        
        
        else {
            logger.error("位置消息,消息ID={}", jt808Msg.getMsgHeader().getMsgId());
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { 
        logger.error("发生异常:{}", cause.getMessage());
        cause.printStackTrace();
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        Session session = Session.buildSession(ctx.channel());
        sessionManager.put(session.getId(), session);
        logger.debug("终端连接:{}", session);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        final String sessionId = ctx.channel().id().asLongText();
        Session session = sessionManager.findBySessionId(sessionId);
        this.sessionManager.removeBySessionId(sessionId);
        logger.debug("终端断开连接:{}", session);
        ctx.channel().close();
        
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (IdleStateEvent.class.isAssignableFrom(evt.getClass())) {
            IdleStateEvent event = (IdleStateEvent) evt;
            if (event.state() == IdleState.READER_IDLE) {
                Session session = this.sessionManager.removeBySessionId(Session.buildId(ctx.channel()));
                logger.error("服务器主动断开连接:{}", session);
                ctx.close();
            }
        }
    }

    private void release(Object msg) {
        try {
            ReferenceCountUtil.release(msg);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}```

### 2.3.3 用到的其他类

```null
package cn.hylexus.jt808.server;

import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.BiConsumer;
import java.util.stream.Collectors;

import cn.hylexus.jt808.vo.Session;

public class SessionManager {

    private static volatile SessionManager instance = null;
    
    private Map<String, Session> sessionIdMap;
    
    private Map<String, String> phoneMap;

    public static SessionManager getInstance() {
        if (instance == null) {
            synchronized (SessionManager.class) {
                if (instance == null) {
                    instance = new SessionManager();
                }
            }
        }
        return instance;
    }

    public SessionManager() {
        this.sessionIdMap = new ConcurrentHashMap<>();
        this.phoneMap = new ConcurrentHashMap<>();
    }

    public boolean containsKey(String sessionId) {
        return sessionIdMap.containsKey(sessionId);
    }

    public boolean containsSession(Session session) {
        return sessionIdMap.containsValue(session);
    }

    public Session findBySessionId(String id) {
        return sessionIdMap.get(id);
    }

    public Session findByTerminalPhone(String phone) {
        String sessionId = this.phoneMap.get(phone);
        if (sessionId == null)
            return null;
        return this.findBySessionId(sessionId);
    }

    public synchronized Session put(String key, Session value) {
        if (value.getTerminalPhone() != null && !"".equals(value.getTerminalPhone().trim())) {
            this.phoneMap.put(value.getTerminalPhone(), value.getId());
        }
        return sessionIdMap.put(key, value);
    }

    public synchronized Session removeBySessionId(String sessionId) {
        if (sessionId == null)
            return null;
        Session session = sessionIdMap.remove(sessionId);
        if (session == null)
            return null;
        if (session.getTerminalPhone() != null)
            this.phoneMap.remove(session.getTerminalPhone());
        return session;
    }

    public Set<String> keySet() {
        return sessionIdMap.keySet();
    }

    public void forEach(BiConsumer<? super String, ? super Session> action) {
        sessionIdMap.forEach(action);
    }

    public Set<Entry<String, Session>> entrySet() {
        return sessionIdMap.entrySet();
    }

    public List<Session> toList() {
        return this.sessionIdMap.entrySet().stream().map(e -> e.getValue()).collect(Collectors.toList());
    }

}```

请移步: [https://github.com/hylexus/jt-808-protocol](https://github.com/hylexus/jt-808-protocol)

另请不要吝啬，在GitHub给个star让小可装装逼…………^\_^

急急忙忙写的博客，先写个大致的思路,有疑问可以联系本人，联系方式：

*   emial: hylexus@163.com 
 [https://blog.csdn.net/hylexus/article/details/54987786#11-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B](https://blog.csdn.net/hylexus/article/details/54987786#11-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
````
