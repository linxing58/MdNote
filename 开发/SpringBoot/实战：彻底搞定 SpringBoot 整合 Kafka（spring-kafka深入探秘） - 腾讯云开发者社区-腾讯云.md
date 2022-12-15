# 实战：彻底搞定 SpringBoot 整合 Kafka（spring-kafka深入探秘） - 腾讯云开发者社区-腾讯云
来源：my.oschina.net/keking/blog/3056698
-------------------------------------

* * *

前言
--

kafka是一个[消息队列](https://cloud.tencent.com/product/cmq?from=10680)产品，基于Topic partitions的设计，能达到非常高的消息发送处理性能。Spring创建了一个项目Spring-kafka，封装了Apache 的Kafka-client，用于在Spring项目里快速集成kafka。

除了简单的收发消息外，Spring-kafka还提供了很多高级功能，下面我们就来一一探秘这些用法。

> 项目地址：https://github.com/spring-projects/spring-kafka

* * *

简单集成
----

### 引入依赖

```
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>2.2.6.RELEASE</version>
</dependency>
```

### 添加配置

```
spring.kafka.producer.bootstrap-servers=127.0.0.1:9092
```

### 测试发送和接收

```
 @SpringBootApplication
@RestController
public class Application {

    private final Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Autowired
    private KafkaTemplate<Object, Object> template;

    @GetMapping("/send/{input}")
    public void sendFoo(@PathVariable String input) {
        this.template.send("topic_input", input);
    }
    @KafkaListener(id = "webGroup", topics = "topic_input")
    public void listen(String input) {
        logger.info("input value: {}" , input);
    }
}
```

启动应用后，在浏览器中输入：http://localhost:8080/send/kl。就可以在控制台看到有日志输出了：input value: "kl"。基础的使用就这么简单。发送消息时注入一个KafkaTemplate，接收消息时添加一个`@KafkaListener`注解即可。

* * *

Spring-kafka-test嵌入式Kafka Server
--------------------------------

不过上面的代码能够启动成功，前提是你已经有了Kafka Server的服务环境，我们知道Kafka是由Scala + Zookeeper构建的，可以从官网下载部署包在本地部署。

但是，我想告诉你，为了简化开发环节验证Kafka相关功能，Spring-Kafka-Test已经封装了Kafka-test提供了注解式的一键开启Kafka Server的功能，使用起来也是超级简单。本文后面的所有测试用例的Kafka都是使用这种嵌入式服务提供的。

### 引入依赖

```
<dependency>
   <groupId>org.springframework.kafka</groupId>
   <artifactId>spring-kafka-test</artifactId>
   <version>2.2.6.RELEASE</version>
   <scope>test</scope>
</dependency>
```

### 启动服务

下面使用Junit测试用例，直接启动一个Kafka Server服务，包含四个Broker节点。

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ApplicationTests.class)
@EmbeddedKafka(count = 4,ports = {9092,9093,9094,9095})
public class ApplicationTests {
    @Test
    public void contextLoads()throws IOException {
        System.in.read();
    }
}
```

如上：只需要一个注解`@EmbeddedKafka`即可，就可以启动一个功能完整的Kafka服务，是不是很酷。默认只写注解不加参数的情况下，是创建一个随机端口的Broker，在启动的日志中会输出具体的端口以及默认的一些配置项。

不过这些我们在Kafka安装包配置文件中的配置项，在注解参数中都可以配置，下面详解下`@EmbeddedKafka`注解中的可设置参数 ：

*   **value：** broker节点数量
*   **count：** 同value作用一样，也是配置的broker的节点数量
*   **controlledShutdown：** 控制关闭开关，主要用来在Broker意外关闭时减少此Broker上Partition的不可用时间

Kafka是多Broker架构的高可用服务，一个Topic对应多个partition，一个Partition可以有多个副本Replication，这些Replication副本保存在多个Broker，用于高可用。

但是，虽然存在多个分区副本集，当前工作副本集却只有一个，默认就是首次分配的副本集【首选副本】为Leader，负责写入和读取数据。当我们升级Broker或者更新Broker配置时需要重启服务，这个时候需要将partition转移到可用的Broker。

下面涉及到三种情况

**1、直接关闭Broker：** 当Broker关闭时，Broker集群会重新进行选主操作，选出一个新的Broker来作为Partition Leader，选举时此Broker上的Partition会短时不可用

**2、开启controlledShutdown：** 当Broker关闭时，Broker本身会先尝试将Leader角色转移到其他可用的Broker上

**3、使用**[**命令行工具**](https://cloud.tencent.com/product/cli?from=10680)**：** 使用bin/kafka-preferred-replica-election.sh，手动触发PartitionLeader角色转移

**ports：** 端口列表，是一个数组。对应了count参数，有几个Broker，就要对应几个端口号

**brokerProperties：** Broker参数设置，是一个数组结构，支持如下方式进行Broker参数设置：

```
@EmbeddedKafka(brokerProperties = {"log.index.interval.bytes = 4096","num.io.threads = 8"})
```

**okerPropertiesLocation：** Broker参数文件设置

功能同上面的brokerProperties，只是Kafka Broker的可设置参数达182个之多，都像上面这样配置肯定不是最优方案，所以提供了加载本地配置文件的功能，如：

```
@EmbeddedKafka(brokerPropertiesLocation = "classpath:application.properties")
```

* * *

创建新的Topic
---------

默认情况下，如果在使用KafkaTemplate发送消息时，Topic不存在，会创建一个新的Topic，默认的分区数和副本数为如下Broker参数来设定

```
num.partitions = 1 #默认Topic分区数
num.replica.fetchers = 1 #默认副本数
```

### 程序启动时创建Topic

```
 @Configuration
