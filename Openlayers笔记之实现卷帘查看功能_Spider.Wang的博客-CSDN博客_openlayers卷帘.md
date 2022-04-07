# Openlayers笔记之实现卷帘查看功能_Spider.Wang的博客-CSDN博客_openlayers卷帘
> 在 `Web GIS` 的开发中，总是会遇到很多新奇的需求，比如卷帘查看功能。  
> 卷帘查看是为了方便比较不同时间、不同类型的两张地图在同一地点的差异性，通过卷帘操作能够快速对比出两张地图的差异性。

## 1. 实现思路

`Openlayers` [API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)文档中提供了控制地图范围的接口 (`Extent`)。通过加载两张地图，移动前端绘制的卷帘轴，实时获取卷帘轴当前的屏幕坐标（x, y)，并实时控制上面层级地图的显示范围，即可实现该功能。

## 2. 具体实现

首先创建图层，加载两张在线地图：

```javascript

function initMap() {
  window.olMap = new window.ol.Map({
    target: 'map',
    layers: [
      new window.ol.layer.Tile({
        
        source: new window.ol.source.TileArcGISRest({
          url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer',
          wrapX: true,
          tileSize: [256, 256]
        }),
        zIndex: 1
      }),
      new window.ol.layer.Tile({
        
        source: new window.ol.source.XYZ({
          url: 'http://wprd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=1&style=6',
          wrapX: true,
          tileSize: [256, 256]
        }),
        zIndex: 1
      })
    ],
    view: new window.ol.View({
      center: window.ol.proj.fromLonLat([120.16, 32.71]),
      zoom: 13,
      enableRotation: true
    }),

    pixelRatio: typeof window.devicePixelRatio !== 'undefined' ? window.devicePixelRatio : 1
  });

  
  setTimeout(() => {
    updateMapExtent({
      x: window.document.documentElement.clientWidth / 2,
      y: 0
    });
  })
}

```

其次根据 `openlayers` 提供的接口，编写更新地图范围的二次开发接口：

```javascript

function updateMapExtent (windowPostion) {
  
  let pos = window.olMap.getCoordinateFromPixel([windowPostion.x, windowPostion.y])

  
  window.olMap.getLayers().getArray()[1].setExtent([0, 0, pos[0], pos[1]])
}

```

最后，控制卷帘轴的移动，实时获取卷帘轴的屏幕坐标，并调用 `updateMapExtent` 方法实现功能！

```javascript

function bindMoveEvent () {
  window.document.addEventListener('mousemove', function (e) {
    document.getElementById('line').style.left = (e.x - 4) + 'px';
    updateMapExtent({
      x: e.x,
      y: 0
    });
  })
}

```

## 3. 最终效果

![](https://img-blog.csdnimg.cn/de5d19cedd394e4099ca42fee0e8fb9a.gif#pic_center)

## 4. 总结

以上是 `Openlayers` 实现卷帘查看的方法，`Cesium` 也可以采用同样的思路实现。

注：示例代码可参见 [OL_RollerView](https://github.com/wangdunwen/ol_roller_show)。

* * *

created by @SpiderWang  
转载请注明作者及链接 
 [https://blog.csdn.net/Spider_wang/article/details/119735990](https://blog.csdn.net/Spider_wang/article/details/119735990)
