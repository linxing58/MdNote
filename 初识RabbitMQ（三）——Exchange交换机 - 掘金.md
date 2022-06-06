# 初识RabbitMQ（三）——Exchange交换机 - 掘金
这是我参与「掘金日新计划 · 6 月更文挑战」的第 4 天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https&#x3A;//juejin.cn/post/7099702781094674468")

### 前言

    大家好，我是程序猿小白 GW_gw，很高兴能和大家一起学习进步。
    复制代码

以下内容部分来自于网络，如有侵权，请联系我删除，本文仅用于学习交流，不用作任何商业用途。

### 摘要

    本文主要带大家了解RabbitMQ的常用的三种交换机(direct、fanout、topic)的类型以及各种交换机的使用特点。
    复制代码

### Exchange 交换机类型

#### 1. Direct 类型交换机

direct 类型的交换机会根据 RoutingKey 和 BindingKey 进行精准一对一匹配，进而把数据放入到相应的队列中。是一种单播模式。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcdb62698ebe47de80cfc866d7b57bea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

#### 2. Fanout 类型交换机

Fanout 类型交换机会把消息转发给所有与自己绑定的队列中，是一种广播模式。但是在发送消息之前，消费者必须要完成对相应队列的监听，才能接收到消息，即如果发送消息时，消费者还未完成监听，那么将接收不到该消息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6b8133807684668acc9021268f2aeb5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

这种模式可能会丢失消息，所以应该应用于消息不是那么重要的，就算接收不到也影响不大，例如 APP 应用的推送消息。

#### 3. topic 类型交换机

topic 类型的交换机也是一种一对多的一种广播模式，但和 Fanout 交换机不同的是，它需要使用 RoutingKey 和 BindingKey 进行模糊匹配。RoutingKey 和 BindingKey 需要为同一个模式才能匹配。RoutingKey 和 BindingKey 的字符串被切分为一个个单词，用点 (.) 来进行分割。有两个通配符可以使用，分别是\*(必须匹配一个单词)，#(匹配 0 个或多个单词)。并且 topic 类型的交换机也是需要消费者先订阅队列之后才能接收到消息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68cb4f1d6c0f42e59a470441a09a5490~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd811c10a256429c83ee48288b8b89a1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

> 注意：以单词为单位来分割，单词使用点 (.) 来分割。

### 小结

以上就是关于 RabbitMq 的常用的三种交换机的使用，希望对读者有所帮助，如有不正之处，欢迎留言指正。 
 [https://juejin.cn/post/7104088857477382158](https://juejin.cn/post/7104088857477382158)
