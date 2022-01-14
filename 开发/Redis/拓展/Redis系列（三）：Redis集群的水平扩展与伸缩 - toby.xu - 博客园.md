# Redis系列（三）：Redis集群的水平扩展与伸缩 - toby.xu - 博客园
## 一、Redis 集群的水平扩展

　　Redis3.0 版本以后，有了集群的功能，提供了比之前版本的哨兵模式更高的性能与可用性，但是集群的水平扩展却比较麻烦，接下来介绍下 Redis 高可用集群如何做水平扩展，在原集群的 6 个节点的基础上新增 2 个节点，由原来的 3 主 3 从变成 4 主 4 从，原先的 3 主 3 从部署详见**[Redis 系列（二）：Redis 高可用集群](https://www.cnblogs.com/toby-xu/p/11960971.html)**，如下图：

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130214226682-1980179140.png)

##  二、水平扩展具体操作

　　**①** 将 redis-5.0.2 文件夹拷贝到新的主机 192.168.160.154 上去，（1）scp -r /usr/local/redis-5.0.2 root@192.168.160.154:/usr/local/ 进去到 192.168.160.154 主机 （2）cd /usr/local

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130214918005-1447892361.png)

 　　**②** 新启动 2 个 redis 实例，然后检查是否启动成功 （1）/usr/local/redis-5.0.2/bin/redis-server /usr/local/redis-5.0.2/redis-cluster/700\*/redis.conf （2）ps -ef | grep redis 查看是否启动成功

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130215134054-1330767141.png)

 　　**③**查看 redis 集群的命令帮助 （1）cd /usr/local/redis-5.0.2 （2）bin/redis-cli --cluster help![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130215603942-1845745274.png)

![](https://common.cnblogs.com/images/copycode.gif)

create：创建一个集群 host1:port1 ... hostN:portN
call：可以执行 redis 命令
add-node：将一个节点添加到集群里，第一个参数为新节点的 ip:port，第二个参数为集群中任意一个已经存在的节点的 ip:port
del-node：移除一个节点
reshard：重新分片
check：检查集群状态

![](https://common.cnblogs.com/images/copycode.gif)

　　**④**使用 add-node 命令新增一个主节点 192.168.160.154:7001(master)，前面的 ip:port 为新增节点，后面的 ip:port 为集群中已存在节点（1）/usr/local/redis-5.0.2/bin/redis-cli --cluster add-node 192.168.160.154:7001 192.168.160.146:7001

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130221134601-397950265.png)

 　　**⑤**（1）连接任意一个客户端即可：./redis-cli -c -h -p (-a 访问服务端密码，-c 表示集群模式，指定 ip 地址和端口号）如：/usr/local/redis-5.0.2/bin/redis-cli -c -h 192.168.160.146 -p 700\* （2）进行验证： cluster info（查看集群信息）、cluster nodes（查看节点列表）

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130222402254-36775487.png)

 　　**⑥** 使用 redis-cli 命令为 192.168.160.154:7001 分配 slots 槽位，找到集群中的任意一个主节点，对其进行重新分片工作。（1）/usr/local/redis-5.0.2/bin/redis-cli --cluster reshard 192.168.160.146:7001

![](https://common.cnblogs.com/images/copycode.gif)

How many slots do you want to move (from 1 to 16384)? 4000 (ps: 需要多少个槽移动到新的节点上，自己设置，比如 4000 个 hash 槽)
What is the receiving node ID? 44b0bd6cf056af7dbbfa0dd9497def1cfc21eb6d
(ps: 把这 4000 个 hash 槽移动到哪个节点上去，需要指定节点 id)
Please enter all the source node IDs.
Type 'all' to use all the nodes as source nodes for the hash slots.
Type 'done' once you entered all the source nodes IDs.
Source node 1:all
(ps: 输入 all 为从所有主节点中分别抽取相应的槽数指定到新节点中，抽取的总槽数为 4000 个; 或者输入原节点 ID 然后输入 done，意思将输入的节点 ID，抽取的总槽数为 4000 个)
Do you want to proceed with the proposed reshard plan (yes/no)? yes
(ps: 输入 yes 确认开始执行分片任务)

![](https://common.cnblogs.com/images/copycode.gif)

　　**⑦**查看下最新的集群状态

 ![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130224505251-707888360.png)

 　　**⑧** 添加从节点 192.168.160.154:7002 到集群中去并查看集群状态

　　（1）/usr/local/redis-5.0.2/bin/redis-cli --cluster add-node 192.168.160.154:7002 192.168.160.146:7001

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130224724934-1437715629.png)

　　（2）cluster nodes

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130224804207-886172845.png)

　　（3）如上图所示，还是一个 master 节点，没有被分配任何的 hash 槽。我们需要执行 replicate 命令来指定当前节点 (从节点) 的主节点 id 为哪个, 首先需要连接新加的 192.168.160.154:7002 节点的客户端，然后使用集群命令进行操作，把当前的 192.168.160.154:7002(slave)节点指定到一个主节点下(这里使用之前创建的 192.168.160.154:7001 主节点)

　　（3-1）/usr/local/redis-5.0.2/bin/redis-cli -c -h 192.168.160.154 -p 7002

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130225435484-206846418.png)

