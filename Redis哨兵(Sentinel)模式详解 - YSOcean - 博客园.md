# Redis哨兵(Sentinel)模式详解 - YSOcean - 博客园
* * *

　　在上一篇博客 ----[Redis 详解（八）------ 主从复制](https://www.cnblogs.com/ysocean/p/9143118.html), 我们简单介绍了 Redis 的主从架构, 但是这种主从架构存在一个问题, 当主服务器宕机, 从服务器不能够自动切换成主服务器, 为了解决这个问题, 我们又介绍了哨兵模式, 本篇博客我们继续深入的介绍一下这种模式.

### 1、架构图

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210163639369-1861604133.png)

### 2、服务器列表

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210163723019-1037639332.png)

### 3、搭建主从模式

　　**①、主要配置项**

　　主服务器 (上图的 Node1) 配置文件 redis.config 主要配置项:

```

```

　　从服务器配置文件主要配置项基本和主服务器保持一致, 需要修改端口 port ; 另外存放位置和日志文件名也可以根据需要修改.

　　为了表示主从关系, 还需要在从服务器配置文件中添加一行**重要配置**:

```

```

　　**②、验证主从关系**

　　配置完成后, 我们通过 redis-server redis.conf 命令启动 Redis. 然后通过 redis-cli -p 端口 分别进入到各台服务器的控制行页面:

　　输入如下命令:

　　三台服务器打印结果如下:

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210171946052-514568169.png)

　　由上图可以看到, Node1 服务器作为主服务器, 节点角色是 master, 另外的两台从服务器, 节点角色都是 slave.

　　另外还可以进行如下测试: 可以在主服务器上添加一条数据, 然后看看从服务器上是否能够查到该数据.

　　**③、问题**

如果对于上面的测试, 主服务器上添加的数据, 从服务器上无法查询到, 可以查看前面配置的目录 / opt/redis/data 日志文件, 有一种错误如下:

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210175353516-1091621637.png)

　　这是由于主服务器设置了登录密码, 从服务器在向主服务器进行数据同步复制时, 由于不知道主服务器密码, 导致连接不上, 从而无法进行同步.

　　解决这个问题, 需要明确两个配置:

　　**一. requreipass**

　　设置 redis 的登录密码.

　　**二. masterauth**

　　针对 master 对应的 slave 节点设置的，在 slave 节点数据同步的时候用到。

　　建议, 如果启用 Redis 密码校验, 最好将各个节点的 masterauth 和 requirepass 设置为相同的密码; 如果不设置为相同的, 要注意 slave 节点 masterauth 和 master 节点 requirepass 的对应关系.

### 4、搭建哨兵模式

　　**①、主要配置项** 　

　　配置文件名称为：sentinel.conf

```

```

　　注意三台服务器的端口配置. 如果 redis 服务器配置了密码连接, 则要增加如下配置:

```

```

　　后面的 123 表示密码. 注意这行配置要配置到 sentinel monitor mymaster ip port 后面, 因为名称 mymaster 要先定义.

　　②**、启动哨兵**

```

```

　　③**、验证主从自动切换**

　　首先 kill 掉 Redis 主节点. 然后查看 sentinel 日志:

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210201842784-528299150.png)

　　上面截图红框框住的几个重要信息, 这里先介绍最后一行, switch-master mymaster 192.168.14.101 6379 192.168.14.103 6381 表示 master 服务器将由 6379 的 redis 服务切换为 6381 端口的 redis 服务器.

　　PS:**+switch-master** 表示切换主节点.

　　然后我们通过 info replication 命令查看 6381 的 redis 服务器:

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210202309053-1947340550.png)

　　我们发现, 6381 的 Redis 服务已经切换成 master 节点了. 

　　另外, 也可以查看 sentinel.conf 配置文件, 里面的 sentinel monitor mymaster 192.168.14.101 6379 2 也自动更改为 sentinel monitor mymaster 192.168.14.103 6381 2 配置了.

### 5、Java 客户端连接哨兵集群

　　这里通过 springboot 项目来连接, 代码地址如下:

　　这里贴一下主要测试代码:

　　PS: 实际上 springboot 已经为我们注入了 RedisTemplate, 我们在实际项目中不用写的像下面代码这么麻烦, 这样写是为了详细的表明连接步骤.

```

```

### 6、Java 客户端连接原理

　　**①**、**结构图**

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200211122445582-11189433.png)

　　**②、连接步骤** 

　　一. 客户端遍历所有的 Sentinel 节点集合, 获取一个可用的 Sentinel 节点.

　　二. 客户端向可用的 Sentinel 节点发送 get-master-addr-by-name 命令, 获取 Redis Master 节点.

　　三. 客户端向 Redis Master 节点发送 role 或 role replication 命令, 来确定其是否是 Master 节点, 并且能够获取其 slave 节点信息.

　　四. 客户端获取到确定的节点信息后, 便可以向 Redis 发送命令来进行后续操作了

　　需要注意的是: 客户端是和 Sentinel 来进行交互的, 通过 Sentinel 来获取真正的 Redis 节点信息, 然后来操作. 实际工作时, Sentinel 内部维护了一个主题队列, 用来保存 Redis 的节点信息, 并实时更新, 客户端订阅了这个主题, 然后实时的去获取这个队列的 Redis 节点信息.

### 7、哨兵模式工作原理

**①、三个定时任务**

