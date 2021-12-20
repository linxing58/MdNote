# Geotrellis入门 - 简书
[![](https://upload.jianshu.io/users/upload_avatars/6954292/78a964a4-68d5-431b-847a-8b9d6fb392bf.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/0fa55627e517)

0.7692018.12.04 09:25:22 字数 1,305 阅读 5,945

-   [shoufengwei 的 Geotrellis 系列文章](http://www.cnblogs.com/shoufengwei/p/5619419.html)
-   [Geotrellis 英文文档](https://docs.geotrellis.io/en/latest/tutorials/quickstart.html)
-   [粥粥 zz 的 CSDN 博客](https://me.csdn.net/qq_32432081)

## 1. 建立工程

  按道理来讲，用 sbt 建立一个 scala 工程，并在 build 里引入依赖，就可以自动下载 geotrellis 的相关包，但我的 sbt 不知道怎么回事，依赖包下载不来，最后还是通过 maven 来解决的。  
[geotrellis 的 maven 列表](https://mvnrepository.com/artifact/org.locationtech.geotrellis?p=1)  
  先用 raster 试试水，pom 中加入

     <dependency>
          <groupId>org.locationtech.geotrellis</groupId>
          <artifactId>geotrellis-spark_2.11</artifactId>
          <version>2.1.0</version>
        </dependency>
        
        <dependency>
          <groupId>org.locationtech.geotrellis</groupId>
          <artifactId>geotrellis-raster_2.11</artifactId>
          <version>2.1.0</version>
        </dependency> 

  然后便是照抄官方文档里的 hello raster 示例。

    package demo

    import geotrellis.raster._
    import geotrellis.raster.mapalgebra.focal.Square
    import geotrellis.spark._

    object Main {
      def helloSentence = "Hello GeoTrellis"

      def helloRaster(): Unit = {
        val nd = NODATA    

        val input = Array[Int](
          nd, 7, 1, 1, 3, 5, 9, 8, 2,
          9, 1, 1, 2, 2, 2, 4, 3, 5,
          3, 8, 1, 3, 3, 3, 1, 2, 2,
          2, 4, 7, 1, nd, 1, 8, 4, 3)

        
        val iat = IntArrayTile(input, 9, 4)

        
        
        val focalNeighborhood = Square(1)
        println(focalNeighborhood)
        val meanTile = iat.focalMean(focalNeighborhood)

        for (i <- 0 to 3) {
          for (j <- 0 to 8) {
            print(meanTile.getDouble(j, i) + " ")
          }
          println()
        }
      }

      def main(args: Array[String]): Unit = {
        helloRaster()
      }
    } 

  输出结果如下

     0  0  0 
     0  0  0 
     0  0  0 

    5.666666666666667 3.8 2.1666666666666665 1.6666666666666667 2.5 4.166666666666667 5.166666666666667 5.166666666666667 4.5 
    5.6 3.875 2.7777777777777777 1.8888888888888888 2.6666666666666665 3.5555555555555554 4.111111111111111 4.0 3.6666666666666665 
    4.5 4.0 3.111111111111111 2.5 2.125 3.0 3.111111111111111 3.5555555555555554 3.1666666666666665 
    4.25 4.166666666666667 4.0 3.0 2.2 3.2 3.1666666666666665 3.3333333333333335 2.75 

## 2. 读本地 Geotiff 文件

  一共四行代码

    import geotrellis.raster.io.geotiff.reader.GeoTiffReader
    import geotrellis.raster.io.geotiff._

    val path: String = "filepath/filename.tif"
    val geoTiff: SinglebandGeoTiff = GeoTiffReader.readSingleband(path) 

  这种方法用于读取但波段 tif 影像，要读取多波段影像，可以用

    val geoTiff: MultibandGeoTiff = GeoTiffReader.readMultiband(path) 

  如果强行用**_GeoTiffReader.readSingleband(path)_**方法去读取一个多波段影像，则最后的返回结果是一个**单波段**影像，且其中的数据为原始影像中的**第一个波段**。  
  此外，也可以用影像路径为参数直接构造一个 Geotiff 对象

    import geotrellis.raster.io.geotiff.SinglebandGeoTiff

    val geoTiff: SinglebandGeoTiff = SinglebandGeoTiff(path) 

## 3.ETL 工具

### 3.1 调用工具，生成金字塔图层

  这个工具貌似是用来进行数据转换的，试着按照各路文档做了一个将 landsat8 的 tif 影像打成金字塔的过程。需要在 maven 中引入

     <dependency>
          <groupId>org.locationtech.geotrellis</groupId>
          <artifactId>geotrellis-spark-etl_2.11</artifactId>
          <version>2.1.0</version>
        </dependency> 

  函数主体很短

    def main(args: Array[String]): Unit = {
        var args = Array[String](
          "--input",
          "D:\\Javaworkspace\\geotrellis\\study\\testdata\\input.json",
          "--output",
          "D:\\Javaworkspace\\geotrellis\\study\\testdata\\output.json",
          "--backend-profiles",
          "D:\\Javaworkspace\\geotrellis\\study\\testdata\\backend-profiles.json"
        );
        Logger.getLogger("org").setLevel(Level.ERROR)
        implicit val sc = SparkUtils.createSparkContext("ETL", new SparkConf(true).setMaster("local[*]"))
        try {
          Etl.ingest[ProjectedExtent, SpatialKey, Tile](args)
        } finally {
          sc.stop
        }
      } 

  重点是 args 里面指定的三个 json 文件的配置，首先是 input.json

    [{
        "format":"geotiff",
        "name":"etlTest1",
        "cache":"NONE",
        "backend":{
            "type":"hadoop",
            "path":"D:\\Javaworkspace\\geotrellis\\study\\testdata\\hzpan.TIF"
        }
    }] 

  这里面制定了输入数据的格式（geotiff），要输出的图层名（etlTest1），读取的方式（type）和数据路径（path）等参数。对于 output.json

    {
        "backend": {
            "type": "file",
            "path": "D:\\Javaworkspace\\geotrellis\\study\\testdata\\output1"
        },
        "reprojectMethod": "buffered",
        "pyramid": true,
        "tileSize": 256,
        "keyIndexMethod": {
            "type": "zorder"
        },
        "resampleMethod": "nearest-neighbor",
        "layoutScheme": "zoomed",
        "crs": "EPSG:4326"
    } 

  我的 windows 本地跑 spark 和 hadoop 有个问题一直没解决，导致无法用 hadoop 的输出接口，因此将 type 属性指定为 file。其他属性都可根据实际情况配置。这里官方文档有一个坑，当 pyramid 设置为 true 的时候，resampleMethod 就不能设置为 cubic-spline，否则会报错。最后是 backend-profiles.json

    {
        "backend-profiles":[]
    } 

  貌似是当你使用了 accumulo 或 cassandra 来进行输入输出的时候需要配置，一般情况下设个空数组就行。运行成功后在输出目录下新建了两个文件夹

-   attributes
-   etlTest1  
      第一个文件夹里存放的似乎是切片后每一个层级的元数据信息，第二个文件夹的命名来自于 input.json 中的 name，其中存放了切片后的图层文件。  
      然而，上面的代码只能用于处理单波段影响，即便输入的是多波段影像，该工具也只会对第一个波段进行转换。如果想要处理多波段影像，至少应该修改两个地方：  
    （1）首先是 input.json 的 format：


    "format":"multiband-geotiff" 

 （2）在调用 ETL 的核心代码中，需要修改切片类型（Tile -> MultibandTile）：

    Etl.ingest[ProjectedExtent, SpatialKey, MultibandTile](args) 

### 3.2 渲染切片

  上一步中生成的切片文件不能用普通查看图片的方法打开，需要借助 Geotrellis 提供的类，发布成地图服务或者渲染成图片格式文件（如 jpg、png 等）。这一步中，尝试将其渲染为图片。  
  第一步是读取图层。Geotrellis 支持多种读取方式，如 Accumulo、h3 等，这里使用最基本的文件读取。

     val zoomId = 9  
        implicit val sc = SparkUtils.createSparkContext("ReadLayer", new SparkConf(true).setMaster("local[*]"))
        val path = "D:\\Javaworkspace\\geotrellis\\study\\testdata\\output1"  
        val store = FileAttributeStore(path)
        val reader = FileLayerReader(path)
        val layerId = LayerId("etlTest1", zoomId)  
        val layers: TileLayerRDD[SpatialKey] = reader.read[SpatialKey, Tile, TileLayerMetadata[SpatialKey]](layerId) 

  正确读进来，layers 将会是个 TileLayerRDD 对象。![](https://math.jianshu.com/math?formula=%5Ccolor%7Bred%7D%7B%E9%87%8D%E7%82%B9%7D)
，这里有个坑，在 import 的时候，要把 geotrellis.spark.io 下的所有类文件都引进来，不要只引用到的 AttributeStore, LayerReader，否则会报找不到隐式参数的错。

     import geotrellis.spark.io.{AttributeStore, LayerReader}

    import geotrellis.spark.io._ 

  获取到 RDD 后，可以利用 spark 的 foreach 算子将其输出到本地的文件目录中

     val colorMap1 = ColorMap(Map(
              0 -> RGB(0,0,0),
              1 -> RGB(255,255,255)
          ))
        val colorRamp = ColorRamp(RGB(0,0,0), RGB(255,255,255))
            .stops(100)
            .setAlphaGradient(0xFF, 0xAA)
        val outputPath = "D:\\Javaworkspace\\geotrellis\\study\\testdata\\output1\\render\\" + zoomId  
        val zoomDir: File = new File(outputPath)
        if (!zoomDir.exists()) {
          zoomDir.mkdirs()
        }
        layers.foreach(layer => {
          val key = layer._1
          val tile = layer._2
          val layerPath = outputPath + "\\" + key.row + "_" + key.col + ".jpg"
          tile.renderJpg(colorRamp).write(layerPath)  
        })
        sc.stop() 

  程序正确运行后，在相应的输出目录可以看到如下文件

![](https://upload-images.jianshu.io/upload_images/6954292-a9ca48098f373c70.png)

图层渲染结果

  将上面的代码稍加修改，就可以将切片导出成 tiff 影像。首先，需要构造一个 GeoTiff 对象，它有这样一个构造函数

    def apply(tile: Tile, extent: Extent, crs: CRS): SinglebandGeoTiff =
        SinglebandGeoTiff(tile, extent, crs) 

  tile 类型的对象可以像上面一样通过 RDD 的 foreach 算子取得，CRS 参数表示坐标系，可以通过 EPSG 取得，也可以自己构造，同时在 geotrellis 里也有预定义几个：

     object LatLng extends CRS with ObjectNameToString {
      lazy val proj4jCrs = factory.createFromName("EPSG:4326")

      def epsgCode: Option[Int] = Some(4326)
    }

    object WebMercator extends CRS with ObjectNameToString {
      lazy val proj4jCrs = factory.createFromName("EPSG:3857")

      def epsgCode: Option[Int] = Some(3857)
    }

    object Sinusoidal extends CRS with ObjectNameToString {
      lazy val proj4jCrs: CoordinateReferenceSystem = factory.createFromParameters(null,
        "+proj=sinu +lon_0=0 +x_0=0 +y_0=0 +a=6371007.181 +b=6371007.181 +units=m +no_defs")
      val epsgCode: Option[Int] = None
    } 

  最后的 Extent 对象，也可以通过 TileLayerRDD 取得，但首先需要取得该切片对应的图层定义：

    val layoutDefinition: LayoutDefinition = layers.metadata.layout 

  layoutDefinition 中有一个 mapTransform 方法，传入一个 SpatialKey，就能得到其对应的格网范围 Extend，因此，将字节码图层文件转换为带有空间信息的 tiff 影像的代码如下：

    import geotrellis.raster.io.geotiff.GeoTiff

    val layers: TileLayerRDD[SpatialKey] = 
    val layoutDefinition: LayoutDefinition = layers.metadata.layout

    layers.foreach(layer => {
      val key = layer._1
      val tile = layer._2
      val tiffPath = outputPath + "\\" + key.row + "_" + key.col + ".tiff
      GeoTiff(tile, layoutDefinition.mapTransform(key), LatLng).write(tiffPath)
    }) 

  采用的是 4326 即 WGS84 坐标，输出结果与上面的渲染成 jpg 类似，但具有空间信息，能被 EnVI 等遥感影像处理软件正确加载。

     import geotrellis.vector.{Polygon}
        import geotrellis.vector.io._
        import geotrellis.vector.io.json._
        
        val json = Polygon((10.0,10.0),(10.0,20.0),(30.0,30.0),(10.0,10.0)).toGeoJson
        println(json)
        
        val fc: String = """{
                           |  "type": "FeatureCollection",
                           |  "features": [
                           |    {
                           |      "type": "Feature",
                           |      "geometry": { "type": "Point", "coordinates": [1.0, 2.0] },
                           |      "properties": { "someProp": 14 },
                           |      "id": "target_12a53e"
                           |    }, {
                           |      "type": "Feature",
                           |      "geometry": { "type": "Point", "coordinates": [2.0, 7.0] },
                           |      "properties": { "someProp": 5 },
                           |      "id": "target_32a63e"
                           |    }
                           |  ]
                           |}""".stripMargin
        val geos = fc.parseGeoJson[JsonFeatureCollectionMap]
        val points1 = geos.getAllPoints()
        for (k <- points1.keySet){
          println(k + ": " + points1.get(k).get)
        } 

  输出结果如下

    {"type":"Polygon","coordinates":[[[10.0,10.0],[10.0,20.0],[30.0,30.0],[10.0,10.0]]]}
    target_12a53e: POINT (1 2)
    target_32a63e: POINT (2 7) 

#### _未完待续_

更多精彩内容，就在简书 APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](https://upload.jianshu.io/users/upload_avatars/6954292/78a964a4-68d5-431b-847a-8b9d6fb392bf.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/0fa55627e517)

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   Spark SQL, DataFrames and Datasets Guide Overview SQL Dat...

    [![](https://cdn2.jianshu.io/assets/default_avatar/4-3397163ecdb3855a0a4139c34a695885.jpg)
    Joyyx](https://www.jianshu.com/u/5d6219efd1b8)阅读 7,124 评论 0 赞 16
-   Spark SQL, DataFrames and Datasets Guide Overview SQL Dat...


-   Spring Cloud 为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智...

    [![](https://upload-images.jianshu.io/upload_images/7328262-54f7992145380c10.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/46fd0faecac1)
-   pyspark.sql 模块 模块上下文 Spark SQL 和 DataFrames 的重要类： pyspark.sql...

    [![](https://cdn2.jianshu.io/assets/default_avatar/4-3397163ecdb3855a0a4139c34a695885.jpg)
    mpro](https://www.jianshu.com/u/7c205898f6fd)阅读 8,190 评论 0 赞 11
-   为贯彻落实智能电网发展战略，进一步促进输配电设备安装调试和运维管理的创新和发展，并全面提升企业在电力及输配电设备试...
       [![](https://cdn2.jianshu.io/assets/default_avatar/6-fd30f34c8641f6f32f5494df5d6b8f3c.jpg)
       木垚一夕](https://www.jianshu.com/u/f8b7ddbfb836)阅读 1,150 评论 0 赞 0 
    [https://www.jianshu.com/p/1eda79747648](https://www.jianshu.com/p/1eda79747648)
