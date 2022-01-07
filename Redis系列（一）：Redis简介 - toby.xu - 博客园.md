# Redis系列（一）：Redis简介 - toby.xu - 博客园
## 一、Redis 概述

　　Redis 是一个开源 (遵循 BSD 协议)Key-Value 数据结构的内存存储系统，用作数据库、缓存和消息代理。它支持 5 种数据结构：字符串 string、哈希 hash、列表 list、集合 set 和有序的集合 sorted-set。Redis 支持 Lua 脚本，哨兵机制和集群实现高可用。适用场景：缓存、投票、抽奖、分布式 session、排行榜、计数、队列、发布订阅等；具体介绍见**[Redis 官网](https://redis.io/)**。

## 二、Redis 安装

　　**①** 下载地址：[https://redis.io/download](https://redis.io/download)

　　**②**  安装 gcc：yum install gcc

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021205625508-533094371.png)

 　　**③** 把第一步下载好的 redis‐5.0.2.tar.gz 上传到服务器的 / root/svr/packages 目录或者直接 cd /root/svr/packages 然后执行：wget [http://download.redis.io/releases/redis-5.0.2.tar.gz](http://download.redis.io/releases/redis-5.0.2.tar.gz)

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021210426124-161653964.png)

 　　**④** 执行 cp redis‐5.0.2.tar.gz ../

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021210809540-552865683.png)

 　　**⑤** cd /root/svr 然后执行：tar -xvf redis‐5.0.2.tar.gz：

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021211028226-586433620.png)

 　　cd redis‐5.0.2：

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021211130530-750609649.png)

 　　**⑥** 执行：make install PREFIX=/root/svr/redis-5.0.2

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021211519756-1144404429.png)

 　　**⑦** 启动 redis 执行：bin/redis-server ../redis.conf （**注意：如果要后台启动需要把 redis.conf 配置里面的 daemonize 改为 yes**）

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021211902300-164640938.png)

 　　**⑧** 验证是否启动成功 ps -ef|grep redis

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021212028655-1168653366.png)

 　　**⑨** 进去 redis 客户端：bin/redis-cli

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021212148030-2096215386.png)

 　　**⑩** 退出客户端：quit

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191021212238973-957314639.png)

##  三、redis.conf 主要配置详解

| 参数 | 解释 |
| bind | 指定 Redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求 |
| port | 监听端口，默认 6379 |
| timeout | 设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接 |
| daemonize | 默认情况下，redis 不是在后台运行的，如果需要在后台运行，把该项的值更改为 yes |
| loglevel | log 等级分为 4 级，debug, verbose, notice, 和 warning。生产环境下一般开启 notice |
| logfile | 配置 log 文件地址，默认使用标准输出，即打印在命令行终端的窗口上 |
| save | save <seconds> <changes>比如 save 60 10000 意思 60 秒（1 分钟）内至少 10000 个 key 值改变（则进行数据库保存 -- 持久化 rdb） |
| dbfilename | rdb 文件的名称 |
| dir | 数据目录，2 种持久化 rdb、aof 文件就在这个目录 |
| replicaof | replicaof <masterip> <masterport>：该配置是主从的配置表示该 redis 实例是 masterip：masterport 的从节点 |
| masterauth | master 连接密码 |
| replica-serve-stale-data | 

当 slave 跟 master 失去连接或者正在同步数据，slave 有两种运行方式：

1) 如果 replica-serve-stale-data 设置为 yes(默认设置)，slave 会继续响应客户端的请求。

2) 如果 replica-serve-stale-data 设置为 no，除去指定的命令之外的任何请求都会返回一个错误”SYNC with master in progress”

 \|
| replica-read-only | 是否设置 slave 只读 |
| repl-diskless-sync | 同步策略: 磁盘或 socket，默认磁盘方式 |
| repl-diskless-sync-delay | 如果非磁盘同步方式开启，可以配置同步延迟时间，以等待 master 产生子进程通过 socket 传输 RDB 数据给 slave。默认值为 5 秒，设置为 0 秒则每次传输无延迟 |
| repl-ping-replica-period | slave 根据指定的时间间隔向 master 发送 ping 请求。默认 10 秒 |
| repl-timeout | 同步的超时时间 |
| repl-disable-tcp-nodelay | 是否在 slave 套接字发送 SYNC 之后禁用 TCP_NODELAY |
| repl-backlog-size | 设置数据备份的 backlog 大小 |
| repl-backlog-ttl | slave 断开开始计时多少秒后，backlog 缓冲将会释放 |
| replica-priority | slave 的优先级，当 master 挂了，优先级数字小的 salve 会优先考虑提升为 master，0 作为一个特殊的优先级，标识这个 slave 不能作为 master |
| requirepass | 客户端在处理任何命令时都要密码验证 |
| rename-command | 命令重命名，可以给危险命令改变名字 |
| maxclients | 设置最多同时连接的客户端数量，默认这个限制是 10000 个客户端。 |
| maxmemory | 设置最大内存，一旦内存使用达到最大内存，redis 会根据选定的回收策略（maxmemmory-policy）删除 key |
| maxmemory-policy | 

