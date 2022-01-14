# Redis集群模式详解 - YSOcean - 博客园
* * *

　　在上一篇博客我们介绍了 ------[Redis 哨兵 (Sentinel) 模式](https://www.cnblogs.com/ysocean/p/12290364.html), 哨兵模式主要是解决高可用问题, 在 master 节点宕机时, slave 节点能够自动切换成为 master 节点

　　本篇博客我们来介绍 Redis 的另外一种模式 ------ 集群模式.

　　**PS：我这里搭建演示的版本是 redis-5.0.5，这个版本对于集群搭建会有很大的简化，比如最常用的 redis-trib.rb 脚本功能已经集成到 redis-cli 工具中了，具体下面会详细介绍。** 

### 1、为什么需要集群?

　　**①、并发量**

　　通常来说, 单台 Redis 能够执行**10 万 / 秒**的命令, 这个并发基本上能够满足我们所有需求了, 但有时候比如做离线计算, 为了更快的得出结果, 有时候我们希望超过这个并发, 那这个时候单机就不满足我们需求了, 就需要集群了.

　　**②、数据量**

　　通常来说, 单台服务器的内存大概在 16G-256G 之间, 前面我们说 Redis 数据量都是存在内存中的, 那如果实际业务要保存在 Redis 的数据量超过了单台机器的内存, 这个时候最简单的方法是增加服务器内存, 但是单台服务器内存不可能无限制的增加, 纵向扩展不了了, 便想到如何进行横向扩展. 这时候我们就会想将这些业务数据分散存储在多台 Redis 服务器中, 但是要保证多台 Redis 服务器能够无障碍的进行内存数据沟通, 这也就是 Redis 集群.

### 2、数据分区方式

　　对于集群来说, 如何将原来单台机器上的数据拆分, 然后尽量均匀的分布到多台机器上, 这是我们创建集群首先要考虑的一个问题, 通常来说, 有如下两种数据分区方式.

　　**①、顺序分布**

　　比如我们有 100W 条数据, 有 3 台服务器, 我们可以将 100W/3 的结果分别存储到三台服务器上, 如下所示:

 　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219113027677-291096148.png)

　　特点：键值业务相关；数据分散，但是容易造成访问倾斜；支持顺序访问；支持批量操作

　　**②、哈希分布**

　　同样是 100W 条数据, 有 3 台服务器, 通过自定义一个哈希函数, 比如节点取余的方法, 余数为 0 的存在第一台服务器, 余数为 1 的存在第二台服务器, 余数为 2 的存储在第三台服务器. 如下所示:

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219113122731-1156066369.png)

　　特点：数据分散度高；键值分布与业务无关；不支持顺序访问；支持批量操作。

### **3、一致性哈希分布**

**问题：** 对于上面介绍的哈希分布，大家可以想一下，如果向集群中增加节点，或者集群中有节点宕机，这个时候应该怎么处理？

　　**①、增加节点**

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219122548809-1622152427.png)

 　　如上图所示，总共 10 个数据通过节点取余 hash(key)%/3 的方式分布到 3 个节点，这时候由于访问量变大，要进行扩容，由 3 个节点变为 4 个节点。

　　我们发现，如图所示，数据除了标红的 1 2 没有进行迁移，别的数据都要进行变动，达到了 80%，如果这时候并发很高，80% 的数据都要从下层节点（比如数据库）获取，会给下层节点造成很大的访问压力，这是不能接受的。

　　即使我们进行**翻倍扩容**，从 3 个节点增加到 6 个节点，其数据迁移也在 50% 左右。

　　**②、删除节点**

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219185204068-967184292.png)

　　上图其实不管是哪一个节点宕机，其数据迁移量都会超过 50%。基本上也是我们所不能接受的。

　　**那么如何使得集群中新增节点或者删除节点时，数据迁移量最少？——一致性哈希算法诞生。** 

PS：关于一致性哈希算法，我会另外写一篇博客进行详细介绍，这里只是大概介绍一下。

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219191534059-1951744286.png)

 　　假设有一个哈希环，从 0 到 2 的 32 次方，均匀的分成三份，中间存放三个节点，沿着顺时针旋转，从 Node1 到 Node2 之间的数据，存放在 Node2 节点上；从 Node2 到 Node3 之间的数据，存放在 Node3 节点上，依次类推。

