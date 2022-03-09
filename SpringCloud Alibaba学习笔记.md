# SpringCloud Alibaba学习笔记
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

**`Srpingcloud Alibaba`是什么？**

1.  `Spring Cloud Alibaba` 是**阿里巴巴**提供的微服务开发一站式解决方案，是**阿里巴巴开源中间件**与 `Spring Cloud` 体系的融合。
2.  在 Spring Cloud 项目中孵化，很可能成为 Spring Cloud 第二代的标准实现。
3.  在业界广泛使用，已有很多成功案例。

**那么很多人要问了，为什么已经有了`SpringCloud` 还要去研发 `SpringCloud Alibaba？`**

-   `SpringCloud`:

1.  部分组件**停止维护和更新**，给开发带来不便;
2.  `SpringCloud`部分**环境搭建复杂**，没有完善的可视化界面, 我们需要大量的二次开发和定制；
3.  `SpringCloud`**配置复杂**，难以上手，部分配置差别难以区分和合理应用

-   `Srpingcloud Alibaba`：阿里使用过的组件经历了考验，**性能强悍**，**设计合理**，现在开源出来成套的产品搭配完善的可视化界面给开发运维带来极大的便利，**搭建简单**，**学习曲线低**。

![](https://img-blog.csdnimg.cn/20210318140704558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

**真实应用场景**

1.  大型复杂的系统，例如大型电商系统
2.  高并发系统，例如大型门户，秒杀系统
3.  需求不明确，且变更很快的系统，例如创业公司业务系统。

**适合哪些人群学习？**

-   拥有 1-3 年的工作经验，对 Spring Cloud Alibaba 有浓厚的兴趣，
-   正在冲击大厂岗位
-   有 1 年以上工作经验，对微服务开发有兴趣有需求的开发人员
-   从事传统开发，想要转型做互联网业务、中间件 开发、架构设计方向的程序员
-   想要了解微服务，对 Spring Cloud Alibaba 诸多开发高级特性及阅读源码感兴趣，且再工作 / 面试中遇到问题无从下手的
-   想进一步提升架构设计认知和源码阅读的其他职位
-   …

**这份学习笔记的优势在哪里？**

1.  经典案例 + 知识点思维导图的讲解的方式
2.  深入浅出，从基础到进阶，再到实战，全方位解析

> **由于内容较多，本次将只展示部分笔记内容，如果看得不过瘾想更加深入地了解本笔记彻底掌握 `SpringCloud Alibaba` 可在`文末`了解详情。** 

下面我们来了解一下这份 **全网首发下载秒破万的阿里 P7 私传 “SpringCloud Alibaba” 学习笔记** 到底有多厉害？  
![](https://img-blog.csdnimg.cn/20210318141628749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  系统架构演变
2.  微服务架构介绍
3.  SpringCloud Alibaba 介绍

![](https://img-blog.csdnimg.cn/20210318142035855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  案例准备
2.  创建父工程
3.  创建基础模块
4.  创建用户微服务
5.  创建商品微服务
6.  创建订单微服务

![](https://img-blog.csdnimg.cn/20210318142307119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  服务治理介绍
2.  nacos 简介
3.  nacos 实战入门
4.  实现服务调用的负载均衡
5.  基于 Feign 实现服务调用

![](https://img-blog.csdnimg.cn/20210318142556925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  高并发带来的问题
2.  服务雪崩效应
3.  常见容错方案
4.  Sentinel 入门
5.  Sentinel 的概念和功能
6.  Sentinel 规则
7.  @SentinelResource 的使用
8.  Sentinel 规则持久化
9.  Feign 整合 Sentinel

![](https://img-blog.csdnimg.cn/20210318143031585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  网关简介
2.  Gateway 简介
3.  Gateway 快速入门
4.  Gateway 核心架构
5.  断言
6.  过滤器
7.  网关限流

![](https://img-blog.csdnimg.cn/20210318143410288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  链路追踪介绍
2.  Sleuth 入门
3.  Zipkin 的集成
4.  Zipkin 数据持久化

![](https://img-blog.csdnimg.cn/20210318143654655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  MQ 简介
2.  Rocketmq 入门
3.  消息发送和接收演示
4.  案例
5.  发送不同类型的消息
6.  消息消费要注意的细节

![](https://img-blog.csdnimg.cn/20210318143935193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  短信服务介绍
2.  短信服务使用
3.  下单之后发送短信

![](https://img-blog.csdnimg.cn/20210318144109672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  服务配置中心介绍
2.  Nacos Config 入门
3.  Nacos Config 深入
4.  nacos 的几个概念

![](https://img-blog.csdnimg.cn/20210318144442365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

1.  分布式事务基础
2.  分布式事务解决方案
3.  Seata 介绍
4.  Seata 实现分布式事务控制

![](https://img-blog.csdnimg.cn/20210318144718391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfQ2FpeW8=,size_16,color_FFFFFF,t_70#pic_center)

> 你比别人强的地方，不是你做过多少年的 CRUD 工作，而是你比别人掌握了更多深入的技能。不要总停留在 CRUD 的表面工作，理解并掌握底层原理并熟悉源码实现，并形成自己的抽象思维能力，做到灵活运用，才是你突破瓶颈，脱颖而出的重要方向！

由于字数篇幅原因，为了不影响阅读在这就展示了整个目录和内容截图，有需要这份已经整理成完整文档的微服务架构学习笔记麻烦 **一键三连** 后，**扫描下方二维码获取免费获取方式！**

![](https://img-blog.csdnimg.cn/20210412092710599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfUG9pc29u,size_16,color_FFFFFF,t_70#pic_center) 
 [https://blog.csdn.net/Java_Poison/article/details/115612360](https://blog.csdn.net/Java_Poison/article/details/115612360)