public class KafkaConfig {
    @Bean
    public KafkaAdmin admin(KafkaProperties properties){
        KafkaAdmin admin = new KafkaAdmin(properties.buildAdminProperties());
        admin.setFatalIfBrokerNotAvailable(true);
        return admin;
    }
    @Bean
    public NewTopic topic2() {
        return new NewTopic("topic-kl", 1, (short) 1);
    }
}
```

如果Kafka Broker支持（1.0.0或更高版本），则如果发现现有Topic的Partition 数少于设置的Partition 数，则会新增新的Partition分区。

**关于KafkaAdmin有几个常用的用法如下：** 

**setFatalIfBrokerNotAvailable(true)：** 默认这个值是False的，在Broker不可用时，不影响Spring 上下文的初始化。如果你觉得Broker不可用影响正常业务需要显示的将这个值设置为True

**setAutoCreate(false) :** 默认值为True，也就是Kafka实例化后会自动创建已经实例化的NewTopic对象

**initialize()：** 当setAutoCreate为false时，需要我们程序显示的调用admin的initialize()方法来初始化NewTopic对象

### 代码逻辑中创建

有时候我们在程序启动时并不知道某个Topic需要多少Partition数合适，但是又不能一股脑的直接使用Broker的默认设置，这个时候就需要使用Kafka-Client自带的AdminClient来进行处理。

上面的Spring封装的KafkaAdmin也是使用的AdminClient来处理的。如：

```
 @Autowired
    private KafkaProperties properties;
    @Test
    public void testCreateToipc(){
        AdminClient client = AdminClient.create(properties.buildAdminProperties());
        if(client !=null){
            try {
                Collection<NewTopic> newTopics = new ArrayList<>(1);
                newTopics.add(new NewTopic("topic-kl",1,(short) 1));
                client.createTopics(newTopics);
            }catch (Throwable e){
                e.printStackTrace();
            }finally {
                client.close();
            }
        }
    }
```

### ps:其他的方式创建Topic

上面的这些创建Topic方式前提是你的spring boot版本到2.x以上了，因为spring-kafka2.x版本只支持spring boot2.x的版本。在1.x的版本中还没有这些api。下面补充一种在程序中通过Kafka\_2.10创建Topic的方式

### 引入依赖

```
 <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.10</artifactId>
            <version>0.8.2.2</version>
        </dependency>
```

### api方式创建

```
 @Test
    public void testCreateTopic()throws Exception{
        ZkClient zkClient =new ZkClient("127.0.0.1:2181", 3000, 3000, ZKStringSerializer$.MODULE$)
        String topicName = "topic-kl";
        int partitions = 1;
        int replication = 1;
        AdminUtils.createTopic(zkClient,topicName,partitions,replication,new Properties());
    }
```

注意下ZkClient最后一个构造入参，是一个序列化反序列化的接口实现，博主测试如果不填的话，创建的Topic在ZK上的数据是有问题的，默认的Kafka实现也很简单，就是做了字符串UTF-8编码处理。

`ZKStringSerializer$`是Kafka中已经实现好的一个接口实例，是一个Scala的伴生对象，在Java中直接调用点MODULE$就可以得到一个实例

### 命令方式创建

```
 @Test
    public void testCreateTopic(){
        String [] options= new String[]{
                "--create",
                "--zookeeper","127.0.0.1:2181",
                "--replication-factor", "3",
                "--partitions", "3",
                "--topic", "topic-kl"
        };
        TopicCommand.main(options);
    }