　　假设 Node1 节点宕机，那么原来 Node3 到 Node1 之间的数据这时候改为存放到 Node2 节点上，Node2 到 Node3 之间数据保持不变，原来 Node1 到 Node2 之间的数据还是存放在 Node2 上，也就是只影响三分之一的数据，节点越多，影响数据越少。

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219192124649-191606285.png)

　　同理，假设增加一个节点，影响的数据甚至更少。

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219192227109-9495373.png)

 　　当然，实际业务中并不是你节点均匀分布，访问就会很平均，这时候容易造成访问倾斜的问题，这里就会引出虚拟节点的定义。我这里就不做详解了。

### 4、Redis Cluster 虚拟槽分区

　　Redis 集群数据分布没有使用一致性哈希分布，而是使用虚拟槽分区概念。

　　Redis 内部内置了序号 0-16383 个槽位，每个槽位可以用来存储一个数据集合，将这些槽位按顺序分配到集群中的各个节点。每次新的数据到来，会通过哈希函数 CRC16(key) 算出将要存储的槽位下标，然后通过该下标找到前面分配的 Redis 节点，最后将数据存储到该节点中。

　　具体情况如下图：（以集群有 3 个节点为例）

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200219213851498-507099931.png)

 　　至于为什么 Redis 不使用一致性哈希分布，而是虚拟槽分区。因为虚拟槽分区虽然没有一致性哈希那么灵活，但是 CRC16(key)%16384 已经分布很均匀了，并且对于后面节点增删操作起来也很方便。

### 5、原生搭建 Redis Cluster

　　集群以**三主三从**的模式来搭建。

#### ①、服务器列表

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220194010583-1216911180.png)

#### ②、配置各个节点参数

![](https://common.cnblogs.com/images/copycode.gif)

\#配置端口
port ${port}
\# 以守护进程模式启动
daemonize yes
\#pid 的存放文件
pidfile /var/run/redis\_${port}.pid
\# 日志文件名
logfile "redis\_${port}.log" #存放备份文件以及日志等文件的目录
dir "/opt/redis/data" #rdb 备份文件名
dbfilename "dump\_${port}.rdb" #开启集群功能
cluster-enabled yes
\# 集群配置文件，节点自动维护
cluster-config-file nodes-${port}.conf
\# 集群能够运行不需要集群中所有节点都是成功的
cluster-require-full-coverage no

![](https://common.cnblogs.com/images/copycode.gif)

　　配置完成后，通过 redis-server redis.conf 命令启动这六个节点。

　　启动之后，进程后面会有 cluster 的字样：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220202304911-776847031.png)

#### ③、建立各个节点通信

 　　这里有 6 个节点，我们只需要拉通 1 个节点和另外 5 个节点之间通信，那么每两个节点就能够通信了。命令如下：

redis-cli -h -p ${port1} -a ${password} cluster meet ${ip2}  ${port2} 

　　这里的 -a 参数表示该 Redis 节点有密码，如果没有可以不用加此参数。

　　实例中的 6 个节点，分别进行如下命令：

redis-cli -p 6379 -a 123 cluster meet 192.168.14.101 6382 redis-cli -p 6379 -a 123 cluster meet 192.168.14.102 6380 redis-cli -p 6379 -a 123 cluster meet 192.168.14.102 6383 redis-cli -p 6379 -a 123 cluster meet 192.168.14.103 6381 redis-cli -p 6379 -a 123 cluster meet 192.168.14.103 6384

　　执行完毕后，可以查看节点通信信息：

redis-cli -p 6379 -a 123 cluster nodes

　　结果如下：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220204303934-1662569257.png)

 　　或者执行如下命令：

redis-cli -p 6379 -a 123 cluster info

　　结果如下：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220204409071-26441393.png)

#### ④、分配槽位

　　由于我们是三主三从的架构，所以只需要对主服务器分配槽位即可。三个节点，分配序号为 0-16383 ，总共 16384 个槽位。

Node1:0~5460 Node2:5461~10922 Node3:10923~16383

　　分配槽位的命令如下：

redis-cli -p ${port} -a ${password} cluster addslots {${startSlot}..${endSlot}}

