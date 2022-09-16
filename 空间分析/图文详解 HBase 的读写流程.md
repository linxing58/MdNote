# 图文详解 HBase 的读写流程
**这是我参与 8 月更文挑战的第 3 天，活动详情查看：[8 月更文挑战](https://juejin.cn/post/6987962113788493831 "https&#x3A;//juejin.cn/post/6987962113788493831")**

请结合我的这篇博客来理解本文： [一篇文章搞懂 HBase 的内部原理](https://juejin.cn/post/6994012960465109023 "https&#x3A;//juejin.cn/post/6994012960465109023")

HBase 采用 **LSM 树架构**，天生适用于写多读少的应用场景。

在真实生产线环境中，也正是因为 HBase 集群出色的写入能力，オ能支持当下很多数据激增的业务。

需要说明的是， HBase 服务端并没有提供 update 、 delete 接口， HBase 中对数据的更新、删除操作在服务器端也认为是写入操作，不同的是，更新操作会写入一个最新版本数据，删除操作会写入一条标记为 deleted 的 KV 数据。

所以 **HBase 中更新、刪除操作的流程与写入流程完全一致。** 

当然， HBase 数据写入的整个流程随着版本的迭代在不断优化，但总体流程变化不大。

## 3 个阶段

从整体架构的视角来看，写入流程可以概括为三个阶段。

1.  客户端处理阶段: 客户端将用户的写入请求进行预处理，并根据集群元数据定位写入数据所在的 RegionServer ，将请求发送给对应的 RegionServer .
2.  Region 写入阶段: RegionServer 接收到写入请求之后将数据解析出来，首先写入 WAL ，再写入对应 Region 列簇的 MemStore 。
3.  MemStore Flush 阶段: 当 Region 中 MemStore 容量超过一定阈值，系统会异步执行 flush 操作，将内存中的数据写入文件，形成 HFile 。

## 6 个步骤

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bff7b7d879a941cba4d6f993fb07ec7d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

1.  首先从 ZooKeeper 找到 hbase:meta 表的 Region 位置，然后读取 hbase:meta 表中的数据， hbase:meta 表中存储了用户表的 Region 信息
2.  根据 namespace 、表名和 rowkey 信息找到写入数据对应的 Region 信息
3.  找到这个 Region 对应的 RegionServer ，然后发送请求
4.  把数据分别写到 HLog (WriteAheadLog) 和 MemStore 各一份
5.  MemStore 达到阈值后把数据刷到磁盘，生成 StoreFile 文件
6.  删除 HLog 中的历史数据。

和写流程相比， HBase 读数据的流程更加复杂。

主要基于两个方面的原因:

一是因为 HBase 一次范围査询可能会涉及多个 Region 、多块缓存甚至多个数据存储文件; 二是因为 HBase 中更新操作以及删除操作的实现都很简单，更新操作并没有更新原有数据，而是使用时间戳属性实现了多版本; 删除操作也并没有真正删除原有数据，只是插入了一条标记为 "deleted" 标签的数据，而真正的数据删除发生在系统异步执行 Major Compact 的时候。

很显然，这种实现思路大大简化了数据更新、删除流程，但是对于数据读取来说却意味着套上了层层枷锁: **读取过程需要根据版本进行过滤，对已经标记删除的数据也要进行过滤**。

## 4 个阶段

读流程从头到尾可以分为如下 4 个阶段:

1.  Client - Server 读取交互逻辑
2.  Server 端 Scan 框架体系
3.  过滤淘汰不符合查询条件的 HFile
4.  从 HFile 中读取待查找 Key

其中 Client - Server 交互逻辑主要介绍 **HBase 客户端在整个 scan 请求的过程中是如何与服务器端进行交互的**，理解这点对于使用 Hbase Scan API 进行数据读取非常重要。

了解 Server 端 Scan 框架体系，从宏观上介绍 **HbaseRegionServer 如何逐步处理一次 scan 请求**。

## 6 个步骤

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/703536dc83bd476a81e88e259d7032d1~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

1.  首先从 ZooKeeper 找到 meta 表的 region 位置，然后读取 hbase:meta 表中的数据， hbase:meta 表中存储了用户表的 region 信息
2.  根据要查询的 namespace 、表名和 rowkey 信息，找到写入数据对应的 Region 信息
3.  找到这个 Region 对应的 RegionServer ，然后发送请求
4.  查找对应的 Region
5.  先从 MemStore 查找数据，如果没有，再从 BlockCache 上读取

> HBase 上 RegionServer 的内存分为两个部分一部分作为 MemStore ，主要用来写;。

另外一部分作为 BlockCache ，主要用于读数据;

6.  如果 BlockCache 中也没有找到，再到 StoreFile(HFile) 上进行读取

> 从 StoreFile 中读取到数据之后，不是直接把结果数据返回给客户端，而是把数据先写入到 BlockCache 中，目的是为了加快后续的查询; 然后在返回结果给客户端。 
>  [https://juejin.cn/post/6994830726759710750](https://juejin.cn/post/6994830726759710750)
