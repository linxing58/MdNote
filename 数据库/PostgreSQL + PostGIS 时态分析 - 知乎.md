# PostgreSQL + PostGIS 时态分析 - 知乎
## 背景

假设我们有一些物体的轨迹数据（经纬度、measure(通常存为 epoch 时间戳)），比如车辆、人、传感器等。

给定一个物体在某个时间范围的轨迹数据，查找有没有与这个物体接触的轨迹，并按亲密度排序。

[http://postgis.net/docs/manual-2.4/geometry_distance_cpa.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/geometry_distance_cpa.html)

需求：

1、判断两个轨迹是否有亲密接触的可能。

2、如果有亲密接触的可能，那么是在什么时间点（measure）发生的。

3、如果有亲密接触的可能，那么他们接触的距离有多近。

4、如果有亲密接触的可能，那么他们最近距离接触的点是哪个。

使用 PostGIS，可以满足相应的需求。

[http://postgis.net/docs/manual-2.4/reference.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/reference.html)

```text
8.13. Temporal Support  
  
ST_IsValidTrajectory — Returns true if the geometry is a valid trajectory.  
  
ST_ClosestPointOfApproach — Returns the measure at which points interpolated along two lines are closest.  
  
ST_DistanceCPA — Returns the distance between closest points of approach in two trajectories.  
  
ST_CPAWithin — Returns true if the trajectories' closest points of approach are within the specified distance.  

```

时态分析可以做很多事情，比如地下活动。

## 1、轨迹格式

轨迹为带有 measure 的 linestring，包括：经纬度 (支持 Z 轴，3D 坐标)、measure(递增)。

通常使用时间戳 epoch 来存储 measure 的内容。

例如：

```text
postgres=# select extract(epoch from now());  
    date_part       
------------------  
 1528347341.02161  
(1 row)  
  
postgres=# select extract(epoch from now());  
    date_part       
------------------  
 1528347342.99521  
(1 row)  
  
postgres=# select ST_MakeLine(ST_MakePointM(-350,300,1528347341),ST_MakePointM(-410,490,1528347342));  
                                                    st_makeline                                                       
--------------------------------------------------------------------------------------------------------------------  
 0102000040020000000000000000E075C00000000000C07240000040B32EC6D6410000000000A079C00000000000A07E40000080B32EC6D641  
(1 row)  
  
postgres=# select st_astext(ST_MakeLine(ST_MakePointM(-350,300,1528347341),ST_MakePointM(-410,490,1528347342)));  
                       st_astext                          
--------------------------------------------------------  
 LINESTRING M (-350 300 1528347341,-410 490 1528347342)  
(1 row)
```

## 2、计算两个轨迹是否有近距离接触

指定距离阈值，判断两个轨迹是否有近距离接触。

[http://postgis.net/docs/manual-2.4/ST_CPAWithin.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/ST_CPAWithin.html)

Name

```text
ST_CPAWithin — Returns true if the trajectories' closest points of approach are within the specified distance.  

```

Synopsis

```text
float8 ST_CPAWithin(geometry track1, geometry track2, float8 maxdist);
```

例子

```text
WITH inp AS ( SELECT  
  ST_AddMeasure('LINESTRING Z (0 0 0, 10 0 5)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) a,  
  ST_AddMeasure('LINESTRING Z (0 2 10, 12 1 2)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) b  
)  
SELECT ST_CPAWithin(a,b,2), ST_DistanceCPA(a,b) distance FROM inp;  
  
 st_cpawithin |     distance  
--------------+------------------  
 t            | 1.96521473776207
```

## 3、计算两个轨迹近距离接触的 MEASURE(时间戳)

[http://postgis.net/docs/manual-2.4/ST_ClosestPointOfApproach.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/ST_ClosestPointOfApproach.html)

Name

```text
ST_ClosestPointOfApproach — Returns the measure at which points interpolated along two lines are closest.
```

Synopsis

```text
float8 ST_ClosestPointOfApproach(geometry track1, geometry track2);
```

例子

```text
-- Return the time in which two objects moving between 10:00 and 11:00  
-- are closest to each other and their distance at that point  
WITH inp AS ( SELECT  
  ST_AddMeasure('LINESTRING Z (0 0 0, 10 0 5)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) a,  
  ST_AddMeasure('LINESTRING Z (0 2 10, 12 1 2)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) b  
), cpa AS (  
  SELECT ST_ClosestPointOfApproach(a,b) m FROM inp  
), points AS (  
  SELECT ST_Force3DZ(ST_GeometryN(ST_LocateAlong(a,m),1)) pa,  
         ST_Force3DZ(ST_GeometryN(ST_LocateAlong(b,m),1)) pb  
  FROM inp, cpa  
)  
SELECT to_timestamp(m) t,  
       ST_Distance(pa,pb) distance  
FROM points, cpa;  
  
               t               |     distance  
-------------------------------+------------------  
 2015-05-26 10:45:31.034483+02 | 1.96036833151395
```

## 4、计算两个轨迹近距离接触的位置

[http://postgis.net/docs/manual-2.4/ST_LocateAlong.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/ST_LocateAlong.html)

Name

```text
ST_LocateAlong — Return a derived geometry collection value with elements that match the specified measure. Polygonal elements are not supported.
```

Synopsis

```text
geometry ST_LocateAlong(geometry ageom_with_measure, float8 a_measure, float8 offset);
```

例子 1

```text
SELECT ST_AsText(the_geom)  
                FROM  
                (SELECT ST_LocateAlong(  
                        ST_GeomFromText('MULTILINESTRINGM((1 2 3, 3 4 2, 9 4 3),  
                (1 2 3, 5 4 5))'),3) As the_geom) As foo;  
  
                                                 st_asewkt  
-----------------------------------------------------------  
 MULTIPOINT M (1 2 3)  
  
--Geometry collections are difficult animals so dump them  
--to make them more digestable  
SELECT ST_AsText((ST_Dump(the_geom)).geom)  
        FROM  
        (SELECT ST_LocateAlong(  
                        ST_GeomFromText('MULTILINESTRINGM((1 2 3, 3 4 2, 9 4 3),  
        (1 2 3, 5 4 5))'),3) As the_geom) As foo;  
  
   st_asewkt  
---------------  
 POINTM(1 2 3)  
 POINTM(9 4 3)  
 POINTM(1 2 3)
```

例子 2

```text
postgres=# WITH inp AS ( SELECT  
postgres(#   ST_AddMeasure('LINESTRING Z (0 0 0, 10 0 5)'::geometry,  
postgres(#     extract(epoch from '2015-05-26 10:00'::timestamptz),  
postgres(#     extract(epoch from '2015-05-26 11:00'::timestamptz)  
postgres(#   ) a,  
postgres(#   ST_AddMeasure('LINESTRING Z (0 2 10, 12 1 2)'::geometry,  
postgres(#     extract(epoch from '2015-05-26 10:00'::timestamptz),  
postgres(#     extract(epoch from '2015-05-26 11:00'::timestamptz)  
postgres(#   ) b  
postgres(# ), cpa AS (  
postgres(#   SELECT ST_ClosestPointOfApproach(a,b) m FROM inp  
postgres(# ) select ST_LocateAlong(a, m), ST_LocateAlong(b, m) from inp,cpa;  
                                        st_locatealong                                        |                                        st_locatealong                                          
----------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------  
 01040000C00100000001010000C03A8EE39E46581E4000000000000000003A8EE39E46580E40F734C292F758D541 | 01040000C00100000001010000C02222222CF7342240E9933E8DB0DCF33FA44FFA34C2720F40F734C292F758D541  
(1 row)  
  
postgres=# WITH inp AS ( SELECT  
  ST_AddMeasure('LINESTRING Z (0 0 0, 10 0 5)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) a,  
  ST_AddMeasure('LINESTRING Z (0 2 10, 12 1 2)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) b  
), cpa AS (  
  SELECT ST_ClosestPointOfApproach(a,b) m FROM inp  
) select st_astext(ST_LocateAlong(a, m)), st_astext(ST_LocateAlong(b, m)) from inp,cpa;  
                              st_astext                               |                                      st_astext                                        
----------------------------------------------------------------------+-------------------------------------------------------------------------------------  
 MULTIPOINT ZM (7.58620689643754 0 3.79310344821877 1432608331.03448) | MULTIPOINT ZM (9.10344827572505 1.24137931035625 3.93103448284997 1432608331.03448)  
(1 row)
```

## 5、计算两个轨迹近距离接触的最近的距离

[http://postgis.net/docs/manual-2.4/ST_DistanceCPA.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/ST_DistanceCPA.html)

Name

```text
ST_DistanceCPA — Returns the distance between closest points of approach in two trajectories.
```

Synopsis

```text
float8 ST_DistanceCPA(geometry track1, geometry track2);
```

例子

```text
-- Return the minimum distance of two objects moving between 10:00 and 11:00  
WITH inp AS ( SELECT  
  ST_AddMeasure('LINESTRING Z (0 0 0, 10 0 5)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) a,  
  ST_AddMeasure('LINESTRING Z (0 2 10, 12 1 2)'::geometry,  
    extract(epoch from '2015-05-26 10:00'::timestamptz),  
    extract(epoch from '2015-05-26 11:00'::timestamptz)  
  ) b  
)  
SELECT ST_DistanceCPA(a,b) distance FROM inp;  
  
     distance  
------------------  
 1.96036833151395
```

## 6、索引加速

[http://postgis.net/docs/manual-2.4/geometry_distance_cpa.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/geometry_distance_cpa.html)

Name

```text
|=| — Returns the distance between A and B trajectories at their closest point of approach.
```

Synopsis

```text
double precision |=|( geometry A , geometry B );
```

Description

```text
The |=| operator returns the 3D distance between two trajectories (See ST_IsValidTrajectory).   
  
This is the same as ST_DistanceCPA but as an operator it can be used for doing nearest   
neightbor searches using an N-dimensional index   
  
(requires PostgreSQL 9.5.0 or higher).  

```

例子

```text
-- Save a literal query trajectory in a psql variable...  
\set qt 'ST_AddMeasure(ST_MakeLine(ST_MakePointM(-350,300,0),ST_MakePointM(-410,490,0)),10,20)'  
-- Run the query !  
SELECT track_id, dist FROM (  
  SELECT track_id, ST_DistanceCPA(tr,:qt) dist  
  FROM trajectories  
  ORDER BY tr |=| :qt  
  LIMIT 5  
) foo;  
 track_id        dist  
----------+-------------------  
      395 | 0.576496831518066  
      380 |  5.06797130410151  
      390 |  7.72262293958322  
      385 |   9.8004461358071  
      405 |  10.9534397988433  
(5 rows)
```

## 压测

1、创建一个生成随机轨迹的函数

```text
create or replace function gen_rand_linestring (  
  seedx float8,    -- 起点x坐标  
  seedy float8,    -- 起点y坐标  
  seedstep float8, -- 运动步调最大值  
  seedts float8,   -- 时间，measure 步调(second 单位)  
  ts1 timestamp,   -- 时间区间最小值  
  ts2 timestamp,   -- 时间区间最大值  
  cnt int          -- 在轨迹中生成几个POINT  
) returns geometry as $$  
declare  
  ts timestamp := ts1 + ((random()*extract(epoch from (ts2-ts1)))||' sec')::interval;  
  ts_epo float8;  
  geo geometry[];  
begin  
  for i in 1..cnt loop   
    ts_epo := extract(epoch from ts+((i*seedts)||' sec')::interval) ;  
    geo := array_cat(geo, array[st_makepointm(seedx + i*(random()*seedstep), seedy + i*(random()*seedstep), ts_epo)]) ;  
  end loop;   
  return st_makeline(geo);   
end;  
$$ language plpgsql strict stable;
```

2、例子

```text
postgres=# select gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10);  
-[ RECORD 1 ]-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
gen_rand_linestring | 01020000400A0000000000503BA9CB124000004062BCE20940A4E1CEC382C6D6410000C0637DE10B400000C0C50AB82F40A4E1CEE182C6D64100005057B1D511400000C4CBE74C3040A4E1CEFF82C6D6410000C0558CD11C400000C09177B72940A4E1CE1D83C6D64100004067EAE646400000884E67E02C40A4E1CE3B83C6D641000088FAE5AA4240000000AD26AF2740A4E1CE5983C6D64100007CD8F8274F400000B8B89E4D2F40A4E1CE7783C6D641000000823E6B27400000902247B34C40A4E1CE9583C6D6410000D863843C56400000C06676921440A4E1CEB383C6D64100000C90922455400000386AD2743D40A4E1CED183C6D641  
  
postgres=# select gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10);  
-[ RECORD 1 ]-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
gen_rand_linestring | 01020000400A0000000000D007926420400000D089B6B52540115366F26EC6D6410000200020CB16400000C83382CD3540115366106FC6D6410000481EF79F32400000643974B938401153662E6FC6D6410000B0230B473C400000806A901F3C401153664C6FC6D6410000F4DE658733400000B85EDB112E401153666A6FC6D6410000C0C5B30603400000802221A63F40115366886FC6D6410000E4045AF23B4000001EDEDB0E4A40115366A66FC6D64100007089A6E048400000D0AD8FBB5140115366C46FC6D641000090384ED9534000007A4EF7515440115366E26FC6D641000084005E3153400000102214F72B401153660070C6D641  
  
  
postgres=# select st_astext(gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10));  
-[ RECORD 1 ]---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
st_astext | LINESTRING M (10.2174849007279 11.9571549287066 1528425468.02122,14.630109699443 3.11233511939645 1528425588.02122,21.6466929381713 21.7603580309078 1528425708.02122,4.62157236784697 22.9575021080673 1528425828.02122,47.6620979132131 23.719867666252 1528425948.02122,15.6148129086941 51.1239671874791 1528426068.02122,47.852089674212 18.3603029865772 1528426188.02122,33.4982615187764 36.1437245160341 1528426308.02122,83.4090811889619 7.33796327654272 1528426428.02122,80.737452045083 19.6353993359953 1528426548.02122)  
  
postgres=# select st_astext(gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10));  
-[ RECORD 1 ]-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
st_astext | LINESTRING M (3.91523572057486 7.61306651681662 1528390545.99534,16.9645196422935 20.0192420165986 1528390665.99534,13.6881912257522 22.0785792060196 1528390785.99534,32.2743593193591 9.22613327205181 1528390905.99534,40.4555864389986 15.4170794393867 1528391025.99534,7.14410931244493 49.0896332990378 1528391145.99534,67.4892951631919 13.0613004816696 1528391265.99534,38.844025567174 50.6820540204644 1528391385.99534,23.3702098755166 91.7289085481316 1528391505.99534,55.1767633520067 70.295524129644 1528391625.99534)  
  
  
postgres=# select st_isvalidtrajectory(gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10));  
 st_isvalidtrajectory   
----------------------  
 t  
(1 row)  
  
postgres=# select st_isvalidtrajectory(gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10));  
 st_isvalidtrajectory   
----------------------  
 t  
(1 row)  
  
postgres=# select st_isvalidtrajectory(gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10));  
 st_isvalidtrajectory   
----------------------  
 t  
(1 row)
```

3、建表

```text
create table tbl_trc(uid int, tc geometry);
```

4、创建空间索引

```text
create index idx_tbl_trc_1 on tbl_trc using gist (tc);
```

5、压测

每 600 秒跟踪 60 个点。

```text
vi test.sql  
  
  
\set uid random(1,100000000)  
\set x random(1,1000)  
\set y random(1,1000)  
insert into tbl_trc values (:uid, gen_rand_linestring(:x,:y,10,600,now()::timestamp,(now()+interval '1 day')::timestamp,60));
pgbench -M prepared -n -r -P 1 -f ./test.sql -c 32 -j 32 -T 1200 -h 127.0.0.1
```

6、查询测试

```text
with a as (select gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10) tc)  
select uid, tbl_trc.tc |=| a.tc, st_astext(tbl_trc.tc), st_astext(a.tc) from tbl_trc,a where ST_CPAWithin(tbl_trc.tc, a.tc, 100) limit 10;
with a as (select gen_rand_linestring(1,2,10,120,now()::timestamp,(now()+interval '1 day')::timestamp,10) tc)  
select uid, tbl_trc.tc |=| a.tc from tbl_trc,a where ST_CPAWithin(tbl_trc.tc, a.tc, 100) limit 10;  
  
  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
NOTICE:  Could not find point with M=1.5284e+09 on first geom  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
NOTICE:  Could not find point with M=1.52839e+09 on first geom  
   uid    |     ?column?       
----------+------------------  
 97041413 | 47.2605527818083  
   627623 | 89.2835207317297  
 65446299 |  75.474589493268  
 86176035 | 52.5219858940626  
 81450777 | 63.8253100431266  
 97175328 | 61.8835334486386  
 99585779 | 34.6884977809673  
 73552807 | 17.7467079053642  
 27098713 | 68.9482850607433  
 84443616 | 62.2849622517964  
(10 rows)
```

## 小结

使用 PostgreSQL + PostGIS 提供的 Temporal Support 以及时态分析索引，可以高效的实现轨迹的时态分析。

适应范围广泛，比如

地下情挖掘，一些无法琢磨的亲密关系，在时态分析上都可以做。又比如动物的发情期活动，哪些动物发生过交配等. 再比如在货运、私有运输行业，存在偷油现象，也可以用时态分析来洞察。

功能点：

1、判断两个轨迹是否有亲密接触的可能。

2、如果有亲密接触的可能，那么是在什么时间点（measure）发生的。

3、如果有亲密接触的可能，那么他们接触的距离有多近。

4、如果有亲密接触的可能，那么他们最近距离接触的点是哪个。

时空数据应用有很多好玩的场景，快来学习 PostgreSQL.

[《Oracle DBA 增值 PostgreSQL,Greenplum 学习计划》](https://link.zhihu.com/?target=https%3A//github.com/digoal/blog/blob/master/201804/20180425_01.md)

## 参考

[《PostgreSQL PostGIS 的 5 种空间距离排序 (knn) 算法》](https://link.zhihu.com/?target=https%3A//github.com/digoal/blog/blob/master/201806/20180605_02.md)

[http://postgis.net/docs/manual-2.4/geometry_distance_cpa.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/geometry_distance_cpa.html)

[http://postgis.net/docs/manual-2.4/reference.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/reference.html)

[http://postgis.net/docs/manual-2.4/ST_LocateAlong.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/ST_LocateAlong.html)

轨迹亲密接触搜索索引加速

[http://postgis.net/docs/manual-2.4/geometry_distance_cpa.html](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-2.4/geometry_distance_cpa.html) 
 [https://zhuanlan.zhihu.com/p/414519140](https://zhuanlan.zhihu.com/p/414519140)