最大内存策略：如果达到内存限制了，redis 如何选择删除 key:  
1)volatile-lru -> 根据 LRU 算法删除设置过期时间的 key  
2)allkeys-lru -> 根据 LRU 算法删除任何 key  
3)volatile-random -> 随机移除设置过过期时间的 key  
4)allkeys-random -> 随机移除任何 key  
5)volatile-ttl -> 移除即将过期的 key(minor TTL)  
6)noeviction -> 不移除任何 key，只返回一个写错误

 \|
| maxmemory-samples | 

设置样本量的个数

 \|
| appendonly | 

是否开启 AOF，如果开启那么在启动时 Redis 将加载 AOF 文件，它更能保证数据的可靠性，aof 的文件内容就是 RESP 协议

 \|
| appendfilename | 

AOF 文件名（默认："appendonly.aof"）

 \|
| appendfsync | 

配置 Redis 多久才将数据 fsync 到磁盘一次  
Redis 支持三种不同的模式：  
1)no：不要立刻刷，只有在操作系统需要刷的时候再刷。比较快。  
2)always：每次写操作都立刻写入到 aof 文件。慢，但是最安全。  
3)everysec：每秒写一次。折中方案。

 \|
| auto-aof-rewrite-percentage | 

自动重写 AOF 文件。如果 AOF 日志文件增大到指定百分比，默认 100。Redis 能够通过 BGREWRITEAOF 自动重写 AOF 日志文件

 \|
| auto-aof-rewrite-min-size | 

自动重写 AOF 文件。如果 AOF 日志文件到达最小的指定大小，默认 64mb

 \|
| aof-use-rdb-preamble | 

Redis 4.0 之后配置混合持久化，需要配置 aof-use-rdb-preamble yes

 \|
| lua-time-limit | 

Lua 脚本的最大执行时间，单位为毫秒

 \|
| cluster-enabled | 

是否开启集群 cluster-enabled yes

 \|
| cluster-config-file | 

redis 自动生成集群配置信息的文件名

 \|
| cluster-node-timeout | 

集群节点超时毫秒数。超时的节点将被视为不可用状态。

 \|
| aof-rewrite-incremental-fsync | 

当一个子进程重写 AOF 文件时，如果配置 aof-rewrite-incremental-fsync yes，则文件每生成 32M，数据会被同步

 \|
| rdb-save-incremental-fsync | 

当 redis 保存 RDB 文件时，如果启用了以下选项，每生成 32MB 数据，文件将被 fsync 到磁盘

 \|

## 四、Redis 五种数据结构

