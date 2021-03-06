# redis的雪崩和穿透
**1. 什么是缓存穿透**

    一般的缓存系统，都是按照 key 值去缓存查询，如果不存在对应的 value，就应该去 DB 中查找 。这个时候，如果请求的并发量很大，就会对后端的 DB 系统造成很大的压力。这就叫做缓存穿透。关键词：缓存 value 为空；并发量很大去访问 DB。

![](https://img-blog.csdn.net/20171023212902089?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHpqMzQ2MjE0NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**造成的原因**

1\. 业务自身代码或数据出现问题；2. 一些恶意攻击、爬虫造成大量空的命中，此时会对数据库造成很大压力。

**解决方法**

1\. 设置布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，  
从避免了对底层存储系统的查询压力。  
2. 如果一个查询返回的数据为空，不管是数据不存在还是系统故障，我们仍然把这个结果进行缓存，但是它的过期时间会很短  
最长不超过 5 分钟。  

**二、雪崩**

**1. 什么是雪崩**

因为缓存层承载了大量的请求，有效的保护了存储 层，但是如果缓存由于某些原因，整体不能够提供服务，于是所有的请求，就会到达存储层，存储层的调用量就会暴增，造成存储层也会挂掉的情况。缓存雪崩的英文解释是奔逃的野牛，指的是缓存层当掉之后，并发流量会像奔腾的野牛一样，大量后端存储。

存在这种问题的一个场景是：当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，大量数据会去直接访问 DB，此时给 DB 很大的压力。

![](https://img-blog.csdn.net/20171023213550913?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHpqMzQ2MjE0NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**2. 解决方法**

（1）设置 redis 集群和 DB 集群的高可用，如果 redis 出现宕机情况，可以立即由别的机器顶替上来。这样可以防止一部分的风险。  

（2）使用互斥锁

在缓存失效后，通过加锁或者队列来控制读和写数据库的线程数量。比如：对某个 key 只允许一个线程查询数据和写缓存，其他线程等待。单机的话，可以使用 synchronized 或者 lock 来解决，如果是分布式环境，可以是用 redis 的 setnx 命令来解决。

（3）不同的 key, 可以设置不同的过期时间，让缓存失效的时间点不一致，尽量达到平均分布。

（4）永远不过期

redis 中设置永久不过期，这样就保证了，不会出现热点问题，也就是物理上不过期。

（5）资源保护

使用 netflix 的 hystrix，可以做各种资源的线程池隔离，从而保护主线程池。

**3. 使用**

四种方案，没有最佳只有最合适， 根据自己项目情况使用不同的解决策略。 
 [https://blog.csdn.net/lzj3462144/article/details/78323589](https://blog.csdn.net/lzj3462144/article/details/78323589)
