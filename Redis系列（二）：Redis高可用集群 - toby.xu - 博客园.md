# Redis系列（二）：Redis高可用集群 - toby.xu - 博客园
## 一、集群模式

　　Redis 集群是一个由多个主从（主从在[**Redis 系列（四）：** **Redis 持久化和主从复制原理**](https://www.cnblogs.com/toby-xu/p/11973477.html)中详细介绍，这里先有个概念 ）节点组成的高可用集群，它具有复制、高可用和分片等特性

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191129233417339-819003963.png)

##  二、集群部署

### 1、环境

　　3 台主机分别是：

　　192.168.160.146

　　192.168.160.152

　　192.168.160.153

　　每台服务器 1 主 1 从，共 3 主 3 从

　　相关安装包存储路径：/usr/local/redis-5.0.2

## 2、部署

　　**①** Redis 安装详见**[Redis 系列（一）：Redis 简介](https://www.cnblogs.com/toby-xu/p/11715704.html)**

**②** 进入 redis-5.0.2 cd /usr/local/redis-5.0.2

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191129235106275-1022232349.png)

 　　**③** 创建集群配置文件夹  mkdir redis-cluster

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191129235257852-1029733712.png)

 　　**④**（1）cd redis-cluster （2） mkdir 7001 和 mkdir 7002

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191129235619359-307618339.png)

 　　**⑤** （1）cp /usr/local/redis-5.0.2/redis.conf 7001/ （2） cd 7001

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191129235805859-1108395291.png)

 　　**⑥** 编辑 redis 配置文件 vim redis.conf 修改内容如下：

![](https://common.cnblogs.com/images/copycode.gif)

**daemonize yes
port 7001
dir /usr/local/redis-5.0.2/redis-cluster/7001/ 这里只是 demo，正式环境把数据跟 redis 安装包分开
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 5000
\#bind 127.0.0.1
protected-mode no
appendonly yes**

![](https://common.cnblogs.com/images/copycode.gif)

　　修改完后，cp redis.conf ../7002/  然后 cd /usr/local/redis-5.0.2/redis-cluster/7002 并修改的配置把所有的 7001 改成 7002

　　**⑦** （1） scp -r /usr/local/redis-5.0.2 root@192.168.160.152:/usr/local/   （2） scp -r /usr/local/redis-5.0.2 root@192.168.160.153:/usr/local/ 将 redis 拷贝到其他 2 台主机上去，（3）分别进到 192.168.160.152 cd /usr/local 和 192.168.160.153 cd /usr/local 如下：

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130002454811-316914070.png)

 　　**⑧** 分别启动 6 个 redis 实例，然后检查是否启动成功 （1）/usr/local/redis-5.0.2/bin/redis-server /usr/local/redis-5.0.2/redis-cluster/700\*/redis.conf （2）ps -ef | grep redis 查看是否启动成功

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130003658091-388936305.png)

 　　**⑨** 用 redis-cli 创建整个 redis 集群 (redis5 以前的版本集群是依靠 ruby 脚本 redis-trib.rb 实现)  /usr/local/redis-5.0.2/bin/redis-cli --cluster create --cluster-replicas 1 192.168.160.146:7001 192.168.160.152:7001 192.168.160.153:7001 192.168.160.146:7002 192.168.160.152:7002 192.168.160.153:7002 

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130004725671-118726230.png)

 　　**⑩** 验证集群： （1）连接任意一个客户端即可：./redis-cli -c -h -p (-a 访问服务端密码，-c 表示集群模式，指定 ip 地址和端口号）如：/usr/local/redis-5.0.2/bin/redis-cli -c -h 192.168.160.146 -p 700\* （2）进行验证： cluster info（查看集群信息）、cluster nodes（查看节点列表） （3）进行数据操作验证 （4）关闭集群则需要逐个进行关闭，使用命令： /usr/local/redis-5.0.2/bin/redis-cli -c -h 192.168.160.146 -p 700\* shutdown

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130005206284-596834218.png)

 　　其中 cluster nodes 信息如下：

![](https://common.cnblogs.com/images/copycode.gif)

9f71f69f061d9e811161d6be8a93f81f53304aed 192.168.160.152:7001@17001 master - 0 1575046303552 2 connected 5461-10922 a5447a0de83b33c028a9374372aa5602a869602e 192.168.160.152:7002@17002 slave dd939a72e405bf1dbd6fd538bc4383642435298e 0 1575046301531 5 connected
e7f80ba80749904838b6d779a0646e7f22313624 192.168.160.146:7001@17001 myself,master - 0 1575046303000 1 connected 0-5460 35582c86fc41f67d0089da2e21e99d9c66164dd3 192.168.160.153:7002@17002 slave e7f80ba80749904838b6d779a0646e7f22313624 0 1575046302443 6 connected
dd939a72e405bf1dbd6fd538bc4383642435298e 192.168.160.153:7001@17001 master - 0 1575046303956 3 connected 10923-16383 eddc783d11a4b46cebf157b5a6488c4346aec541 192.168.160.146:7002@17002 slave 9f71f69f061d9e811161d6be8a93f81f53304aed 0 1575046303452 4 connected
\# 含义 #
\# 节点 ID
\#IP: 端口: 集群端口
\# 标志: master, slave, myself, fail
\# 如果是从节点这里是他对应的主节点 ID
\# 集群最近一次向节点发送 PING 命令之后，过去了多长时间还没接到回复
\# 节点最近一次返回 PONG 回复的时间
\# 本节点的网络连接情况
\# 节点目前包含的槽：例如 192.168.160.146:7001 目前包含 0-5460 个哈希槽 (master)

![](https://common.cnblogs.com/images/copycode.gif)

　　**至此 Redis 集群搭建完成！！！！！**

##  三、**Redis 集群原理**

### **1、16384 个 slots（**槽位**）**

　　Redis Cluster 没有单机的那种 16 个数据库（0-15）的概念，而是分成了 16384 个 slots(槽位)，每个节点负责其中一部分槽位，槽位的信息存储于每个节点中；当客户端来连接集群时，它先得到一份集群的槽位配置信息并将其缓存在客户端本地。这样当客户端要查找某个 key 时，可以直接定位到目标节点。同时因为槽位的信息可能会存在客户端与服务器不一致的情况，还需要纠正机制来实现槽位信息的校验调整。

### 　　2、**槽位定位算法**

　　Redis Cluster 默认会对 key 值使用 CRC16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行取模来得到具体槽位。**HASH_SLOT = CRC16(key) mod 16384**

###  　　3、跳转重定位　

　　当客户端向一个错误的节点发出了指令，该节点会发现指令的 key 所在的槽位并不归自己管理，这时它会向客户端发送一个特殊的跳转指令携带目标操作的节点地址，告诉客户端去连这个节点去获取数据。客户端收到指令后除了跳转到正确的节点上去操作，还会同步更新纠正本地的槽位映射表缓存，后续所有 key 将使用新的槽位映射表。

![](https://img2018.cnblogs.com/i-beta/1761778/201911/1761778-20191130144732436-1865081885.png)

[Redis 持久化和主从复制原理](https://www.cnblogs.com/toby-xu/p/11973477.html) 
 [https://www.cnblogs.com/toby-xu/p/11960971.html](https://www.cnblogs.com/toby-xu/p/11960971.html)
