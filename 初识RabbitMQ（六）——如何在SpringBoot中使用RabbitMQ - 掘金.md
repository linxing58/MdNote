# 初识RabbitMQ（六）——如何在SpringBoot中使用RabbitMQ - 掘金
这是我参与「掘金日新计划 · 6 月更文挑战」的第 7 天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https&#x3A;//juejin.cn/post/7099702781094674468")

### 前言

    大家好，我是程序猿小白 GW_gw，很高兴能和大家一起学习进步。
    复制代码

以下内容部分来自于网络，如有侵权，请联系我删除，本文仅用于学习交流，不用作任何商业用途。

### 摘要

    本文主要带大家了解如何在Springboot中使用RabbitMQ，主要是三种交换机的使用。
    复制代码

### 1. 在 SpringBoot 中集成 RabbitMQ

#### 1.1 添加依赖，配置连接

下面大家主要看第一个依赖，后两个测试用。

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    ​
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
    复制代码

配置 RabbitMQ 的连接信息，这里使用 yml 格式配置，配置 ip，端口和用户名以及密码。

    spring:
      rabbitmq:
        host: 192.168.44.250
        port: 5672
        username: admin
        password: 123456
    复制代码

#### 1.2 使用 direct 类型交换机发送和接收消息

###### 配置类中声明队列和交换机并绑定

    @Configuration
    public class RabbitMQConfig {
    ​
        /**
         * 配置direct类型的交换机
         * @return
         */
        @Bean
        public DirectExchange directExchange(){
    ​
            return new DirectExchange("bootDirectExchange");
        }
    ​
        /**
         * 配置队列
         * @return
         */
        @Bean
        public Queue directQueue(){
            return new Queue("bootDirectQueue");
        }
    ​
        /**
         * 绑定队列和交换机
         * @param directQueue 参数名和某个方法名相同，springboot会进行自动注入
         * @param directExchange 参数名和某个方法名相同，springboot会进行自动注入
         * @return
         */
        @Bean
        public Binding directBinding(Queue directQueue,DirectExchange directExchange){
            return BindingBuilder.bind(directQueue).to(directExchange).with("bootDirectRoutingKey");
        }
    }
    复制代码

> 在配置类中声明队列只能自己指定名称，不适用于 fanout 和 topic 类型的交换机。

###### 发送消息

定义 Send 接口和实现类，定义方法如下：

接口：

    public interface SendService {
        void sendMessage(String message);
    }
    复制代码

实现类：

    @Service
    public class SendServiceImpl implements SendService {
        @Resource
        private AmqpTemplate amqpTemplate;
    ​
        @Override
        public void sendMessage(String message) {
            /*
              发送消息
             */
            amqpTemplate.convertAndSend("bootDirectExchange","bootDirectRoutingKey",message);
        }
    }
    复制代码

> 参数解释：
>
> 参数 1：交换机名
>
> 参数 2：RoutingKey
>
> 参数 3：消息
>
> 这里使用 convertAndSend 方法，而不直接使用 send 方法，该方法会自动将消息进行转换并发送，使用简单。

测试类：

    @SpringBootTest
    public class SendServiceImplTest {
        @Autowired
        private SendService sendService;
        @Test
        void testSendMessage(){
            sendService.sendMessage("springboot测试消息");
        }
    }
    复制代码

###### 接收消息

定义接收接口，定义实现类，定义方法，如下：

接口：

    public interface Receiveervice {
        void receiveMessage();
        void directReceive(String message);
    }
    复制代码

实现类：

    @Service
    public class ReceiveerviceImpl implements Receiveervice {
        @Resource
        private AmqpTemplate amqpTemplate;
    ​
        /**
         * 每运行一次只能接收一条消息
         */
        @Override
        public void receiveMessage() {
            String message = (String) amqpTemplate.receiveAndConvert("bootDirectQueue");
            System.out.println(message);
        }
        /**
         * 使用监听器
         * @param message
         */
        @RabbitListener(queues = {"bootDirectQueue"})
        @Override
        public void directReceive(String message){
            System.out.println(message);
        }
    }
    复制代码

> @RabbitListener 监听器不间断接收消息，不需要手动调用，Spring 会自动调用 如果当前方法正常结束，Spring 会自动确认消息，出现异常则不会确认消息，需要进行防重复处理
>
> 同样，接受时我们使用 receiveAndConvert 方法该方法会自动将消息进行接收并转换。

