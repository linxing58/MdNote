# GeoMesa-Kafka（GeoServer中创建GeoMesa-Kafka数据存储并发布图层）_tiger-hcx的博客-CSDN博客_geomesa kafka
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

* * *

1\. 必须已经在集群中安装了 GeoMesa-Kafka（[CDH6.2.1 集群中安装 geomesa-kafka 3.1.0](https://blog.csdn.net/qq_18188119/article/details/111562769)），并且在 geoserver 中安装了 GeoMesa-Kafka 的插件（[geoserver 中安装 GeoMesa-Kafka 插件](https://blog.csdn.net/qq_18188119/article/details/111573780)）  
2. 这里模拟 100 万车辆的位置。使用 GeoMesa-Kafka 插入 100 万辆车辆的位置信息，然后再 geoserver 上创建存储发布图层，查看图层，测试查询速度。

## [schema](https://so.csdn.net/so/search?q=schema&spm=1001.2101.3001.7020)的创建

```java

final String sftName = "cars8";

final String sftSchema = "car:String,dtg:Date,*geom:Point:srid=4326";
SimpleFeatureType sft = SimpleFeatureTypes.createType(sftName, sftSchema);
ds.createSchema(sft);

```

## 生成并写入

```java
FeatureWriter<SimpleFeatureType, SimpleFeature> writer = ds.getFeatureWriterAppend(sft.getTypeName(), Transaction.AUTO_COMMIT);
SimpleFeature toWrite = writer.next();

for (int i = 1; i <= 1000000; i++) {
    SimpleFeatureBuilder builder = new SimpleFeatureBuilder(sft);
    
    String car = "闽A U"+i;
    builder.set("car", car);
    builder.set("dtg", new Date());
    
    double latv = getRandom(500, 5000)/100.0;
    double lngv = getRandom(9000, 14600)/100.0;
    double lat = new BigDecimal(String.valueOf(latv)).doubleValue();
    double lng = new BigDecimal(String.valueOf(lngv)).doubleValue();
    String point = "POINT ("+lng+ " " +lat+")";
    builder.set("geom",point);
    builder.featureUserData(Hints.USE_PROVIDED_FID, Boolean.TRUE);
    SimpleFeature feature = builder.buildFeature(car);
    toWrite.setAttributes(feature.getAttributes());
    ((FeatureIdImpl) toWrite.getIdentifier()).setID(feature.getID());
    toWrite.getUserData().put(Hints.USE_PROVIDED_FID, Boolean.TRUE);
    toWrite.getUserData().putAll(feature.getUserData());
    writer.write();
}

writer.close();

```

## 在 kafka 中查看是否有数据了

## 命令行查看

进入 kafka 的数据存储路径（cdh 6.2.1 的话是在 / var/local/kafka/data），查看有一个 geomesa-ds-kafka-cars8 开头的，这就是刚刚生成的。topic 的名字为 geomesa-ds-kafka-cars8。

![](https://img-blog.csdnimg.cn/20210111103639880.png)

## kafka 图形化界面查看

这里我们安装的是 kafka eagle。  
![](https://img-blog.csdnimg.cn/20210111103719793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

我这里有 4 台机器，partition 给的也是 4。  
![](https://img-blog.csdnimg.cn/20210111103745129.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

## 创建存储

点击 Kafka（GeoMesa）  
![](https://img-blog.csdnimg.cn/20210111104042953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

填写参数  
![](https://img-blog.csdnimg.cn/20210111104304440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

## 发布图层

点击发布  
![](https://img-blog.csdnimg.cn/20210111104348866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

发布参数设置，整简单点，设置下图层名，以及计算一下图层数据的边框范围  
![](https://img-blog.csdnimg.cn/20210111104531785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210111104510467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

点击保存后就可以去预览图层了  
![](https://img-blog.csdnimg.cn/20210111104619681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

点击 openlayers，咦发现什么都没有。这里就是涉及到了很多的参数设置，这里就不详细展开了，这里面涉及到很多 kafka 的知识。这里是因为我们 geoserver 相当于实现消费者，  
我们是先生产后创建的消费者。这里我们修改一下创建存储的时候的一个参数 kafka.consumer.read-back 为 inf，这个意思是读取这个 topic 中的所有数据。或者也可以重新生产数据。

![](https://img-blog.csdnimg.cn/20210111105037316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

修改参数  
![](https://img-blog.csdnimg.cn/20210111105556501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

## 查看图层

重新查看，图层，100 万点密密麻麻显示着。。  
![](https://img-blog.csdnimg.cn/20210111105805567.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

放大点击查询某个车辆的信息  
查询数据是相当快的。  
![](https://img-blog.csdnimg.cn/20210111110046106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

1\. 这里知识简单的测试一下，其中还有很多细节可以研究。kafka 的知识要事先具备。  
2. 欢迎互相学习，交流讨论，本人的微信：huangchuanxiaa。 
 [https://blog.csdn.net/qq_18188119/article/details/112462016](https://blog.csdn.net/qq_18188119/article/details/112462016)