　　比如，对于 Node1 主节点，我们执行命令如下：

redis-cli -p 6379 -a 123 cluster addslots {0..5462}

　　另外两个节点对于上面的命令更改一下槽位数，然后查看集群信息：　　

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220213038086-1726063690.png)

　　查看 Node1 节点信息：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220212645343-639382864.png)

#### ⑤、主从配置

　　命令如下：

redis-cli -p ${port} -a {password} cluster replicate ${nodeId}

　　前面的 ${port} 表示从节点的端口，这里的 nodeId 表示主节点的 nodeId，如下：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220213659345-738320038.png)

 　　如果弄反了，会报如下错误：

(error) ERR To set a master the node must be empty and without assigned slots.

　　执行三条命令完毕后，查看节点信息：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200220214359351-1357072255.png)

 　　这时候，集群状态是成功了。

#### ⑥、测试

　　经过如上几步操作，集群搭建成功，我们通过如下命令进入客户端：

redis-cli -c -p ${port} -a {password}

　　注意：必须要加 -c 参数，否则进行键值对操作时会报如下错误：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200221105544054-1565661844.png)

 　　正确进入后，可以正确存值和取值。

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200221105631965-1289869897.png)

### 6、脚本搭建 Redis Cluster

 　　上面原生命令安装 Redis Cluster 走下来其实挺费劲的，在实际生产环境中，如果集群数量比较大，操作还是容易出错的。

　　不过 Redis 官方提供了一个安装集群的脚本，在 Redis 安装目录的 src 目录下——redis-trib.rb，使用该脚本可以快速搭建 Redis Cluster 集群。

　　**注意**：redis 版本在 5 之前的集群运行该脚本需要安装 ruby 环境，而 redis5.0 之后已经将 redis-trib.rb 脚本的功能全部集成到 redis-cli 之中了，所以如果当前版本是 Redis5，那么可以不用安装 ruby 环境。

　　下面我分别介绍这两种方法。

#### ①、Redis5 之前使用 redis-trib.rb 脚本搭建

　　redis-trib.rb 脚本使用 ruby 语言编写，所以想要运行次脚本，我们必须安装 Ruby 环境。安装命令如下：

yum -y install centos-release-scl-rh
yum -y install rh-ruby23  
scl enable rh-ruby23 bash
gem install redis

　　安装完成后，我们可以使用 ruby -v 查看版本信息。

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200221181647318-2083171766.png)

 　　Ruby 环境安装完成后。运行如下命令：

redis-trib.rb create --replicas 1 192.168.14.101:6379 192.168.14.102:6380 192.168.14.103:6381 192.168.14.101:6382 192.168.14.102:6383 192.168.14.103:6384

　　关于这个命令的解释下面会一起介绍。

#### ②、Redis5 版本集群搭建　

　　前面我们就说过，redis5.0 之后已经将 redis-trib.rb 脚本的功能全部集成到 redis-cli 中了，所以我们直接使用如下命令即可：

redis-cli -a ${password} --cluster create 192.168.14.101:6379 192.168.14.102:6380 192.168.14.103:6381 192.168.14.101:6382 192.168.14.102:6383 192.168.14.103:6384 --cluster-replicas 1

　　①、${password} 表示连接 Redis 的密码，通常整个集群我们要么不设置密码，要么设置成一样的。

　　②、后面的六个 ip:port，按照顺序，前面三个是主节点，后面三个是从节点，顺序不能错。

　　③、最后数字 1 表示一个主节点只有一个从节点。和前面的配置相对应。

### 7、集群扩容

　　这里新增两个端口分别是 6390、6391 的节点。其中 6391 节点是 6390 节点的从节点。

#### ①、配置新增节点文件

　　比如，我们将 6379 节点的配置文件 redis.conf 拷贝两份，然后将里面的配置文件里面的字符串 6379 分别替换成 6390 和 6391。

　　:**%s/6379/6390/g，:%s/6379/6391/g**

替换完成之后，分别启动这两个节点。

　　这时候这两个节点都不在集群当中，是两个孤儿节点。

#### ②、将新增主节点加入到集群中

　　命令如下：

redis-cli -p existing_port -a ${password} --cluster add-node new_host:new_port existing_host:existing_port