　　一. 每 10 秒每个 sentinel 对 master 和 slave 执行 info 命令: 该命令第一个是用来发现 slave 节点, 第二个是确定主从关系.

　　二. 每 2 秒每个 sentinel 通过 master 节点的 channel(名称为\_sentinel\_:hello) 交换信息 (pub/sub): 用来交互对节点的看法(后面会介绍的节点主观下线和客观下线) 以及自身信息.

　　三. 每 1 秒每个 sentinel 对其他 sentinel 和 redis 执行 ping 命令, 用于心跳检测, 作为节点存活的判断依据.

　　**②、主观下线和客观下线**

一. 主观下线

　　SDOWN:subjectively down, 直接翻译的为” 主观” 失效, 即当前 sentinel 实例认为某个 redis 服务为” 不可用” 状态.

　　二. 客观下线

　　ODOWN:objectively down, 直接翻译为” 客观” 失效, 即多个 sentinel 实例都认为 master 处于”SDOWN” 状态, 那么此时 master 将处于 ODOWN,ODOWN 可以简单理解为 master 已经被集群确定为” 不可用”, 将会开启故障转移机制.

　　结合我们第 4 点搭建主从模式, 验证主从切换时, kill 掉 Redis 主节点, 然后查看 sentinel 日志, 如下:

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200210201842784-528299150.png)

　　发现有类似 sdown 和 odown 的日志. 在结合我们配置 sentinel 时的配置文件来看:

```

```

　　最后的 2 表示投票数, 也就是说当一台 sentinel 发现一个 Redis 服务无法 ping 通时, 就标记为 主观下线 sdown; 同时另外的 sentinel 服务也发现该 Redis 服务宕机, 也标记为 主观下线, 当多台 sentinel (大于等于 2, 上面配置的最后一个) 时, 都标记该 Redis 服务宕机, 这时候就变为客观下线了, 然后进行故障转移.

　　**③、故障转移**

故障转移是由 sentinel 领导者节点来完成的 (只需要一个 sentinel 节点), 关于 sentinel 领导者节点的选取也是每个 sentinel 向其他 sentinel 节点发送我要成为领导者的命令, 超过半数 sentinel 节点同意, 并且也大于 quorum , 那么他将成为领导者, 如果有多个 sentinel 都成为了领导者, 则会过段时间在进行选举.

　　sentinel 领导者节点选举出来后, 会通过如下几步进行故障转移:

　　一. 从 slave 节点中选出一个合适的 节点作为新的 master 节点. 这里的合适包括如下几点:

　　　　1\. 选择 slave-priority(slave 节点优先级) 最高的 slave 节点, 如果存在则返回, 不存在则继续下一步判断.

　　　　2\. 选择复制偏移量最大的 slave 节点 (复制的最完整), 如果存在则返回, 不存在则继续.

　　　　3\. 选择 runId 最小的 slave 节点 (启动最早的节点)

二. 对上面选出来的 slave 节点执行 slaveof no one 命令让其成为新的 master 节点.

　　三. 向剩余的 slave 节点发送命令, 让他们成为新 master 节点的 slave 节点, 复制规则和前面设置的 parallel-syncs 参数有关.

　　四. 更新原来 master 节点配置为 slave 节点, 并保持对其进行关注, 一旦这个节点重新恢复正常后, 会命令它去复制新的 master 节点信息.(注意: 原来的 master 节点恢复后是作为 slave 的角色)

　　可以从 sentinel 日志中出现的几个消息来进行查看故障转移:

　　1\.**+switch-master**: 表示切换主节点 (从节点晋升为主节点)

　　2\.**+sdown**: 主观下线

　　3\.**+odown**: 客观下线

　　4\.**+convert-to-slave**: 切换从节点 (原主节点降为从节点)

### 资料推荐

最近又赶上跳槽的高峰期（招聘旺季），好多读者都问我要有没有最新面试题，找华为朋友整理一份内部资料《第 6 版：互联网大厂面试题》！

整个资料包，包括 Spring、Spring Boot/Cloud、Dubbo、JVM、集合、多线程、JPA、MyBatis、MySQL、大数据、Nginx、Git、Docker、GitHub、Servlet、JavaWeb、IDEA、Redis、算法、面试题等几乎覆盖了 Java 基础和阿里巴巴等大厂面试题等、等技术栈！

![](https://img2020.cnblogs.com/blog/1120165/202106/1120165-20210612225217077-4954963.png)

 另外，还有博主为大家整理的 Java 关键字解析、Java 源码解析

![](https://img2020.cnblogs.com/blog/1120165/202106/1120165-20210612225327187-2090169914.png)

![](https://img2020.cnblogs.com/blog/1120165/202106/1120165-20210612225441168-543048236.png)

据说已经有小伙伴通过这套资料，成功的入职了蚂蚁金服、字节跳动等大厂。

而且，这些资料不是扫描版的，里面的文字都可以直接复制，非常便于我们学习： 

如果你想获得完整 PDF 可以通过以下方式获得

面试大全怎么获取：

1.  关注下方公众号
2.  在下方公众号后台回复 【666】 即可。
3.  ![](https://img2020.cnblogs.com/blog/1120165/202106/1120165-20210612225611359-464449875.png) 
    [https://www.cnblogs.com/ysocean/p/12290364.html](https://www.cnblogs.com/ysocean/p/12290364.html)
