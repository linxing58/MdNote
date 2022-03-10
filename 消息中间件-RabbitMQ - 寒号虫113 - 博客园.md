# 消息中间件-RabbitMQ - 寒号虫113 - 博客园
### 一、基础知识

###### 1. 什么是 RabbitMQ

RabbitMQ 是 2007 年发布，是一个在 AMQP(高级消息队列协议) 基础上完成的，简称 MQ 全称为 Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法，由 Erlang（专门针对于大数据高并发的语言）语言开发，可复用的企业消息系统，是当前最主流的消息中间件之一，具有可靠性、灵活的路由、消息集群简单、队列高可用、多种协议的支持、管理界面、跟踪机制以及插件机制。

###### 2. 什么是消息和队列

1\. 消息 就是数据，增删改查的数据。例如在员工管理系统中增删改查的数据

2\. 队列 指的是一端进数据一端出数据，例如 C# 中 (Queue 数据结构）

###### 3. 什么是消息队列

1\. 消息队列指：一端进消息，一端出消息

2.RabbitMQ 就是实现了消息队列概念的一个组件，以面向对象的思想去理解，消息队列就是类，而 RabbitMQ 就是实例，当然不仅仅只有 RabbitMQ, 例如 ActiveMQ，RocketMQ，Kafka，包括 Redis 也可以实现消息队列。

###### 4. 什么地方使用 RabbitMQ

1\. 在常见的单体架构中，主要流程是用户 UI 操作发起 Http 请求 > 服务器处理 > 然后由服务器直接和数据库交互，最后同步反馈用户结果

