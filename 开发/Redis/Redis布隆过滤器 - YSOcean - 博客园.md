# Redis布隆过滤器 - YSOcean - 博客园
* * *

　　本篇博客我们主要介绍如何用 Redis 实现布隆过滤器，但是在介绍布隆过滤器之前，我们首先介绍一下，为啥要使用布隆过滤器。

### 1、布隆过滤器使用场景

　　比如有如下几个需求：

　　①、原本有 10 亿个号码，现在又来了 10 万个号码，要快速准确判断这 10 万个号码是否在 10 亿个号码库中？

　　解决办法一：将 10 亿个号码存入数据库中，进行数据库查询，准确性有了，但是速度会比较慢。

　　解决办法二：将 10 亿号码放入内存中，比如 Redis 缓存中，这里我们算一下占用内存大小：10 亿\*8 字节 = 8GB，通过内存查询，准确性和速度都有了，但是大约 8gb 的内存空间，挺浪费内存空间的。

　　②、接触过爬虫的，应该有这么一个需求，需要爬虫的网站千千万万，对于一个新的网站 url，我们如何判断这个 url 我们是否已经爬过了？

　　解决办法还是上面的两种，很显然，都不太好。

　　③、同理还有垃圾邮箱的过滤。

　　那么对于类似这种，大数据量集合，如何准确快速的判断某个数据是否在大数据量集合中，并且不占用内存，**布隆过滤器**应运而生了。

### 2、布隆过滤器简介

　　带着上面的几个疑问，我们来看看到底什么是布隆过滤器。

　　布隆过滤器：一种数据结构，是由一串很长的二进制向量组成，可以将其看成一个二进制数组。既然是二进制，那么里面存放的不是 0，就是 1，但是初始默认值都是 0。

　　如下所示：