```

* * *

消息发送之KafkaTemplate探秘
--------------------

### 获取发送结果

**异步获取**

```
 template.send("","").addCallback(new ListenableFutureCallback<SendResult<Object, Object>>() {
            @Override
            public void onFailure(Throwable throwable) {
                ......
            }

            @Override
            public void onSuccess(SendResult<Object, Object> objectObjectSendResult) {
                ....
            }
        });
```

**同步获取**

```
 ListenableFuture<SendResult<Object,Object>> future = template.send("topic-kl","kl");
        try {
            SendResult<Object,Object> result = future.get();
        }catch (Throwable e){
            e.printStackTrace();
        }
```

### kafka事务消息

默认情况下，Spring-kafka自动生成的KafkaTemplate实例，是不具有事务消息发送能力的。需要使用如下配置激活事务特性。事务激活后，所有的消息发送只能在发生事务的方法内执行了，不然就会抛一个没有事务交易的异常

```
spring.kafka.producer.transaction-id-prefix=kafka_tx.
```

当发送消息有事务要求时，比如，当所有消息发送成功才算成功，如下面的例子：假设第一条消费发送后，在发第二条消息前出现了异常，那么第一条已经发送的消息也会回滚。

而且正常情况下，假设在消息一发送后休眠一段时间，在发送第二条消息，消费端也只有在事务方法执行完成后才会接收到消息

```
 @GetMapping("/send/{input}")
    public void sendFoo(@PathVariable String input) {
        template.executeInTransaction(t ->{
            t.send("topic_input","kl");
            if("error".equals(input)){
                throw new RuntimeException("failed");
            }
            t.send("topic_input","ckl");
            return true;
        });
    }
```

当事务特性激活时，同样，在方法上面加`@Transactional`注解也会生效

```
 @GetMapping("/send/{input}")
    @Transactional(rollbackFor = RuntimeException.class)
    public void sendFoo(@PathVariable String input) {
        template.send("topic_input", "kl");
        if ("error".equals(input)) {
            throw new RuntimeException("failed");
        }
        template.send("topic_input", "ckl");
    }
```

Spring-Kafka的事务消息是基于Kafka提供的事务消息功能的。而Kafka Broker默认的配置针对的三个或以上Broker高可用服务而设置的。这边在测试的时候为了简单方便，使用了嵌入式服务新建了一个单Broker的Kafka服务，出现了一些问题：如

**1、事务日志副本集大于Broker数量，会抛如下异常：** 

> Number of alive brokers '1' does not meet the required replication factor '3' for the transactions state topic (configured via 'transaction.state.log.replication.factor'). This error can be ignored if the cluster is starting up and not all brokers are up yet.

默认Broker的配置`transaction.state.log.replication.factor=3`，单节点只能调整为1

**2、副本数小于副本同步队列数目，会抛如下异常**

> Number of insync replicas for partition \_\_transaction\_state-13 is \[1\], below required minimum \[2\]

默认Broker的配置transaction.state.log.min.isr=2，单节点只能调整为1

### ReplyingKafkaTemplate获得消息回复

ReplyingKafkaTemplate是KafkaTemplate的一个子类，除了继承父类的方法，新增了一个方法sendAndReceive，实现了消息发送\\回复语义

```
RequestReplyFuture<K, V, R> sendAndReceive(ProducerRecord<K, V> record);
```

也就是我发送一条消息，能够拿到消费者给我返回的结果。就像传统的RPC交互那样。当消息的发送者需要知道消息消费者的具体的消费情况，非常适合这个api。

如，一条消息中发送一批数据，需要知道消费者成功处理了哪些数据。下面代码演示了怎么集成以及使用ReplyingKafkaTemplate

```
 @SpringBootApplication
@RestController
public class Application {
    private final Logger logger = LoggerFactory.getLogger(Application.class);
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    @Bean
    public ConcurrentMessageListenerContainer<String, String> repliesContainer(ConcurrentKafkaListenerContainerFactory<String, String> containerFactory) {
        ConcurrentMessageListenerContainer<String, String> repliesContainer = containerFactory.createContainer("replies");
        repliesContainer.getContainerProperties().setGroupId("repliesGroup");
        repliesContainer.setAutoStartup(false);
        return repliesContainer;
    }

