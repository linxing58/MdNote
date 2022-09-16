# timescaledb简介及性能测试
[![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)
](https://www.jianshu.com/u/136525c3d5b5)

2021.12.03 15:47:24 字数 4,630 阅读 66

## 一、背景

随着物联网的发展，时序数据库的需求越来越多，比如水文监控、工厂的设备监控、国家安全相关的数据监控、通讯监控、金融行业指标数据、传感器数据等。  
在互联网行业中，也有着非常多的时序数据，例如用户访问网站的行为轨迹，应用程序产生的日志数据等等。

时序数据有几个特点：

-   基本上都是插入，没有更新的需求。
-   数据基本上都有时间属性，随着时间的推移不断产生新的数据，旧的数据不需要保存太久。

业务方对时序数据通常有几个查询需求。

-   获取最新状态，查询最近的数据（例如传感器最新的状态）。
-   展示区间统计，指定时间范围，查询统计信息，例如平均值，最大值，最小值，计数等。
-   获取异常数据，根据指定条件，筛选异常数据。

## 二、时序数据库应该具备的特点

-   压缩能力  
    通常用得上时序数据库的业务，传感器产生的数据量都是非常庞大的，数据压缩可以降低存储成本。
-   自动 rotate  
    时序数据通常对历史数据的保留时间间隔是有规定的，例如一个线上时序数据业务，可能只需要保留最近 1 周的数据。为了方便使用，时序数据库必须有数据自动 rotate 的能力。
-   支持分片，水平扩展  
    因为涉及的传感器可能很多，单个节点可能比较容易成为瓶颈，所以时序数据库应该具备水平扩展的能力，例如分表应该支持水平分区。
-   自动扩展分区，  
    业务对时序数据的查询，往往都会带上对时间区间进行过滤，因此时序数据通常在分区时，一定会有一个时间分区的概念。时序数据库务必能够支持自动扩展分区，减少用户的管理量，不需要人为的干预自动扩展分区。例如 1 月份月末，自动创建 2 月份的分区。
-   插入性能  
    时序数据，插入是一个强需求。对于插入性能要求较高。
-   分区可删除  
    分区可以被删除，例如保留 1 个月的数据，1 个月以前的分区都可以删除掉。
-   易用性 (SQL 接口)  
    SQL 是目前最通用的数据库访问语言，如果时序数据库能支持 SQL 是最好的。
-   类型丰富  
    物联网的终端各异，会有越来越多的非标准类型的支持需求。例如采集图像的传感器，数据库中至少要能够存取图像的特征值。而对于其他垂直行业也是如此，为了最大程度的诠释业务，必须要有精准的数据类型来支撑。
-   索引接口  
    支持索引，毫无疑问是为了加速查询而引入的。
-   高效分析能力  
    时序数据，除了单条的查询，更多的是报表分析或者其他的分析类需求。这对时序数据库的统计能力也是一个挑战。
-   其他特色  
    （1）支持丰富的数据类型，数组、范围类型、JSON 类型、K-V 类型、GIS 类型、图类型等。满足更多的工业化需求，例如传感器的位置信息、传感器上传的数据值的范围，批量以数组或 JSON 的形式上传，传感器甚至可能上传图片特征值，便于图片的分析。（例如国家安全相关），轨迹数据的上层则带有 GIS 属性。  
    这个世界需要的是支持类型丰富的时序数据库，而不是仅仅支持简单类型的时序数据库。  
    （2）支持丰富的索引接口，因为类型丰富了，普通的 B-TREE 索引可能无法满足快速的检索需求，需要更多的索引来支持 数组、JSON、GIS、图特征值、K-V、范围类型等。 (例如 PostgreSQL 的 gin, gist, sp-gist, brin, rum, bloom, hash 索引接口)。  
    这两点可以继承 PostgreSQL 数据库的已有功能，已完全满足。

## 三、TimescaleDB 介绍

TimescaleDB 是基于 PostgreSQL 数据库打造的一款时序数据库，插件化的形式，随着 PostgreSQL 的版本升级而升级，不会因为另立分支带来麻烦。

![](https://upload-images.jianshu.io/upload_images/20224274-89e371d157166555.png)

图 1 timescaleDB 结构

数据自动按时间和空间分片（chunk）

-   partition key（分区键）  
    是由一个表上的一个列或者多个列组成，用于确定某一行特定数据分布在哪个分区上，用 create table 语句来定义。
-   chunks  
    分区在 TimescaleDB 中被称为 chunk。
-   query across time  
    跨时间查询
-   query across space  
    跨空间查询

**透明自动分区特性**  
在时序数据的应用场景下，其记录数往往是非常庞大的，很容易就达到 数以亿计 。而对于 PG 来说，由于大量的还是使用 B+tree 索引，所以当数据量到达一定量级后其写入性能就会出现明显的下降（这通常是由于索引本身变得非常庞大且复杂）。这样的性能下降对于时序数据的应用场景而言是不能忍受的，而 TimescaleDB 最核心的自动分区特性解决的就是这个问题。这个特性希望达到的目标如下：

-   随着数据写入的不断增加，将时序数据表的数据分区存放，保证每一个分区的索引维持在一个较小规模，从而维持住写入性能。
-   基于时序数据的查询场景，自动分区时以时序数据的时间戳为分区键，从而确保查询时可以快速定位到所需的数据分区，保证查询性能。
-   分区过程对用户透明，从而达到自动扩展的效果。

![](https://upload-images.jianshu.io/upload_images/20224274-2e716e6de7fc49c4.png)

图 2 PG 库分区

上图是 PG10 声明式分区示例，我们创建了一个表树，以主表为根，接下来是下一个子级别的四个设备表，然后是任意数量的时间表第三个子层次。上图显示了 PG 库多维表分区的概念视图。

PostgresSQL 分区的意思是把逻辑上的一个大表分割成物理上的几块儿。在 PG 里表分区是通过表继承来实现的，一般都是建立一个主表，里面是空，然后每个分区都去继承它。无论何时，都应保证主表里面是空的。

PostgreSQL 10.x 之前的版本提供了一种 “手动” 方式使用分区表的方式，需要使用继承 + 触发器的来实现分区表，步骤较为繁琐，需要定义附表、子表、子表的约束、创建子表索引，创建分区删除、修改，触发器等。[PostgreSQL 分区表（Table Partitioning）应用](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fmchina%2Farchive%2F2013%2F04%2F09%2F2973427.html)

PostgreSQL 10.x 开始提供了内置分区表（内置是相对于 10.x 之前的手动方式）。内置分区简化了操作，将部分操作内置，最终简单三步就能够创建分区表。但是只支持范围分区（RANGE）和列表分区（LIST），11.x 版本添加了对 HASH 分区。[PostgreSQL 10 分区表性能测试](https://www.jianshu.com/p/c646f26feb04)

从上图可以看出来，需要每个时间间隔创建四个新表，每个设备一个。额外的设备将使分区策略进一步复杂化，所有这些额外的表都可能对性能和可伸缩性产生负面影响，因为需要为插入和查询处理更多的表。

TimescaleDB 也依赖于表继承，但避免了深度嵌套的多级继承树。相反，它直接在根创建一个浅层叶块树，而不管分区维度的数量，如下所示。

![](https://upload-images.jianshu.io/upload_images/20224274-17719fd143925fda.png)

图 3 TimescaleDB 分区

彩色框代表不同的” 空间” 键（例如，我们示例中的设备）。这样子的设计减少了树中表的数量并避免了嵌套。通过这样做，TimescaleDB 提高了插入和查询性能，简化了重新分区，并使表管理（和保留）更容易。

### 四、性能测试

环境：

-   操作系统：CentOS 7.8（虚拟机）
-   Postgresql 版本：12.1
-   TimescaleDB 版本：2.2.0
-   CPU：Intel Core i7 9xx 1 核心 4 线程
-   内存：4G
-   硬盘：100G
-   时序库安装参考 [离线安装 Postgres、时序库 (timescaledb)](https://www.jianshu.com/p/f6061ac99ef4)

**安装工具**

1、golang 环境安装详见[centos 7 安装 golang](https://www.jianshu.com/p/35a161738d83)  
2、TSBS 安装  
TSBS 用于对批量加载性能和查询执行性能进行基准测试。（目前不测量并发插入和查询性能）[github 上的 tsbs](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ftimescale%2Ftsbs)

    cd $GOPATH
    yum install git
    git clone https://gitee.com/mirrors_toddlipcon/tsbs.git
    cd tsbs/cmd
    go get ./... 

3、安装 postgres 缓存清理扩展 pg_dropcache,![](https://math.jianshu.com/math?formula=%5Ccolor%7Bred%7D%7B%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E6%85%8E%E7%94%A8%EF%BC%81%EF%BC%81%EF%BC%81%EF%BC%81%EF%BC%81%7D)

    #进入contrib目录下,下载pg_dropcache
    git clone git://github.com/zilder/pg_dropcache
    cd pg_dropcache
    make 
    make install USE_PGXS=1 

**生成测试数据**

    tsbs_generate_data --use-case="cpu-only" --seed=123 --scale=4000 --timestamp-start="2021-01-01T00:00:00Z" --timestamp-end="2021-01-04T00:00:00Z" --log-interval="10s" --format="timescaledb" | gzip > /tmp/timescaledb-data.gz 

> 需要的变量：  
> 1.--use-case：分为 iot、cpu-only 和 devops，例如 cpu-only  
> 2.--seed：一种用于确定性生成的 PRNG 种子，例如 123  
> 3.--scale：需要生成的设备数量，例如 4000  
> 4.--timestamp-start：数据生成的开始时间，例如 2016-01-01T00:00:00Z  
> 5.--timestamp-end：数据生成的结束时间，例如 2016-01-04T00:00:00Z  
> 6.--log-interval：以秒为单位读取每个设备之间的时间间隔，例如 10  
> 7.--format：需要生成的数据库， 例如 timescaledb (cassandra、clickhouse、cratedb、influx、mongo、siridb、timescaledb)

上面的示例将生成一个伪 CSV 文件，可用于将数据批量加载到 TimescaleDB 中。上面的配置将生成超过 1 亿行。时间段每增加一天将增加大约 3300 万行，30 天将产生 10 亿行。

**生成查询脚本**  
本测试没有使用这些测试脚本，因为生成的脚本使用的 time_bucket() 是 timescaledb 专属方法。不过借鉴了相关的查询方法。

    [root@panghu tsbs]# FORMATS="timescaledb" SCALE=4000 SEED=123 TS_START="2021-01-01T00:00:00Z" TS_END="2021-01-04T00:00:01Z" QUERIES=1000 QUERY_TYPES="single-groupby-1-1-1 single-groupby-1-1-12 single-groupby-1-8-1 single-groupby-5-1-1 single-groupby-5-1-12 single-groupby-5-8-1 cpu-max-all-1 cpu-max-all-8 double-groupby-1 double-groupby-5 double-groupby-all high-cpu-all high-cpu-1 lastpoint groupby-orderby-limit" BULK_DATA_DIR="/tmp/queries" scripts/generate_queries.sh 

**写入数据**  
-- 普通表  
在 tsbs 目录下执行一下命令，其参数表示是单线程执行，创建普通表，每 30s 输出一次报告，并将报告输出到 normal.log

    NUM_WORKERS=1 USE_HYPERTABLE=false REPORTING_PERIOD=30s BULK_DATA_DIR=/tmp \ 
    DATABASE_NAME=normal scripts/load_timescaledb.sh > normal.log 

\-- 超表  
其创建的是以 12 小时分块的超表

    NUM_WORKERS=1 CHUNK_TIME=12h USE_HYPERTABLE=true REPORTING_PERIOD=30s \
    DATABASE_NAME=hyper BULK_DATA_DIR=/tmp scripts/load_timescaledb.sh > hypertable.log 

经过 1 亿数据量的测试，并没有发现 timescaledb 在 insert 方面的优势。可能是机器配置或数据库配置问题，也可能是测试数据不够大。相比来说 pg 速度比 timescaledb 快，但 timescaledb 的插入速度一直都比较稳定。timescaledb 官网上也只有一张图，并没有详细的测试场景介绍。

![](https://upload-images.jianshu.io/upload_images/20224274-447ec9ccdc5a0a2b.png)

图 4 官网测试

按照官网的解释，当向 PostgreSQL 中插入新的数据行时，数据库需要更新表的每个索引列的索引（一般来说是 BTREE）。一旦索引太大而无法放入内存——根据分配的内存，当表每 10s 内插入上百万行数据时，通常会发生这种情况——这需要从磁盘交换一页或多页。正是这种磁盘交换造成了插入瓶颈。在这个问题上投入更多的内存不可避免地造成延迟。

TimescaleDB 通过大量利用自动时空分区解决了这一问题，即使是运行在一台机器上。从本质上讲，所有在最近时间间隔内的写入都只对保留在内存中的表进行。这使得在大规模插入数据时，插入性能比 PostgreSQL 提高了 20 倍。

**查询性能测试**  
首先在两个库中引入 pg_dropcache

    [root@panghu pg_dropcache]# su - postgres -c psql
    postgres=# \c normal
    normal=# create extension pg_dropcache;
    normal=# \c hypertable 
    hypertable=# create extension pg_dropcache; 

每次查询前都执行

**简单查询**  
1、1 台设备在 1 小时内每 1 分钟的一个指标 (一列) 最大值

    explain analyze select date_trunc('minute', time) as minute, max(usage_user) as max_usage_user from cpu where tags_id in (select id from tags where hostname in('host_249')) and time >='2021-01-02 00:00:00' and time < '2021-01-02 01:00:00' group by minute order by minute asc; 

\-- 普通表

     GroupAggregate  (cost=1529.58..1536.98 rows=370 width=16) (actual time=5.159..5.396 rows=60 loops=1)
       Group Key: (date_trunc('minute'::text, cpu."time"))
       ->  Sort  (cost=1529.58..1530.51 rows=370 width=16) (actual time=5.136..5.213 rows=360 loops=1)
             Sort Key: (date_trunc('minute'::text, cpu."time"))
             Sort Method: quicksort  Memory: 41kB
             ->  Nested Loop  (cost=0.85..1513.80 rows=370 width=16) (actual time=0.140..4.806 rows=360 loops=1)
                   ->  Index Scan using tags_hostname_idx on tags  (cost=0.28..8.30 rows=1 width=4) (actual time=0.060..0.062 rows=1 l
    oops=1)
                         Index Cond: (hostname = 'host_249'::text)
                   ->  Index Scan using cpu_tags_id_time_idx on cpu  (cost=0.57..1500.88 rows=370 width=20) (actual time=0.063..4.039 
    rows=360 loops=1)
                         Index Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("tim
    e" < '2021-01-02 01:00:00+08'::timestamp with time zone))
     Planning Time: 1.079 ms
     Execution Time: 5.498 ms 

\-- 超表

     Sort  (cost=1477.89..1478.39 rows=200 width=16) (actual time=5.172..5.179 rows=60 loops=1)
       Sort Key: (date_trunc('minute'::text, _hyper_1_2_chunk."time"))
       Sort Method: quicksort  Memory: 27kB
       ->  HashAggregate  (cost=1467.74..1470.24 rows=200 width=16) (actual time=5.110..5.134 rows=60 loops=1)
             Group Key: date_trunc('minute'::text, _hyper_1_2_chunk."time")
             ->  Nested Loop  (cost=17.52..1465.91 rows=367 width=16) (actual time=0.416..4.783 rows=360 loops=1)
                   ->  Index Scan using tags_hostname_idx on tags  (cost=0.28..8.30 rows=1 width=4) (actual time=0.193..0.199 rows=1 l
    oops=1)
                         Index Cond: (hostname = 'host_249'::text)
                   ->  Bitmap Heap Scan on _hyper_1_2_chunk  (cost=17.24..1453.02 rows=367 width=20) (actual time=0.209..4.104 rows=36
    0 loops=1)
                         Recheck Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("t
    ime" < '2021-01-02 01:00:00+08'::timestamp with time zone))
                         Heap Blocks: exact=360
                         ->  Bitmap Index Scan on _hyper_1_2_chunk_cpu_tags_id_time_idx  (cost=0.00..17.15 rows=367 width=0) (actual t
    ime=0.149..0.149 rows=360 loops=1)
                               Index Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND
     ("time" < '2021-01-02 01:00:00+08'::timestamp with time zone))
     Planning Time: 1.235 ms
     Execution Time: 5.264 ms 

2、扩大查询时间，12 小时内，每分钟的数据最大值

    explain analyze select date_trunc('minute', time) as minute, max(usage_user) as max_usage_user from cpu where tags_id in (select id from tags where hostname in('host_555')) and time >='2021-01-02 00:00:00' and time < '2021-01-02 12:00:00' group by minute order by minute asc; 

\-- 普通表

     GroupAggregate  (cost=18034.27..18124.93 rows=4533 width=16) (actual time=54.607..57.942 rows=720 loops=1)
       Group Key: (date_trunc('minute'::text, cpu."time"))
       ->  Sort  (cost=18034.27..18045.61 rows=4533 width=16) (actual time=54.586..55.601 rows=4320 loops=1)
             Sort Key: (date_trunc('minute'::text, cpu."time"))
             Sort Method: quicksort  Memory: 395kB
             ->  Nested Loop  (cost=186.64..17758.98 rows=4533 width=16) (actual time=3.339..51.775 rows=4320 loops=1)
                   ->  Index Scan using tags_hostname_idx on tags  (cost=0.28..8.30 rows=1 width=4) (actual time=0.059..0.066 rows=1 loops=1)
                         Index Cond: (hostname = 'host_555'::text)
                   ->  Bitmap Heap Scan on cpu  (cost=186.36..17694.02 rows=4533 width=20) (actual time=3.260..45.543 rows=4320 loops=1)
                         Recheck Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                         Heap Blocks: exact=4320
                         ->  Bitmap Index Scan on cpu_tags_id_time_idx  (cost=0.00..185.23 rows=4533 width=0) (actual time=2.229..2.229 rows=4320 loops=1)
                               Index Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
     Planning Time: 0.883 ms
     Execution Time: 58.365 ms 

\-- 超表

     Sort  (cost=17181.49..17181.99 rows=200 width=16) (actual time=55.692..55.797 rows=720 loops=1)
       Sort Key: (date_trunc('minute'::text, _hyper_1_2_chunk."time"))
       Sort Method: quicksort  Memory: 58kB
       ->  HashAggregate  (cost=17171.34..17173.84 rows=200 width=16) (actual time=55.070..55.374 rows=720 loops=1)
             Group Key: date_trunc('minute'::text, _hyper_1_2_chunk."time")
             ->  Nested Loop  (cost=116.85..17148.76 rows=4517 width=16) (actual time=2.083..51.022 rows=4320 loops=1)
                   ->  Index Scan using tags_hostname_idx on tags  (cost=0.28..8.30 rows=1 width=4) (actual time=0.081..0.087 rows=1 loops=1)
                         Index Cond: (hostname = 'host_555'::text)
                   ->  Append  (cost=116.57..17084.01 rows=4516 width=20) (actual time=1.989..45.468 rows=4320 loops=1)
                         ->  Bitmap Heap Scan on _hyper_1_2_chunk  (cost=116.57..10590.86 rows=2824 width=20) (actual time=1.983..30.447 rows=2880 loops=1)
                               Recheck Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                               Heap Blocks: exact=2880
                               ->  Bitmap Index Scan on _hyper_1_2_chunk_cpu_tags_id_time_idx  (cost=0.00..115.86 rows=2824 width=0) (actual time=1.323..1.323 rows=2880 loops=1)
                                     Index Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                         ->  Bitmap Heap Scan on _hyper_1_3_chunk  (cost=70.14..6470.57 rows=1692 width=20) (actual time=0.866..13.523 rows=1440 loops=1)
                               Recheck Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                               Heap Blocks: exact=1440
                               ->  Bitmap Index Scan on _hyper_1_3_chunk_cpu_tags_id_time_idx  (cost=0.00..69.71 rows=1692 width=0) (actual time=0.588..0.589 rows=1440 loops=1)
                                     Index Cond: ((tags_id = tags.id) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
     Planning Time: 1.978 ms
     Execution Time: 56.013 ms 

在单磁盘机器上，许多只执行索引查找或表扫描的简单查询在 TimescaleDB 上会慢 1-3 毫秒。例如，在我们带有索引时间、主机名和 cpu 使用信息的 1 亿 行表中，每个数据库将花费不到 3ms，但在 TimescaleDB 上需要额外 1ms。由于计划时间开销稍大，大多数通常需要 &lt;20 毫秒的简单查询（例如，索引查找）在 TimescaleDB 上会慢几毫秒。

**基于时间的查询**  
查询在半天里 CPU 某个指标超过 90 的数据

    explain analyze select * from cpu where usage_user > 90.0 and  time >='2021-01-02 00:00:00' and time < '2021-01-02 12:00:00'; 

\-- 普通表

     Index Scan using cpu_time_idx on cpu  (cost=0.57..921587.58 rows=1661968 width=133) (actual time=93.180..157431.245 rows=1654719 loops=1)
       Index Cond: (("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
       Filter: (usage_user > '90'::double precision)
       Rows Removed by Filter: 15625281
     Planning Time: 0.259 ms
     Execution Time: 157719.448 ms 

\-- 超表

     Gather  (cost=1000.00..902892.82 rows=1795682 width=133) (actual time=10107.044..111039.684 rows=1654719 loops=1)
       Workers Planned: 2
       Workers Launched: 2
       ->  Parallel Append  (cost=0.00..722324.62 rows=748201 width=133) (actual time=6100.415..106707.848 rows=551573 loops=3)
             ->  Parallel Index Scan using _hyper_1_3_chunk_cpu_time_idx on _hyper_1_3_chunk  (cost=0.44..294651.21 rows=275748 width=133) (actual time=55.335..81420.184 rows=180439 loops=3)
                   Index Cond: (("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                   Filter: (usage_user > '90'::double precision)
                   Rows Removed by Filter: 1739561
             ->  Parallel Seq Scan on _hyper_1_2_chunk  (cost=0.00..423932.41 rows=472453 width=133) (actual time=9092.177..37698.496 rows=556702 loops=2)
                   Filter: ((usage_user > '90'::double precision) AND ("time" >= '2021-01-02 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 12:00:00+08'::timestamp with time zone))
                   Rows Removed by Filter: 8083298
     Planning Time: 97.874 ms
     Execution Time: 111278.058 ms 

![](https://upload-images.jianshu.io/upload_images/20224274-1a7e25410cb614b4.png)

图 5 两者查询数据量较大时查询时间对比

基于时间的查询，特别是在数据量较大时，timescaleDB 通常可以达到 postgre 大约 1.2 到 5 倍的性能。

**基于时间的聚合**  
查询一天内每个设备没小时某个测值平均值

    explain analyze select date_trunc('hour', time) as hour, hostname, avg(usage_user) as mean_usage_user from cpu where time >= '2021-01-01 00:00:00' and time < '2021-01-02 00:00:00' group by hour, hostname order by hour; 

\-- 普通表

     Finalize GroupAggregate  (cost=2805761.31..5565569.39 rows=9714638 width=25) (actual time=297183.377..302860.122 rows=64000 loops=1)
       Group Key: (date_trunc('hour'::text, "time")), hostname
       ->  Gather Merge  (cost=2805761.31..5274130.25 rows=19429276 width=49) (actual time=297183.100..303492.096 rows=192000 loops=1)
             Workers Planned: 2
             Workers Launched: 2
             ->  Partial GroupAggregate  (cost=2804761.28..3030509.69 rows=9714638 width=49) (actual time=296854.664..302114.907 rows=64000 loops=3)
                   Group Key: (date_trunc('hour'::text, "time")), hostname
                   ->  Sort  (cost=2804761.28..2830840.14 rows=10431543 width=25) (actual time=296854.443..299567.644 rows=7680000 loops=3)
                         Sort Key: (date_trunc('hour'::text, "time")), hostname
                         Sort Method: external merge  Disk: 310456kB
                         Worker 0:  Sort Method: external merge  Disk: 317048kB
                         Worker 1:  Sort Method: external merge  Disk: 315008kB
                         ->  Parallel Index Scan using cpu_time_idx on cpu  (cost=0.57..1089562.88 rows=10431543 width=25) (actual time=100.705..210288.786 rows=7680000 loops=3)
                               Index Cond: (("time" >= '2021-01-01 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 00:00:00+08'::timestamp with time zone))
     Planning Time: 2.233 ms
     Execution Time: 303763.514 ms 

\-- 超表

     Finalize GroupAggregate  (cost=2419133.87..2526990.01 rows=40000 width=25) (actual time=188301.739..196638.338 rows=64000 loops=1)
       Group Key: (date_trunc('hour'::text, _hyper_1_2_chunk."time")), _hyper_1_2_chunk.hostname
       ->  Gather Merge  (cost=2419133.87..2525790.01 rows=80000 width=49) (actual time=188301.571..196564.337 rows=136000 loops=1)
             Workers Planned: 2
             Workers Launched: 2
             ->  Partial GroupAggregate  (cost=2418133.85..2515556.00 rows=40000 width=49) (actual time=176953.907..181966.568 rows=45333 loops=3)
                   Group Key: (date_trunc('hour'::text, _hyper_1_2_chunk."time")), _hyper_1_2_chunk.hostname
                   ->  Sort  (cost=2418133.85..2442364.39 rows=9692215 width=25) (actual time=176919.752..179504.723 rows=7680000 loops=3)
                         Sort Key: (date_trunc('hour'::text, _hyper_1_2_chunk."time")), _hyper_1_2_chunk.hostname
                         Sort Method: external merge  Disk: 394720kB
                         Worker 0:  Sort Method: external merge  Disk: 150496kB
                         Worker 1:  Sort Method: external merge  Disk: 397320kB
                         ->  Result  (cost=0.00..829638.08 rows=9692215 width=25) (actual time=116.879..94302.311 rows=7680000 loops=3)
                               ->  Parallel Append  (cost=0.00..708485.39 rows=9692215 width=25) (actual time=116.857..88726.340 rows=7680000 loops=3)
                                     ->  Parallel Index Scan using _hyper_1_2_chunk_cpu_time_idx on _hyper_1_2_chunk  (cost=0.44..254091.97 rows=2492192 width=25) (actual time=208.773..59973.717 rows=1920000 loops=3)
                                           Index Cond: (("time" >= '2021-01-01 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 00:00:00+08'::timestamp with time zone))
                                     ->  Parallel Seq Scan on _hyper_1_1_chunk  (cost=0.00..405932.35 rows=7200023 width=25) (actual time=19.255..40519.150 rows=8640000 loops=2)
                                           Filter: (("time" >= '2021-01-01 00:00:00+08'::timestamp with time zone) AND ("time" < '2021-01-02 00:00:00+08'::timestamp with time zone))
     Planning Time: 276.695 ms
     Execution Time: 196843.845 ms 

![](https://upload-images.jianshu.io/upload_images/20224274-405b919333f84b3c.png)

图 6 基于时间聚合的对比

由于 TimescaleDB 的时空分区，我们的数据集自然地被划分成块，每个块都有自己的索引。这在使用基于时间的聚合处理查询时有几个好处。在各种块上使用约束排除，TimescaleDB 可以扫描更少的数据，允许数据库选择更合适的计划：例如，完全在内存中执行计算与溢出到磁盘，使用更小的索引（因此可以遍历更少的索引值），使用更好的算法，如 HashAggregates 与 GroupAggregates 等。

**基于时间排序的查询**

1、分组查询后 limit 获取

    explain analyze select date_trunc('minute', time) as minute, max(usage_user) from cpu where time < '2021-01-01 19:00:00' group by minute order by minute desc limit 5; 

\-- 普通表

     Limit  (cost=743688.63..743689.91 rows=5 width=16) (actual time=160970.872..161024.248 rows=5 loops=1)
       ->  Finalize GroupAggregate  (cost=743688.63..747817.53 rows=16138 width=16) (actual time=160970.866..160970.876 rows=5 loops=1
    )
             Group Key: (date_trunc('minute'::text, "time"))
             ->  Gather Merge  (cost=743688.63..747454.42 rows=32276 width=16) (actual time=160970.840..161024.208 rows=16 loops=1)
                   Workers Planned: 2
                   Workers Launched: 2
                   ->  Sort  (cost=742688.60..742728.95 rows=16138 width=16) (actual time=160904.119..160904.186 rows=442 loops=3)
                         Sort Key: (date_trunc('minute'::text, "time")) DESC
                         Sort Method: quicksort  Memory: 55kB
                         Worker 0:  Sort Method: quicksort  Memory: 55kB
                         Worker 1:  Sort Method: quicksort  Memory: 55kB
                         ->  Partial HashAggregate  (cost=741358.98..741560.70 rows=16138 width=16) (actual time=160903.347..160903.77
    6 rows=660 loops=3)
                               Group Key: date_trunc('minute'::text, "time")
                               ->  Parallel Index Scan using cpu_time_idx on cpu  (cost=0.57..705526.81 rows=7166434 width=16) (actual
     time=43.759..158098.601 rows=5280000 loops=3)
                                     Index Cond: ("time" < '2021-01-01 19:00:00+08'::timestamp with time zone)
     Planning Time: 136.133 ms
     Execution Time: 161025.264 ms 

\-- 超表

     Limit  (cost=0.44..22819.45 rows=5 width=16) (actual time=296.532..1300.620 rows=5 loops=1)
       ->  GroupAggregate  (cost=0.44..912761.09 rows=200 width=16) (actual time=296.527..1300.604 rows=5 loops=1)
             Group Key: (date_trunc('minute'::text, cpu."time"))
             ->  Custom Scan (ChunkAppend) on cpu  (cost=0.44..827155.96 rows=17120526 width=16) (actual time=32.262..1270.585 rows=12
    0001 loops=1)
                   Order: date_trunc('minute'::text, cpu."time") DESC
                   ->  Index Scan using _hyper_1_1_chunk_cpu_time_idx on _hyper_1_1_chunk  (cost=0.44..784354.64 rows=17120526 width=1
    6) (actual time=32.257..1230.772 rows=120001 loops=1)
                         Index Cond: ("time" < '2021-01-01 19:00:00+08'::timestamp with time zone)
     Planning Time: 261.787 ms
     Execution Time: 1300.790 ms 

![](https://upload-images.jianshu.io/upload_images/20224274-d40c1127d16bc93d.png)

图 7 分组查询后 limit 获取

数据库数据量越大，这个时间差会拉开得越大

**删除数据库数据**

时间序列数据通常建立得非常快，因此需要数据保留策略，例如 “仅将原始数据存储一周”。TimescaleDB 提供了数据管理工具，使实施数据保留策略变得容易，而且性能明显高于 PostgreSQL。

事实上，将数据保留策略与聚合的使用相结合是很常见的，因此有时可能保留两个超级表：一个包含原始数据，另一个包含数据汇总到每分钟或每小时的聚合表中。然后，可以在两个超级表上定义不同的保留策略，例如，将聚合数据存储更长时间，而只保留原始数据一个月。

TimescaleDB 通过 drop_chunks() 功能在块级别而不是行级别有效删除旧数据。比如之前我们这 1 亿数据，在超表中以 12h 一个块存储，会有 6 个块。下面将比较删除一天数据（因为前面数据是以 UTC 为准，所以超表向后推 8 小时）  
-- 普通表

    normal=# explain analyze delete from cpu where time < '2021-01-02 00:00:00';
                                                                        QUERY PLAN                                                    
                    
    ----------------------------------------------------------------------------------------------------------------------------------
    ----------------
     Delete on cpu  (cost=0.57..1146936.37 rows=25035703 width=6) (actual time=388605.156..388605.157 rows=0 loops=1)
       ->  Index Scan using cpu_time_idx on cpu  (cost=0.57..1146936.37 rows=25035703 width=6) (actual time=65.239..343970.066 rows=23
    040000 loops=1)
             Index Cond: ("time" < '2021-01-02 00:00:00+08'::timestamp with time zone)
     Planning Time: 82.617 ms
     Execution Time: 388693.189 ms 

\-- 超表

    hypertable=# explain analyse select drop_chunks('cpu','2021-01-02 00:00:00');
                                                QUERY PLAN                                            
    --------------------------------------------------------------------------------------------------
     ProjectSet  (cost=0.00..5.02 rows=1000 width=32) (actual time=1107.801..1107.812 rows=1 loops=1)
       ->  Result  (cost=0.00..0.01 rows=1 width=0) (actual time=0.007..0.007 rows=1 loops=1)
     Planning Time: 0.168 ms
     Execution Time: 1119.857 ms 

![](https://upload-images.jianshu.io/upload_images/20224274-288b386956c6ebaa.png)

图 8 清理数据

TimescaleDB 的 drop_chunks 从超表中删除所有只包含早于指定持续时间的数据的块。因为块是单独的表，删除的结果是简单地从文件系统中删除一个文件，因此非常快，在 10 毫秒内完成。同时，PostgreSQL 需要几分钟才能完成，因为它必须删除表中的单数据行。

TimescaleDB 的方法还避免了底层数据库文件中的碎片，从而避免了在非常大的表中可能额外昂贵（即耗时）的清理需要。

## 五、用于时间序列操作的新功能

在单磁盘机器上，许多只执行索引查找或表扫描的简单查询在 PostgreSQL 和 TimescaleDB 之间的性能相似。例如上述性能测试的简单查询。

TimescaleDB 除了支持完整的 SQL 外，还有许多定制的 SQL 函数，可以帮助我们以更少的代码行执行时间序列分析。

1、time_bucket() - 用于分析任意时间间隔的数据

可以将 time_bucket 看作 PostgreSQL date_trunc 函数的增强版。time_bucket 允许任意时间间隔（例如 10 秒、15 分钟、6 小时），而 date_trunc 仅提供标准日、分、小时。

示例：每个 host 在 1 小时内每 5 分钟的某个 cpu 指标平均值  
这是 postgres 中用于时间段的解决方法之一 [Stack Overflow](https://links.jianshu.com/go?to=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F12045600%2Fpostgresql-sql-group-by-time-interval-with-arbitrary-accuracy-down-to-milli-sec)

    SELECT
        c.hostname, i.start_time, AVG ( C.usage_user ) 
    FROM cpu AS C 
    RIGHT JOIN (
        SELECT
            ( SELECT MIN ( TIME ) :: DATE FROM cpu ) + ( n || ' minutes' ) :: INTERVAL start_time,
            ( SELECT MIN ( TIME ) :: DATE FROM cpu ) + ( ( n + 5 ) || ' minutes' ) :: INTERVAL end_time 
        FROM
            generate_series ( 0, ( 24 * 60 ), 5 ) n 
    ) AS i ON C.TIME >= i.start_time AND C.TIME < i.end_time 
    WHERE
        C.TIME >= '2021-01-02 12:00:00' AND C.TIME < '2021-01-02 13:00:00' 
    GROUP BY
        i.start_time, C.hostname 
    ORDER BY
        c.hostname asc, i.start_time ASC; 

下面是 timescaleDB 解决方法：

    SELECT
        hostname,
        time_bucket ( '5 minute', TIME ) AS fifteen,
        AVG ( usage_user ) AS avg_usage_user 
    FROM
        cpu 
    WHERE
        TIME >= '2021-01-02 12:00:00' 
        AND TIME < '2021-01-02 13:00:00' 
    GROUP BY
        fifteen,
        hostname 
    ORDER BY
        hostname ASC, fifteen ASC 

不仅更简洁，而且查询速度也差了几十倍。

2、last()，first() - 用于分析聚合中的数据

last() 和 first() 一般用于聚合函数中，可用于查询组中的最后一条 / 最后一条。

示例：1 小时内，每个 host 每分钟最后一条数据。  
postgres

    SELECT
        hostname, time, dt, usage_user 
    FROM
        (
        SELECT ROW_NUMBER ( ) OVER ( PARTITION BY dt, hostname ORDER BY TIME DESC ) row_id,
            dt, hostname, time, usage_user 
        FROM
            ( SELECT hostname, time, usage_user, date_trunc( 'minute', TIME ) dt FROM cpu WHERE time> '2021-01-03 00:00:00' AND time< '2021-01-03 01:00:00' ) AS t 
        ) AS rs
    WHERE
        rs.row_id = 1 
    ORDER BY
        hostname,
    TIME 

timescaleDB

    SELECT
        time_bucket ( '1 minute', time) one_min, hostname, LAST ( usage_user, time) 
    FROM cpu 
    WHERE
        time>= '2021-01-03 00:00:00' AND time< '2021-01-03 01:00:00'
    GROUP BY
        one_min, hostname 
    ORDER BY
        hostname, one_min 

3、time_bucket_gapfill() - 填充间隙

有时我们的数据中间丢失了一段，会导致数据不连续，如果有这方面的需求，就必须通过一些手段生成缺失数据。  
例如：获取上周每个设备每天的温度，插入缺失的读数：

    SELECT
      time_bucket_gapfill('1 day', time, now() - INTERVAL '1 week', now()) AS day,
      device_id,
      avg(temperature) AS value,
      interpolate(avg(temperature))
    FROM metrics
    WHERE time > now () - INTERVAL '1 week'
    GROUP BY day, device_id
    ORDER BY day;

               day          | device_id | value | interpolate
    ------------------------+-----------+-------+-------------
     2019-01-10 01:00:00+01 |         1 |       |
     2019-01-11 01:00:00+01 |         1 |   5.0 |         5.0
     2019-01-12 01:00:00+01 |         1 |       |         6.0
     2019-01-13 01:00:00+01 |         1 |   7.0 |         7.0
     2019-01-14 01:00:00+01 |         1 |       |         7.5
     2019-01-15 01:00:00+01 |         1 |   8.0 |         8.0
     2019-01-16 01:00:00+01 |         1 |   9.0 |         9.0
    (7 row) 

一般来说，单独使用 time_bucket_gapfill() 只会填充缺失时间点，但是不会填充数据，要配合 locf() 和 interpolate() 来使用。

-   locf()：可让将空值填充成聚合分组中前一个分组的值。
-   interpolate()：对缺失值进行线性插值。

timescaleDB 还提供了很多聚合、统计、计算等 API，可前往官网查阅，此处不一一赘述。[TimescaleDB API 参考](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.timescale.com%2Fapi%2Flatest%2F%23apis-grouped-by-related-feature)

更多精彩内容，就在简书 APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)
](https://www.jianshu.com/u/136525c3d5b5)

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   一、测试环境 操作系统：CentOS 7.8（虚拟机）Postgresql 版本：10.11.3CPU：Intel(...

    [![](https://upload-images.jianshu.io/upload_images/20224274-1d4aa56a189d6f5b.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/c646f26feb04)
-   PostgreSQL , TimescaleDB , 时间序列 , 物联网 , IoT 背景 随着物联网的发展，时...


-   最近看到这个开源时序数据库, 基于 postgrSQL,github 已经 2600 多颗星, 适合传感器采集�数据的存储以及...

    [![](https://upload.jianshu.io/users/upload_avatars/1650675/64e266f08b82.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    卓\_然](https://www.jianshu.com/u/0a3d1330fd66)阅读 7,645 评论 0 赞 4

    [![](https://upload-images.jianshu.io/upload_images/1650675-503691e8b6bd0fb2.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/3e2e5847f70a)
-   转载自：[https://www.cnblogs.com/Detector/p/10104254.html](https://www.cnblogs.com/Detector/p/10104254.html) http...

    [![](https://upload-images.jianshu.io/upload_images/19660942-2ff32291e4e7790d.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/cb807170d648)
-   作者：Micheal，腾讯资深后台开发工程师。 WeTest 导读服务器性能测试是一项非常重要而且必要的工作，本文是...

    [![](https://cdn2.jianshu.io/assets/default_avatar/8-a356878e44b45ab268a3b0bbaaadeeb7.jpg)
    饭盒](https://www.jianshu.com/u/9b6a6b2cd6e6)阅读 816 评论 1 赞 11

    [![](https://upload-images.jianshu.io/upload_images/1944350-cf661d98482226c2.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/5fd97d9eee51)
-   1\. 服务器性能测试小结 讲到服务器性能大部分人会想到这个服务器的架构是什么样子的，用的什么 epoll，select...


-   引言 谈到性能测试，部分公司连专门用于性能测试的环境都没有，更别提性能测试框架 / 平台了。下面，笔者就 “基于 Jmet...

    [![](https://upload-images.jianshu.io/upload_images/5997551-30048e5f8cb6a803.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/3d2d0ee350d4)
-   前言 InfluxDB 是一个年轻的时序数据库，是用同样很年轻的语言 GO 开发出来的。小数据量的时候还性能还不错，但是...

    [![](https://upload-images.jianshu.io/upload_images/22668799-69270ec1cf3a5bbb.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/f91a98bbb7b3)
-   当前有一个服务的日志表，需要每日进行清理。当前的做法是 truncate，但是会导致业务 1 分钟左右不能正常提供服务。...
-   本文我们将介绍如何使用 JMeter+InfluxDB+Grafana 打造压测可视化实时监控。 InfluxDB 是一...


-   性能指标分类 在进行性能测试的指标监控和结果分析时，可以关注以下这几个维度的指标： 系统性能指标 资源性能指标 数...

    [![](https://upload.jianshu.io/users/upload_avatars/13826085/c52262b7-8387-497e-88f6-02e52cd3e5f9?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    猪儿打滚](https://www.jianshu.com/u/2b4190d632dd)阅读 1,284 评论 0 赞 6

    [![](https://upload-images.jianshu.io/upload_images/13826085-dc84673c6968453e.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/791ffdacdc97)

-   版权声明: 本文由神州数码云基地团队整理撰写，若转载请注明出处。 一、前言 TiDB 作为 New SQL 数据库的...

    [![](https://upload-images.jianshu.io/upload_images/25446753-fe02059bb630004f.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/56f4cb7ffc8b)

-   今天给大家介绍的就是阿 贝云主机，阿 贝云是提供免 费虚拟主机免 费云服务器的，希望大家不要错过了。 讲到服务器性...

    [![](https://upload-images.jianshu.io/upload_images/2231755-13af85545b5161ad?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/54cb64a73d3f)

-   性能测试简介 概念: 通过自动化测试工具模拟多种正常、峰值以及异常负载条件来对系统的各项性能指标进行测试。 Why 评...

    [![](https://upload-images.jianshu.io/upload_images/10819934-f69c6ebce69e50a2.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/10896d189376)

-   !\[Flask](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAW...

-   双胎妊娠有家族遗传倾向，随母系遗传。有研究表明，如果孕妇本人是双胎之一，她生双胎的机率为 1/58；若孕妇的父亲或母...

-   今天理好了行李，看到快要九点了，就很匆忙的洗头洗澡，（心存一份念想，你总会打给我的🐶）然后把洗头液当成沐浴液了😨，...

-   那年我们 15，像阳光一样温暖的年纪。每天我都会骑自行车上学，路过田野，工厂，医院，村庄，有微风，有阳光，有绿...


-   最近身边有两对小两口在冷战，感觉战事随时能升级到要分开的地步。不幸的人各有各的不幸，然而在刚刚在一起的时候他们并不...

    [![](https://cdn2.jianshu.io/assets/default_avatar/9-cceda3cf5072bcdd77e8ca4f21c40998.jpg)
    苏觅觅](https://www.jianshu.com/u/6cf62758fa37)阅读 468 评论 0 赞 1

    [![](https://upload-images.jianshu.io/upload_images/884232-47152b5779268c17.jpeg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/90395c96f270)
-   嗟乎！余将弱冠也矣！ 时序轮替，白驹过隙。光阴荏苒，岁逢年关。弃勺舞而就象，久志学而将冠。 东隅已逝，桑榆恐迟。资...
       [![](https://upload-images.jianshu.io/upload_images/794326-7ed02761507f9d09.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
       ](https://www.jianshu.com/p/3d742eead224) 
    [https://www.jianshu.com/p/125ff16d79f9](https://www.jianshu.com/p/125ff16d79f9)
