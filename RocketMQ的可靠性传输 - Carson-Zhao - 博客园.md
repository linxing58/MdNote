# RocketMQ的可靠性传输 - Carson-Zhao - 博客园
## 整体

![](https://img2022.cnblogs.com/blog/2183293/202203/2183293-20220310163930448-1856102611.png)

## 分析：

**需确保一发一存一消费这些过程均无消息丢失**

利用**ACK 机制**保证每个阶段需要执行的操作成功后，再往下一个阶段推动（放行）

消息处理过程：

![](https://img2022.cnblogs.com/blog/2183293/202203/2183293-20220310170046973-926146131.png)

由上图分析可知：

消息丢失，可能发生在三个阶段，生产阶段、存储阶段、消费阶段

如下，为每个阶段保证消息不丢失：

**消息生产阶段**：

利用 MQ 的 ack 确认机制，在 try-catch 中处理好 Broker 的返回值，如果返回失败，则进行重试，若重试次数过多，则进行报警日志打印，排查解决问题

**消息存储阶段**：

刷盘存储的消息进行多副本备份处理，从高可用角度取设计中间件，搭建集群；同时，中间件也会进行备份，至少两个节点以上备份成功之后才会给生产者返回 ack 确认消息

**消息消费阶段**：

消费者从消费队列中拉去消息后，不是立马给 Broker 返回 ack 确认消息，而是等待业务代码顺利执行完成之后，再给 Broker 返回 ack 确认消息

## 实现：

### Producer——>Broker

-   发送方式

    -   同步发送

        -   Producer 向 broker 发送消息，会阻塞当前线程等待 broker 响应结果

        ````null
        public class SyncProducer {	public static void main(String[] args) throws Exception {    	// 实例化消息生产者Producer        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");    	// 设置NameServer的地址	    	producer.setNamesrvAddr("localhost:9876");    	// 启动Producer实例        producer.start();    	for (int i = 0; i < 100; i++) {    	    // 创建消息，并指定Topic，Tag和消息体    	    Message msg = new Message("TopicTest" /* Topic */,        	"TagA" /* Tag */,        	("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */        	);        	// 发送消息到一个Broker            SendResult sendResult = producer.send(msg);            // 通过sendResult返回消息是否成功送达            System.out.printf("%s%n", sendResult);    	}    	// 如果不再发送消息，关闭Producer实例。    	producer.shutdown();    }}```

        ````
    -   异步发送

        -   Producer 首先构建一个向 broker 发送消息的任务，把该任务提交给线程池，等执行完该任务时，回调用户自定义的回调函数，执行处理结果

        ````null
        public class AsyncProducer {	public static void main(String[] args) throws Exception {    	// 实例化消息生产者Producer        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");    	// 设置NameServer的地址        producer.setNamesrvAddr("localhost:9876");    	// 启动Producer实例        producer.start();        producer.setRetryTimesWhenSendAsyncFailed(0);		int messageCount = 100;        // 根据消息数量实例化倒计时计算器	final CountDownLatch2 countDownLatch = new CountDownLatch2(messageCount);    	for (int i = 0; i < messageCount; i++) {                final int index = i;            	// 创建消息，并指定Topic，Tag和消息体                Message msg = new Message("TopicTest",                    "TagA",                    "OrderID188",                    "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));                // SendCallback接收异步返回结果的回调                producer.send(msg, new SendCallback() {                    @Override                    public void onSuccess(SendResult sendResult) {                        countDownLatch.countDown();                        System.out.printf("%-10d OK %s %n", index,                            sendResult.getMsgId());                    }                    @Override                    public void onException(Throwable e) {                        countDownLatch.countDown();      	                System.out.printf("%-10d Exception %s %n", index, e);      	                e.printStackTrace();                    }            	});    	}	// 等待5s	countDownLatch.await(5, TimeUnit.SECONDS);    	// 如果不再发送消息，关闭Producer实例。    	producer.shutdown();    }}```

        ````
    -   Oneway

        -   Oneway 方式只负责发送请求，不等待应答，Producer 只负责把请求发出去，不会处理响应结果

        ````null
        public class OnewayProducer {	public static void main(String[] args) throws Exception{    	// 实例化消息生产者Producer        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");    	// 设置NameServer的地址        producer.setNamesrvAddr("localhost:9876");    	// 启动Producer实例        producer.start();    	for (int i = 0; i < 100; i++) {        	// 创建消息，并指定Topic，Tag和消息体        	Message msg = new Message("TopicTest" /* Topic */,                "TagA" /* Tag */,                ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */        	);        	// 发送单向消息，没有任何返回结果        	producer.sendOneway(msg);     	}    	// 如果不再发送消息，关闭Producer实例。    	producer.shutdown();    }}```

        ````
-   推荐

    同步发送：

    -   同步发送会返回四个状态码
        -   SEND_OK：消息发送成功
        -   FLUSH_DISK_TIMEOUT：消息发送成功但是消息刷盘超时
        -   FLUSH_SLAVE_TIMEOUT：消息发送成功但是消息同步到 slave 节点时超时
        -   SLAVE_NOT_AVAILABLE：消息发送成功但是 broker 的 slave 节点不可用
    -   处理
        -   根据返回的状态码，进行消息重试，默认设置为 3 次，可以通过设置调整

            > producer.setRetryTimesWhenSendFailed(重试次数);

    异步发送：

    -   在 onException() 方法中处理，如果发送失败，则在这里执行重试

    额外问题：

    -   如果 Broker 收到消息后，就因为某些原因宕机了，就算 Producer 再怎么重试都是无法解决消息丢失的问题，该如何处理？

    👉 利用**多主模式**，挂了一个，就换一个 master 继续消息发送

### 总结：

保证 Producer——>Broker 消息不丢失的方案

### Broker 存储及备份

-   刷盘

    ![](https://img2022.cnblogs.com/blog/2183293/202203/2183293-20220310164016571-1673152557.png)

    -   同步刷盘

        -   消息写入内存后，立刻调用刷盘线程进行刷盘
        -   如果消息在约定的时间内未刷盘成功（默认 5s），则返回 FLUSH_DISK_TIMEOUT，Producer 收到后进行重试
    -   异步刷盘（**默认**）

        -   消息写入 CommitLog 时，不会直接写入磁盘，而是先写到 PageCache 缓存后返回成功
        -   启用后台线程异步将消息刷入磁盘

-   高可用
    -   多主
        -   多个 Master 节点，防止单主宕机，丢失消息问题
    -   主从 + 双写
        -   主从的情况下（写入 master 成功后立即 ACK 给 Producer），会发生，master——>slave 时，主节点 Broker 宕机，同步失败，从而导致消息丢失
        -   开启双写，只有等 master 和 slave 都写入成功，即双写成功后才会 ACK 给 Producer，否则，会触发 Producer 的重试机制

### 总结

保证 Broker 存储及备份阶段，消息不丢失

### Broker——>Consumer

-   消息确认

    -   消费者从 Broker 中拉去消息后，不是立马给 Broker 返回 ack 确认消息，而是等待业务代码顺利执行完成之后，再给 Broker 返回 ack 确认消息
-   消息重试

    -   消息消费失败后，需提供重试消息的能力，RocketMQ 本身提供了重新消费的能力

    ### 总结

    保证 Broker——>Consumer 阶段，消息不丢失

## 最终方案：

posted on 2022-03-10 16:54  [Carson-Zhao](https://www.cnblogs.com/zhaorongbiao/)  阅读 (182)  评论 ()  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=15990277)  收藏  举报 
 [https://www.cnblogs.com/zhaorongbiao/p/15990277.html](https://www.cnblogs.com/zhaorongbiao/p/15990277.html)
