# NEO4J空间索引_马超的博客的博客-CSDN博客
> Neo4j空间索引可以对数据进行空间索引，例如在指定区域内以某个感兴趣的点作为起始，搜索指定距离内其它感兴趣的点。可以方便地将空间索引的节点数据与已有图数据结合分析。

### 1、创建图层

```sql
CALL spatial.addPointLayer('geom')

```

![](https://img-blog.csdnimg.cn/20190422185355755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1cGVybWFuX3h4eA==,size_16,color_FFFFFF,t_70)

### 2、查看已经创建的图层列表

```sql
CALL spatial.layers()

```

![](https://img-blog.csdnimg.cn/20190422185503702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1cGVybWFuX3h4eA==,size_16,color_FFFFFF,t_70)

### 3、建立空间点并将新创建的点加入到geom图层中

```sql
MERGE (n:Node {longitude:15.2,latitude:60.1}) WITH n 
CALL spatial.addNode('geom',n) YIELD node RETURN node

```

![](https://img-blog.csdnimg.cn/20190422185543930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1cGVybWFuX3h4eA==,size_16,color_FFFFFF,t_70)

### 4、查询[维度](https://so.csdn.net/so/search?q=%E7%BB%B4%E5%BA%A6&spm=1001.2101.3001.7020)在60.0到60.2之间，经度在15.0到15.3之间的空间点

```sql
CALL spatial.bbox('geom',{longitude:15.0,latitude:60.0},{longitude:15.3,latitude:60.2}) YIELD node RETURN node

```

![](https://img-blog.csdnimg.cn/20190422185617204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1cGVybWFuX3h4eA==,size_16,color_FFFFFF,t_70)

### 5、导入全国公路shp文件

[数据下载地址](https://malagis.com/gis-data-china-road-shp.html)

```sql

CALL spatial.addWKTLayer('layer_roads','geometry')
CALL spatial.importShapefileToLayer('layer_roads','roa_4m.shp')

```

### 6、查询一个矩形内的图形语句

```sql
CALL spatial.bbox('layer_roads',{longitude:14.0,latitude:60.0},{longitude:19.3,latitude:81.0}) YIELD node RETURN node.name as name

```

### 7、查询一个多边形内的点

```sql
WITH "POLYGON((15.3 60.0,15.3 62.0,15.2 60.2,15.4 65.0))" as  polygon
CALL spatial.intersects('layer_roads', polygon) YIELD node RETURN node.name as name

```

### 8、WithinDistance - 查询点周边distance（0.1km）以内的点

```sql
CALL spatial.withinDistance('geom',{longitude:15.2,latitude:60.1},0.1) YIELD node RETURN node LIMIT 10

```

![](https://img-blog.csdnimg.cn/20190422190157925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1cGVybWFuX3h4eA==,size_16,color_FFFFFF,t_70)

### 9、批量节点构建空间索引

```sql
CALL spatial.addPointLayer('geom')
UNWIND [{name:'a',latitude:60.1,longitude:15.2},{name:'b',latitude:60.3,longitude:15.5}] as point CREATE (n:Node) SET n += point WITH n 
CALL spatial.addNode('geom',n) YIELD node RETURN node.name as name

```

[neo4j-spatial-plugin downloa](https://github.com/neo4j-contrib/spatial/releases)  
neo4j根据经纬度计算两点之间的距离