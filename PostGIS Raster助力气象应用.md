# PostGIS Raster助力气象应用
众所周知，气象行业由于其专业性和封闭性，大量格点数据都以文件系统存储，如常见的nc、grib数据格式，这类格式一般面向专业气象软件。随着气象应用于各行各业，越来越多的行业应用开发者接触气象数据，其特有的格式给习惯以数据库作为数据存储的技术人员带来比较大的挑战，如何合理设计存储、分析、可视化是一个很大的架构问题，笔者进行过大量的技术研究最终选择PostGIS Raster作为气象应用数据库的基石，本文作简要技术分析。

**一 格点与栅格关系**

气象数据大部分是格点数据，格点数据一点也不神秘，熟悉GIS的朋友肯定知道数据分矢量和栅格两大类，气象格点数据就是GIS里的栅格数据。GIS里会经常进行矢量栅格化，栅格矢量化等格式转换应用于分析和可视化，而格点数据其实就是GIS这种思想在气象行业里的一个具体应用。

如果我们把一张格网的全部格点投影到一张图像上，格点位置对应像素位置，根据格点值的不同给像素值填不同的颜色，最终会得到一个图像：

![](https://pic2.zhimg.com/v2-f49c8a01a81d8f2778b367a98d666071_b.jpg)

格点数据类似GIS里的栅格数据

与普通图像不同，这种图像具有地理范围，具有地理坐标系，因此它是一个非常正宗的地理栅格数据。

**二 栅格数据在气象行业中的应用**

与气象专业不同，气象行业应用通常不需要专业的气象格式，大部分行业的需求就2点：

*   气象可视化

以栅格数据存储，简单显示需求，既可以通过在服务端对栅格数据着色，通过专业的GIS服务器以Web Tile形式发布，前端地图直接叠加显示。复杂的动画、特效展示需求可以将栅格数据转灰度图提交前端由webgl渲染。详细参考：

> [格点数据应用之二 WebGIS等值面可视化篇](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzg2OTUxMzM2MA%3D%3D%26mid%3D2247483709%26idx%3D1%26sn%3Dd386cb42e94b715b86dda86444d3fba0%26chksm%3Dce9aa0c2f9ed29d497648f98698b6270c9a2b89a809a90c329789fc733f993eb6782400d7560%26token%3D1032199986%26lang%3Dzh_CN%23rd)

当然，还有等值线的展示需求，可以通过d3-contour计算（效果一般般），还可以通过数据库引擎生成，下文介绍。

*   气象空间分析

既有日常生活里常见的任意点的时序穿透查询，比如天气预报，任一点就是城市的位置点，穿透查询未来几个日期的气象数据信息：

![](https://pic3.zhimg.com/v2-14ae743418cd2edc47dab9fb66d46afe_b.jpg)

也有电力、交通等行业的典型气象灾害预警分析：

![](https://pic1.zhimg.com/v2-aa4a7e62deec3c7f9c467ff497143854_b.jpg)

针对这些典型气象业务场景，选择一个既能支持可视化又能支持栅格查询、矢量栅格混合空间查询的数据库就很有意义了。

**三 栅格数据库为什么选PostGIS**

在GIS行业里，栅格数据是以文件形式存储的，使用开源的GDAL库对栅格数据做各种处理和分析。当然如果IT足够专业，不可避免我们会选择一个数据库去存储这样的文件数据，本文要介绍的主角PostGIS Raster登场了。

*   支持栅格查询，矢量栅格混合查询

PostGIS Raster是PostGIS的半边天，其实宏观意义上的PostGIS是矢量和栅格两部分组成的，矢量是由底层Geos库进行数据处理和分析，栅格是由底层GDAL库进行处理和分析，所以实际上PostGIS处理栅格数据的能力仍然是GDAL库赋予的。在PostGIS 3.0之后，矢量和栅格的扩展分家了，所以如果您在3.0之后启用扩展，通常执行如下语句：

```text
-- 矢量postgis
create extension postgis;
-- 栅格postgis
create extension postgis_raster;
```

在PostGIS里，我们可以直接进行栅格查询和矢量栅格混合查询。

例如，我们将连续n个时刻的预报数据写入栅格数据，栅格数据的每个band记录一个时刻的格网图像，那么当我们按时间依次切换band显示的时候大概如下：

![](https://pic1.zhimg.com/v2-1e826938a93d1dc31de320579c47b0b4_b.gif)

当我们进行格点穿透查询的时候直接可以使用如下的sql就能快速得到格点穿透查询的数值：

```text
--查询一个位置对应6个通道的像素值，每个通道像素值代表一个预报时刻。
WITH pt AS (
  ST_SetSRID(ST_MakePoint(-80, 35), 4326) AS pt
)
SELECT b, ST_Value(rast, b, pt.pt) as pop 
FROM pop12
CROSS JOIN pt
CROSS JOIN generate_series(1,6) b
WHERE ST_Intersects(pt.pt, st_convexhull(rast));
```

那么能得到这样的结果如下：

```text
b | pop 
---+-----
 1 |   0
 2 |   0
 3 |  71
 4 |  90
 5 |  16
 6 |   2
```

![](https://pic2.zhimg.com/v2-d9298b28447bb31999ec8518c2ba7ca1_b.jpg)

当与行业应用叠加分析的时候，比如统计若干地点每个地点的气象值，并输出前几个地点名称和气象值。

![](https://pic3.zhimg.com/v2-2f869f0eb1c88868d42a42a3c62754a2_b.jpg)

地点数据表Places

sql写起来大概就这个样子的：

```text
SELECT 
  places.name, places.adm1name, 
  ST_Value(rast, 5, places.geom) AS pop 
FROM pop12
JOIN places 
  ON ST_Intersects(places.geom, ST_ConvexHull(rast))
WHERE places.sov_a3 = 'USA'
ORDER BY pop DESC 
LIMIT 10;
  name     |   adm1name    | pop 
--------------+---------------+-----
 Scranton     | Pennsylvania  | 100
 Wilkes Barre | Pennsylvania  | 100
 Binghamton   | New York      |  99
 New London   | Connecticut   |  99
 Elmira       | New York      |  99
 New Haven    | Connecticut   |  98
 Waterbury    | Connecticut   |  97
 Bridgeport   | Connecticut   |  97
 Stamford     | Connecticut   |  97
 New Bedford  | Massachusetts |  96
```

很明显在数据分析应用中在PostGIS非常有优势，开发更简洁，开发人员只需sql就能快速开发业务应用。

*   支持可视化

PostGIS的raster的api可以支持将栅格导出为图片，如ST\_AsJPEG、ST\_AsPNG，也有一些灰度图的API，如ST_Grayscale，可把栅格转灰度图图片作为webgl纹理渲染，

![](https://pic3.zhimg.com/v2-9d448b8058296852adcc368dc12a56aa_b.jpg)

ST_Grayscale

当然也可以后端直接上色气象栅格tif，通过geoserver等软件发布地理服务简单叠加：

![](https://pic3.zhimg.com/v2-ed2ab48caf953769aaeac2b1a432092e_b.jpg)

![](https://pic3.zhimg.com/v2-1cde2ca5f96524764aa3722a9332f712_b.jpg)

尤其令人兴奋的时，在即将发布的PostGIS3.2版本，竟然有直接支持生成等值线ST_Contour，效果如下图：

![](https://pic2.zhimg.com/v2-f02a0a42a1fae5c95708d3fd37a75d59_b.jpg)

ST_Contour

综述，当使用PostGIS Raster后，使用一种栅格格式（GeoTiff），同时支持业务的穿透查询，矢量栅格混合空间查询，服务端渲染可视化，客户端渲染可视化，还有等值线等，全方位覆盖气象业务应用。

**四 PostGIS让气象业务腾飞**

上文介绍了单机版的PostGIS Raster在功能方面的覆盖，但实际工程问题会比功能问题更复杂。例如，为什么不用文件系统存储nc而使用数据库存储geotiff咧？因为数据库非常成熟，在互联网场景下的负载均衡水平分布都有很多成熟的框架可直接复用，同样，云计算场景下还有存储和计算分离等各种场景，这都不是简单文件系统可比的，幸运的是，结合PostGIS，气象业务应用可以直接插上翅膀，复用数据库领域最新的技术进行业务快速扩张。

*   分布式部署

PostGIS仅仅是PostgreSQL的一个插件，PG社区有很多分布式产品如greenplum、citus等，使用PostGIS的气象存储天然具有分布式部署和计算的能力。

*   存储计算分离

气象数据通常非常的大，数据库的存储设备非常昂贵，如云数据库，如果把数据库都存数据库内部肯定不是好方案，其实PostGIS Raster提供了两种栅格存储方式，即"in-db"和"out-db"，

in-db：栅格数据存储在PostGIS内，默认，通常不选用，占用昂贵的数据库存储资源。

out-db：栅格数据存储在数据库外，数据库存储其元数据，如数据名称，存储文件目录（或者网络服务url），程序访问数据库，数据库访问栅格数据，该方式实现了计算和存储的分离。

![](https://pic3.zhimg.com/v2-ea7e0ba8768c2cbdecb34389fad258b6_b.jpg)

*   云计算

最新版的GDAL已经支持访问远程文件，借助这个能力，PostGIS集成GDAL可以实现对远程气象栅格数据的访问，业务设计者可以把气象数据存储在云存储桶上，由PostGIS存储元数据，查询虽然比本地慢一点，但是从云计算和计算存储分离角度而言非常有意义，能做出更有意思的业务设计来。

![](https://pic2.zhimg.com/v2-1ea9199afb7ceaa93c90427cd1b835c1_b.jpg)

详细的in-db,out-db我们后文会介绍其使用特点。

总的来说，在气象作行业应用时，一定要跳出气象领域的圈子，放弃他们的超算、数据格式（这些东西是气象专业使用而不是气象行业应用使用的），用it人的思路去设计系统和数据架构，用GIS人的思路去设计功能和优化分析算法。

更多关于WebGIS和PostGIS的架构和技术交流，可通过qq群445307545一起分享更有意思的知识。