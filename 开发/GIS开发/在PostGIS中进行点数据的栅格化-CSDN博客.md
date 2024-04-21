# 在PostGIS中进行点数据的栅格化-CSDN博客
说明
--

介绍在PotGIS中将点[数据转换](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E8%BD%AC%E6%8D%A2&spm=1001.2101.3001.7020)为栅格数据。

**关键字：**  `raster`、`point`、`PostGIS`

### 环境准备

*   Postgresql版本：`PostgreSQL 14.0, 64-bit`
*   PostGIS版本：`POSTGIS="3.3.2"`
*   QGIS版本：`3.28.3-Firenze`

### 基本步骤

#### 一、数据准备

[测试数据](https://so.csdn.net/so/search?q=%E6%B5%8B%E8%AF%95%E6%95%B0%E6%8D%AE&spm=1001.2101.3001.7020)中有一张点数据表，坐标系`3857`。

```
`CREATE TABLE IF NOT EXISTS public.test_point
(
    geom geometry(Point,3857),
    id bigint,
    h numeric,--指定要转换为栅格值的列，如高程、温度、风速等
    CONSTRAINT test_point_pkey PRIMARY KEY (id)
)

TABLESPACE pg_default;

ALTER TABLE public.test_point
    OWNER to postgres;` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12


```

#### 二、二维点数据转换为三维点

```
`DROP TABLE IF EXISTS test_point_3d;
CREATE TABLE test_point_3d
    as select h,ST_Force3D(ST_Transform(geom,3857),tp.h) as geom 
    from test_point tp;` 

*   1
*   2
*   3
*   4


```

![](https://img-blog.csdnimg.cn/dd91766cc7514baa8c6724de392011e0.png)

#### 三、将三维点数据转换为栅格

```
`WITH inputs AS 
    (SELECT 16::INTEGER AS pixelsize,'linear:radius:100' AS algorithm, 
    ST_Collect(geom) AS geom, ST_Expand(ST_Collect(geom), 10) AS ext
    FROM test_point_3d ),sizes AS 
    (SELECT ceil((ST_XMax(ext) - ST_XMin(ext))/pixelsize)::integer AS width,
         ceil((ST_YMax(ext) - ST_YMin(ext))/pixelsize)::integer AS height,
         ST_XMin(ext) AS upperleftx,ST_YMax(ext) AS upperlefty
    FROM inputs )SELECT 1 AS rid,
         ST_InterpolateRaster( geom,
         algorithm,
         ST_SetSRID(ST_AddBand(ST_MakeEmptyRaster(width,
         height,upperleftx,upperlefty,pixelsize),
         '32BF'), ST_SRID(geom)) ) AS rast FROM sizes, inputs;` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13


```

![](https://img-blog.csdnimg.cn/8924795211ef423a800773f5361b05e7.png)

其中`ST_InterpolateRaster 参见`:

[https://postgis.net/docs/RT\_ST\_InterpolateRaster.html](https://postgis.net/docs/RT_ST_InterpolateRaster.html)

其参数`algorithm`参见：

[https://gdal.org/programs/gdal_grid.html#interpolation-algorithms](https://gdal.org/programs/gdal_grid.html#interpolation-algorithms)