    @Bean
    public ReplyingKafkaTemplate<String, String, String> replyingTemplate(ProducerFactory<String, String> pf, ConcurrentMessageListenerContainer<String, String> repliesContainer) {
        return new ReplyingKafkaTemplate(pf, repliesContainer);
    }

    @Bean
    public KafkaTemplate kafkaTemplate(ProducerFactory<String, String> pf) {
        return new KafkaTemplate(pf);
    }

    @Autowired
    private ReplyingKafkaTemplate template;

    @GetMapping("/send/{input}")
    @Transactional(rollbackFor = RuntimeException.class)
    public void sendFoo(@PathVariable String input) throws Exception {
        ProducerRecord<String, String> record = new ProducerRecord<>("topic-kl", input);
        RequestReplyFuture<String, String, String> replyFuture = template.sendAndReceive(record);
        ConsumerRecord<String, String> consumerRecord = replyFuture.get();
        System.err.println("Return value: " + consumerRecord.value());
    }

    @KafkaListener(id = "webGroup", topics = "topic-kl")
    @SendTo
    public String listen(String input) {
        logger.info("input value: {}", input);
        return "successful";
    }
}
```

* * *

Spring-kafka消息消费用法探秘
--------------------

### @KafkaListener的使用

前面在简单集成中已经演示过了`@KafkaListener`接收消息的能力，但是`@KafkaListener`的功能不止如此，其他的比较常见的，使用场景比较多的功能点如下：

*   显示的指定消费哪些Topic和分区的消息，
*   设置每个Topic以及分区初始化的偏移量，
*   设置消费线程并发度
*   设置消息异常处理器

```
 @KafkaListener(id = "webGroup", topicPartitions = {
            @TopicPartition(topic = "topic1", partitions = {"0", "1"}),
                    @TopicPartition(topic = "topic2", partitions = "0",
                            partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100"))
            },concurrency = "6",errorHandler = "myErrorHandler")
    public String listen(String input) {
        logger.info("input value: {}", input);
        return "successful";
    }
```

其他的注解参数都很好理解，errorHandler需要说明下，设置这个参数需要实现一个接口`KafkaListenerErrorHandler`。而且注解里的配置，是你自定义实现实例在spring上下文中的Name。比如，上面配置为`errorHandler = "myErrorHandler"`。则在spring上线中应该存在这样一个实例：

```
 @Service("myErrorHandler")
public class MyKafkaListenerErrorHandler implements KafkaListenerErrorHandler {
    Logger logger =LoggerFactory.getLogger(getClass());
    @Override
    public Object handleError(Message<?> message, ListenerExecutionFailedException exception) {
        logger.info(message.getPayload().toString());
        return null;
    }
    @Override
    public Object handleError(Message<?> message, ListenerExecutionFailedException exception, Consumer<?, ?> consumer) {
        logger.info(message.getPayload().toString());
        return null;
    }
}
```

### 手动Ack模式

手动ACK模式，由业务逻辑控制提交偏移量。比如程序在消费时，有这种语义，特别异常情况下不确认ack，也就是不提交偏移量，那么你只能使用手动Ack模式来做了。开启手动首先需要关闭自动提交，然后设置下consumer的消费模式

```
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.listener.ack-mode=manual
```

上面的设置好后，在消费时，只需要在`@KafkaListener`监听方法的入参加入Acknowledgment 即可，执行到`ack.acknowledge()`代表提交了偏移量

```
 @KafkaListener(id = "webGroup", topics = "topic-kl")
    public String listen(String input, Acknowledgment ack) {
        logger.info("input value: {}", input);
        if ("kl".equals(input)) {
            ack.acknowledge();
        }
        return "successful";
    }
```

### @KafkaListener注解监听器生命周期

`@KafkaListener`注解的监听器的生命周期是可以控制的，默认情况下，`@KafkaListener`的参数`autoStartup = "true"`。也就是自动启动消费，但是也可以同过`KafkaListenerEndpointRegistry`来干预他的生命周期。

`KafkaListenerEndpointRegistry`有三个动作方法分别如：`start()`,`pause()`,`resume()`/启动，停止，继续。如下代码详细演示了这种功能。

```
 @SpringBootApplication
@RestController
public class Application {
    private final Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Autowired
    private KafkaTemplate template;

    @GetMapping("/send/{input}")
    @Transactional(rollbackFor = RuntimeException.class)
    public void sendFoo(@PathVariable String input) throws Exception {
        ProducerRecord<String, String> record = new ProducerRecord<>("topic-kl", input);
        template.send(record);
    }

    @Autowired
    private KafkaListenerEndpointRegistry registry;

    @GetMapping("/stop/{listenerID}")
    public void stop(@PathVariable String listenerID){
        registry.getListenerContainer(listenerID).pause();
    }
    @GetMapping("/resume/{listenerID}")
    public void resume(@PathVariable String listenerID){
        registry.getListenerContainer(listenerID).resume();
    }
    @GetMapping("/start/{listenerID}")
    public void start(@PathVariable String listenerID){
        registry.getListenerContainer(listenerID).start();
    }
    @KafkaListener(id = "webGroup", topics = "topic-kl",autoStartup = "false")
    public String listen(String input) {
        logger.info("input value: {}", input);
        return "successful";
    }
}
```

在上面的代码中，listenerID就是@KafkaListener中的id值“webGroup”。项目启动好后，分别执行如下url，就可以看到效果了。

先发送一条消息：http://localhost:8081/send/ckl。因为autoStartup = "false"，所以并不会看到有消息进入监听器。

接着启动监听器：http://localhost:8081/start/webGroup。可以看到有一条消息进来了。

暂停和继续消费的效果使用类似方法就可以测试出来了。

### SendTo消息转发

前面的消息发送响应应用里面已经见过@SendTo,其实除了做发送响应语义外，@SendTo注解还可以带一个参数，指定转发的Topic队列。

常见的场景如，一个消息需要做多重加工，不同的加工耗费的cup等资源不一致，那么就可以通过跨不同Topic和部署在不同主机上的consumer来解决了。如：

```
 @KafkaListener(id = "webGroup", topics = "topic-kl")
    @SendTo("topic-ckl")
    public String listen(String input) {
        logger.info("input value: {}", input);
        return input + "hello!";
    }

    @KafkaListener(id = "webGroup1", topics = "topic-ckl")
    public void listen2(String input) {
        logger.info("input value: {}", input);
    }
```

### 消息重试和死信队列的应用

除了上面谈到的通过手动Ack模式来控制消息偏移量外，其实Spring-kafka内部还封装了可重试消费消息的语义，也就是可以设置为当消费数据出现异常时，重试这个消息。而且可以设置重试达到多少次后，让消息进入预定好的Topic。也就是死信队列里。

下面代码演示了这种效果：

```
 @Autowired
    private KafkaTemplate template;

    @Bean
    public ConcurrentKafkaListenerContainerFactory<?, ?> kafkaListenerContainerFactory(
            ConcurrentKafkaListenerContainerFactoryConfigurer configurer,
            ConsumerFactory<Object, Object> kafkaConsumerFactory,
            KafkaTemplate<Object, Object> template) {
        ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
        configurer.configure(factory, kafkaConsumerFactory);
        
        factory.setErrorHandler(new SeekToCurrentErrorHandler(new DeadLetterPublishingRecoverer(template), 3));
        return factory;
    }

    @GetMapping("/send/{input}")
    public void sendFoo(@PathVariable String input) {
        template.send("topic-kl", input);
    }

    @KafkaListener(id = "webGroup", topics = "topic-kl")
    public String listen(String input) {
        logger.info("input value: {}", input);
        throw new RuntimeException("dlt");
    }

    @KafkaListener(id = "dltGroup", topics = "topic-kl.DLT")
    public void dltListen(String input) {
        logger.info("Received from DLT: " + input);
    }
```

上面应用，在topic-kl监听到消息会，会触发运行时异常，然后监听器会尝试三次调用，当到达最大的重试次数后。消息就会被丢掉重试死信队列里面去。死信队列的Topic的规则是，业务Topic名字+“.DLT”。

如上面业务Topic的name为“topic-kl”，那么对应的死信队列的Topic就是“topic-kl.DLT”

* * *

文末结语
----

最近业务上使用了kafka用到了Spring-kafka，所以系统性的探索了下Spring-kafka的各种用法，发现了很多好玩很酷的特性，比如，一个注解开启嵌入式的Kafka服务、像RPC调用一样的发送\\响应语义调用、事务消息等功能。

希望此博文能够帮助那些正在使用Spring-kafka或即将使用的人少走一些弯路少踩一点坑。

扫描上方二维码获取更多Java干货