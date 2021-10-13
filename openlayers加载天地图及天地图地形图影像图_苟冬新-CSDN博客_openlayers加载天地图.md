# openlayers加载天地图及天地图地形图影像图_苟冬新-CSDN博客_openlayers加载天地图
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[苟冬新](https://blog.csdn.net/weixin_40187450) 2020-05-31 17:06:52 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 5089 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 收藏  14 

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

**openlayer 加载天地图、天地图地形图、天地图影像图，相关代码有注释。** 

**加载效果：** 

##### 天地图底图

![](https://img-blog.csdnimg.cn/2020053116582179.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE4NzQ1MA==,size_16,color_FFFFFF,t_70#pic_center)

##### 天地图地形图

![](https://img-blog.csdnimg.cn/20200531165901923.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE4NzQ1MA==,size_16,color_FFFFFF,t_70#pic_center)

##### 天地图影像图

![](https://img-blog.csdnimg.cn/20200531165935782.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE4NzQ1MA==,size_16,color_FFFFFF,t_70#pic_center)

**相关代码：** 

```javascript
import {XYZ,TileImage} from 'ol/source';

export function tianditu(map) {
  
  
  
  var source = new XYZ({
    url: "http://t4.tianditu.com/DataServer?T=vec_w&tk=申请的天地图key&x={x}&y={y}&l={z}"
  });
  var tileLayer = new TileLayer({
    id: "tileLayer",
    title: "天地图",
    layerName:"baseMap",
    source: source
  });
  
  var sourceMark = new XYZ({
    url: 'http://t3.tianditu.com/DataServer?T=cva_w&tk=申请的天地图key&x={x}&y={y}&l={z}'
  });
  var tileMark = new TileLayer({
    id: "tileMark",
    title: "标注图层",
    layerName:"baseMap",
    source: sourceMark,

  });
  
  var sourceSatellite = new XYZ({
    url: 'http://t3.tianditu.com/DataServer?T=img_w&tk=申请的天地图key&x={x}&y={y}&l={z}'
  });
  var tileSatellite = new TileLayer({
    id: "tileSatellite",
    title: "卫星图",
    layerName:"baseMap",
    source: sourceSatellite

  });
  
  var map_ter = new TileLayer({
    id: "map_ter",
    title: "天地图地形",
    layerName:"baseMap",
    source: new XYZ({
      url: "http://t4.tianditu.com/DataServer?T=ter_w&tk=申请的天地图key&x={x}&y={y}&l={z}"
    })
  })
  var map_cta = new TileLayer({
    id: "map_cta",
    title: "天地图标注",
    layerName:"baseMap",
    source: new XYZ({
      url: "http://t4.tianditu.com/DataServer?T=cva_w&tk=申请的天地图key&x={x}&y={y}&l={z}"
    })
  });

  return {
    "tileLayer": tileLayer,
    "tileMark": tileMark,
    "tileSatellite": tileSatellite,
    "map_ter": map_ter,
    "map_cta": map_cta
  };
}

```

 [https://blog.csdn.net/weixin_40187450/article/details/106456488](https://blog.csdn.net/weixin_40187450/article/details/106456488)