![](https://img2018.cnblogs.com/blog/1761778/201910/1761778-20191022091931196-50291973.png)

### 　　1、String

#### 　　1.1 字符串操作

![](https://common.cnblogs.com/images/copycode.gif)

SET      key  value                            // 设置字符串键值对
MSET     key  value \[key value ...] // 批量设置字符串键值对
SETNX    key  value // 设置字符串键值对，当 key 存在就不做处理直接返回 0，否则跟 set 命令一样
GET      key // 获取一个字符串键值
MGET     key  \[key ...] // 批量获取字符串键值
DEL      key  \[key ...] // 删除一个键
EXPIRE   key  seconds // 设置一个键的过期时间 (秒)

![](https://common.cnblogs.com/images/copycode.gif)

#### 　　1.2 原子操作

INCR     key                                    // 将 key 中储存的数字值加 1
DECR     key // 将 key 中储存的数字值减 1
INCRBY   key  increment // 将 key 所储存的值加上 increment
DECRBY   key  decrement // 将 key 所储存的值减去 decrement

### 　　2、Hash

#### 　　2.1 常用操作

![](https://common.cnblogs.com/images/copycode.gif)

HSET     key  field  value                       // 设置哈希表 key 中的字段 field 的值设为 value HSETNX   key  field  value // 设置哈希表 key 中的字段 field 的值设为 value，当 key 存在不做处理，返回 0，否则跟 hset 命令一样
HMSET    key  field  value \[field value ...] // 批量设置哈希表 key 中的字段 field 的值设为 value HGET     key  field // 获取哈希表 key 对应的 field 字段的值
HMGET    key  field  \[field ...] // 批量获取哈希表 key 中多个 field 字段的值
HDEL     key  field  \[field ...] // 删除哈希表 key 中的 field 字段
HLEN     key // 返回哈希表 key 中 field 的数量
HGETALL  key // 返回哈希表 key 中所有的键值
HINCRBY  key  field  increment // 为哈希表 key 中 field 键的值加上增量 increment

![](https://common.cnblogs.com/images/copycode.gif)

### 　　3、List

#### 　　3.1 常用操作

![](https://common.cnblogs.com/images/copycode.gif)

LPUSH   key  value \[value ...]                   // 将一个或多个值 value 插入到 key 列表的表头 (最左边)
RPUSH   key  value \[value ...] // 将一个或多个值 value 插入到 key 列表的表尾 (最右边)
LPOP    key // 移除并返回 key 列表的头元素
RPOP    key // 移除并返回 key 列表的尾元素
LRANGE  key  start  stop // 返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定
BLPOP   key  \[key ...]  timeout // 从 key 列表表头弹出一个元素，若列表中没有元素，阻塞等待 timeout 秒, 如果 timeout=0, 一直阻塞等待
BRPOP   key  \[key ...]  timeout // 从 key 列表表尾弹出一个元素，若列表中没有元素，阻塞等待 timeout 秒, 如果 timeout=0, 一直阻塞等待

![](https://common.cnblogs.com/images/copycode.gif)

### 　　4、Set

#### 　　4.1 常用操作

![](https://common.cnblogs.com/images/copycode.gif)

SADD         key  member  \[member ...]            // 往集合 key 中存入元素，元素存在则忽略，key 不存在则新建
SREM         key  member  \[member ...] // 从集合 key 中删除元素
SMEMBERS     key // 获取集合 key 中所有元素
SCARD        key // 获取集合 key 的元素个数
SISMEMBER    key  member // 判断 member 元素是否存在于集合 key 中
SRANDMEMBER  key  \[count] // 从集合 key 中选出 count 个元素，元素不从 key 中删除
SPOP         key  \[count] // 从集合 key 中选出 count 个元素，元素从 key 中删除

![](https://common.cnblogs.com/images/copycode.gif)

#### 　　4.2 运算操作

![](https://common.cnblogs.com/images/copycode.gif)

SINTER       key  \[key ...]                       // 交集运算
SINTERSTORE  destination  key  \[key ..] // 将交集结果存入新集合 destination 中
SUNION       key  \[key ..] // 并集运算
SUNIONSTORE  destination  key  \[key ...] // 将并集结果存入新集合 destination 中
SDIFF        key  \[key ...] // 差集运算
SDIFFSTORE   destination  key  \[key ...] // 将差集结果存入新集合 destination 中

![](https://common.cnblogs.com/images/copycode.gif)

### 　　5、Sorted-Set

#### 　　5.1 常用操作

![](https://common.cnblogs.com/images/copycode.gif)

ZADD         key score member \[\[score member]...]  // 往有序集合 key 中加入带分值元素
ZREM         key member \[member ...] // 从有序集合 key 中删除元素
ZSCORE       key member // 返回有序集合 key 中元素 member 的分值
ZINCRBY      key increment member // 为有序集合 key 中元素 member 的分值加上 increment 
ZCARD        key // 返回有序集合 key 中元素个数
ZRANGE       key start stop \[WITHSCORES] // 正序获取有序集合 key 从 start 下标到 stop 下标的元素
ZREVRANGE    key start stop \[WITHSCORES] // 倒序获取有序集合 key 从 start 下标到 stop 下标的元素

![](https://common.cnblogs.com/images/copycode.gif)

#### 　　5.2 运算操作

ZUNIONSTORE  destkey numkeys key \[key ...]         // 并集计算
ZINTERSTORE  destkey numkeys key \[key ...] // 交集计算

## 五、Redis 核心原理

### 　　1、Redis 单线程为什么还能这么快

　　Redis 所有的数据都是在内存中，所有的运算都是内存级别的运算，而且单线程避免了多线程的切换性能损耗的问题。正因为 Redis 是单线程，所以要小心使用 Redis 指令，对于那些耗时的指令 (比如 keys)，一定要谨慎使用，一不小心就可能会导致 Redis 卡顿。

### 　　2、Redis 单线程为何能处理那么多的并发客户端连接

　　Redis 的 IO 多路复用：redis 利用 epoll 来实现 IO 多路复用，将连接信息和事件放到队列中，依次放到文件事件分派器，事件分派器将事件分发给事件处理器。（IO 多路复用在后续的 netty 系列里面详细讲解）

　　总结：Redis 是一个 Key-Value 数据结构的内存存储系统，他支持 5 种数据结构，分别是 String 结构、Hash 结构、List 结构、Set 结构、Sorted-Set 结构；Redis 支持 Lua 脚本，哨兵机制和集群实现高可用；介绍了安装 Redis 的步骤，详细的解释了 redis.conf 的主要配置，以及 Redis 的核心原理。 
 [https://www.cnblogs.com/toby-xu/p/11715704.html](https://www.cnblogs.com/toby-xu/p/11715704.html)
