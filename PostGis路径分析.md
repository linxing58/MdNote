# PostGis路径分析
一、导入数据
------

1.  建立PostGis数据库。  
    使用sample数据库做模板。![](https://img-blog.csdnimg.cn/20200915221226902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
    
2.  导入纽约公路矢量图层到PostGis。[地图下载位置](https://docs.geoserver.org/latest/en/user/_downloads/30e405b790e068c43354367cb08e71bc/nyc_roads.zip)  
    ![](https://img-blog.csdnimg.cn/20200915220757659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
      
    ![](https://img-blog.csdnimg.cn/20200915221753288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
    

二、生成拓扑
------

要生成最佳路径，首先要生成合法的拓扑。  
生成拓扑前，需要添加两个字段，用来存储线段的首尾编号

```sql

ALTER TABLE nyc_roads ADD COLUMN "source" integer;
ALTER TABLE nyc_roads ADD COLUMN "target" integer;

```

调用pgr_createTopology生成拓扑，注意就是生成线段的首位编号的过程

```sql
pgr_createTopology(
'<table>',   
float tolerance,   
'<geometry column>',   
'<gid>')  

```

容错值：例如线段的端不能完全吻合时，允许多少误差，单位一般为角度或公里数

官方说明：[https://docs.pgrouting.org/3.1/en/pgr_createTopology.html](https://docs.pgrouting.org/3.1/en/pgr_createTopology.html)

例子

```sql

SELECT pgr_createTopology('nyc_roads', 0.00001, 'geom', 'gid');

```

三、生成最佳路径
--------

pgrouting支持的最佳路径算法很多。  
官方说明：[https://docs.pgrouting.org/3.1/en/search.html?q=+shortest+path&check_keywords=yes&area=default](https://docs.pgrouting.org/3.1/en/search.html?q=%20shortest%20path&check_keywords=yes&area=default)  
这里以Shortest Path A*和Shortest Path Dijkstra（狄克斯特拉）为例，介绍如何生成最佳路径

如果考虑回程成本的话，需要增加回程成本的字段，并设置为公里数。

```sql
ALTER TABLE nyc_roads ADD COLUMN reverse_cost double precision;
UPDATE nyc_roadsSET reverse_cost = length;

```

### 1\. Shortest Path Dijkstra算法举例

函数说明：

```sql
pgr_dijkstra(
text sql, 
          
integer source,   
integer target,   
boolean directed   
);  

```

官方说明：[https://docs.pgrouting.org/3.1/en/pgr_dijkstra.html](https://docs.pgrouting.org/3.1/en/pgr_dijkstra.html)

返回结果：(seq, path\_seq ,node, edge, cost, agg\_cost)  
node:起点id  
edge:目标ID, -1表示终点

```sql
SELECT * from  public.pgr_dijkstra(
	'SELECT
	gid AS id,
	source::integer,
	target::integer,
	length::double precision AS cost,
	reverse_cost
	FROM nyc_roads', 
	1, 
	9, 
	false
) 

```

![](https://img-blog.csdnimg.cn/20200915215514531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

### 2\. Shortest Path A*算法举例

与Shortest Path Dijkstra算法类似，只是SQL需要用到每条线段的起点和重点的坐标，其他参数和pgr_dijkstra都一样。

```sql
ALTER TABLE nyc_roads ADD COLUMN x1 double precision;
ALTER TABLE nyc_roads ADD COLUMN y1 double precision;
ALTER TABLE nyc_roads ADD COLUMN x2 double precision;
ALTER TABLE nyc_roads ADD COLUMN y2 double precision;
 
UPDATE nyc_roads SET x1 = ST_x(ST_PointN(geom, 1));  
UPDATE nyc_roads SET y1 = ST_y(ST_PointN(geom, 1));  
 
UPDATE nyc_roads SET x2 = ST_x(ST_PointN(geom, ST_NumPoints(geom)));  
UPDATE nyc_roads SET y2 = ST_y(ST_PointN(geom, ST_NumPoints(geom)));  

```

函数说明：

```sql
pgr_astar(
sql text,     
source integer,   
target integer, 
directed boolean, 
has_rcost boolean  
);

```

官方说明：[https://docs.pgrouting.org/3.1/en/pgr_aStar.html](https://docs.pgrouting.org/3.1/en/pgr_aStar.html)

返回结果与pgr_dijkstra一样

例子：

```sql
SELECT * FROM pgr_astar('
	SELECT gid AS id,
			 source::integer,
			 target::integer,
			 length::double precision AS cost,
			 x1, y1, x2, y2
			FROM nyc_roads',
	1, 9, false);

```

结果：  
![](https://img-blog.csdnimg.cn/20200915215844964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)