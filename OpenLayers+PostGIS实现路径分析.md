# OpenLayers+PostGIS实现路径分析
[![](https://upload.jianshu.io/users/upload_avatars/10338988/dae08ed6-09eb-409f-b237-89ca5b086603?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/4cc481eed7b7)

0.9662018.12.28 22:43:18字数 647阅读 3,891

  本文内容默认已在PostGIS中添加了PgRouting扩展模块。

  路径分析功能是针对路网图层进行的空间分析应用，在将shp文件导入数据库之前，需要对其进行拓扑检验，主要是道路的自相交以及路段之间的道路节点的相离的检验。可以利用ArcGIS对文件进行拓扑的构建和检验，保证数据的准确性。

  利用PostGIS自带的工具将文件入库。如下图所示，1是将编码方式设置为GBK，一般Shapfile文件的编码方式均为GBK，若选用其他编码方式，可能会出现乱码。2是对线要素进行简化，Pgrouting不支持对多线的路径计算。

  

![](https://upload-images.jianshu.io/upload_images/10338988-68563bf9e5f88832.png)

图1

设定空间参考系统标识（SRID）常用的3857或者4326，见下图。

![](https://upload-images.jianshu.io/upload_images/10338988-e1574b8ca5d2ae61.png)

图2

  要生成路径，首先要生成合法的拓扑。通过创建拓扑关系，可以完成最短路径查询函数的编写。

2.1 构建拓扑关系：pgr_createTopology
-----------------------------

  这里创建的是路段道路节点与弧段之间的关系，通过节点间可达性可以完成路径的导航。

```
1、 添加起止点，存储线段的首尾编号
ALTER TABLE prjroad ADD COLUMN source integer;
ALTER TABLE prjroad ADD COLUMN target integer;
2、添加道路权重值
ALTER TABLE prjroad ADD COLUMN length double precision;
3、创建拓扑，生成线段的首尾编号
SELECT pgr_createTopology(prjroad,0.00001, 'geom', 'gid');

第一步默认创建起止点的索引，如果没有创建，可以执行以下sql创建
CREATE INDEX source_idx ON prjroad ("source");
CREATE INDEX target_idx ON prjroad ("target");
4、为权重赋值，这里将路段的长度赋值给权重值
UPDATE prjroad SET length =st_length(geom);
5、回程成本设置，则可支持回程
ALTER TABLE prjroad ADD COLUMN reverse_cost double precision;
UPDATE prjroad SET reverse_cost = length; 
```

2.2 验证可行性
---------

pgr_dijkstra函数详见：

[http://docs.pgrouting.org/2.2/en/src/dijkstra/doc/pgr_dijkstra.html#pgr-dijkstra](http://docs.pgrouting.org/2.2/en/src/dijkstra/doc/pgr_dijkstra.html#pgr-dijkstra)

```
pgr_dijkstra(

text sql, __用于计算最佳路径的数据来源，用SOL表示

integer source, __规划路径的起点

integer target, __规划路径的终点

Boolean directed, __ if the graph is directed 有向图

Boolean has_rcost __if true 需要添加reverse_cost计算回程路径

) 
```

![](https://upload-images.jianshu.io/upload_images/10338988-d1c7583aae56fe5d.png)

查询节点2到节点36之间的最短路径，写入表dijkstra_res中。

```
SELECT seq, id1 AS node, id2 AS edge, cost,geom into dijkstra_res FROM pgr_dijkstra('

SELECT gid AS id,                   

source::integer,                      

target::integer,                     

length::double precision AS cost

FROM prjroad',

2, 36, false, false) as di

join prjroad pt

on di.id2 = pt.gid; 
```

![](https://upload-images.jianshu.io/upload_images/10338988-f2fc7ff9ae24bf79.png)

图2.1 可行性验证结果

将表导出，加载到qgis中，可看到最短路径如图2.2：

![](https://upload-images.jianshu.io/upload_images/10338988-9d5cf80387c07292.png)

图2.2 验证结果展示

2.3 功能函数fromAtoB
----------------

1、若使用A*算法，则需要知道起止点的坐标值，可以为prjroad表添加字段并赋值

```
ALTER TABLE prjroad ADD COLUMN x1 double precision;

ALTER TABLE prjroad ADD COLUMN y1 double precision;

ALTER TABLE prjroad ADD COLUMN x2 double precision;

ALTER TABLE prjroad ADD COLUMN y2 double precision;

UPDATE prjroad SET x1 =ST_x(ST_PointN(geom, 1));

UPDATE prjroad SET y1 =ST_y(ST_PointN(geom, 1));

UPDATE prjroad SET x2 =ST_x(ST_PointN(geom, ST_NumPoints(geom)));

UPDATE prjroad SET y2 =ST_y(ST_PointN(geom, ST_NumPoints(geom))); 
```

2、创建函数

```
//create function

--

--DROP FUNCTION pgr_fromAtoB(varchar, double precision, double precision,

-- double precision, double precision);

CREATE OR REPLACE FUNCTION pgr_fromAtoB(

 IN tbl varchar,

 IN x1 double precision,

 IN y1 double precision,

 IN x2 double precision,

 IN y2 double precision,

 OUT seq integer,

 OUT gid integer,

 OUT heading double precision,

 OUT cost double precision,

 OUT geom geometry

 )

 RETURNS SETOF record AS

$BODY$

DECLARE

 sql text;

 rec record;

 source integer;

 target integer;

 point integer;

BEGIN

 -- 查询距离出发点最近的道路节点

 EXECUTE 'SELECT id::integer FROM line_vertices_pgr

 ORDER BY the_geom <-> ST_GeometryFromText(''POINT('

 || x1 || ' ' || y1 || ')'',3857) LIMIT 1' INTO rec;

 source := rec.id;

 -- 查询距离目的地最近的道路节点

 EXECUTE 'SELECT id::integer FROM line_vertices_pgr

 ORDER BY the_geom <-> ST_GeometryFromText(''POINT('

 || x2 || ' ' || y2 || ')'',3857) LIMIT 1' INTO rec;

 target := rec.id;

 -- 最短路径查询

 seq := 0;

 sql := 'SELECT gid, geom, cost, source, target,

 ST_Reverse(geom) AS flip_geom FROM ' ||

 'pgr_dijkstra(''SELECT gid as id, source::int, target::int, '

 //这里可以省略坐标值                  || 'length::float AS cost,x1,y1,x2,y2 FROM '

 || quote_ident(tbl) || ''', '

 || source || ', ' || target

 || ' ,false, false), '

 || quote_ident(tbl) || ' WHERE id2 = gid ORDER BY seq';

 -- Remember start point

 point := source;

 FOR rec IN EXECUTE sql

 LOOP

 -- Flip geometry (if required)

 IF ( point != rec.source ) THEN

 rec.geom := rec.flip_geom;

 point := rec.source;

 ELSE

 point := rec.target;

 END IF;

 -- Calculate heading (simplified)

 EXECUTE 'SELECT degrees( ST_Azimuth(

 ST_StartPoint(''' || rec.geom::text || '''),

 ST_EndPoint(''' || rec.geom::text || ''') ) )'

 INTO heading;

 -- Return record

 seq := seq + 1;

 gid := rec.gid;

 cost := rec.cost;

 geom := rec.geom;

 RETURN NEXT;

 END LOOP;

 RETURN;

END;

$BODY$

LANGUAGE 'plpgsql' VOLATILE STRICT; 
```

坐标系要保持一致性。

3.1 发布服务
--------

在geoserver中发布服务

数据存储——添加新的数据存储——PostGIS

![](https://upload-images.jianshu.io/upload_images/10338988-850a063ba68b48a6.png)

图3.1 配置信息

输入相关信息，点击保存

![](https://upload-images.jianshu.io/upload_images/10338988-d1e9fb8eb3d71c81.png)

图3.2 SQL配置

如图3.2，选择“配置新的SQL视图”

![](https://upload-images.jianshu.io/upload_images/10338988-61df0336c35004be.png)

图3.3 SQL配置步骤

1、 输入视图名称  
2、 输入SQL语句

```
SELECT ST_MakeLine(route.geom) geom FROM (

 SELECT geom FROM pgr_fromAtoB_3('prjroad', %x1%, %y1%, %x2%, %y2%)

ORDER BY seq) AS route 
```

3、 从SQL猜想的参数  
4、 获取到参数，输入默认值和正则表达式  
5、 刷新，得到SQL语句返回的结果  
6、 设定类型和SRID  
7、 保存  
8、 计算边框，保存

![](https://upload-images.jianshu.io/upload_images/10338988-c533a6ff9eabd20c.png)

图3.4 计算边框

3.2 功能实现
--------

传入起止点坐标，计算最短路径

![](https://upload-images.jianshu.io/upload_images/10338988-801578c36dbdd766.png)

图3.5 功能展示

核心代码如下：

```
var viewparams = ['x1:' + cordx, 'y1:' + cordy, 'x2:' + cordx1, 'y2:' + cordy1];
viewparams = viewparams.join(';');
testLayer1 = new OpenLayers.Layer.Vector("result3", {
    request: new OpenLayers.Request.GET({
        url: "http://192.168.15.97:8080/hgisserver/network/wfs?service=WFS&version=1.1.0&request=GetFeature" + "&typeName=network:newview&outputformat=JSON" + "&viewparams=" + viewparams,
        success: function (req) {
            var jsonStr = req.responseText;
            var jsonFormat = new OpenLayers.Format.GeoJSON();
            var featureArray = jsonFormat.read(jsonStr);
            testLayer1.addFeatures(featureArray);
            testLayer1.redraw();
        }
    })
});
testLayer1.style = {
    strokeWidth: 3,
    strokeOpacity: 1,
    strokeColor: "#FF0000"
}
map.addLayer(testLayer1); 
```

更多精彩内容，就在简书APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](https://upload.jianshu.io/users/upload_avatars/10338988/dae08ed6-09eb-409f-b237-89ca5b086603?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/4cc481eed7b7)

总资产17共写了5411字获得120个赞共77个粉丝

### 被以下专题收入，发现更多相似内容