　　（3-2）cluster replicate 44b0bd6cf056af7dbbfa0dd9497def1cfc21eb6d #后面这串 id 为 192.168.160.154:7001 的节点 id

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130225706634-1972451892.png)

 　　（3-2）cluster nodes

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130225743838-979759253.png)

##  三、水平伸缩具体操作

　　目的还原成原始集群，如下图：

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130231115497-861253827.png)

 　　**①** 删除 192.168.160.154:7002 从节点，用 del-node 删除从节点 192.168.160.154:7002，指定删除节点 ip 和端口，以及节点 id(192.168.160.154:7002 节点 id) 

　　（1）/usr/local/redis-5.0.2/bin/redis-cli --cluster del-node 192.168.160.154:7002 564963541c243365cbb20aed69e98048d21d68fd

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130231451108-2058158341.png)

 　　（2）cluster nodes

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130231600761-1443151188.png)

 　　**②** 删除 192.168.160.154:7001 主节点，这个步骤相对麻烦一些，因为主节点的里面是有分配了 slots 槽位，所以必须先把 192.168.160.154:7001 里的 slots 槽位放入到其他的可用主节点中去，然后再进行移除节点操作，不然会出现数据丢失问题 (目前只能把 master 的数据迁移到一个节点上，暂时做不了平均分配功能)

　　（1）/usr/local/redis-5.0.2/bin/redis-cli --cluster reshard 192.168.160.154:7001

![](https://common.cnblogs.com/images/copycode.gif)

How many slots do you want to move (from 1 to 16384)? 4000 (ps: 需要多少个槽移动到新的节点上，自己设置，比如 4000 个 hash 槽)
What is the receiving node ID? e7f80ba80749904838b6d779a0646e7f22313624
(ps: 把这 4000 个 hash 槽移动到哪个节点上去，需要指定节点 id)
Please enter all the source node IDs.
Type 'all' to use all the nodes as source nodes for the hash slots.
Type 'done' once you entered all the source nodes IDs.
Source node 1:44b0bd6cf056af7dbbfa0dd9497def1cfc21eb6d
Source node 1:done
(ps: 输入 all 为从所有主节点中分别抽取相应的槽数指定到新节点中，抽取的总槽数为 4000 个; 或者输入原节点 ID 然后输入 done，意思将输入的节点 ID，抽取的总槽数为 4000 个)
Do you want to proceed with the proposed reshard plan (yes/no)? yes
(ps: 输入 yes 确认开始执行分片任务)

![](https://common.cnblogs.com/images/copycode.gif)

　　（2）cluster nodes 已经成功的把 192.168.160.154:7001 主节点的数据迁移到 192.168.160.146:7001 上去了

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130232447904-1139465162.png)

 　　　（3）最后我们直接使用 del-node 命令删除 192.168.160.154:7001 主节点即可

 ![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130232710101-1151147311.png)

 　　（4）最后执行 cluster nodes

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130232749682-939056987.png) 
 [https://www.cnblogs.com/toby-xu/p/11964409.html](https://www.cnblogs.com/toby-xu/p/11964409.html)
