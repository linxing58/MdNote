# 在PostGIS中进行点数据的等值线提取-CSDN博客
说明
--

介绍在PostGIS中从点[数据提取](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E6%8F%90%E5%8F%96&spm=1001.2101.3001.7020)等值线。

**关键字：**  `raster`、`point`、`PostGIS`、`等值线`

### 环境准备

*   Postgresql版本：`PostgreSQL 14.0, 64-bit`
*   PostGIS版本：`POSTGIS="3.3.2"`
*   QGIS版本：`3.28.3-Firenze`（验证用）

### 基本步骤

![](https://img-blog.csdnimg.cn/919fc17596bf43a7982230c4ddfe9881.png)

#### 一、数据准备

略.

#### 二、二维点数据转换为三维点

略.

#### 三、将三维点数据转换为栅格

略.

以上三步参见[《在PostGIS中进行点数据的栅格化》](https://blog.csdn.net/eqmaster/article/details/134499687)

#### 四、从栅格数据中提取等值线

关键SQL如下：

```
`Create table _ResultTableName as
    WITH c AS (
    SELECT (ST_Contour(rast, 1, fixed_levels => 100,polygonize=>false)).*
    FROM _TheRastTableName )
    SELECT geom, id, value FROM c;` 

*   1
*   2
*   3
*   4
*   5


```

![](https://img-blog.csdnimg.cn/5ba48deb8d95400a965d9ecf1ffecacf.png)

其中`ST_Contour`详细参数说明参见:

[https://postgis.net/docs/RT\_ST\_Contour.html](https://postgis.net/docs/RT_ST_Contour.html)

按照说明参数`polygonize=true`时，可以生成等值面，但在实际使用时依旧会生成线，所以用了另外一种方法提取等值面，参加其他文档。