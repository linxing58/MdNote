# 初识RabbitMQ（四）——如何在Java代码使用RabbitMQ - 掘金
这是我参与「掘金日新计划 · 6 月更文挑战」的第 5 天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https&#x3A;//juejin.cn/post/7099702781094674468")

### 前言

    大家好，我是程序猿小白 GW_gw，很高兴能和大家一起学习进步。
    复制代码

以下内容部分来自于网络，如有侵权，请联系我删除，本文仅用于学习交流，不用作任何商业用途。

### 摘要

    本文主要带大家了解RabbitMQ如何使用Java代码进行简单的消息发送接收和使用三种交换机进行消息的发送与接收。
    复制代码

### 1. Java 发送和接收 queue

#### 1.1 创建工程，添加依赖

    <!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.9.0</version>
    </dependency>
    复制代码

#### 1.2 编写发送代码

步骤如下：

1.  创建连接工厂
2.  配置连接工厂信息
3.  获取连接
4.  获取通道
5.  声明队列
6.  发送消息
7.  关闭资源

具体代码如下：

    public class Send {
        public static void main(String[] args) {
            //创建连接工厂
            ConnectionFactory factory = new ConnectionFactory();
            //配置连接信息
            factory.setHost("192.168.44.250");
            factory.setPort(5672);
            factory.setUsername("admin");
            factory.setPassword("123456");
    ​
            Connection connection = null;
            Channel channel = null;
            try {
                //获取连接
                connection = factory.newConnection();
                //获取通道
                channel = connection.createChannel();
    ​
               channel.queueDeclare("myQueue",true,false,false,null);
                String message = "发送到MQ的测试消息2";
               
                channel.basicPublish("","myQueue",null,message.getBytes(StandardCharsets.UTF_8));
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }finally {
                //关闭资源
                if(channel!=null){
                    try {
                        channel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (TimeoutException e) {
                        e.printStackTrace();
                    }
                }
                if(connection!=null){
                    try {
                        connection.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
    ​
        }
    }
    复制代码

参数解释：

> /\* 声明一个队列：
>
> 1.  参数 1：队列名
> 2.  参数 2：是否是持久化
> 3.  参数 3：是否排外，如果排外则这个队列只允许一个消费者监听
> 4.  参数 4：是否自动删除，如果没有消费者连接时，就会自动删除该队列
> 5.  参数 5：为队列的一些属性设置, 一般为 null
>
> 注意：
>
> 1.  如果声明的队列已经存在也不会报错，如果不存在则会创建一个新的队列
> 2.  在发送消息前一定要有该队列 (可以不在这里声明，直接在管控台进行声明)。 \*/ channel.queueDeclare("myQueue",true,false,false,null);
>
> 发送消息到 mq
>
> 1.  参数 1：为交换机名称，空字符串表示不使用交换机
> 2.  参数 2：为队列名称或 RoutingKey，如果指定了交换机名称就是 RoutingKey
> 3.  参数 3：具体消息属性，一般为空。
> 4.  参数 4：具体消息字节数组 channel.basicPublish("","myQueue",null,message.getBytes(StandardCharsets.UTF_8));

#### 1.3 编写接收代码

步骤如下：

1.  创建连接工厂
2.  配置连接工厂信息
3.  获取连接
4.  获取通道
5.  声明队列
6.  接收消息

具体代码如下：

    public class Receive {
        public static void main(String[] args) {
            //创建连接工厂
            ConnectionFactory factory = new ConnectionFactory();
            //配置连接信息
            factory.setHost("192.168.44.250");
            factory.setPort(5672);
            factory.setUsername("admin");
            factory.setPassword("123456");
    ​
            //创建连接
            Connection connection = null;
            Channel channel = null;
            try {
                connection = factory.newConnection();
                //获取管道
                channel = connection.createChannel();
                //声明队列
                channel.queueDeclare("myQueue",true,false,false,null);
                //接收消息
                channel.basicConsume("myQueue",true,"",new DefaultConsumer(channel){
                    @Override
                    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                        String message = new String(body, StandardCharsets.UTF_8);
                        System.out.println(message);
                    }
                });
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
    ​
        }
    }
    复制代码

参数解释：

> 1.  参数 1：当前消费者监听的队列名
> 2.  参数 2：是否自动确认，自动确认表示接受完消息后会将消息从队列中自动删除。
> 3.  参数 3：tag，消费者的标签，通常为空字符串
> 4.  参数 4：消息接收后的回调方法
>
> 注意：使用 basicConsume 方法后，会开启一个线程持续监听队列
>
> channel.basicConsume("myQueue",true,"",new DefaultConsumer(channel){ @Override public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte\[] body) throws IOException { String message = new String(body, StandardCharsets.UTF_8); System.out.println(message); } });

### 2. 使用 Direct 类型的交换机发送和接收消息

#### 2.1 发送消息

步骤如下：

1.  创建连接工厂
2.  配置连接工厂信息
3.  获取连接
4.  获取通道
5.  声明队列
6.  创建交换机
7.  将队列绑定到交换机
8.  发送消息
9.  关闭资源

具体代码如下：

    public class Send {
        public static void main(String[] args) {
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("192.168.44.250");
            factory.setPort(5672);
            factory.setUsername("admin");
            factory.setPassword("123456");
            Connection connection = null;
            Channel channel = null;
            try {
                connection = factory.newConnection();
                channel = connection.createChannel();
                channel.queueDeclare("MyDirectQueue", true, false, false, null);
    ​
                //创建交换机
                channel.exchangeDeclare("directExchange","direct",true);
               
                channel.queueBind("MyDirectQueue","directExchange","directRoutingKey");
                String message = "direct交换机测试消息";
                //发送消息
                channel.basicPublish("directExchange","directRoutingKey",null,message.getBytes(StandardCharsets.UTF_8));
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }finally {
                //关闭资源
                if(channel!=null){
                    try {
                        channel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (TimeoutException e) {
                        e.printStackTrace();
                    }
                }
                if(connection!=null){
                    try {
                        connection.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    复制代码

参数解释：

> 1.  参数 1： 交换机名称
> 2.  参数 2： 交换机类型，direct、fanout、topic、headers
> 3.  参数 3： 是否持久化
>
> channel.exchangeDeclare("directExchange","direct",true);
>
> 将队列绑定到交换机
>
> 1.  参数 1：队列名称
> 2.  参数 2：交换机名称
> 3.  参数 3：消息的 RoutingKey（就是 BandingKey）
>
> channel.queueBind("MyDirectQueue","directExchange","directRoutingKey");

#### 2.2 接收消息

步骤如下：

1.  创建连接工厂
2.  配置连接工厂信息
3.  获取连接
4.  获取通道
5.  声明队列
6.  创建交换机
7.  将队列绑定到交换机
8.  接收消息

具体代码如下：

    public class Receive {
        public static void main(String[] args) {
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("192.168.44.250");
            factory.setPort(5672);
            factory.setUsername("admin");
            factory.setPassword("123456");
            Connection connection = null;
            Channel channel = null;
            try {
                 connection = factory.newConnection();
                 channel = connection.createChannel();
                 //声明队列
                 channel.queueDeclare("MyDirectQueue",true,false,false,null);
                 //声明交换机
                 channel.exchangeDeclare("directExchange","direct",true);
                 //绑定队列和交换机
                 channel.queueBind("MyDirectQueue","directExchange","directRoutingKey");
    ​
                 //接收消息
                channel.basicConsume("MyDirectQueue",true,"",new DefaultConsumer(channel){
                    @Override
                    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                        String message = new String(body, StandardCharsets.UTF_8);
                        System.out.println(message);
                    }
                });
            } catch (IOException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
        }
    }
    复制代码

### 3. 使用 Fanout 类型交换机发送和接收消息

使用 Fanout 类型交换机我们只需要简单修改代码即可，Fanout 类型的交换机消费者需要先监听队列才能获得消息，因此我们先来写消费者。

#### 3.1 接收消息

    String queueName = channel.queueDeclare().getQueue();
    //声明交换机
    channel.exchangeDeclare("fanoutExchange","fanout",true);
    //绑定队列和交换机
    channel.queueBind(queueName,"fanoutExchange","");
    ​
    //接收消息
    channel.basicConsume(queueName,true,"",new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            String message = new String(body, StandardCharsets.UTF_8);
            System.out.println(message);
        }
    });
    复制代码

参数解释：

> /\* 使用无参方式声明队列，默认是随机队列名称， 数据非持久化， 排外，(只允许一个消费者监听当前队列) 自动删除 (没有消费者监听队列时，自动删除) getQueue() 方法获取随机队列名称 \*/
>
> String queueName = channel.queueDeclare().getQueue();

#### 3.2 发送消息

    channel.exchangeDeclare("fanoutExchange","fanout",true);
    String message = "fanout交换机测试消息";
    //发送消息    
    channel.basicPublish("fanoutExchange","",null,message.getBytes(StandardCharsets.UTF_8));
    复制代码

指定交换机类型，不需要声明队列和绑定交换机，这些操作在消费者中做即可。

### 4. 使用 topic 类型交换机发送和接收消息

#### 4.1 接收消息

指定队列和交换机，绑定队列和交换机。

    //声明队列
    channel.queueDeclare("topicQueue01",true,false,false,null);
    //声明交换机
    channel.exchangeDeclare("topicExchange","topic",true);
    //绑定队列和交换机
    channel.queueBind("topicQueue01","topicExchange","topicRoutingKey.aa");
    ​
    //接收消息
    channel.basicConsume("topicQueue01",true,"",new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            String message = new String(body, StandardCharsets.UTF_8);
            System.out.println(message);
        }
    });
    复制代码

#### 4.2 发送消息

声明交换机，发送消息。

    //声明交换机
    channel.exchangeDeclare("topicExchange","topic",true);
    String message = "topic交换机测试消息";
    //发送消息
    channel.basicPublish("topicExchange","topicRoutingKey.aa",null,message.getBytes(StandardCharsets.UTF_8));
    复制代码

> Topic 和 Fanout 交换机的区别
>
> Topic 类型的交换机适用于不同模块接收同一消息。例如下单成功后的消息发送给其他模块。
>
> Fanout 类型的交换机适用于同一功能的不同进程接收消息。例如 App 的消息推送，多个用户的 app 进程都可以接收到消息。

### 小结

以上就是关于如何使用 Java 代码进行消息的发送和接受，以及如何使用三种交换机，希望以上内容对读者有所帮助。 
 [https://juejin.cn/post/7104440437309440013](https://juejin.cn/post/7104440437309440013)