测试类：

    @Autowired
    private Receiveervice receiveervice;
    @Test
    void testReceiveMessage(){
        receiveervice.receiveMessage();
    }
    复制代码

> 不使用监听器需要显示调用接收消息。

#### 1.3 使用 fanout 类型交换机发送和接收消息

###### 配置类中声明交换机

    /**
     * 配置fanout类型的交换机
     * @return
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanoutExchange");
    }
    复制代码

###### 接收消息

定义接收接口，定义实现类，定义方法，如下：

接口中方法：

    void fanoutReceive(String message);
    void fanoutReceive02(String message);
    复制代码

实现类方法：

    /**
     * @QueueBinding 绑定队列和交换机
     * @Queue() 创建随机名称队列
     * @Exchange() 创建一个交换机
     * @param message
     */
    @RabbitListener(bindings = {
            @QueueBinding(value = @Queue(),
                    exchange = @Exchange(name = "fanoutExchange",type = "fanout"))})
    @Override
    public void fanoutReceive(String message){
        System.out.println(message);
    }
    ​
    @RabbitListener(bindings = {
            @QueueBinding(value = @Queue(),
                    exchange = @Exchange(name = "fanoutExchange",type = "fanout"))})
    @Override
    public void fanoutReceive02(String message){
        System.out.println(message);
    }
    复制代码

> 使用全注解方式绑定交换机和队列，@RabbitListener 注解开启自动监听，不需要显示调用。

###### 发送消息

定义接收接口，定义实现类，定义方法，如下：

接口中方法：

    void fanoutSendMessage(String message);
    复制代码

实现类中方法：

    @Override
    public void fanoutSendMessage(String message){
        amqpTemplate.convertAndSend("fanoutExchange","",message);
    }
    复制代码

> 不需要指定路由键。

测试类：

    @Test
    void testFanoutSendMessage(){
        sendService.fanoutSendMessage("fanout测试消息");
    }
    复制代码

#### 1.4 使用 topic 类型交换机发送和接收消息

###### 配置类中声明交换机

    /**
    * 配置topic类型的交换机
    * @return
    */
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange("topicExchange");
    }
    复制代码

###### 接收消息

定义接收接口，定义实现类，定义方法，如下：

接口中方法：

    void topicReceive(String message);
    void topicReceive02(String message);
    void topicReceive03(String message);
    复制代码

实现类方法：

    @Override
    @RabbitListener(bindings = {
        @QueueBinding(value = @Queue("topic01"),key = {"aa"},
                      exchange = @Exchange(name = "topicExchange",type = "topic"))})
    public void topicReceive(String message){
        System.out.println("topic01---aa"+message);
    }
    @Override
    @RabbitListener(bindings = {
        @QueueBinding(value = @Queue("topic02"),key = {"aa.*"},
                      exchange = @Exchange(name = "topicExchange",type = "topic"))})
    public void topicReceive02(String message){
        System.out.println("topic02---aa.*"+message);
    }
    @Override
    @RabbitListener(bindings = {
        @QueueBinding(value = @Queue("topic03"),key = {"aa.#"},
                      exchange = @Exchange(name = "topicExchange",type = "topic"))})
    public void topicReceive03(String message){
        System.out.println("topic03---aa.#"+message);
    }
    复制代码

> 使用全注解方式绑定交换机和队列，开启自动监听，并指定路由键。

###### 发送消息

定义接收接口，定义实现类，定义方法，如下：

接口中方法：

    void topicSendMessage(String message);
    复制代码

实现类中方法：

    @Override
    public void topicSendMessage(String message){
        amqpTemplate.convertAndSend("topicExchange","aa.bb",message);
    }
    复制代码

> 需要指定路由键。

测试类：

    @Test
    void testTopicSendMessage(){
        sendService.topicSendMessage("topic测试消息");
    }
    复制代码

### 小结

以上就是关于如何让在 SpringBoot 中使用 RabbitMQ 的相关介绍，希望能对读者有所帮助，如有不正之处，欢迎留言指正。 
 [https://juejin.cn/post/7105311466202333198](https://juejin.cn/post/7105311466202333198)