　　我这里是将新增的主节点 6390 添加到原来的集群中。

redis-cli -p 6379 -a 123 --cluster add-node 192.168.14.101:6390 192.168.14.101:6379

　　添加完毕后，这时候查看集群状态

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200222183438002-971520853.png)

 　　6390 节点已经存在集群中了，但是还没有分配槽位。

#### ③、为新增主节点分配槽位

　　分配命令如下：

redis-cli -p existing_port -a ${password} --cluster reshard existing_host:existing_port

　　后面的 existing_host:existing_port 表示原来集群中的任意一个节点，这个命令表示将源节点的一部分槽位分配个新增的节点。

　　在分配过程中，会出现如下几个提示：

![](https://common.cnblogs.com/images/copycode.gif)

\#后面的 2000 表示分配 2000 个槽位给新增节点
How many slots do you want to move (from 1 to 16384)? 2000 #表示接受节点的 NodeId, 填新增节点 6390 的
What is the receiving node ID? 64a0779c7baef78c8fd0f2bb6e73f29375e00133d
\# 这里填槽的来源，要么填 all，表示所有 master 节点都拿出一部分槽位分配给新增节点；
\# 要么填某个原有 NodeId，表示这个节点拿出一部分槽位给新增节点
Please enter all the source node IDs.
Type 'all' to use all the nodes as source nodes for the hash slots.
Type 'done' once you entered all the source nodes IDs.
Source node #1: all

![](https://common.cnblogs.com/images/copycode.gif)

　　分配成功后，我们查看节点信息：

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200222190838680-1831758053.png)

 　　我们发现已经给该节点分配了槽位。

#### ④、将新增的从节点添加到集群中

redis-cli -p 6379 -a 123 --cluster add-node 192.168.14.101:6391 192.168.14.101:6379

#### ⑤、建立新增节点的主从关系

　　命令如下：

redis-cli -p ${port} -a {password} cluster replicate ${nodeId}

　　前面的 ${port} 表示从节点的端口，这里的 nodeId 表示主节点的 nodeId。

#### ⑥、测试

　　查看节点信息，发现 4 主 4 从。

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200222193148506-1319733266.png)

 　　在 6379 节点新增一个字符串 (k4,v4)，然后到 6390 节点查看：

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200222193354897-1139119437.png)

 　　自此，大功告成。

### 8、集群收缩

　　这里我们将上一步添加的主从节点 6390 和 6391 从集群中移除。

#### ①、迁移待移除节点的槽位

　　移除之前的节点信息：

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200222193148506-1319733266.png)

redis-cli -p existing_port -a {Redis 登录密码} --cluster reshard --cluster-from {待移除的 NodeId} --cluster-to {接受移除节点的 NodeId} --cluster-slots {移除的槽位个数} existing_host:existing_port

　　比如，我这里要移除主节点 6390 的所有槽位，给 6379 节点。

redis-cli -p 6379 -a 123 --cluster reshard --cluster-from 4a0779c7baef78c8fd0f2bb6e73f29375e00133d --cluster-to 001a22b1edae6ea1699b753d193871824723f375 --cluster-slots 2000 192.168.14.101:6379

　　移除完后，查看节点信息，发现 6390 已经没有槽位了。

![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200223201225319-513091646.png)

####  ②、移除待删除主从节点

　　**注意：要首先移除从节点，然后再移除主节点，因为如果你先移除主节点，会触发集群的故障转移。** 

所以，我们应该先移除 6391 从节点，然后在移除 6390 主节点。移除命令如下：

redis-cli -p existing_port -a {Redis 登录密码} --cluster del-node host:port {待删除的 NodeId}

　　删除 6391 从节点：

redis-cli -p 6379 -a 123 --cluster del-node 192.168.14.101:6379 3622ec34956b624358722e6f4a2b762574d35bf0

　　删除 6390 主节点：

redis-cli -p 6379 -a 123 --cluster del-node 192.168.14.101:6379 4a0779c7baef78c8fd0f2bb6e73f29375e00133d

　　![](https://img2018.cnblogs.com/i-beta/1120165/202002/1120165-20200223202136963-1485783798.png) 
 [https://www.cnblogs.com/ysocean/p/12328088.html](https://www.cnblogs.com/ysocean/p/12328088.html)
