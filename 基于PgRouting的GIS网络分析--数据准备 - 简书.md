# 基于PgRouting的GIS网络分析--数据准备 - 简书
[![](https://upload.jianshu.io/users/upload_avatars/68979/ade680e57161?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/2cbf98ef6104)

12017.04.17 14:31:05字数 1,470阅读 9,538

PgRouting是基于开源空间数据库PostGIS用于网络分析的扩展模块，最初它被称作pgDijkstra，因为它只是利用Dijkstra算法实现最短路径搜索，之后慢慢添加了其他的路径分析算法，如A_算法，双向A_算法，Dijkstra算法，双向Dijkstra算法，tsp货郎担算法等，然后被更名为pgRouting\[1\]。该扩展库依托PostGIS自身的gist索引，丰富的坐标系与图形类型，强大的几何处理能力，如空间查询，空间处理，线性参考等优势，能保障在较大数据级别下的网络分析效果更快更好。

　　 　　PostGIS早已奠定了最优秀的开源空间数据库地位，在新时代GIS中的应用将会越来越普遍。其实，网络分析算法很多服务端语言如java，C#等虽能实现，但基于真实城市道路数据量较大且查询分析操作步骤复杂与数据库交互频繁，以这类服务端频繁访问数据库导致数据库开销压力较大，分析较慢，故选择PgRouting在数据库内部实现算法，提升分析效率。最后，路径分析不仅仅是最短路径，在实际应用中还有最短耗时，最近距离，道路对车辆类型限制，道路对速度限制等因素，交通事故、市政事故导致的交通障碍点等问题，所有的问题本质其实是对路径分析权重（Weight）的设置问题。

windows安装过程比较简单，安装PostgreSQL，PostGIS一直Next即可，安装完自带PgRouting扩展，这里主要以Centos7介绍安装过程：

1.  安装PostgreSQL，参考 [《Centos7安装PostgreSQL》](https://www.jianshu.com/p/639ebb43bfb4)；
2.  安装PostGIS，PgRouting，具体安装步骤参考[《CentOS 7源码安装PostGIS(包含SFCGAL,PgRouting)》](https://www.jianshu.com/p/e08dbc60a3b2)；

3.1 创建测试数据库
-----------

```
[root@localhost opt]# su - postgres
[postgres@localhost ~]$ psql
psql (9.6.1)
Type "help" for help.
postgres=# create database network;
CREATE DATABASE
postgres=# \c network
You are now connected to database "network" as user "postgres".
network=# create extension postgis;
CREATE EXTENSION
network=# create extension pgrouting;
CREATE EXTENSION 
```

3.2 导入测试路网数据
------------

PgRouting需要使用道路线型数据，建立道路连通性topo关系。由于路网分析的特殊性，只支持LineString类型，不支持MultiLineString类型。测试数据从[OSM下载](http://download.geofabrik.de/asia.html)得来。

![](https://upload-images.jianshu.io/upload_images/68979-c2c3cf8a39924f99.png)

数据下载.png

　　一般下载shp，使用PostGIS自带的shp2pgsql导入即可。其他格式可以使用osm2pgrouting工具，本文不做详述。笔者下载后，将路网数据使用ArcMap只截取了南京市范围内路网后，从epsg:4326转换成了epsg:3857坐标系，然后导入数据库，做测试数据。

```
[postgres@localhost ~]$ shp2pgsql -c -g geom -D -s 3857 -S -i -I /opt/roads.shp road | psql -d network
Shapefile type: Arc
Postgis type: LINESTRING[2]
SET
SET
BEGIN
CREATE TABLE
ALTER TABLE
                addgeometrycolumn                  
----------------------------------------------------
public.road.geom SRID:3857 TYPE:LINESTRING DIMS:2 
(1 row)

COPY 9731
CREATE INDEX
COMMIT
ANALYZE 
```

关于shp2pgsql参数问题，参考[http://www.jianshu.com/p/1251fdc603ac](https://www.jianshu.com/p/1251fdc603ac)。  
　　注意：使用shp2pgsql工具，shp路径不要太深，不要有中文。对于含有中文乱码的，可使用-W gbk等转码。

PgRouting提供pgr_createTopology方法，对道路数据创建拓扑关系，比如单一线要素，建立与其连通的source,tartget连通点。详细步骤如下：

```
#在network数据库中对导入的road表建立source,target字段
network=# alter table road add column source int;
ALTER TABLE
network=# alter table road add column target int;
ALTER TABLE
#创建连通性topo
#road是表名称，geom是该表的图形字段名称，gid是改变的主键id。
#一般我们使用shp2pgsql工具会自动创建gid为主键，geom为图形。
#如果是自己其他形式建立的表，注意参数写自己对应的字段
network=# SELECT pgr_createTopology('road', 0.00001, 'geom', 'gid'); 
```

正常情况下顺利完成以上步骤。  
　　由于在网络分析中频繁读取source,target字段的topo关系值，为了提升查询效率，需要对这两个字段添加索引：

```
network=# create index road_source_idx on road("source");
network=# create index road_target_idx on road("target"); 
```

3.4 路网元数据说明
-----------

到此为止，全部数据准备工作已经完成了，查看本文用于测试路径分析的数据描述如下：

```
network=
                                    Table "public.road"
  Column   |           Type            |                     Modifiers                      
-----------+---------------------------+----------------------------------------------------
 gid       | integer                   | not null default nextval('road_gid_seq'::regclass)
 direction | character varying(1)      | 
 roadname  | character varying(40)     | 
 oneway    | character varying(50)     | 
 geom      | geometry(LineString,3857) | 
 source    | integer                   | 
 target    | integer                   | 
Indexes:
    "road_pkey" PRIMARY KEY, btree (gid)
    "road_geom_idx" gist (geom)
    "road_source_idx" btree (source)
    "road_target_idx" btree (target) 
```

由此看出，road表拥有主键索引，geom的gist索引，source，target节点处索引，还有一般性的道路名称字段，比较重要的是direction和oneway字段（有一个即可），这两个字段都是说明道路真实方向的。

| direction | oneway | 含义 |
| --- | --- | --- |
| 0,1 | 空值 | 道路双向通行 |
| 2 | FT | 道路真实方向与数字化方向一致 |
| 3 | TF | 道路真实方向与数字化方向相反 |
| 4 | N | 道路禁止通行 |

3.5 通行成本权重设置
------------

在算法中分为有向图，无向图，图的path长度一般设置为权重，网络分析中，具体到比如交通领域，也分为双向通行道路，单向通行道路，交通事故导致的临时交通阻塞无法通行（障碍点），不同等级道路对车辆类型限制，比如高架，高速只允许机动车，乡间道路允许非机动车（条件限制因素），本节只是举个例子说明下不同的条件下如何设置通行成本权重：

*   双向通行：

```
update road set length=st_length(geom),rev_length=st_length(geom) where oneway is null;
--采用真实地理距离是这样：
update road set lenght=st_length(st_transform(geom),4326),true),rev_length=st_length((st_transform(geom),4326),true) where oneway is null; 
```

*   单向通行

```
#FT是道路方向与数字化方向一致，那么正向通行成本为道路长度，反向成本为正无穷（以极大值代替）
update road set length=st_length(geom),rev_length=99999999999 where oneway='FT';
update road set length=99999999999,rev_length=st_length(geom) where oneway='TF'; 
```

*   障碍点

```
#假设gid=20的道路因事故，修路暂时不能通行
update road set lenght=99999999999,rev_length=99999999999 where gid=20; 
```

*   限制通行  
    假设当前是一辆大货车，通过有限高限重的道路，在为他做规划时，先获取车辆类型，再查询road表中是否有对其限制的因素（以下纯逻辑描述sql）

```
#假设道路表有字段restrict，该字段是array，记录了不可通行的车辆类型
update road set lenght=99999999999,rev_length=99999999999 where 'lorry'=any(restrict); 
```

言归正传，本文从入门角度只阐述最简单的单向，双向通行道路的例子，其他概不设置限制。

4.1 创建cost
----------

```
alter table road add column length numeric;
alter table road add column rev_length numeric;
update road set length=ST_Length(ST_TransForm(geom,4326),true),rev_length=ST_Length(ST_TransForm(geom,4326),true) where oneway is null;
update road set length=st_length(ST_TransForm(geom,4326),true),rev_length=99999999999 where oneway='FT';
update road set length=99999999999,rev_length=st_length(ST_TransForm(geom,4326),true) where oneway='TF'; 
```

4.2 创建路径分析方法
------------

*   单点到单点  
    建立好网络topo的数据一定会有source,target字段，这是表达线的走向和连通性的关系，一条线的target，是与它尾部相连的其他线的source。

```
--忽视道路方向，每条路正反都可以通行
SELECT * FROM pgr_dijkstra(
    'SELECT gid AS id,
         source,
         target,
         cost
        FROM roads',
    1060, 1661,
    directed := false);
--遵循道路真实方向，有的道路是单行道，如高速的一侧，有的路是正反都可以通行，他们由cost,revcost决定的
SELECT * FROM pgr_dijkstra(
    'SELECT gid AS id,
         source,
         target,
         cost,rev_cost as reverse_cost 
        FROM roads',
    1060, 1661,
    directed := true); 
```

所有的方法都有单\-单，单\-多，多\-单，多\- 多，详细参考官方api

```
pgr_dijkstra(edges_sql, start_vid,  end_vid)
pgr_dijkstra(edges_sql, start_vid,  end_vid  [, directed])
pgr_dijkstra(edges_sql, start_vid,  end_vids [, directed])
pgr_dijkstra(edges_sql, start_vids, end_vid  [, directed])
pgr_dijkstra(edges_sql, start_vids, end_vids [, directed]) 
```

本例子上述使用的是：

```
--单-单，无方向
pgr_dijkstra(edges_sql, start_vid,  end_vid)
----单-单，有方向
pgr_dijkstra(edges_sql, start_vid,  end_vid, true) 
```

更多其他使用，请具体参考官方api，本文的方向，障碍点，限制车辆类型，限制车辆速度等等仅提供辅助参考作用，本质上都仅仅是设置roads的cost和rev_cost参数而已。

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

![](https://cdn2.jianshu.io/assets/default_avatar/wechat-771371d5741b0c32cd805caeb48ad6c0.png)
![](https://cdn2.jianshu.io/assets/default_avatar/wechat-771371d5741b0c32cd805caeb48ad6c0.png)
[![](https://cdn2.jianshu.io/assets/default_avatar/3-9a2bcc21a5d89e21dafc73b39dc5f582.jpg)
](https://www.jianshu.com/u/e847f401f821)共3人赞赏

[![](https://upload.jianshu.io/users/upload_avatars/68979/ade680e57161?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/2cbf98ef6104)

[遥想公瑾当年](https://www.jianshu.com/u/2cbf98ef6104 "遥想公瑾当年")专注于OpenSource GIS开发，从事开源gis开发与架构设计。<br>1 相关专业开发...

总资产194共写了4.4W字获得641个赞共886个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

*   //我所经历的大数据平台发展史（三）：互联网时代 • 上篇http://www.infoq.com/cn/arti...
    
    [![](https://upload-images.jianshu.io/upload_images/2569324-7b30ac45f279654a.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/cdbed82a34bc)
*   Android 自定义View的各种姿势1 Activity的显示之ViewRootImpl详解 Activity...
    

*   初冬的夜 星星隐匿，月亮也躲进云层 寒冷散去了街道的热闹嘈杂 昏黄路灯下 归家的影子匆匆聚集，又被迅速拉长 熟悉的...
    
    [![](https://upload.jianshu.io/users/upload_avatars/8918212/7fe1b5b4-9917-4257-ad9b-a3b8717f5fcb.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    雨夜行舟](https://www.jianshu.com/u/282ffd18384e)阅读 170评论 0赞 4