![](https://img2022.cnblogs.com/blog/1264751/202202/1264751-20220225234143356-459832411.png)

2\. 在微服务架构中，UI 与微服务通信，主要是通过 Http 或者 gRPC 同步通信

![](https://img2022.cnblogs.com/blog/1264751/202202/1264751-20220225234309618-2074066291.png)

`问题分析`

在上述 2 种情况下，我们发现在 UI 请求时都是同步操作 ，第 2 种架构虽然将整体服务按业务拆分成不同的微服务并且对应各自的数据库，但是在用户与微服务通信时，存在的问题依然没有解决，例如数据库的承载能力只能处理 10w 个请求，如果遇到高并发情况下，UI 发起 50w 请求，那数据库是远远承载不了的, 从而导致如下问题。

1\. 高并发请求导致系统性能下降响应慢，同时数据库承载风险加大

2\. 扩展性不强 UI 操作的交互对业务的依赖较大，导致用户体验下降

3\. 瞬时流量涌入巨大的话，服务器可能直接挂了

`解决方案`

![](https://img2022.cnblogs.com/blog/1264751/202202/1264751-20220225235023108-1097177772.png)

-   为了解决性能瓶颈问题。我们需要将同步通信换成异步通信方式。因此就使用消息队列，用户在 UI 中操作直接写入 RabbitMQ 然后直接返回，剩下的业务操作由消息队列和各自的微服务来完成

`RabbitMQ 的优势`

1.  异步处理，响应快，增加了数据库（服务器的承载能力）
2.  削峰，可以把流量的高峰分解到不同的时间段来处理
3.  解耦（扩展性就更强），让 UI 和业务独立演化
4.  高可用，处理器如果发生故障了，对其他的处理器没有影响

`RabbitMQ 的不足`

1.  增加了系统复杂性，不方便调试和开发，在使用 RabbitMQ 以前前端直接和服务交互，现在加了一层
2.  即时性降低了, 在某一程度上提升了用户操作体验，也降低了用户体验，但是避免不了，取长补短
3.  更加依赖消息队列了

###### 5.RabbitMQ 组成概念

1.ConnectionFactory 为 Connection 的制造工厂。

2.Connection 是 RabbitMQ 的 socket 链接，它封装了 socket 协议相关部分逻辑。

3.Channel 是我们与 RabbitMQ 打交道的最重要的一个接口，我们大部分的业务操作是在 Channel 这个接口中完成的，包括定义 Queue、定义 Exchange、绑定 Queue 与 Exchange、发布消息等。

4.Exchange（交换机） 我们通常认为生产者将消息投递到 Queue 中，实际上实际的情况是，生产者将消息发送到 Exchange，由 Exchange 将消息路由到一个或多个 Queue 中（或者丢弃），而在 RabbitMQ 中的 Exchange 一共有 4 种策略，分别为：fanout（扇形）、direct（直连）、topic（主题）、headers（头部）

* * *

### 二、如何落地 RabbitMQ

###### 1.RabbitMQ 环境安装

1\.[下载 RabbitMQ](https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.13/rabbitmq-server-3.9.13.exe)

2\.[运行环境 erlang](https://github.com/erlang/otp/releases/download/OTP-24.2.1/otp_win64_24.2.1.exe)

3\. 安装完成之后，加载 RabbitMQ 管理插件

    rabbitmq-plugins enable rabbitmq_management 

4\. 安装成功访问 RabbitMQ 管理后台[http://localhost:15672](http://localhost:15672)

###### 2. 创建系统业务

1\. 分别创建考勤服务，请假服务，计算薪酬服务，邮件服务，短信服务消费者角色

2\. 创建员工管理网站用于模拟前端调用，主要充当生产者角色

3\. 在员工管理网站和每一个模拟微服务中通过 nuget 引入 RabbitMQ.Client

4\. 在员工管理网站中创建模拟添加考勤的控制器并加入生产者代码

     //创建连接
     using (var connection = factory.CreateConnection())
     {
         //创建通道
         var channel = connection.CreateModel();
         //定义队列
         channel.QueueDeclare("CreateAttendance", false, false, false, null);

         string json = JsonConvert.SerializeObject(attendanceDto);

         //创建内容对象
         var properties = channel.CreateBasicProperties();
         //发送消息
         channel.BasicPublish(exchange: "",routingKey: "CreateAttendance",basicProperties: properties,body: Encoding.UTF8.GetBytes(json));
     } 

5\. 在考勤微服务中创建接口，并在接口中加入消费者代码

    var connection = factory.CreateConnection();
    var channel = connection.CreateModel();   
    //创建消费者事件
    var consumer = new EventingBasicConsumer(channel);
    consumer.Received += (model, ea) =>
    {
        var body = ea.Body;
        // 1、逻辑代码，添加到数据库
        var message = Encoding.UTF8.GetString(body.ToArray());
        object json = JsonConvert.DeserializeObject(message);
        Console.WriteLine(" [x] 创建考勤信息 {0}", message);
    };
    //设置消费者属性
    //p1.监听队列p2.消息确认ACK p3.消费者实例赋值
    channel.BasicConsume(queue: "CreateAttendance",autoAck: false,consumer:consumer); 

* * *

### 三、Exchange 交换机及实例分析

###### 1.Fanout Exchange (扇形交换机)

fanout 类型的 Exchange 路由规则非常简单，工作方式类似于多播一对多，它会把所有发送到该 Exchange 的消息路由到所有与它绑定的 Queue 中。

1\. 生产者一个 Exchange 对应多个 Queue，或者不声明 Queue

2\. 消费者定义 Exchange, 如果生产者定义了 Queue，那必须将 exchange 和 queue 绑定，如果没有定义队列，那消费者自己声明一个随机 Queue 用于接收消费消息

`业务实例`

当我们有员工需要请假，在员工管理系统提交请假，但是由于公司规定普通员工请假，需要发送短信到他的主管领导，针对此业务场景我们需要调用请假服务的同时去发送短信，这时需要两个消费者（请假服务，短信服务）来消费同一条消息，其实本质就是往 RabbitMQ 写入一个能被多个消费者接收的消息，所以可以使用 `扇形交换机`，一个生产者，多个消费者.

`生产者模拟使用调用控制器来实现`

    [HttpPost]
    public IEnumerable<bool> CreateLeave(CreateLeaveDto createLeaveDto)
    {
        var factory = new ConnectionFactory()
        {
            HostName = "192.168.0.106",
            Port = 5672,
            Password = "guest",
            UserName = "guest",
            VirtualHost = "/"
        };
        using (var connection = factory.CreateConnection())
        {
            var channel = connection.CreateModel();
            //定义交换机
            channel.ExchangeDeclare(exchange: "Leave_fanout", type: "fanout");
            string productJson = JsonConvert.SerializeObject(createLeaveDto);
            var body = Encoding.UTF8.GetBytes(productJson);
            var properties = channel.CreateBasicProperties();
            //设置消息持久化
            properties.Persistent = true;

            channel.BasicPublish(exchange: "Leave_fanout", routingKey: "",  basicProperties: properties,body: body);
        }

    } 

`消费者实现 IHostedService 接口创建一个监听主机`

    public class RabbitmqHostService : IHostedService
    {
    	  public Task StartAsync(CancellationToken cancellationToken)
          {
                var factory = new ConnectionFactory()
                {
                    HostName = "localhost",
                    Port = 5672,
                    Password = "guest",
                    UserName = "guest",
                    VirtualHost = "/"
                };
               	var connection = factory.CreateConnection();
                var channel = connection.CreateModel();

                // 1、定义交换机
                channel.ExchangeDeclare(exchange: "Leave_fanout", type: ExchangeType.Fanout);
                //定义随机队列
                var queueName = channel.QueueDeclare().QueueName;	   
                //队列和交换机绑定
                channel.QueueBind(queueName,"Leave_fanout",routingKey: "");

               var consumer = new EventingBasicConsumer(channel);
               consumer.Received += (model, ea) =>
               {
                   Console.WriteLine($"model:{model}");
                   var body = ea.Body;
                   // 1、业务逻辑
                   var message = Encoding.UTF8.GetString(body.ToArray());
                   Console.WriteLine(" [x] 创建请假 {0}", message);

                   // 1、自动确认机制缺陷，消息是否正常添加到数据库当中,所以需要使用手工确认
                   channel.BasicAck(ea.DeliveryTag, true);
              };
              
              // Qos(防止多个消费者，能力不一致，导致的系统质量问题。
              // 每一次一个消费者只成功消费一个)
              channel.BasicQos(0, 1, false);
               // 消息确认(防止消息消费失败)
              channel.BasicConsume(queue: queueName ,autoAck: false,consumer: consumer);
          }
          
          public Task StopAsync(CancellationToken cancellationToken)
          {
             // 1、关闭rabbitmq的连接
             throw new NotImplementedException();
          }
    } 

###### 2.Direct Exchange (直连交换机)

直接交换器，工作方式类似于单播一对一，Exchange 会将消息发送完全匹配 ROUTING_KEY 的 Queue，缺陷是无法实现多生产者对一个消费者

1\. 生产者一个 Exchange 对应一个 routingKey 绑定，也可以声明队列并绑定，然后向指定的队列发送消息。

2\. 消费者需要定义 Exchange 和 routingKey，如果生产者声明并绑定了队列，那消费者必须绑定生产者指定的 Queue 来接收消息，如果没有指定 Queue，那消费者需要自己声明一个随机 Queue 然后绑定用于接收消息

当我们员工管理系统需要计算薪资并将结果以发送短信的方式告诉员工，这个时候我们就不太适合用 “扇形交换机” 了，因为换做是你，你也不想你的工资全公司都知道吧？这个时候就需要定制了一对一的场景了，那就在生产消息时使用`直连交换机`根据 routingKey 发送指定的消费者.

`生产者模拟使用调用控制器来实现`

    public IEnumerable<bool> SendCalculateSalary(CalculateSalaryDto calculateSalaryDto)
    {
     var factory = new ConnectionFactory()
     {
         HostName = "192.168.0.106",
         Port = 5672,
         Password = "admin",
         UserName = "admin",
         VirtualHost = "/"
     };

    using (var connection = factory.CreateConnection())
     {
         var channel = connection.CreateModel();
         //2、定义交换机
         channel.ExchangeDeclare(exchange: "CalculateSalary_direct", type: "direct");

         string calculateSalaryDtoJson = JsonConvert.SerializeObject(calculateSalaryDto);
         var body = Encoding.UTF8.GetBytes(calculateSalaryDtoJson);

         //3、发送消息
         var properties = channel.CreateBasicProperties();
         properties.Persistent = true; // 设置消息持久化
         //p1 指定交换机
         //p2 routingKey 
         channel.BasicPublish(exchange: "CalculateSalary_direct",routingKey: "product-sms",basicProperties: properties,body: body);
     }
    } 

`消费者实现 IHostedService 接口创建一个监听主机`

    public class RabbitmqHostService : IHostedService
    {
          public Task StartAsync(CancellationToken cancellationToken)
          {
             var factory = new ConnectionFactory()
             {
                 HostName = "localhost",
                 Port = 5672,
                 Password = "guest",
                 UserName = "guest",
                 VirtualHost = "/"
             };
               
    	var connection = factory.CreateConnection();
    	var channel = connection.CreateModel();

    	// 1、定义交换机
    	channel.ExchangeDeclare(exchange: "CalculateSalary_direct", type: ExchangeType.Direct);

    	// 2、定义随机队列
    	var queueName = channel.QueueDeclare().QueueName;

    	// 3、队列要和交换机绑定起来
    	channel.QueueBind(queueName,"CalculateSalary_direct",routingKey: "product-sms");

    	var consumer = new EventingBasicConsumer(channel);
            consumer.Received += (model, ea) =>
            {
                Console.WriteLine($"model:{model}");
                var body = ea.Body;
                // 1、业务逻辑
                var message = Encoding.UTF8.GetString(body.ToArray());
                Console.WriteLine(" [x] 发送短信 {0}", message);
                // 1、消息是否正常添加到数据库当中,所以需要使用手工确认
                channel.BasicAck(ea.DeliveryTag, true);
            };
                // 3、消费消息
                channel.BasicQos(0, 1, false); // Qos(防止多个消费者，能力不一致，导致的系统质量问题。
                // autoAck设为false 不进行自动确认                     
                channel.BasicConsume(queue: queueName,autoAck: false, consumer: consumer);
          }
          
          public Task StopAsync(CancellationToken cancellationToken)
          {
             // 1、关闭rabbitmq的连接
             throw new NotImplementedException();
          }
    } 

###### 3.Topic Exchange （主题交换机）

Exchange 绑定队列需要制定 Key; Key 可以有自己的规则；Key 可以有占位符；_或者# ，_匹配一个单词、# 匹配多个单词，在 Direct 基础上加上模糊匹配；多生产者一个消费者, 可以多对对，也可以多对 1， 真实项目当中，使用主题交换机。可以满足所有场景

1\. 生产者定义 Exchange，然后不同的 routingKey 绑定

2\. 消费者定义 Exchange，如果生产者定义了 Queue，那必须将 exchange 和 queue 以及 routingKey 绑定，如果没有定义队列，那消费者自己声明一个随机 Queue 用于接收消费消息，

3\. 消费者 routingKey 的模糊匹配，生产者发送消息时 routingKey 定义以 sms. 开头， \* 号只能匹配的 routingKey 为一级，例如 (sms.A) 或(sms.B)的发送的消息，# 能够匹配的 routingKey 为一级及多级以上 ，例如 (sms.A)或者(sms.A.QWE.IOP)

在月底的时候我们需要把员工存在异常考勤信息，薪资结算信息，请假信息分别以邮件的形式发送给我们的员工查阅，我们知道这是一个典型的多个生产者，一个消费者场景，异常考勤信息，薪资结算信息，请假信息分别需要生产消息发送到 RabbitMQ，然后供我们员工消费

`分别模拟 3 个生产者：异常考勤信息，薪资结算信息，请假信息`

    var factory = new ConnectionFactory()
     {
         HostName = "192.168.0.106",
         Port = 5672,
         Password = "admin",
         UserName = "admin",
         VirtualHost = "/"
     };
     //计算薪资生产者
    public IEnumerable<bool> SendCalculateSalary(CalculateSalaryDto calculateSalaryDto)
    {
    using (var connection = factory.CreateConnection())
     {
         var channel = connection.CreateModel();
         //2、定义topic交换机
         channel.ExchangeDeclare(exchange: "sms_topic", type: "topic");

         string calculateSalaryDtoJson = JsonConvert.SerializeObject(calculateSalaryDto);
         var body = Encoding.UTF8.GetBytes(calculateSalaryDtoJson);

         //3、发送消息
         var properties = channel.CreateBasicProperties();
         properties.Persistent = true; // 设置消息持久化
         //p1 指定交换机
         //p2 routingKey 
         channel.BasicPublish(exchange: "sms_topic",routingKey: "sms.CalculateSalary",basicProperties: properties,body: body);
     }
    }

    //考勤生产者
    public IEnumerable<bool> SendCalculateAttendance(CalculateAttendanceDto calculateAttendance)
    {
    using (var connection = factory.CreateConnection())
     {
         var channel = connection.CreateModel();
         //2、定义topic交换机
         channel.ExchangeDeclare(exchange: "sms_topic", type: "topic");

         string calculateAttendanceDtoJson = JsonConvert.SerializeObject(calculateAttendance);
         var body = Encoding.UTF8.GetBytes(calculateAttendanceDtoJson);

         //3、发送消息
         var properties = channel.CreateBasicProperties();
         properties.Persistent = true; // 设置消息持久化
         //p1 指定交换机
         //p2 routingKey 
         channel.BasicPublish(exchange: "sms_topic",routingKey: "sms.CalculateAttendance",basicProperties: properties,body: body);
     }
    }

    //请假信息生产者
    public IEnumerable<bool> SendCalculateLeave(CalculateLeaveDto calculateLeave)
    {
    using (var connection = factory.CreateConnection())
     {
         var channel = connection.CreateModel();
         //2、定义topic交换机
         channel.ExchangeDeclare(exchange: "sms_topic", type: "topic");

         string calculateLeaveJson = JsonConvert.SerializeObject(calculateLeave);
         var body = Encoding.UTF8.GetBytes(calculateLeaveJson);

         //3、发送消息
         var properties = channel.CreateBasicProperties();
         properties.Persistent = true; // 设置消息持久化
         //p1 指定交换机
         //p2 routingKey 
         channel.BasicPublish(exchange: "sms_topic",routingKey: "sms.CalculateAttendance",basicProperties: properties,body: body);
     }
    } 

    public class RabbitmqHostService : IHostedService
    {
    	  public Task StartAsync(CancellationToken cancellationToken)
          {
                var factory = new ConnectionFactory()
                {
                    HostName = "localhost",
                    Port = 5672,
                    Password = "guest",
                    UserName = "guest",
                    VirtualHost = "/"
                };
               
    	              var connection = factory.CreateConnection();
                          var channel = connection.CreateModel();

    			// 1、定义交换机
    			channel.ExchangeDeclare(exchange: "sms_topic", type: ExchangeType.Topic);

    			// 2、定义随机队列
    			var queueName = channel.QueueDeclare().QueueName;

    			// 3、队列要和交换机绑定起来
    			// * 号的缺陷：只能匹配一级
                // # 能够匹配一级及多级以上 
    			channel.QueueBind(queueName,"sms_topic",routingKey: "sms.#");

    			var consumer = new EventingBasicConsumer(channel);
                consumer.Received += (model, ea) =>
                {
                    Console.WriteLine($"model:{model}");
                    var body = ea.Body;
                    // 1、业务逻辑
                    var message = Encoding.UTF8.GetString(body.ToArray());
                    Console.WriteLine(" [x] 发送短信 {0}", message);
                    // 1、消息是否正常添加到数据库当中,所以需要使用手工确认
                    channel.BasicAck(ea.DeliveryTag, true);
                };
                // 3、消费消息
                channel.BasicQos(0, 1, false); // Qos(防止多个消费者，能力不一致，导致的系统质量问题。
                // autoAck设为false 不进行自动确认                     
                channel.BasicConsume(queue: queueName,autoAck: false, consumer: consumer);
          }
          
          public Task StopAsync(CancellationToken cancellationToken)
          {
             // 1、关闭rabbitmq的连接
             throw new NotImplementedException();
          }
    } 

headers 类型的 Exchange 不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配。  
在绑定 Queue 与 Exchange 时指定一组键值对以及 x-match 参数，x-match 参数是字符串类型，可以设置为 any 或者 all。如果设置为 any，意思就是只要匹配到了 headers 表中的任何一对键值即可，all 则代表需要全部匹配。

1\. 不需要依赖 Key

2\. 更多的时候，像这种 Key Value 的键值，可能会存储在数据库中，那么我们就可以定义一个动态规则来拼装这个 Key value ，从而达到消息灵活转发到不同的队列中去

* * *

### 四、RabbitMQ 消息确认

我们根据上面的业务和代码简单实现了由生产者到消费者的一个业务流程，我们可以总结出知道，整个消息的收发过程包含有三个角色，生产者（员工管理网站）、RabbitMQ（Broker）、消费者（微服务）, 在理想状态下，按照这样实现，整个流程以及系统的稳定性，可能不会发生太大的问题，但是真正在实际应用中我们要去思考可能存在的问题，主要从三个大的方面去分析，然后发散。

1\. 生产端

2\. 存储端

3\. 消费端

###### 1. 消息生产端

我们在给 RabbitMQ 发送消息时，如何去保证消息一定到达呢，我们可以使用 RabbitMQ 提供了 2 种生产端的消息确认机制

| 模式         | 描述                                                         | 实现方式                           |
| ---------- | ---------------------------------------------------------- | ------------------------------ |
| Confirm 模式 | 应答模式，生产者发送一条消息之后，Rabbitmq 服务器做了个响应，表示消息确认收到                | 异步模式，在应答之前，可以继续发送消息，单条消息、批量消息  |
| Tx 事务模式    | 基于 AMQP 协议；可以把 channel 设置成一个带事务的通道道，分为三步：1. 开启事务，提交事务，回滚事务 | 同步模式，在事务提交之前不能继续发送消息，事务模式效率差一些 |

-   `1.Confirm 实现`


    using (var connection = factory.CreateConnection())
    {
        var channel = connection.CreateModel();
        //2、定义topic交换机
        channel.ExchangeDeclare(exchange: "sms_topic", type: "topic");

        string calculateAttendanceDtoJson = JsonConvert.SerializeObject(calculateAttendance);
        var body = Encoding.UTF8.GetBytes(calculateAttendanceDtoJson);

        //3、发送消息
        var properties = channel.CreateBasicProperties();
        properties.Persistent = true; // 设置消息持久化

        try
        {
            //开启消息确认模式
            channel.ConfirmSelect();
            channel.BasicPublish(exchange: "sms_topic", 
            routingKey: "sms.CalculateAttendance", basicProperties: properties, body: body);
            //如果一条消息或多消息都确认发送
    	    if (channel.WaitForConfirms()) 
            {
               Console.WriteLine($"【{message}】发送到Broke成功！");
            }
            else
            { 
               //可以记录个日志，重试一下；
            }
            //如果所有消息发送成功 就正常执行；如果有消息发送失败；就抛出异常；
            channel.WaitForConfirmsOrDie();
        }
        catch (Exception ex)
        {	
        	 Console.WriteLine($"【{message}】发送到Broker失败！");
        }
    } 

-   `2.Tx 事务 实现`


     using (var connection = factory.CreateConnection())
      {
          var channel = connection.CreateModel();
          //2、定义topic交换机
          channel.ExchangeDeclare(exchange: "sms_topic", type: "topic");
      
          string calculateAttendanceDtoJson = JsonConvert.SerializeObject(calculateAttendance);
          var body = Encoding.UTF8.GetBytes(calculateAttendanceDtoJson);
      
          //3、发送消息
          var properties = channel.CreateBasicProperties();
          properties.Persistent = true; // 设置消息持久化
      
          try
          {
              //开启事务机制，AMQP协议支持
              channel.TxSelect(); //事务是协议支持的
              channel.BasicPublish(exchange: "sms_topic", 
              routingKey: "sms.CalculateAttendance", basicProperties: properties, body: body);
              //提交事务 只有事务提交了才会真正写入队列
              channel.TxCommit();
          }
          catch (Exception ex)
          {	
          	//事务回滚
        		 channel.TxRollback(); 
          }
      } 

###### 2. 消息存储端

我们生产端给 RabbitMQ 发送消息成功后，如果 RabbitMQ 宕机了，会导致 RabbitMQ 中消息丢失，如何解决消息丢失问题，针对 RabbitMQ 消息丢失，我们可以在生产者中使用

1\. 持久化消息

2\. 集群

###### 3. 消息消费端

-   1\. 消费者宕机，导致消息丢失
-   2\. 执行业务逻辑失败，但是消息已经被消费

​ 当生产者写入消息到 RabbitMQ 后，消费服务接收消息期间，服务器宕机, 导致消息丢失了，这个时候我们就应该使用 RabbitMQ 的`消费端消息确认机制`

| 模式           | 描述                                                                                                                         | 特点      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- | ------- |
| 自动确认 autoAck | 自动确认，是消费消息的时候，只要收到消息，就直接回执给 RabbitMQ，已经收到一切正常； 直接总览所有了，如果有 1w 条消息，只是消费成功了一条消息，RabbitMQ 也会认为你是全部成功了，会将所有消息从队列中移除；这样会导致消息的丢失 | 处理很快    |
| 手动确认         | 消费者消费一条，回执给 RabbitMQ 一条消息，RabbitMQ 只删除当前这一条消息, 相当于是一条消费了，删除一条消息；                                                           | 性能稍微低一些 |

`1. 自动确认`

    // 消息自动确认机制
    channel.BasicConsume(queue: "CreateAttendance",autoAck: true, consumer: consumer); 

`2. 手动确认`

消费者收到消息。消费者发送确认消息给 rabbitmq 期间。执行业务逻辑失败了，但是消息已经确认被消费了，我们应该在我们的消费者接收消息回调执行业务逻辑后面，执行使用手动确认消息机制, 保证消息不被丢失

    var connection = factory.CreateConnection();
    var channel = connection.CreateModel();
    channel.ExchangeDeclare(exchange: "sms_topic", type: ExchangeType.Topic);
    var queueName = channel.QueueDeclare().QueueName;
    channel.QueueBind(queueName,"sms_topic",routingKey: "sms.#");

    var consumer = new EventingBasicConsumer(channel);
    consumer.Received += (model, ea) =>
    {
          var message = Encoding.UTF8.GetString(ea.Body.ToArray());
          //执行业务逻辑
          
          //手工确认告诉borker可以删除消息了
          channel.BasicAck(ea.DeliveryTag, true);

          //否定：告诉Broker，这个消息我没有正常消费；  requeue: true：重新写入到队列里去； false:你还是删除掉；
          //channel.BasicReject(deliveryTag: ea.DeliveryTag, requeue: true);
    };

    // autoAck设为false 不进行自动确认                     
    channel.BasicConsume(queue: queueName,autoAck: false, consumer: consumer); 

-   3\. 由于服务器性能不一致导致消息堆积

    生产者发送高并发消息，消费者来不及处理，导致消息堆积，如何解决消息堆积问题？可以使用消费服务集群，将压力分散到不同的服务实例能解决这个问题，但是又产生了一个新的集群缺陷问题，假设集群服务器的强弱不一致，比较弱的服务器处理消息慢，就会导致大部分消息堆积在这台性能较差的服务器，那又该如何解决呢？  
    我们可以采用 RabbitMQ 的`QOS`功能，俗称限流，他的意思就是消费者一次可以拉取指定数量的消息，在这些消息未处理完毕之前，不会再向队列拉取消息。


    // Qos(防止多个消费者，能力不一致，导致的系统质量问题。
    // 每一次一个消费者只成功消费一个)
    channel.BasicQos(0, 1, false); 

-   4\. 如何保证消息不被重复消费（幂等性）
       `1. 生产时消息重复`
       由于生产者发送消息给 MQ，在 MQ 确认的时候出现了网络波动，生产者没有收到确认，实际上 MQ  
       已经接收到了消息。这时候生产者就会重新发送一遍这条消息。生产者中如果消息未被确认，或确  
       认失败，我们可以使用定时任务 +（redis/db）来进行消息重试。
       `2. 消费时消息重复`
       消费者消费成功后，再给 MQ 确认的时候出现了网络波动，MQ 没有接收到确认，为了保证消息被消费，MQ 就会继续给消费者投递之前的消息。这时候消费者就接收到了两条一样的消息。  
       我们可以让每个消息携带一个全局的唯一 ID，即可保证消息的幂等性消费者获取到消息后先根据 id 去查询 redis/db 是否存在该消息。如果不存在，则正常消费，消费完毕后写入 redis/db。  
       如果存在，则证明消息被消费过，直接丢弃。 
    [https://www.cnblogs.com/yuxl01/p/15978229.html](https://www.cnblogs.com/yuxl01/p/15978229.html)
