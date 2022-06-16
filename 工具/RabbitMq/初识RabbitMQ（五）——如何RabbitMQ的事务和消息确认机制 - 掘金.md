# 初识RabbitMQ（五）——如何RabbitMQ的事务和消息确认机制 - 掘金
这是我参与「掘金日新计划 · 6 月更文挑战」的第 6 天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https&#x3A;//juejin.cn/post/7099702781094674468")

### 前言

    大家好，我是程序猿小白 GW_gw，很高兴能和大家一起学习进步。
    复制代码

以下内容部分来自于网络，如有侵权，请联系我删除，本文仅用于学习交流，不用作任何商业用途。

### 摘要

    本文主要带大家了解RabbitMQ在Java中如何使用事务和进行消息确认，以及进行消息的防重复处理。
    复制代码

### 1. 事务

对于某些消息，我们应该一次全部发送成功或失败，这时我们就应该使用事务。

代码很简单，如下：

在发送消息之前显式声明事务：

    channel.txSelect();
    复制代码

在发送所有消息之后提交事务：

    channel.txCommit();
    复制代码

> 如果不提交事务，那消息只会存放在内存中而不会被发送。

关闭资源时回滚事务：

    channel.txRollback();
    复制代码

对于在内存中未被发送的事务进行内存的释放。

### 2. 发送者消息确认模式

对于出错的概率一般是很低的，使用事务相对来说效率会降低，我们可以使用发送者确认模式来代替事务。

简单来说就是发送消息后，接收服务端的确认消息来确认消息来判断是否发送成功。

#### 2.1 简单确认模式

每次发送一条消息都要进行确认。

    //启用发送者确认模式
    channel.confirmSelect();
    //发送消息
    channel.basicPublish("directConfirmExchange","confirmRoutingKey",null,message.getBytes(StandardCharsets.UTF_8));
    //阻塞线程等待服务器响应
    boolean b = channel.waitForConfirms();
    //指定等待服务器确认的超时时间
    //boolean b1 = channel.waitForConfirms(3000L);
    复制代码

>     普通确认，每次调用该方法只能确认一条消息
>     返回false或抛出异常(超过超时时间)都有可能发送成功或没有发送成功，可以采用
>     递归或Redis+定时任务来完成补发消息
>     复制代码

#### 2.2 批量确认模式

    //启用发送者确认模式
    channel.confirmSelect();        channel.basicPublish("directConfirmExchange","confirmRoutingKey",null,message.getBytes(StandardCharsets.UTF_8));
    //阻塞线程等待服务器响应       
    channel.waitForConfirmsOrDie();
    //指定等待服务器确认的超时时间
    //channel.waitForConfirmsOrDie(3000L);
    复制代码

>     批量消息确认。
>     它会确认当前信道的所有消息是否成功写入，
>     如果有一条消息没有被写入或服务器不可访问都会抛出异常，需要进行全部消息的补发。
>     复制代码

#### 2.3 异步消息确认模式

对于异步消息确认模式我们需要先启动监听。

关注两个方法：

1.  消息确认后的回调方法 handleAck(long l, boolean b)
2.  消息未确认的回调方法 handleNack(long l, boolean b)

参数解释：

> handleAck(long l, boolean b)
>
> 参数 1：表示被确认的消息的编号，从 1 开始
>
> 参数 2：表示是否确认了多条消息，ture 表示在当前消息编号之前和上一次确认的消息编号之间的消息已经被确认，false 表示只确认了当前消息
>
> handleNack(long l, boolean b)
>
> 参数 1：表示未被确认的消息的编号，从 1 开始
>
> 参数 2：表示是否确认了多条消息，ture 表示在当前消息编号之前和上一次确认的消息编号之间的消息未被确认，false 表示当前消息未被确认

    //启用发送者确认模式
    channel.confirmSelect();
    //异步的监听器需要先启动，放在发送消息之前
    channel.addConfirmListener(new ConfirmListener() {
        //消息确认后的回调方法
        @Override
        public void handleAck(long l, boolean b) throws IOException {
            System.out.println("被确认的当前消息编号是:" + l + "是否确认了多条消息:" + b);
        }
        //消息未确认的回调方法
        @Override
        public void handleNack(long l, boolean b) throws IOException {
            System.out.println("未被确认的当前消息编号是:" + l + "是否确认了多条消息:" + b);
        }
    });
    //发送测试消息
    for (int i = 0; i < 1000; i++) {
        channel.basicPublish("directConfirmExchange", "confirmRoutingKey", null, message.getBytes(StandardCharsets.UTF_8));
    }
    复制代码

### 3. 消费者手动确认消息和防重复处理

> 参数 2 设置为 false，手动确认消息。
>
> c.basicAck(tag,true); 手动确认消息。
>
> 参数 1：当前消息编号。 参数 2：是否确认多条消息，true 表示确认小于等于当前编号的消息，false 表示确认当前消息。
>
> 使用 isRedeliver() 方法判断消息是否处理过，进行防重复处理。

    channel.basicConsume("confirmQueue",false,"",new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            //获取信道
            Channel c = this.getChannel();
            boolean redeliver = envelope.isRedeliver();
            //防止重复处理
            if(!redeliver){
                String message = new String(body, StandardCharsets.UTF_8);
                System.out.println(message);
                //获取当前消息的编号
                long tag = envelope.getDeliveryTag();
                //手动确认消息
                c.basicAck(tag,true);
            }
            else{
    ​
            }
        }
    });
    复制代码

注意：

> 消费者为手动确认模式且使用事务时，需要进行提交事务，否则就算手动确认了消息也无效，但是消息被认为是处理过的。

## 小结

以上就是关于 RabbitMQ 如何使用事务，以及消息确认的几种模式和消费者如何进行防重复处理，希望能对读者有所帮助，觉好留赞哦。 
 [https://juejin.cn/post/7104917591474307109](https://juejin.cn/post/7104917591474307109)
