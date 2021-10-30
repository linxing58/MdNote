# Java版gdal 初探 - 简书
[![](https://upload.jianshu.io/users/upload_avatars/6954292/78a964a4-68d5-431b-847a-8b9d6fb392bf.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/0fa55627e517)

12018.12.13 23:07:06 字数 2,337 阅读 9,150

**最新更新：2019.11.08：四. Spark 并行化规整裁切**

* * *

  刚看了没几天的 geotrellis 就被叫去看 gdal 了，还好以前有一丢丢基础，借着这个机会复习一下。

  部署方法可以参见另外两篇文章：[windows 下 java 版 gdal 部署](https://www.jianshu.com/p/efd50d9a996a)和[linux 下 java 版 gdal 部署](https://www.jianshu.com/p/4b655fc91cd7)。  
  其中，在 windows 的部署文章中提到，要复制**gdalconstjni.dll、gdaljni.dll、ogrjni.dll、osrjni.dll**这四个 dll，但本文使用的 gdal 版本是 2.3.2，这四个 dll 已经被整合成了一个**gdalalljni.dll**。

  首先扔一个[gdal 的 java API 文档](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.gdal.org%2Fjava%2Foverview-summary.html)，该文档适用于 gdal 1.7.0 或以上版本。  
  首先是所有 gdal 程序都需要的注册语句：

  之前的版本还需要加这句话来支持中文路径：

     gdal.SetConfigOption("GDAL_FILENAME_IS_UTF8","YES"); 

  但在 2.3.2 中，我没有加这句话，也能正常地在中文路径下读写影像。读取影像基本信息的代码如下：

     Dataset rds = gdal.Open("影像路径", gdalconst.GA_ReadOnly);
        
        int x = rds.getRasterXSize();
        int y = rds.getRasterYSize();
        int b = rds.getRasterCount();

        
        double[] geoTransform = rds.GetGeoTransform();
        
        double[] ulCoord = new double[2];
        ulCoord[0] = geoTransform[0];
        ulCoord[1] = geoTransform[3];
        
        double[] brCoord = new double[2];
        brCoord[0] = geoTransform[0] + x * geoTransform[1] + y * geoTransform[2];
        brCoord[1] = geoTransform[3] + x * geoTransform[4] + y * geoTransform[5];

        
        String proj = rds.GetProjection(); 

  这里有个数组 geoTransform，容量为 6，代表的是仿射变换六参数，其含义如下：

-   geoTransform\[0]：左上角 x 坐标
-   geoTransform\[1]：东西方向空间分辨率
-   geoTransform\[2]：x 方向旋转角
-   geoTransform\[3]：左上角 y 坐标
-   geoTransform\[4]：y 方向旋转角
-   geoTransform\[5]：南北方向空间分辨率  
      影像中任意一个像素的坐标可以用下式计算：

    ![](https://upload-images.jianshu.io/upload_images/6954292-9e39e93821a77482.png)

    任意像素的坐标计算公式

      其中六参数分别为 {dx, a11, a12, dy, a21, a22}，正常的影像，正北方向朝上，两个旋转角的值都是 0。

  在 gdal 中，影像裁切的基本思想，是把原始影像中想要保留的部分进行另存为。因此这部分要做的事情，就是在定好要裁切的部分后，读取相应的部分并保存即可，其中涉及到一些简单的位置计算。代码如下：

     gdal.AllRegister();
        Dataset rds = gdal.Open("影像路径", gdalconst.GA_ReadOnly);
        
        int b = rds.getRasterCount();
        
        int dataType = rds.GetRasterBand(1).GetRasterDataType();

        
        double[] geoTransform = rds.GetGeoTransform();

        
        
        
        int startX = 5000;
        int startY = 5000;
        int clipX = 1000;
        int clipY = 1000;

        
        geoTransform[0] = geoTransform[0] + startX * geoTransform[1] + startY * geoTransform[2];
        geoTransform[3] = geoTransform[3] + startX * geoTransform[4] + startY * geoTransform[5];

        
        Driver driver = gdal.GetDriverByName("GTIFF");
        Dataset outputDs = driver.Create("D:\\Javaworkspace\\gdal\\output\\clip1.tif", clipX, clipY, b, dataType);
        outputDs.SetGeoTransform(geoTransform);
        outputDs.SetProjection(rds.GetProjection());

        
        for(int i = 0; i < clipY; i++){
          
          for(int j = 1; j <= b; j++){
            Band orgBand = rds.GetRasterBand(j);
            int[] cache = new int[clipX];
            
            orgBand.ReadRaster(startX, startY + i, clipX, 1, cache);
            Band desBand = outputDs.GetRasterBand(j);
            
            desBand.WriteRaster(0, i, clipX, 1, cache);
            desBand.FlushCache();
          }
        }

        
        rds.delete();
        outputDs.delete(); 

  除了 Band 对象中的**ReadRaster**和**WriteRaster**接口外，Dataset 对象也有**ReadRaster**和**WriteRaster**的接口，但是用法有所不同。要使用 Dataset 中的接口，需要先构建波段数组，表示在读写过程中对哪些波段进行操作。

     int[] bands = new int[]{1}; 

  然后在实例化 cache 数组的时候，数组容量就不是简单的要读写的影像窗口的 x\*y，而是 x\*y\*b，b 表示波段数。而数组的类型也需要根据影像的数据类型进行调整。gdal 中的数据类型以常量的形式存放在**org.gdal.gdalconst**这个类中。从接口文档中能查到有这么些个：  

![](https://upload-images.jianshu.io/upload_images/6954292-48ffd627a637a205.png)

gdal 数据类型

  正常来讲，可以使用位数更高的数据类型的数组来读取位数较低的数据。比如在我自己的影像中，datatype 的值为 2，对应的数据类型为 GDT_UInt16，即无符号 16 位整型，但我在使用 int 类型数组来读数据的时候，报了数据类型不匹配的错（仅限 Dataset 的接口），但是输出的影像好像没有问题。报错信息如下：

> ERROR 1: Java array type is not compatible with GDAL data type

  当我改用 short 类型的数组来读写数据后，这个报错就消失了，不是很懂 gdal 的这一波操作。最后利用 Dataset 对象读写数据的代码如下，也是按行来执行：

     int[] bands = new int[]{1};
        for (int i = 0; i < clipY; i++)
        {  
            short[] cache = new short[clipX * b]; 
            rds.ReadRaster(startX, startY + i, clipX, 1, clipX, 1, dataType, cache, bands);  
            outputDs.WriteRaster(0, i, clipX, 1, clipX, 1, dataType, cache, bands); 
            outputDs.FlushCache();
        } 

#### 补充：

  后来发现，有一种比较通用的方式可以读取各种数据类型的影像，即使用 byte 类型的 cache 数组，但其长度还需要再乘以目标数据类型与 byte 类型之间的倍数。举个例子，假设要存储一个像素的 short 类型的数据，正常来讲只需要实例化一个长度为 1 的 short 数组；如果用 byte 型，一个 byte 是 8 位，而一个 short 是 16 位，因此应该实例化一个长度 1\*2=2 的 byte 数组。gdal 中也提供了接口可以用来计算这个倍数，代码如下：

     int dataType = dataset.GetRasterBand(1).GetRasterDataType();

    int typeSize = gdal.GetDataTypeSize(dataType);

    int byteToType = typeSize / 8;

    byte[] datas = new byte[x * y * b * byteToType];

    dataset.Read(0, 0, x, y, x, y, dataType, datas, bandList); 

  这种方法在不对数据进行具体计算，且又想在不确定数据的类型时使程序变得通用的情况下相当有效。但对于某些压缩过的数据，用 short 存储，需要通过元数据信息进行计算复原的话就不适用了。

  这次会使用 gdal，主要就是为了这个需求。利用 spark，将影像裁切成一个个规整的方块。比如我设定 x 方向和 y 方向的切片数量分别为 5 的话，结果应该是这样：  

![](https://upload-images.jianshu.io/upload_images/6954292-bc13c1161065ef20.png)

影像规整裁切结果

  并且，这些影像都应该带有位置信息，在 ENVI 或 ArcGIS 等平台中加载，能够拼成完整的一幅影像。通过上面裁切的 demo，想要实现这个功能并不难，只需要多套两层循环即可。现在需要考虑的，是如何利用 Spark，进行分布式的影像裁切。  
  放了一波寒假，一段时间没碰这个了。。。。。。现在考虑的思路是，建立一个格网索引，将影像分发到它们所在的格网中，并裁出格网与影像相交的部分，最后在每一个格网内将影像进行重叠、拼接，重叠部分则计算平均值。  
  **更新**，很长一段时间没有在看 gdal 了，这次把这部分内容的坑填上吧。Spark 拥有众多算子，想要实现一个目标可以有很多种写方法（光是 wordcount 我所知道的就有 2~3 种）。因此，对于影像的规整裁切，我所采用的思路、方法可能不是最优的，但这里也先贴出来供参考。

1.  输入及元数据信息提取  
      Spark 不能直接读取 GeoTiff 数据，这里还是要靠 GDAL。在我的思路中输入影像应该是可以有多幅的，最终的结果应该呈现出所有输入影像拼接后再裁切的样子（当然实际上并没有拼接的过程）。所以首先统计了输入影像的各类元数据信息，来为进一步切分做准备。

-   首先定义一个需要获得的信息的类


    public class DatasetInfo implements Serializable {
      public String projection; 
      public int dataType; 
      public int bandCount; 
      public double xPixel; 
      public double yPixel; 
      public double xRotate; 
      public double yRotate; 
      public double noData; 
      public Extent extent; 

      public DatasetInfo(String projection, int dataType, int bandCount, double xPixel, double yPixel,
        double xRotate, double yRotate, double noData, Extent extent) {
        this.projection = projection;
        this.dataType = dataType;
        this.bandCount = bandCount;
        this.xPixel = xPixel;
        this.yPixel = yPixel;
        this.xRotate = xRotate;
        this.yRotate = yRotate;
        this.noData = noData;
        this.extent = extent;
      }
    }

    public class Extent implements Serializable {
      public double minX;
      public double minY;
      public double maxX;
      public double maxY;

      public Extent(double minX, double minY, double maxX, double maxY) {
        this.minX = minX;
        this.minY = minY;
        this.maxX = maxX;
        this.maxY = maxY;
      }

      
      public Extent overlap(Extent extent){
        double newMinX = Math.max(minX, extent.minX);
        double newMaxX = Math.min(maxX, extent.maxX);
        double newMinY = Math.max(minY, extent.minY);
        double newMaxY = Math.min(maxY, extent.maxY);
        if (newMinX >= newMaxX || newMinY >= newMaxY) {
          throw new RuntimeException("Not overlap extents!");
        }
        return new Extent(newMinX, newMinY, newMaxX, newMaxY);
      }
      
      public Extent union(Extent extent) {
        double newMinX = Math.min(minX, extent.minX);
        double newMinY = Math.min(minY, extent.minY);
        double newMaxX = Math.max(maxX, extent.maxX);
        double newMaxY = Math.max(maxY, extent.maxY);
        return new Extent(newMinX, newMinY, newMaxX, newMaxY);
      }
    } 

-   根据输入路径参数（输入文件路径的 List）整合元数据  
      思路是先利用 map 算子读取每一幅输入影像的元数据信息，再在 reduce 中统合。统合原则是，外包框进行合并，其他信息则要确保一致，否则抛出异常。


     JavaRDD<String> pathsRDD = sc.parallelize(inputs).cache();
    DatasetInfo datasetInfo = pathsRDD.map(file -> {
          
          
          gdal.AllRegister();
          Dataset ds = gdal.Open(file, gdalconst.GA_ReadOnly);
          double[] geoTransform = ds.GetGeoTransform();
          
          int width = ds.getRasterXSize();
          int height = ds.getRasterYSize();
          double leftUpX = geoTransform[0];
          double leftUpY = geoTransform[3];
          double rightBottomX = leftUpX + width * geoTransform[1] + height * geoTransform[2];
          double rightBottomY = leftUpY + width * geoTransform[4] + height * geoTransform[5];
          Extent extent = new Extent(leftUpX, rightBottomY, rightBottomX, leftUpY);
          String proj = ds.GetProjection();
          int dataType = ds.GetRasterBand(1).GetRasterDataType();
          int bandCount = ds.getRasterCount();
          Double[] val = new Double[1];
          
          ds.GetRasterBand(1).GetNoDataValue(val);
          double noData = DEFAULT_NO_DATA_VALUE;
          if (val[0] != null) {
            noData = val[0];
          }
          ds.delete();
          return new DatasetInfo(proj, dataType, bandCount, geoTransform[1], geoTransform[5],
            geoTransform[2], geoTransform[4], noData, extent);
        }).reduce((info1, info2) -> {
          
          
          if (!info1.projection.equals(info2.projection)
            || info1.dataType != info2.dataType
            || info1.bandCount != info2.bandCount
            || info1.xPixel != info2.xPixel
            || info1.yPixel != info2.yPixel
            || info1.xRotate != info2.xRotate
            || info1.yRotate != info2.yRotate
            || info1.noData != info2.noData) {
            throw new RuntimeException
              ("Input images should have same projection, dataType, bands, resolution, rotation and noData");
          }
          
          Extent extent = info1.extent.union(info2.extent);
          return new DatasetInfo(info1.projection, info1.dataType, info1.bandCount,
            info1.xPixel, info1.yPixel, info2.xRotate, info2.yRotate, info1.noData, extent);
        });
      } 

-   利用统合元数据信息构建格网索引。假设格网是个正方形，其分片数（slices）为格网数量的平方。比如你最后想输出 5x5 的切片，那么格网数量就是 25，分片数就是 5。


    public class GridCell implements Serializable {
      
      private Extent extent;
      
      private int slices;
      
      private int cellXSize;
      private int cellYSize;
      
      private int allXSize;
      private int allYSize;
      
      private double xPixel;
      private double yPixel;

      public GridCell(Extent extent, int slices, double xPixel, double yPixel) {
        this.extent = extent;
        this.slices = slices;
        this.xPixel = xPixel;
        this.yPixel = yPixel;
        this.allXSize = (int) ((extent.maxX - extent.minX) / xPixel);
        this.allYSize = (int) ((extent.minY - extent.maxY) / yPixel);
        this.cellXSize = (int) Math.ceil(allXSize * 1.0 / slices);
        this.cellYSize = (int) Math.ceil(allYSize * 1.0 / slices);
      }

      public int getXIndex(double x) {
        int widthSize = (int) ((x - extent.minX) / xPixel);
        int xIndex = widthSize / cellXSize;
        return Math.min(xIndex, slices - 1);
      }

      public int getYIndex(double y) {
        int heightSize = (int) ((y - extent.maxY) / yPixel);
        int yIndex = heightSize / cellYSize;
        return Math.min(yIndex, slices - 1);
      }

      public Extent getCellExtent(int xIndex, int yIndex) {
        if (xIndex >= slices || yIndex >= slices) {
          return null;
        }
        int minXIndex = xIndex * cellXSize;
        int maxXIndex = Math.min(minXIndex + cellXSize, allXSize);
        int minYIndex = yIndex * cellYSize;
        int maxYIndex = Math.min(minYIndex + cellYSize, allYSize);
        return new Extent(extent.minX + minXIndex * xPixel,
          extent.maxY + maxYIndex * yPixel,
          extent.minX + maxXIndex * xPixel,
          extent.maxY + minYIndex * yPixel);
      }
    }

    GridCell gridCell = new GridCell(datasetInfo.extent, slices, datasetInfo.xPixel, datasetInfo.yPixel); 

  看代码可能有点抽象，具体来讲，假设我有一个 5x5 的影像，想切成 3x3 的格网

![](https://upload-images.jianshu.io/upload_images/6954292-9965e66666b5aae2.png)

GridCell 各属性含义

-   进行裁切  
      准备工作都做完了，接下来开始进行裁切。首先，为了减少对象的复制次数，把必要的东西先进行广播。


     int[] bands = new int[datasetInfo.bandCount];
        for (int i = 0; i < bands.length; i++) {
          bands[i] = i + 1;
        }

        Broadcast<Integer> byteSizeBroadcast = sc.broadcast(BYTESIZE);
        Broadcast<String> outputBroadcast = sc.broadcast(outputDir);
        Broadcast<DatasetInfo> datasetInfoBroadcast = sc.broadcast(datasetInfo);
        Broadcast<GridCell> gridCellBroadcast = sc.broadcast(gridCell);
        Broadcast<int[]> bandsBroadcast = sc.broadcast(bands);
        Broadcast<byte[]> noDataBroadcast = sc.broadcast(nodata); 

  接下来，是裁切的主体逻辑部分。简单来讲就是先对输入数据进行索引，生成 key 为格网号，value 为原始数据的 PairRDD，其中一个输入可以和多个格网相交，因此可以对应多个格网，所以这里采用的算子是 flatMapToPair。然后，将生成的 PairRDD 按照格网号聚集（groupByKey），并在 foreach 算子内对每一个格网内的数据进行处理。对于每一个格网，先生成一个统一的 data 空数组，遍历每一个与之相交的数据，裁剪出相交的部分，并将该部分数据写进 data 中。然后利用 gdal 将 data 数组写到本地文件中即可。如下所示：

![](https://upload-images.jianshu.io/upload_images/6954292-07a1d99ef292fc81.png)

裁切过程

  然后是代码，比较长。

    pathsRDD.flatMapToPair(path -> {
          gdal.AllRegister(); 
          Dataset dataset = gdal.Open(path, gdalconst.GA_ReadOnly);
          double[] geoTransform = dataset.GetGeoTransform();
          
          double minX = geoTransform[0], maxX, minY, maxY = geoTransform[3];
          maxX = minX + dataset.getRasterXSize() * geoTransform[1] + dataset.getRasterYSize() * geoTransform[2];
          minY = maxY + dataset.getRasterXSize() * geoTransform[4] + dataset.getRasterYSize() * geoTransform[5];
          int minXIndex = gridCellBroadcast.value().getXIndex(minX);
          int maxXIndex = gridCellBroadcast.value().getXIndex(maxX);
          int minYIndex = gridCellBroadcast.value().getYIndex(maxY);
          int maxYIndex = gridCellBroadcast.value().getYIndex(minY);
          List<Tuple2<Tuple2<Integer, Integer>, String>> grids = new ArrayList<>();
          for (int x = minXIndex; x <= maxXIndex; x++) {
            for (int y = minYIndex; y <= maxYIndex; y++) {
              grids.add(new Tuple2<>(new Tuple2<>(x, y), path));
            }
          }
          dataset.delete();
          return grids.iterator();
        }).groupByKey().foreach(grids -> {
          gdal.AllRegister();
          DatasetInfo curInfo = datasetInfoBroadcast.value();
          Tuple2<Integer, Integer> grid = grids._1();
          Iterator<String> datasets = grids._2().iterator();
          
          Path path = Files.createTempDirectory("gdalTemp");
          String datasetPath = Paths.get(path.toString(), grid._2() + "_" + grid._1() + ".tif").toString();
          Driver driver = gdal.GetDriverByName("GTiff");
          
          Extent gridExtent = gridCellBroadcast.value().getCellExtent(grid._1(), grid._2());
          int gridWidth = (int) ((gridExtent.maxX - gridExtent.minX) / curInfo.xPixel);
          int gridHeight = (int) ((gridExtent.minY - gridExtent.maxY) / curInfo.yPixel);
          
          Dataset gridDs = driver.Create(datasetPath, gridWidth, gridHeight, curInfo.bandCount, curInfo.dataType);
          gridDs.SetProjection(curInfo.projection);
          gridDs.SetGeoTransform(new double[]{gridExtent.minX, curInfo.xPixel, curInfo.xRotate, gridExtent.maxY,
            curInfo.yRotate, curInfo.yPixel});

          
          int byteToType = gdal.GetDataTypeSize(curInfo.dataType) / byteSizeBroadcast.value();
          byte[] dst = new byte[gridWidth * gridHeight * curInfo.bandCount * byteToType];
          fillWithNodata(dst, noDataBroadcast.value());
          
          while (datasets.hasNext()) {
            Dataset dataset = gdal.Open(datasets.next(), gdalconst.GA_ReadOnly);
            double[] geoTransform = dataset.GetGeoTransform();
            double minX = geoTransform[0];
            double maxX = minX + dataset.getRasterXSize() * geoTransform[1] + dataset.getRasterYSize() * geoTransform[2];
            double maxY = geoTransform[3];
            double minY = maxY + dataset.getRasterXSize() * geoTransform[4] + dataset.getRasterYSize() * geoTransform[5];
            
            Extent overlap = gridExtent.overlap(new Extent(minX, minY, maxX, maxY));
            int overlapWidth = (int) ((overlap.maxX - overlap.minX) / curInfo.xPixel);
            int overlapHeight = (int) ((overlap.minY - overlap.maxY) / curInfo.yPixel);
            
            int srcXOff = (int) ((overlap.minX - minX) / curInfo.xPixel);
            int srcYOff = (int) ((overlap.maxY - maxY) / curInfo.yPixel);
            if (srcXOff + overlapWidth > dataset.getRasterXSize()) {
              overlapWidth = dataset.getRasterXSize() - srcXOff;
            }
            if (srcYOff + overlapHeight > dataset.getRasterYSize()) {
              overlapHeight = dataset.getRasterYSize() - srcYOff;
            }
            byte[] data = new byte[overlapWidth * overlapHeight * curInfo.bandCount * byteToType];
            dataset.ReadRaster(srcXOff, srcYOff, overlapWidth, overlapHeight, overlapWidth, overlapHeight,
              curInfo.dataType, data, bandsBroadcast.value());
            
            int dstXOff = (int) ((overlap.minX - gridExtent.minX) / curInfo.xPixel);
            int dstYOff = (int) ((overlap.maxY - gridExtent.maxY) / curInfo.yPixel);
            dataMerge(data, overlapWidth, overlapHeight, dst, gridWidth, gridHeight, dstXOff, dstYOff, byteToType,
              noDataBroadcast.value());
          }
          
          gridDs.WriteRaster(0, 0, gridWidth, gridHeight, gridWidth, gridHeight,
            curInfo.dataType, dst, bandsBroadcast.value());
          gridDs.FlushCache();
          gridDs.delete();
        }); 

  涉及的辅助函数

     private void fillWithNodata(byte[] arr, byte[] noData) {
        int count = arr.length / noData.length;
        for (int i = 0; i < count; i++) {
          System.arraycopy(noData, 0, arr, noData.length * i, noData.length);
        }
    }

    private void dataMerge(byte[] src, int srcXSize, int srcYSize, byte[] dst, int dstXSize, int dstYSize,
        int xoff, int yoff, int byteToType, byte[] noData) {
        int srcRowPixels = srcXSize * byteToType;
        int srcBandPixels = srcRowPixels * srcYSize;
        int dstRowPixels = dstXSize * byteToType;
        int dstBandPixels = dstRowPixels * dstYSize;
        int bandCount = src.length / srcBandPixels;
        for (int i = 0; i < bandCount; i++) {
          for (int r = 0; r < srcYSize; r++) {
            int dstR = r + yoff;
            if (dstR >= dstYSize) {
              continue;
            }
            for (int c = 0; c < srcXSize; c++) {
              int dstC = c + xoff;
              if (dstC >= dstXSize) {
                continue;
              }
              int srcFirstIndex = srcBandPixels * i + srcRowPixels * r + byteToType * c;
              int dstFirstIndex = dstBandPixels * i + dstRowPixels * dstR + byteToType * dstC;
              
              if (ArrayUtils.
                isEquals(noData, ArrayUtils.subarray(src, srcFirstIndex, srcFirstIndex + byteToType))) {
                continue;
              } else if (ArrayUtils.
                isEquals(noData, ArrayUtils.subarray(dst, dstFirstIndex, dstFirstIndex + byteToType))) {
                for (int b = 0; b < byteToType; b++) {
                  dst[dstFirstIndex + b] = src[srcFirstIndex + b];
                }
              } else {
                for (int b = 0; b < byteToType; b++) {
                  dst[dstFirstIndex + b] = (byte) ((dst[dstFirstIndex + b] + src[srcFirstIndex + b]) / 2);
                }
              }
            }
          }
        }
      } 

-   一些注意事项  
    1）GDAL 中的对象都没有实现 Serializable 接口，不能在 driver 和 Executor、各 Executor 之间传递，所以传递时使用了原始数据路径而不是 Dataset 对象。此外，也尽量在同一个 task 中完成 Dataset 的创建和写入。  
    2）文中的 merge 方法在取平均时逻辑并不严密，假如同一像元先后要写入三个值：a、b、c，得到的结果是 (((a + b) / 2 ) + c ) / 2。要得到 (a + b + c) / 3 的结果，需要另外声明一个与 buff 同样长度的 count 数组来记录个数。  
    3）如前文所说，这个并行逻辑可能不是最佳的，应该会有更高效的写法。欢迎交流。

  gdal 提供了重投影的接口，且有多个重载：

![](https://upload-images.jianshu.io/upload_images/6954292-1feb12174b1fc77a.png)

gdal 重投影接口

我自己一般使用

> ReprojectImage​(Dataset src_ds, Dataset dst_ds, java.lang.String  
> src_wkt, java.lang.String dst_wkt, int resampleAlg)

  在使用该接口之前，需要自行构建**dst_ds**即目标数据集，其波段数、数据类型可以与原数据**src_ds**保持一致，投影与目标投影**dst_wkt**相同，但其宽、高、空间范围等都需要重新计算。  
  首先计算空间范围，方法是先单独对原始影像的四个角点进行重投影，得到新影像的角点值。

     SpatialReference src = new SpatialReference();
    src.ImportFromWkt(srcPrj);
    SpatialReference dst = new SpatialReference();
    dst.ImportFromWkt(dstPrj);
    CoordinateTransformation coordinateTransformation = osr.CreateCoordinateTransformation(src, dst);

    double[][] coords = new double{}

    coordinateTransformation.TransformPoints(coords);

    double minX = Double.MAX_VALUE, minY = Double.MAX_VALUE, maxX = Double.MIN_VALUE, maxY = Double.MIN_VALUE;
    for (int i = 0; i < coords.length; i++) {
       if (coords[i][0] < minX) minX = coords[i][0];
       if (coords[i][0] > maxX) maxX = coords[i][0];
       if (coords[i][1] < minY) minY = coords[i][1];
       if (coords[i][1] > maxY) maxY = coords[i][1];
    } 

  遗憾的是，目标影像的宽高需要自己根据实际情况指定，也可以指定一个分辨率，并根据坐标范围来计算。这样一来，就可以通过输出路径、影像宽高、波段数、数据类型来创建一个新的 Dataset，利用上面计算出来的左上角坐标和自己指定的分辨率来设置该 Dataset 的 geoTransform（y 方向的分辨率是 x 方向的负数，rotate 一般为 0），并设置好参考系即可。  
  最后是投影过程中采用的重采样方法**resampleAlg**，重采样的含义及常用方法可自行百度，在 gdalconst 中提供了如下方法：  

![](https://upload-images.jianshu.io/upload_images/6954292-8ffe9c34cbf235f2.png)

gdal 中的重采样方法

  其中，**GRA_NearestNeighbour**（最邻近法）是重投影时的默认方法。所有参数都创建好后，直接调用即可：

    gdal.ReprojectImage(srcDs, dstDs, srcPrj, dstPrj, reasampleAlg);
    dstDs.FlushCache(); 

![](https://upload-images.jianshu.io/upload_images/6954292-94115ae4bffc5c1a.png)

咕咕咕

更多精彩内容，就在简书 APP

"小礼物走一走，来简书关注我"

[![](https://upload.jianshu.io/users/upload_avatars/13698230/d3327d57-573b-4bf0-a36b-5c6a088c102e?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
](https://www.jianshu.com/u/77455133afd8)共 1 人赞赏

[![](https://upload.jianshu.io/users/upload_avatars/6954292/78a964a4-68d5-431b-847a-8b9d6fb392bf.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/0fa55627e517)

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   1、通过 CocoaPods 安装项目名称项目信息 AFNetworking 网络请求组件 FMDB 本地数据库组件 SD...

-   开始之前 交互沙盘是美国加州大学戴维斯分校 (UC Davis) 的开源项目(GNU GPL 协议), 项目的官方页面:...

    [![](https://cdn2.jianshu.io/assets/default_avatar/12-aeeea4bedf10f2a12c0d50d626951489.jpg)
    杜传](https://www.jianshu.com/u/958bec1def1d)阅读 10,396 评论 3 赞 12

    [![](https://upload-images.jianshu.io/upload_images/1246531-83fe77e739575f87.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/887997e502bc)

-   各位朋友大家好，今天小编又要来唠叨唠叨了！唠叨些什么内容呢? 跟着小编一起往下看吧。 自由职业人是现在许多年轻朋友的...

-   米兰 · 昆德拉在《生命中不能承受之轻》里说，从现在起，我开始谨慎地选择我的生活，我不再轻易让自己迷失在各种诱惑里，我...

    [![](https://cdn2.jianshu.io/assets/default_avatar/10-e691107df16746d4a9f3fe9496fd1848.jpg)
    不尽之思](https://www.jianshu.com/u/503bd1e2ab0f)阅读 133 评论 0 赞 0

-   花儿泼墨，绚烂了夏，落叶题词，静美了秋！ 滴滴清雨，和着八月燥热的气息如天水沁润而来，宛如一副展开的花卷，赏心悦目...
       [![](https://cdn2.jianshu.io/assets/default_avatar/13-394c31a9cb492fcb39c27422ca7d2815.jpg)
       秋雨寒](https://www.jianshu.com/u/755fed67d0ad)阅读 145 评论 1 赞 3 
    [https://www.jianshu.com/p/c25f9360459f](https://www.jianshu.com/p/c25f9360459f)