　　![](https://img2020.cnblogs.com/blog/1120165/202003/1120165-20200330220824117-1290183653.png)

　　**①、添加数据**

　　介绍概念的时候，我们说可以将布隆过滤器看成一个容器，那么如何向布隆过滤器中添加一个数据呢？

　　如下图所示：当要向布隆过滤器中添加一个元素 key 时，我们通过多个 hash 函数，算出一个值，然后将这个值所在的方格置为 1。

　　比如，下图 hash1(key)=1，那么在第 2 个格子将 0 变为 1（数组是从 0 开始计数的），hash2(key)=7，那么将第 8 个格子置位 1，依次类推。

　　![](https://img2020.cnblogs.com/blog/1120165/202003/1120165-20200330221613591-2062171492.png)

　　**②、判断数据是否存在？**

　　知道了如何向布隆过滤器中添加一个数据，那么新来一个数据，我们如何判断其是否存在于这个布隆过滤器中呢？

　　很简单，我们只需要将这个新的数据通过上面自定义的几个哈希函数，分别算出各个值，然后看其对应的地方是否都是 1，如果存在一个不是 1 的情况，那么我们可以说，该新数据一定不存在于这个布隆过滤器中。

　　反过来说，如果通过哈希函数算出来的值，对应的地方都是 1，那么我们能够肯定的得出：这个数据一定存在于这个布隆过滤器中吗？

　　答案是否定的，因为多个不同的数据通过 hash 函数算出来的结果是会有重复的，所以会存在某个位置是别的数据通过 hash 函数置为的 1。

　　我们可以得到一个结论：**布隆过滤器可以判断某个数据一定不存在，但是无法判断一定存在**。

　　**③、布隆过滤器优缺点**

　　优点：优点很明显，二进制组成的数组，占用内存极少，并且插入和查询速度都足够快。

　　缺点：随着数据的增加，误判率会增加；还有无法判断数据一定存在；另外还有一个重要缺点，无法删除数据。

### 3、Redis 实现布隆过滤器

#### ①、bitmaps

　　我们知道计算机是以二进制位作为底层存储的基础单位，一个字节等于 8 位。

　　比如 “big” 字符串是由三个字符组成的，这三个字符对应的 ASCII 码分为是 98、105、103，对应的二进制存储如下：

　　![](https://img2020.cnblogs.com/blog/1120165/202004/1120165-20200404213453130-714545258.png)

　　在 Redis 中，Bitmaps 提供了一套命令用来操作类似上面字符串中的每一个位。

　　**一、设置值**

　　![](https://img2020.cnblogs.com/blog/1120165/202004/1120165-20200404214003953-5622706.png)

 　　我们知道 "b" 的二进制表示为 0110 0010，我们将第 7 位（从 0 开始）设置为 1，那 0110 0011 表示的就是字符 “c”，所以最后的字符 “big” 变成了“cig”。

　　**二、获取值**

　　![](https://img2020.cnblogs.com/blog/1120165/202004/1120165-20200404214216788-374691466.png)

 　　**三、获取位图指定范围值为 1 的个数**

　　如果不指定，那就是获取全部值为 1 的个数。

　　注意：start 和 end 指定的是**字节的个数**，而不是位数组下标。

　　![](https://img2020.cnblogs.com/blog/1120165/202004/1120165-20200404214812688-1075855704.png)

#### ②、Redisson

　　Redis 实现布隆过滤器的底层就是通过 bitmap 这种数据结构，至于如何实现，这里就不重复造轮子了，介绍业界比较好用的一个客户端工具——Redisson。

　　Redisson 是用于在 Java 程序中操作 Redis 的库，利用 Redisson 我们可以在程序中轻松地使用 Redis。

　　下面我们就通过 Redisson 来构造布隆过滤器。

![](https://common.cnblogs.com/images/copycode.gif)

 1 package com.ys.rediscluster.bloomfilter.redisson; 2 
 3 import org.redisson.Redisson; 4 import org.redisson.api.RBloomFilter; 5 import org.redisson.api.RedissonClient; 6 import org.redisson.config.Config; 7 
 8 public class RedissonBloomFilter { 9 
10     public static void main(String\[] args) {11         Config config = new Config(); 12         config.useSingleServer().setAddress("redis://192.168.14.104:6379"); 13         config.useSingleServer().setPassword("123"); 14         // 构造 Redisson
15         RedissonClient redisson = Redisson.create(config); 16 
17         RBloomFilter<String> bloomFilter = redisson.getBloomFilter("phoneList"); 18         // 初始化布隆过滤器：预计元素为 100000000L, 误差率为 3%
19         bloomFilter.tryInit(100000000L,0.03); 20         // 将号码 10086 插入到布隆过滤器中
21         bloomFilter.add("10086"); 22 
23         // 判断下面号码是否在布隆过滤器中
24         System.out.println(bloomFilter.contains("123456"));//false
25         System.out.println(bloomFilter.contains("10086"));//true
26 } 27 }

![](https://common.cnblogs.com/images/copycode.gif)

　　这是单节点的 Redis 实现方式，如果数据量比较大，期望的误差率又很低，那单节点所提供的内存是无法满足的，这时候可以使用分布式布隆过滤器，同样也可以用 Redisson 来实现，这里我就不做代码演示了，大家有兴趣可以试试。

### 4、guava 工具

　　最后提一下不用 Redis 如何来实现布隆过滤器。

　　guava 工具包相信大家都用过，这是谷歌公司提供的，里面也提供了布隆过滤器的实现。

![](https://common.cnblogs.com/images/copycode.gif)

 1 package com.ys.rediscluster.bloomfilter; 2 
 3 import com.google.common.base.Charsets; 4 import com.google.common.hash.BloomFilter; 5 import com.google.common.hash.Funnel; 6 import com.google.common.hash.Funnels; 7 
 8 public class GuavaBloomFilter { 9     public static void main(String\[] args) { 10         BloomFilter<String> bloomFilter = BloomFilter.create(Funnels.stringFunnel(Charsets.UTF_8),100000,0.01); 11 
12         bloomFilter.put("10086"); 13 
14         System.out.println(bloomFilter.mightContain("123456")); 15         System.out.println(bloomFilter.mightContain("10086")); 16 } 17 }

![](https://common.cnblogs.com/images/copycode.gif)

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

 [https://www.cnblogs.com/ysocean/p/12594982.html](https://www.cnblogs.com/ysocean/p/12594982.html)
