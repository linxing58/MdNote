# 使用pgRouting生成可达圈（等时圈） - 简书
[![](https://upload.jianshu.io/users/upload_avatars/12249756/c093d919-190f-4d9b-bc73-a3e632658b83?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/e320cc4a3572)

[圣瓦伦](https://www.jianshu.com/u/e320cc4a3572)IP属地: 浙江

2019.11.03 17:54:28字数 265阅读 2,552

pgRouting是地理空间数据库PostGIS的扩展插件，提供了路径规划的功能。

输入一个起始点（经纬度坐标）和驾车（步行、骑行）的时间，根据已有的路网，输出能到达的最大范围，并获得范围内的兴趣点。

主要使用pgRouting的pgr\_drivingDistance + ST\_ConcaveHull生成包围圈。

1.  [安装PostgreSQL/PostGIS/pgRouting](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fytwy%2Fp%2F6817179.html)
2.  [导入路网数据，生成拓扑](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fzhaoquanfeng%2Farticle%2Fdetails%2F84863889)

### 1.1 计算离目标点最近的节点

<-\> ：返回两点之间的距离

```
SELECT * FROM shenzhen_vertices_pgr
    ORDER BY the_geom <-> ST_GeometryFromText('POINT(113.954396 22.580677)',4326) 
    LIMIT 1; 
```

### 1.2 以该节点作为起点，给定时间成本，计算可达路网

pgr_drivingDistance :详情查看[pgRouting手册](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.pgrouting.org%2F2.3%2Fen%2Fsrc%2Fdriving_distance%2Fdoc%2Fpgr_drivingDistance.html)

```
with drivingDistance as(
SELECT
    sz.geom
FROM
    pgr_drivingDistance (
        'SELECT gid as id, source, target, distance/mean_speed as cost FROM shenzhen',  --路网表
        1034,  --节点的ID
        0.5*3600,    --总成本（时间）
        FALSE
    ) dd,
    shenzhen as sz
WHERE sz.gid = dd.edge
) 
```

![](https://upload-images.jianshu.io/upload_images/12249756-78f368713aa94d48.png)

路网范围

### 1.3 根据路网计算可达圈

```
SELECT  st_concavehull(st_collect(array(select * from drivingDistance)),0.7) 
```

st_concavehull：返回坐标点的包围圈，[详情](https://links.jianshu.com/go?to=http%3A%2F%2Fpostgis.net%2Fdocs%2FST_ConcaveHull.html)

![](https://upload-images.jianshu.io/upload_images/12249756-1a8ba32913fa9af5.png)

可达圈

路网和可达圈叠加：

```
SELECT  st_concavehull(st_collect(array(select * from drivingDistance)),0.7)
union
SELECT * FROM drivingDistance; 
```

![](https://upload-images.jianshu.io/upload_images/12249756-0c18756c28c3e42b.png)

叠加

### 1.4 根据可达圈计算圈内的POI（兴趣点）

主要用到的是st_contains函数

```
select t.* from 
(SELECT st_concavehull(st_collect(array(select * from drivingDistance)),0.9)) c 
left join traffic_facility t 
on st_contains(c.st_concavehull,t.geom)
where t.station_type IN ('SUBWAY'); 
```

![](https://upload-images.jianshu.io/upload_images/12249756-7de66302739c2a71.png)

POI和可达圈叠加

参考资料：
-----

[使用pgRouting可达范围计算](https://www.jianshu.com/p/96f135ba775e)  
[官网](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.pgrouting.org%2F2.3%2Fen%2Fsrc%2Fdriving_distance%2Fdoc%2Fpgr_drivingDistance.html)  
PostGIS介绍：[https://www.jianshu.com/p/66ea816f6fee](https://www.jianshu.com/p/66ea816f6fee)
