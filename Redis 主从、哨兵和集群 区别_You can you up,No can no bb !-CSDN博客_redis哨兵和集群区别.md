# Redis 主从、哨兵和集群 区别_You can you up,No can no bb !-CSDN博客_redis哨兵和集群区别
             Redis 主从、哨兵和集群 区别\_You can you up,No can no bb !-CSDN博客\_redis哨兵和集群区别       

[![](https://img-home.csdnimg.cn/images/20201124032511.png)
](https://www.csdn.net/)

-   [博客](https://blog.csdn.net/)
-   [课程](https://edu.csdn.net/)
-   [开发者商城](https://mall.csdn.net/)
-   [问答](https://ask.csdn.net/)
-   [社区](https://bbs.csdn.net/)
-   [开源](https://git.csdn.net/)

CSDNGreener 正在守护您的浏览体验

 搜索 

[![](https://profile.csdnimg.cn/D/D/9/2_lx286225928)
](https://blog.csdn.net/lx286225928)

[_--_粉丝](https://blog.csdn.net/lx286225928?type=sub&subType=fans) [_30_关注](https://blog.csdn.net/lx286225928?type=sub) [_--_获赞](https://blog.csdn.net/lx286225928)

-   [个人中心](https://i.csdn.net/#/user-center/profile)
-   [内容管理](https://mp.csdn.net/mp_blog/manage/article?spm=1011.2124.3001.5298)
-   [学习平台](https://edu.csdn.net/)
-   [我的订单](https://mall.csdn.net/myorder)
-   [我的钱包](https://i.csdn.net/#/wallet/index)
-   [我的 API](https://portal-data.csdn.net/my/api)
-   [我的认证](https://ac.csdn.net/user/myExam.html)
-   [签到抽奖](https://i.csdn.net/#/uc/reward)
-   [退出](javascript:;)

[会员中心 ![](https://img-home.csdnimg.cn/images/20210918025138.gif)](https://mall.csdn.net/vip) 

会员特权

[  
限时抽奖](https://vip.csdn.net/welfarecenter?utm_source=vip_hyzx_hytbcj#banner)[  
领券中心](https://vip.csdn.net/welfarecenter?utm_source=vip_hyzx_hytblq#discount_center)[  
加赠 1 年](https://mall.csdn.net/vip?utm_source=vip_hyzx_fc_xsjz)[  
会员购](https://vip.csdn.net/welfarecenter?utm_source=vip_hyzx_hytbhyg#vip_shop)

[领取限时优惠券，最高可减 80 元](https://mall.csdn.net/vip) [领券开通](https://mall.csdn.net/vip)

[收藏](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blink.csdn.net)

[消息](https://i.csdn.net/#/msg/index)

[评论](https://i.csdn.net/#/msg/index) [关注](https://i.csdn.net/#/msg/attention) [点赞](https://i.csdn.net/#/msg/like) [私信](https://im.csdn.net/im/main.html) [系统通知](https://i.csdn.net/#/msg/notice) [消息设置](https://i.csdn.net/#/msg/setting)

[创作](https://mp.csdn.net)

# Redis 主从、哨兵和集群 区别

![](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[草鱼狂飙](https://blog.csdn.net/u014527619) 2019-03-06 13:39:50 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 8338 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png)
 收藏  34 

分类专栏： [运维经验](https://blog.csdn.net/u014527619/category_7703566.html)

 [![](https://img-blog.csdnimg.cn/20201014180756918.png?x-oss-process=image/resize,m_fixed,h_64,w_64)
 运维经验](https://blog.csdn.net/u014527619/category_7703566.html "运维经验") 专栏收录该内容

14 篇文章 0 订阅

订阅专栏

# 主从模式 (master-slave)

![](https://img-blog.csdnimg.cn/20190306140233493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1Mjc2MTk=,size_16,color_FFFFFF,t_70)

备份数据、[负载均衡](https://so.csdn.net/so/search?q=%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)，一个 Master 可以有多个 Slaves。  
主从模式强调 数据备份，读写分离等

Redis 复制功能的几个重要方面：

1.  一个主服务器可以有多个从服务器。
2.  不仅主服务器可以有从服务器， 从服务器也可以有自己的从服务器， 多个从服务器之间可以构成一个图状结构。
3.  复制功能不会阻塞主服务器： 即使有一个或多个从服务器正在进行初次同步， 主服务器也可以继续处理命令请求。
4.  复制功能也不会阻塞从服务器： 只要在 redis.conf 文件中进行了相应的设置， 即使从服务器正在进行初次同步， 服务器也可以使用旧版本的数据集来处理命令查询。  
    不过， 在从服务器删除旧版本数据集并载入新版本数据集的那段时间内， 连接请求会被阻塞。  
    你还可以配置从服务器， 让它在与主服务器之间的连接断开时， 向客户端发送一个错误。
5.  复制功能可以单纯地用于数据冗余（data redundancy）， 也可以通过让多个从服务器处理只读命令请求来提升扩展性（scalability）： 比如说， 繁重的 SORT 命令可以交给附属节点去运行。
6.  可以通过复制功能来让主服务器免于执行持久化操作： 只要关闭主服务器的持久化功能， 然后由从服务器去执行持久化操作即可。

# 哨兵模式 (sentinel)

![](https://img-blog.csdnimg.cn/20190306140335640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1Mjc2MTk=,size_16,color_FFFFFF,t_70)

sentinel 发现 master 挂了后，就会从 slave 中重新选举一个 master。  
哨兵模式强调高可用

Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

**监控（Monitoring）**： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。

**提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

**自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

客户端中不会记录 redis 的地址（某个 IP），而是记录 sentinel 的地址，这样我们可以直接从 sentinel 获取的 redis 地址，因为 sentinel 会对所有的 master、slave 进行监控，它是知道到底谁才是真正的 master 的，例如我们故障转移，这时候对于 sentinel 来说，master 是变了的，然后通知客户端。而客户端根本不用关心到底谁才是真正的 master，只关心 sentinel 告知的 master。

# 集群模式 （cluster）

![](https://img-blog.csdnimg.cn/20190306140411969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1Mjc2MTk=,size_16,color_FFFFFF,t_70)

cluster 是为了解决单机 Redis 容量有限的问题，将数据按一定的规则分配到多台机器。

集群模式提高并发量。

* * *

# Redis 持久化

redis 支持两种方式的持久化，可以单独使用或者结合起来使用。  
1、第一种：RDB 方式 redis 默认的持久化方式（快照）  
2、第二种：AOF 方式（日志追加）

RDB 快照模式 (snapshot):

```
save 900 1  //900秒内如果超过1个key被修改，则发起快照保存
save 300 10 //300秒内容如超过10个key被修改，则发起快照保存
save 60 10000  //(这3个选项都屏蔽,则rdb禁用)

stop-writes-on-bgsave-error yes  // 后台备份进程出错时,主进程停不停止写入
rdbcompression yes      // 导出的rdb文件是否压缩
Rdbchecksum   yes       //导入rbd恢复时数据时,要不要检验rdb的完整性
dbfilename dump.rdb     //导出来的rdb文件名
dir /var/lib/redis/     //rdb的放置路径

```

AOF 日志模式:

```
appendonly yes              //启用aof持久化方式
# appendfsync always        //每收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用
appendfsync everysec        //每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，推荐
# appendfsync no            //完全依赖os，性能最好,持久化没保证（操作系统自身的同步）
no-appendfsync-on-rewrite  yes   //正在导出rdb快照的过程中,要不要停止同步aof
auto-aof-rewrite-percentage 100  //aof文件大小比起上次重写时的大小,增长率100%时,重写
auto-aof-rewrite-min-size 64mb   //aof文件,至少超过64M时,重写

```

注意：  
1、AOF 和 RDB 同时存在的时候，优先使用 AOF 恢复数据，因为 AOF 保存的数据集更完整；  
2、AOF 和 RDB 恢复的时候，RDB 更快，直接拷贝数据，AOF 还要执行语句；  
3、建议同时使用 AOF 和 RDB

参考资料：  
[Redis 主从同步，哨兵](https://jasonhzy.github.io/2017/12/12/redis/)  
[redis sentinel 架构详解](https://my.oschina.net/u/3371837/blog/1789790)

显示推荐内容

[![](https://profile.csdnimg.cn/2/C/5/3_u014527619)
草鱼狂飙](https://blog.csdn.net/u014527619)

[关注](javascript:;) 关注

-   ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUpactive.png)
     ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newHeart2021Active.png)
     ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newHeart2021White.png)
      3 

    点赞
-   ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newUnHeart2021Active.png)
     ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newUnHeart2021White.png) 

    踩
-   [![](https://csdnimg.cn/release/blogv2/dist/pc/img/newComment2021White.png)
      3](#commentBox) 

    评论
-   [![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png)
     ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCollectWhite.png)
     ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCollectActive.png)
      34](javascript:;) 

    收藏
-   [![](https://csdnimg.cn/release/blogv2/dist/pc/img/newHealthCompanies1Active.gif)](javascript:;) 

    一键三连


-   [![](https://csdnimg.cn/release/blogv2/dist/pc/img/newShareWhite.png)](javascript:;) 

    扫一扫，分享海报

    ![](data:image/png;base64,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)

专栏目录

COPY BUTTON

 [![](https://profile.csdnimg.cn/D/D/9/3_lx286225928)](https://blog.csdn.net/lx286225928) 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/emoticon.png)

插入表情

添加代码片

-   HTML/XML
-   objective-c
-   Ruby
-   PHP
-   C
-   C++
-   JavaScript
-   Python
-   Java
-   CSS
-   SQL
-   其它

还能输入_1000_个字符 

-   [![](https://profile.csdnimg.cn/5/B/7/3_zhangyusheng091)
    ](https://blog.csdn.net/zhangyusheng091)

    [大海 VIP](https://blog.csdn.net/zhangyusheng091): 是，相当于数据库分库吧 10 月前回复![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentMore.png)
    举报

    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentUnHeart.png)
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)


-   [![](https://profile.csdnimg.cn/9/C/3/3_weixin_43182029)
    ](https://blog.csdn.net/weixin_43182029)

    [weixin_43182029](https://blog.csdn.net/weixin_43182029):cluster 是为了解决单机 Redis 容量有限的问题，将数据按一定的规则分配到多台机器。 集群模式提高并发量。集群模式每台 redis 服务器上的内容是不一样的？2 年前回复![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentMore.png)
    举报

    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentUnHeart.png)
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)
    ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)
-   -   [![](https://profile.csdnimg.cn/9/C/3/3_weixin_43182029)
        ](https://blog.csdn.net/weixin_43182029)

        [weixin_43182029](https://blog.csdn.net/weixin_43182029)回复: Redis 的哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，所以在 redis3.0 上加入了 Cluster 集群模式，实现了 Redis 的分布式存储，也就是说每台 Redis 节点上存储不同的内容。 作者：猴哥 链接：[https://juejin.im/post/5ed51964e51d457889261c13](https://juejin.im/post/5ed51964e51d457889261c13) 来源：掘金 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。2 年前回复![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentMore.png)
        举报

        ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentUnHeart.png)
        ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)
        ![](https://csdnimg.cn/release/blogv2/dist/pc/img/commentActiveHeart.png)


-   &lt;
-   1
-   \>

\[

..._集群_的_区别_\_u014761412 的博客\__redis**哨兵**和**集群**区别_

]([https://blog.csdn.net/u014761412/article/details/90408173](https://blog.csdn.net/u014761412/article/details/90408173))

1-3

\[

三、_集群_ 即使使用_哨兵_,_redis_每个实例也是全量存储, 每个_redis_存储的内容都是完整的数据, 浪费内存且有木桶效应。为了最大化利用内存, 可以采用_集群_, 就是分布式存储。即每台_redis_存储不同的内容, ...

]([https://blog.csdn.net/u014761412/article/details/90408173](https://blog.csdn.net/u014761412/article/details/90408173))

\[

..._哨兵_模式、_集群_模式\_SiyualChen 的博客\__redis\_\_集群_模式...

]([https://blog.csdn.net/weixin_40245787/article/details/87858380](https://blog.csdn.net/weixin_40245787/article/details/87858380))

1-4

\[

_Redis_-benchmark: 性能测试工具, 可以在自己本子运行, 看看自己本子性能如何 (服务启动起来后执行) _Redis_-check-aof: 修复有问题的 AOF 文件, 持久化 _Redis_-check-dump: 修复有问题的 dump.rdb 文件 _Redis_-sentinel:_Redis\_\_集群_使用,_哨兵_ ...

]([https://blog.csdn.net/weixin_40245787/article/details/87858380](https://blog.csdn.net/weixin_40245787/article/details/87858380))

\[

_Redis_必学（二）_redis\_\_主从_模式_和\_\_哨兵_模式

]([https://blog.csdn.net/u012133048/article/details/88558657](https://blog.csdn.net/u012133048/article/details/88558657))

[u012133048 的博客](https://blog.csdn.net/u012133048)

03-16 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 1 万 + 

\[

1 概述 一般的文档，都把_redis_的_集群_方式分成三种：_主从_、_哨兵_、_集群_（这里的_集群_只是广义_集群_的一种）。但是这么分类很不严谨，_哨兵_模式，单独使用是没有意义的，_哨兵_的作用有两个： 监控：监控主节点_和_从节点是否正常运行 提醒：当被监控的某个_Redis_节点出现问题时, _哨兵_(sentinel) 可以通过 API 向管理员或者其他应用程序发送通知。 故障迁移：主数据库出现故障时自动将从数据库转换为主数...

]([https://blog.csdn.net/u012133048/article/details/88558657](https://blog.csdn.net/u012133048/article/details/88558657))

\[

关于_redis_的_主从_、_哨兵_、_集群_

]([https://blog.csdn.net/weixin_33997389/article/details/93164185](https://blog.csdn.net/weixin_33997389/article/details/93164185))

[weixin_33997389 的博客](https://blog.csdn.net/weixin_33997389)

09-09 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 850 

\[

关于_redis\_\_主从_、_哨兵_、_集群_的介绍网上很多，这里就不赘述了。 一、_主从_ 通过持久化功能，_Redis_保证了即使在服务器重启的情况下也不会损失（或少量损失）数据，因为持久化会把内存中数据保存到硬盘上，重启会从硬盘上加载数据。 。但是由于数据是存储在一台服务器上的，如果这台服务器出现硬盘故障等问题，也会导致数据丢失。为了避免单点故障，通常的做法是将数据库复制多个副本以部署在不同的服...

]([https://blog.csdn.net/weixin_33997389/article/details/93164185](https://blog.csdn.net/weixin_33997389/article/details/93164185))

\[

_redis_ _哨兵_模式 cluster 模式_区别_\__Redis_系列 _主从_库模式...

]([https://blog.csdn.net/weixin_39955700/article/details/111172924](https://blog.csdn.net/weixin_39955700/article/details/111172924))

1-6

\[

_哨兵_与没一个主库_和_从库相连接, 并且_哨兵_节点之间行成_集群_。 监控 _哨兵_与没一个_Redis_节点相连接并通过 PING 命令检测它自己_和_主、从库的网络连接情况, 用来判断实例的状态。没一个_Redis_节点的状态可以分为” 主观下线 “_和_” 客观下线 “。

]([https://blog.csdn.net/weixin_39955700/article/details/111172924](https://blog.csdn.net/weixin_39955700/article/details/111172924))

\[

java 技术 -- _Redis\_\_主从_复制、_哨兵**和**集群_这三者原理_和\_\_区别_

]([https://blog.csdn.net/qq591009234/article/details/105414266](https://blog.csdn.net/qq591009234/article/details/105414266))

1-1

\[

4\._Redis_ _主从_复制、_哨兵**和**集群_这三个有什么_区别_ (1)_主从_模式:&lt;1> 读写分离, 备份, 一个 Master 可以有多个 Slaves (2)_哨兵\_sentinel:&lt;1> 监控, 自动转移,_哨兵_发现主服务器挂了后, 就会从 slave 中重新选举一个主服务器 (3)_集群\_:&lt;1> 为了解决...

]([https://blog.csdn.net/qq591009234/article/details/105414266](https://blog.csdn.net/qq591009234/article/details/105414266))

\[

_Redis\_\_主从_、_哨兵_、_集群_

]([https://blog.csdn.net/zhaoqingquanajax/article/details/118061049](https://blog.csdn.net/zhaoqingquanajax/article/details/118061049))

[zhaoqingquanajax 的博客](https://blog.csdn.net/zhaoqingquanajax)

06-20 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 106 

\[

文章目录_Redis\_\_主从_复制_哨兵_模式分片_集群_ _Redis\_\_主从_复制 概念 _主从_复制，是指将一台_Redis_服务器的数据，复制到其他_Redis_服务器。前者称为主节点（master/leader），后者称为从结点（slave/follower）；数据的复制是单向的，只能由主节点到从结点。Master 以写为主，Slave 以读为主，实现读写分离。 默认情况下，每台_Redis_服务都是主节点，且一个主节点可以有多个从结点，但一个从结点只能由一个主节点。 _主从_复制 数据冗余：_主从_复制实现了数据的热备份，是持久化之外的一种数据

]([https://blog.csdn.net/zhaoqingquanajax/article/details/118061049](https://blog.csdn.net/zhaoqingquanajax/article/details/118061049))

\[

_redis_从单点、_主从_、_哨兵_、到_集群_的总结（阶段性总结）

]([https://blog.csdn.net/u012045045/article/details/86623691](https://blog.csdn.net/u012045045/article/details/86623691))

[u012045045 的博客](https://blog.csdn.net/u012045045)

01-24 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 7268 

\[

我最近看项目：发现我们当前项目用的_redis_是_主从_，但是跟单点其实没有什么_区别_，因为我们在应用层面没有做读写分离，所以其实从服务器只是做了一个_主从_复制的工作，其他的什么都没有做。 那么如果我们的系统升级，用户量上升，那么一主一从可能扛不住那么大的压力，可能需要一主多从做备机，那么假如主服务器宕机了，选举哪台从服务器做主呢？这就是一个问题，需要一个第三个人来解决，所以我查了一下，_哨兵_模式可以解决这...

]([https://blog.csdn.net/u012045045/article/details/86623691](https://blog.csdn.net/u012045045/article/details/86623691))

\[

_Redis\_\_哨兵_、复制、_集群_的设计原理, 以及_区别_\_似非。的博客

]([https://blog.csdn.net/weixin_44980838/article/details/105041602](https://blog.csdn.net/weixin_44980838/article/details/105041602))

11-19

\[

_Redis_ _主从_复制、_哨兵**和**集群_这三个有什么_区别_ 1._主从_模式: 读写分离, 备份, 一个 Master 可以有多个 Slaves。 2._哨兵\_sentinel: 监控, 自动转移,_哨兵\_发现主服务器挂了后, 就会从 slave 中重新选举一个主服务器。

]([https://blog.csdn.net/weixin_44980838/article/details/105041602](https://blog.csdn.net/weixin_44980838/article/details/105041602))

\[

Linux 下 _Redis\_\_集群_搭建详解（_主从_+_哨兵_）

]([https://blog.csdn.net/xch_yang/article/details/104019552](https://blog.csdn.net/xch_yang/article/details/104019552))

[程序员大佬超的博客](https://blog.csdn.net/xch_yang)

01-17 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 1 万 + 

\[

前言 由于项目需要，搭建了一个 _Redis_ 服务器_集群_，实现了_主从_配置_和_容灾部署，使得主机出现故障时，可自动进行容灾切换，下面就详细讲解一下如何利用 _Redis_ 来实现。 文章重点 1、_Redis_ 入门简介 2、_Redis_ 安装部署 3、_Redis_ _集群_整体架构 4、_Redis_ _主从_配置及数据同步 5、_Redis_ _哨兵_模式搭建 一、_Redis_ 入门简介 _Redis_（\_Re_mote \_Di_ctiona...

]([https://blog.csdn.net/xch_yang/article/details/104019552](https://blog.csdn.net/xch_yang/article/details/104019552))

\[

七、_Redis\_\_哨兵_模式_和_高可用_集群_模式比较

]([https://blog.csdn.net/zwj1030711290/article/details/116232584](https://blog.csdn.net/zwj1030711290/article/details/116232584))

[zwj1030711290 的 CSDN](https://blog.csdn.net/zwj1030711290)

04-28 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 993 

\[

一、_哨兵_模式 在_redis_3.0 以前的版本要实现_集群_一般是借助_哨兵_sentinel 工具来监控 master 节点的状态，如果 master 节点异常，则会做_主从_切换，将某一台 slave 作为 master，_哨兵_的配置略微复杂，并且性能_和_高可用性等各方面表现一般，特别是在_主从_切换的瞬间存在访问瞬断的情况，而且_哨兵_模式只有一个主节点对外提供服务，没法支持很高的并发，且单个主节点内存也不宜设置得过大，否则会导致持久化文件过大，影响数据恢复或_主从\_同步的效率 搭建可以参考：[https://www.suibibk.com/t](https://www.suibibk.com/t)

]([https://blog.csdn.net/zwj1030711290/article/details/116232584](https://blog.csdn.net/zwj1030711290/article/details/116232584))

\[

_redis_中_主从_、_哨兵**和**集群_这三个有什么_区别_

最新发布

]([https://blog.csdn.net/qq_41589293/article/details/118411491](https://blog.csdn.net/qq_41589293/article/details/118411491))

[小旋风\_亮宇](https://blog.csdn.net/qq_41589293)

07-02 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 227 

\[

_主从_模式是最简单的实现高可用的方案，核心就是_主从_同步。 _哨兵_(sentinel) 的功能比单纯的_主从_架构全面的多了，它具备自动故障转移、_集群_监控、消息通知等功能 如果说依靠_哨兵_可以实现_redis_的高可用，如果还想在支持高并发同时容纳海量的数据，那就需要_redis\_\_集群_。_redis\_\_集群_是_redis_提供的分布式数据存储方案，_集群_通过数据分片 shar_di_ng 来进行数据的共享，同时提供复制_和\_故障转移的功能。 ...

]([https://blog.csdn.net/qq_41589293/article/details/118411491](https://blog.csdn.net/qq_41589293/article/details/118411491))

\[

_redis_3.0_集群\_特性

]([https://itguang.blog.csdn.net/article/details/76096279](https://itguang.blog.csdn.net/article/details/76096279))

[只有变秃 才能变强](https://blog.csdn.net/itguangit)

07-25 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 1932 

\[

_redis_3.0_集群_特性_主从_复制（读写分离）_主从_复制的好处有 2 点： 1、 避免\_redis_单点故障 2、 构建读写分离架构，满足读多写少的应用场景 设置_主从_创建 6379、6380、6381 目录，分别将安装目录下的_redis_.conf 拷贝到这三个目录下。分别进入这三个目录，分别修改配置文件，将端口分别设置为：6379（Master）、6380（Slave）、6381（Slave）。同时要设置

]([https://itguang.blog.csdn.net/article/details/76096279](https://itguang.blog.csdn.net/article/details/76096279))

\[

【_集群_】_Redis_的_哨兵_模式_和\_\_集群_模式

]([https://blog.csdn.net/weixin_30396699/article/details/97334176](https://blog.csdn.net/weixin_30396699/article/details/97334176))

[weixin_30396699 的博客](https://blog.csdn.net/weixin_30396699)

06-27 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 143 

\[

_哨兵_模式 _哨兵_模式是_redis_高可用的实现方式之一 使用一个或者多个_哨兵_(Sentinel) 实例组成的系统，对_redis_节点进行监控，在主节点出现故障的情况下，能将从节点中的一个升级为主节点，进行故障转义，保证系统的可用性。 _哨兵_们是怎么感知整个系统中的所有节点 (主节点 / 从节点 /_哨兵_节点) 的 ...

]([https://blog.csdn.net/weixin_30396699/article/details/97334176](https://blog.csdn.net/weixin_30396699/article/details/97334176))

\[

_redis_ _哨兵_模式 cluster 模式_区别_\__Redis\_\_主从_复制及_哨兵_模式、_集群_模式实现的优缺点总结

]([https://blog.csdn.net/weixin_39974932/article/details/110515602](https://blog.csdn.net/weixin_39974932/article/details/110515602))

[weixin_39974932 的博客](https://blog.csdn.net/weixin_39974932)

11-26 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 3143 

\[

一、_主从_复制 1 概念_Redis_多副本，采用_主从_(_re_plication) 部署结构，相较于单副本而言最大的特点就是_主从_实例间数据实时同步，并且提供数据持久化_和_备份策略。_主从_实例部署在不同的物理服务器上，根据公司的基础环境配置，可以实现同时对外提供服务_和_读写分离策略。一般来说，要将\_Redis_运用于工程项目中，只使用一台_Redis_是万万不能的，原因如下： 从结构上，单个_Redis_服务器会发生单点故障，并...

]([https://blog.csdn.net/weixin_39974932/article/details/110515602](https://blog.csdn.net/weixin_39974932/article/details/110515602))

\[

_Redis_系列 - _主从_库模式、_哨兵\_\_和_分片_集群_

]([https://blog.csdn.net/u011485472/article/details/109348122](https://blog.csdn.net/u011485472/article/details/109348122))

[u011485472 的博客](https://blog.csdn.net/u011485472)

10-29 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 382 

\[

_Redis_系列 - _主从_库模式、_哨兵\_\_和_分片_集群_ _主从_库模式 _Redis_的高可靠性主要包括两方面： 数据尽量少丢失：RDB & AOF 机制 服务尽量少中断：增加副本冗余 _主从_模式 _Redis_提供了_主从_库模式，增加冗余的副本来提高_Redis\_\_集群_的高可靠性。_主从_库之间采用读写分离的方式，写请求只能在主库，读请求在_主从_库都可以完成。 读操作：主库、从库 写操作：主库 --> 主库写完后同步从库 写请求为什么只能在主库上，若从库_和_主库上都可以进行读写会发生什么？ 1. 若有 3 个写请求

]([https://blog.csdn.net/u011485472/article/details/109348122](https://blog.csdn.net/u011485472/article/details/109348122))

\[

_redis_的单机、_哨兵_、_集群_模式对比

]([https://blog.csdn.net/qq_44703805/article/details/109223888](https://blog.csdn.net/qq_44703805/article/details/109223888))

[qq_44703805 的博客](https://blog.csdn.net/qq_44703805)

10-22 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 1228 

\[

_redis_作为集中式的数据库中间件，它具有持久化的能力，但是需要容许少量数据的丢失。 单机版 生成环境不建议使用，有单点故障风险。一旦单台结点出现故障，可能会导致整个服务不可用 sentinal_哨兵_模式 另外引入一台结点_redis_ sentinal（_哨兵_），会与其他的_redis_结点保持长连接（心跳机制），它实时地感应到其他_redis_结点的状态，客户端进行访问时，会向_redis_ sentinal 询问可用的_redis_服务结点，_redis_ sentinal 会返回可用的_redis_结点的信息。客户端获取到信息后再去访

]([https://blog.csdn.net/qq_44703805/article/details/109223888](https://blog.csdn.net/qq_44703805/article/details/109223888))

\[

_Redis\_\_主从_与_集群_

]([https://blog.csdn.net/qq_41901047/article/details/104989834](https://blog.csdn.net/qq_41901047/article/details/104989834))

[qq_41901047 的博客](https://blog.csdn.net/qq_41901047)

03-20 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 261 

\[

一、_Redis_ _集群_概述 _Redis_ _主从_复制 到 目前 为止，我们所学习的 _Redis_ 都是单机版的，这也就意味着一旦我们所依赖的 _Redis_ 服务宕机了，我们的主流程也会受到一定的影响，这当然是我们不能够接受的。 所以一开始我们的想法是：搞一台备用机。这样我们就可以在一台服务器出现问题的时候切换动态地到另一台去： 幸运的是，两个节点数据的同步我们可以使用 _Redis_ 的_主从_同...

]([https://blog.csdn.net/qq_41901047/article/details/104989834](https://blog.csdn.net/qq_41901047/article/details/104989834))

\[

_redis_系列之——高可用（_主从_、_哨兵_、_集群_）

]([https://blog.csdn.net/wuxiaolongah/article/details/107306679](https://blog.csdn.net/wuxiaolongah/article/details/107306679))

[诸葛小猿](https://blog.csdn.net/wuxiaolongah)

07-12 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 1 万 + 

\[

_redis_系列之——高可用（_主从_、_哨兵_、_集群_） 所谓的高可用，也叫 HA（High Availability），是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。 如果在实际生产中，如果_redis_只部署一个节点，当机器故障时，整改服务都不能提供服务了。这就是我们常说的单点故障。 如果_redis_部署了多台，当一台或几台故障时，整个系统依然可以对外提供服务，这样就提高了服务的可用性。 今天我们就聊聊_redis_高可用的三种模式：_主从_模式，_哨兵_模式，_集群_模式。 一、_主从_模式 一

]([https://blog.csdn.net/wuxiaolongah/article/details/107306679](https://blog.csdn.net/wuxiaolongah/article/details/107306679))

\[

mysql _redis\_\_集群_ 同步\__redis**集群**和**redis**主从_同步的_区别_

]([https://blog.csdn.net/weixin_29092787/article/details/113910241](https://blog.csdn.net/weixin_29092787/article/details/113910241))

[weixin_29092787 的博客](https://blog.csdn.net/weixin_29092787)

02-07 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 224 

\[

很多人认为_redis\_\_集群_就是_redis\_\_主从_同步，其实_redis\_\_集群_跟_redis\_\_主从_同步的机制完全不一样。1、_redis\_\_集群_包含_主从_同步：假如你配置了 6 个节点的_redis_-server 做_集群_，那么使用创建_集群_命令后，会自动指定其中三台为 master(主_redis_)，其他三台为 slave(从_redis_)。2、_redis\_\_主从_同步: 当 master(主_redis_) 发生改变后，slave(从_redis_) 会自动...

]([https://blog.csdn.net/weixin_29092787/article/details/113910241](https://blog.csdn.net/weixin_29092787/article/details/113910241))

\[

_Redis**集群**主从_复制（一主两从）搭建配置教程【Windows 环境】

热门推荐

]([https://aflyun.blog.csdn.net/article/details/79427606](https://aflyun.blog.csdn.net/article/details/79427606))

[阿飞云](https://blog.csdn.net/u010648555)

03-03 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 2 万 + 

\[

如何学会在合适的场景使用合适的技术方案，这值得思考。 由于本地环境的使用，所以搭建一个本地的_Redis\_\_集群_，本篇讲解_Redis\_\_主从_复制_集群_的搭建，使用的平台是 Windows，搭建的思路_和\_Linux 上基本一致！ （精读阅读本篇可能花费您 15 分钟，略读需 5 分钟左右） \_Redis\_\_主从_复制简单介绍 为了使得_集群_在一部分节点下线或者无法与_集群_的大多数节点进行通讯的情况下， 仍然可以正常运...

]([https://aflyun.blog.csdn.net/article/details/79427606](https://aflyun.blog.csdn.net/article/details/79427606))

\[

_redis_ _哨兵_与_集群_小结

]([https://blog.csdn.net/gsc1456/article/details/102466763](https://blog.csdn.net/gsc1456/article/details/102466763))

[gsc1456 的博客](https://blog.csdn.net/gsc1456)

10-09 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png)
 76 

\[

日常学习小结，不计格式、不计内容，只做笔记之用，若有不对请留言赐教！！！ 1、产生背景 _哨兵_ _redis\_\_主从_复制结构可以实现读写分离、数据备份、宕机恢复等功能，但主节点发生故障，需要故障转移时，需要人工干预。 应用方无法感知_redis_主节点变化，主节点发生故障，应用方仍会继续向主节点发送数据。 人工干预会导致实时性_和_准确性无法保证。 为此需要自动化故障转移策略 – _哨兵_ _集群_ 当_redis_中...

]([https://blog.csdn.net/gsc1456/article/details/102466763](https://blog.csdn.net/gsc1456/article/details/102466763))

### 目录

1.  [主从模式 (master-slave)](#t0)
2.  [哨兵模式 (sentinel)](#t1)
3.  [集群模式 （cluster）](#t2)
4.  [Redis 持久化](#t3)

    [https://blog.csdn.net/u014527619/article/details/88232178](https://blog.csdn.net/u014527619/article/details/88232178)
