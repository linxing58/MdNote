# GeoMesa时空基础及应⽤场景
推荐视频讲解 1h [https://yq.aliyun.com/live/793](https://yq.aliyun.com/live/793)

## 基础概念

### 数据库时空引擎

![](https://img-blog.csdnimg.cn/20200517152511591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### OGC 空间要素对象表达

-   SimpleFeature : 时空要素的抽象表达，默认还有 Geometry 字段
-   SimpleFeatureTpye: 要素元数据描述，包括：字段名、类型、空间参考等，类比数据库表结构
-   WKT： Well-known text, 用来描述 SimpleFeature 对象  
    ![](https://img-blog.csdnimg.cn/20200517152405941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

## GeoMesa 简介

-   GeoMesa 是⼀一款开源的基于分布式计算系统的⾯面向海海量量时空数据查询与分析的⼯工具包
-   GeoMesa 基于 GeoTools API 进⾏行行设计，与 GeoServer 等进⾏行行集成提供 OGC 标准的服务
-   ⽀支持多种可扩展的、基于云端的数据存储架构，包括 Apache Accumulo, HBase，Cassandra，Google Bigtable，以及⽤用于流计算的 Apache Kafka 。
-   提供了了 Spark，并增加了了正对空间数据的 UDT、UDF 和 UDAF，⽅方便便⽤用户直接使⽤用 Spark SQL  
    进⾏行行空间数据查询与分析

官网 [https://www.geomesa.org/](https://www.geomesa.org/)  
![](https://img-blog.csdnimg.cn/20200517152910502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### GeoMesa 体系架构

GeoMesa 的源码可以在 GitHub 官方页面下到。GeoMesa 代码库本身比较复杂，包含了大量的子模块，除了自身所带的一些基础工具模块外，还提供了基于目前主流的分布式存储系统的扩展模块，但是在了解了 GeoMesa 的架构之后，读者应该不难看出整个工程还是有规律可循的，比如 geomesa-index-api 提供了最核心的空间数据索接口 (GeoMesaFeatureIndex) 与数据访问 (GeoMesaDataStore) 等基础接口类，然后基于此模块分别有 geomesa-accumulo,geomesa-hbase,geomesa-cassandra 等扩展模块。两外还包括一些其他的辅助模块，如 geomea-jobs 提供了对 MapReduce 的支持等，geomea-utils 提供了一些被广泛使用的工具类

详细见博客 [https://blog.csdn.net/u011596455/article/details/85869199](https://blog.csdn.net/u011596455/article/details/85869199)

### 时空基础 ： 空间填充曲线与 GeoHash

#### 时空索引 R 树

解决了在高维空间搜索等问题，Oracle Spatial、Mysql Spatial、PostgreSql(PostGIS) 都是基于 R 树进行空间搜索操作，即对时空字段（Geometry Column） 创建 R 树索引。  
详细见博客  
[https://blog.csdn.net/qq\\\_18298439/article/details/96278997](https://blog.csdn.net/qq\_18298439/article/details/96278997)

为什么使用空间填充曲线  
R 树存在的问题:

-   单独创建索引文件
-   数据更新问题  
    为了达到平衡状态，新掺入数据需要更新整个 R 树
-   不适合 NoSQL 的存储结构  
    Hbase 本身只提供基于行键和全表扫面的查询，而行键索引单一，对于多维度的查询困难

`空间填充曲线的优点是将多维空间转换成一维曲线`

### 空间填充曲线

![](https://img-blog.csdnimg.cn/2020051715521833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

Z 曲线 Hibert 曲线最常用

### 空间查询

![](https://img-blog.csdnimg.cn/20200517155336538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

-   用户定义查询窗口
-   层次划分
-   计算查询范围（Range）

### GeoHash 原理

GeoHash 将⼆二维的经纬度转换成字符串串（Base32 编码），如下图展示了了 9 个区域的 GeoHash 字符串串，  
分别是 WX4ER，WX4G2、WX4G3 等，每⼀一个字符串串代表了了某⼀一矩形区域。  
![](https://img-blog.csdnimg.cn/20200517155501177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

-   字符串越长，表示的范围越小越精确；字符串长度越小，表示的范围越大越宽泛；
-   字符串越相似表示距离越相近；

## GeoMesa Hbase 时空索引

### GeoMesa 时空索引

GeoMesa 使用了基于 Z-order 填充曲线的 GeoHash 空间索引技术，  
并针对时间维度进行了扩展，具体分为：  
• Z2：空间，点索引  
• Z3：时间 + 空间，点索引  
• XZ2 ：空间，线\\面索引  
• XZ3 ：时间 + 空间，线\\面索引  
Z2 ![](https://img-blog.csdnimg.cn/20200517155937396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

Z3  
![](https://img-blog.csdnimg.cn/20200517155948194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

GeoMesa 时空索引具体实现：  
[https://github.com/locationtech/sfcurve](https://github.com/locationtech/sfcurve)  
[https://github.com/locationtech/geomesa/tree/master/geomesa-z3](https://github.com/locationtech/geomesa/tree/master/geomesa-z3)

## GeoMesa HBase 索引

![](https://img-blog.csdnimg.cn/2020051716035967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### RowKey 设计

![](https://img-blog.csdnimg.cn/20200517160458324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### 时空查询 - Query Planning

![](https://img-blog.csdnimg.cn/20200517174744718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

索引选择顺序

1.  Feature ID predicates using the ID index
2.  High-cardinality attribute predicates using the attribute index
3.  Attribute equality predicates using the attribute index
4.  Spatio-temporal predicates using the Z3/XZ3 index
5.  Attribute range predicates using the attribute index
6.  Spatial predicates using the Z2/XZ2 index
7.  Temporal predicates using the Z3/XZ3 index

![](https://img-blog.csdnimg.cn/20200517174830241.png)

## GeoMesa HBase 应⽤用场景

[https://blog.csdn.net/xiaof22a/article/details/80215787](https://blog.csdn.net/xiaof22a/article/details/80215787)

GeoTools DataStore 接⼝口  
• SimpleFeature：空尽要素的抽象表达，默认含有 Geometry 字段  
• SimpleFeatureType：要素[元数据](https://so.csdn.net/so/search?q=%E5%85%83%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)描述，包括：字段名、类型、空间参考等，类似表结构  
• DataStore：要素数据集，对应 RDBMS 中的数据库，定义了了⽤用户操作数据的接⼝口  
• FeatureSource：⽤用于数据查询  
• FeatureStore： FeatureSource ⼦子类，增加数据更更新功能  
• SimpleFeatureCollection：数据要素结合，按需加载  
• Query：数据查询类，封装了了查询条件  
![](https://img-blog.csdnimg.cn/20200517175339999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### GeoMesa Kafka DataStore

-   使⽤用 Kafka 作为数据存储 DataStore
-   通过 GeoTools DataStore 标准接⼝口进⾏行行访问
-   Consumer 与 Producer 可以分布在不不同 server
-   ⽀支持要素缓存，定时写⼊入 kafka  
    -![](https://img-blog.csdnimg.cn/20200517175412402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

### GeoMesa Lambda DataStore

• 数据存储在两个层：transient tier （Kafka）  
和 a persistent tier（HBase）  
• 数据定时写⼊入持久层  
• 使⽤用 ZK 同步数据缓存状态，保证数据⼀一次  
写⼊入  
• 进⾏行行数据查询会从两个存储层分别进⾏行行，  
然后合并查询结果返回给⽤用户

![](https://img-blog.csdnimg.cn/20200517175449703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

[https://www.geomesa.org/documentation/user/lambda/index.html](https://www.geomesa.org/documentation/user/lambda/index.html)

### GeoMesa GeoJSON/REST API

优点：

1.  系统便于部署
2.  基于 REST 服务，操作灵活方便
3.  全部使用 GeoJSON 进行编码，方便与  
    其他系统集成  
    缺点：
4.  功能比 GeoTools API 弱，不支持属  
    性索引、排序等高级功能
5.  Server 端容易成为系统瓶颈  
    ![](https://img-blog.csdnimg.cn/20200517175523936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)
    [https://www.geomesa.org/documentation/user/geojson.html](https://www.geomesa.org/documentation/user/geojson.html)

### 支持 Spark ⼤大数据分析

提供了用于空间数据分析的 SpatialRDD 模型

-   提供了多种时空函数实现，如  
    buffer，contains 等
-   扩展 Spark SQL 以支持 ISO SQL/MM 标  
    准与 OGC SF/SQL 标准的时空查询  
    ![](https://img-blog.csdnimg.cn/20200517175633685.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

阿⾥里里云 HBase Ganos  
![](https://img-blog.csdnimg.cn/20200517175652577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Jvb21MZWU=,size_16,color_FFFFFF,t_70)

访问：[https://cn.aliyun.com/product/hbase](https://cn.aliyun.com/product/hbase) 查询详细信息 
 [https://blog.csdn.net/BoomLee/article/details/106174431](https://blog.csdn.net/BoomLee/article/details/106174431)
