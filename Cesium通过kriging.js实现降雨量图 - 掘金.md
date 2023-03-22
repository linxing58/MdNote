# Cesium通过kriging.js实现降雨量图 - 掘金
最近项目上需要前端实现一个降雨量图，一开始一头蒙，这不是gis干的事吗？挠了一天头，后面经过几番度娘，找到了解决办法，就是kriging.js。先给大家看下降雨量图大致的样子，我找了深圳市气象局的一张图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cc811a04f9342ffb28d5657413d9a76~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

克里金法（Kriging）是依据[协方差函数](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%258D%258F%25E6%2596%25B9%25E5%25B7%25AE%25E5%2587%25BD%25E6%2595%25B0%2F3737807%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E5%8D%8F%E6%96%B9%E5%B7%AE%E5%87%BD%E6%95%B0/3737807?fromModule=lemma_inlink")对[随机过程](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%259A%258F%25E6%259C%25BA%25E8%25BF%2587%25E7%25A8%258B%2F368895%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E9%9A%8F%E6%9C%BA%E8%BF%87%E7%A8%8B/368895?fromModule=lemma_inlink")/随机场进行空间建模和预测（插值）的回归算法 。在特定的随机过程，例如固有平稳过程中，克里金法能够给出最优线性无偏估计（Best Linear Unbiased Prediction, [BLUP](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FBLUP%2F10172213%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/BLUP/10172213?fromModule=lemma_inlink")），因此在[地统计学](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%259C%25B0%25E7%25BB%259F%25E8%25AE%25A1%25E5%25AD%25A6%2F2892595%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E5%9C%B0%E7%BB%9F%E8%AE%A1%E5%AD%A6/2892595?fromModule=lemma_inlink")中也被称为空间最优无偏估计器（spatial BLUP） 。

对克里金法的研究可以追溯至二十世纪60年代，其算法原型被称为普通克里金（Ordinary Kriging, OK），常见的改进算法包括泛克里金（Universal Kriging, UK）、协同克里金（Co-Kriging, CK）和析取克里金（Disjunctive Kriging, DK）；克里金法能够与其它模型组成混合算法。

若协方差函数的形式等价，且建模对象是平稳[高斯过程](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%25AB%2598%25E6%2596%25AF%25E8%25BF%2587%25E7%25A8%258B%2F4535435%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E9%AB%98%E6%96%AF%E8%BF%87%E7%A8%8B/4535435?fromModule=lemma_inlink")，普通克里金的输出与[高斯过程回归](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%25AB%2598%25E6%2596%25AF%25E8%25BF%2587%25E7%25A8%258B%25E5%259B%259E%25E5%25BD%2592%2F23271644%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E9%AB%98%E6%96%AF%E8%BF%87%E7%A8%8B%E5%9B%9E%E5%BD%92/23271644?fromModule=lemma_inlink")（Gaussian Process Regression, GPR）在正态似然下输出的[均值](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%259D%2587%25E5%2580%25BC%2F5922988%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E5%9D%87%E5%80%BC/5922988?fromModule=lemma_inlink")和[置信区间](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25BD%25AE%25E4%25BF%25A1%25E5%258C%25BA%25E9%2597%25B4%2F7442583%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E7%BD%AE%E4%BF%A1%E5%8C%BA%E9%97%B4/7442583?fromModule=lemma_inlink")相同，有稳定的预测效果。克里金法是典型的地统计学算法，被应用于[地理科学](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%259C%25B0%25E7%2590%2586%25E7%25A7%2591%25E5%25AD%25A6%2F4683237%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E5%9C%B0%E7%90%86%E7%A7%91%E5%AD%A6/4683237?fromModule=lemma_inlink")、[环境科学](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%258E%25AF%25E5%25A2%2583%25E7%25A7%2591%25E5%25AD%25A6%2F337267%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E7%8E%AF%E5%A2%83%E7%A7%91%E5%AD%A6/337267?fromModule=lemma_inlink")、[大气科学](https://link.juejin.cn/?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%25A4%25A7%25E6%25B0%2594%25E7%25A7%2591%25E5%25AD%25A6%2F85598%3FfromModule%3Dlemma_inlink "https://baike.baidu.com/item/%E5%A4%A7%E6%B0%94%E7%A7%91%E5%AD%A6/85598?fromModule=lemma_inlink")等领域 。 一大堆官方解释我也看的头晕，这些都是官方说法，我们只需要知道克里金算法在我们实现如降雨量图时起到了什么作用,我总结下来就是克里金算法在特定区域内对区域化变量进行无偏最优估计,能够顺滑衔接。

克里金插值算法有开源的js，GitHub地址[kriging.js](https://link.juejin.cn/?target=https%3A%2F%2Fgitcode.net%2Fmirrors%2Foeo4b%2Fkriging.js%3Futm_source%3Dcsdn_github_accelerator "https://gitcode.net/mirrors/oeo4b/kriging.js?utm_source=csdn_github_accelerator")

kriging.train(t, x, y, model, sigma2, alpha)：使用gaussian、exponential或spherical模型对数据集进行训练，返回的是一个variogram对象；

kriging.grid(polygons,variogram,width)：使用刚才的variogram对象使polygons描述的地理位置内的格网元素具备不一样的预测值；

kriging.plot(canvas,grid,xlim,ylim,colors): 将得到的格网grid渲染至canvas上。

1、站点数据
------

站点可以理解成监测点的位置或范围，多站点的数据形成一个特定范围,样例如下：

```js
const sitesData = [
  { name: "广坤街道", lon: 113.7878622, lat: 22.68899985 },
  { name: "刘能街道", lon: 113.8817965, lat: 22.76980065 },
  { name: "老四街道", lon: 114.0154123999, lat: 22.7211864899 },
  { name: "晓峰街道", lon: 113.9847207999, lat: 22.5880693099 },
  { name: "大拿街道", lon: 113.961574, lat: 22.7522136099 },
  { name: "老徐街道", lon: 113.7898898, lat: 22.73634492 }
]

```

1、站点值
-----

每个站点值和上面的站点一一对应,样例如下：

```js
const siteValues = [52.4, 0.5, 5.5, 11.7, 0, 13]

```

2、定义色带
------

根据自己的具体需求定义每个范围的最大最小值，并设置一个颜色

```js
const colors = [
  { min: 0, max: 0.1, color: "#008FFF" },
  { min: 0.1, max: 10, color: "#72D66B" },
  { min: 10, max: 20, color: "#3DB83D" },
  { min: 20, max: 30, color: "#3DB83D" },
  { min: 30, max: 40, color: "#3DB83D" },
  { min: 70, max: 300, color: "#3DB83D" }
]

```

3、修改源码，以获取自定义色值
---------------

```js

kriging.getColor = function (colors, z) {
  let l = colors.length;
  for (let i = 0; i < l; i++) {
    if (z >= colors[i].min && z < colors[i].max) return colors[i].color;
  }
  if (z < 0) {
    return colors[0].color;
  }
};

```

4、添加站点到Cesium中
--------------

通过遍历站点数据添加Cesium的实体entity中，并用数组保存下来，并且调用绘制方法

```js
import { drawKriging } from "./drawKriging";
let sitesEntity = []
sitesData.forEach((item) => {
    let entity = viewer.entities.add({
      id: item.id,
      position: Cesium.Cartesian3.fromDegrees(item.lon, item.lat, height),
      label: {},
      monitoItems: {
        ...item
      }
    });
    sitesEntity.push(entity);
});
drawKriging(sitesEntity, viewer, values, colors, "KrigingRain");

```

5、drawKriging方法
---------------

```js
function getCanvas(siteValue, lngs, lats, ex, minx, maxy, maxx, miny, colors) {
  
  let variogram = kriging.train(siteValue, lngs, lats, "exponential", 0, 100);
  
  let grid = kriging.grid(ex, variogram, (maxy - miny) / 500);
  let canvas = document.createElement("canvas");
  canvas.width = 1000;
  canvas.height = 1000;
  canvas.style.display = "block";
  canvas.getContext("2d").globalAlpha = 1; 
  
  kriging.plot(canvas, grid, [minx, maxx], [miny, maxy], colors);
  return canvas;
}
export function drawKriging(sites, viewer, values, colors, layerName) {
  
  let KrigingRain = viewer.entities.getById(layerName);
  viewer.entities.remove(KrigingRain);
  let lngs = []; 
  let lats = []; 
  let siteValue = []; 
  let coords = []; 
  let ex = []; 
  ex = json.features[0].geometry.coordinates[0];
  for (let i = 0; i < sites.length; i++) {
    
    let ellipsoid = viewer.scene.globe.ellipsoid;
    let cartesian3 = new Cesium.Cartesian3(
      sites[i]._position._value.x,
      sites[i]._position._value.y,
      sites[i]._position._value.z
    );
    let cartographic = ellipsoid.cartesianToCartographic(cartesian3);
    let lat = Cesium.Math.toDegrees(cartographic.latitude);
    let lng = Cesium.Math.toDegrees(cartographic.longitude);
    siteValue.push(values[i]);
    lngs.push(lng);
    lats.push(lat);
  }
  let coordinates = json.features[0].geometry.coordinates[0][0];
  for (let deg = 0; deg < coordinates.length; deg++) {
    coords.push(coordinates[deg][0]);
    coords.push(coordinates[deg][1]);
  }
  if (siteValue.length > 2) {
    new Cesium.PolygonGeometry({
      polygonHierarchy: new Cesium.PolygonHierarchy(Cesium.Cartesian3.fromDegreesArray(coords))
    }); 
    let extent = Cesium.PolygonGeometry.computeRectangle({
      polygonHierarchy: new Cesium.PolygonHierarchy(Cesium.Cartesian3.fromDegreesArray(coords))
    }); 
    let minx = Cesium.Math.toDegrees(extent.west); 
    let miny = Cesium.Math.toDegrees(extent.south);
    let maxx = Cesium.Math.toDegrees(extent.east);
    let maxy = Cesium.Math.toDegrees(extent.north);
    let canvas = null; 
    canvas = getCanvas(siteValue, lngs, lats, ex, minx, maxy, maxx, miny, colors);
    if (canvas !== null) {
      viewer.entities.add({
        id: layerName,
        polygon: {
          clampToGround: true,
          height: 0,
          hierarchy: {
            positions: Cesium.Cartesian3.fromDegreesArray(coords)
          },
          material: new Cesium.ImageMaterialProperty({
            image: canvas 
          })
        }
      });
    }
  }
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1aa04f822cd344fdb03eff7331e6b8eb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

通过克里金我们实现了降雨量图，我们还能实现气温图、湿度图、气压图等等，也了解了克里金函数的作用，以及和Cesium如何相结合使用。实际开发中，大家可以结合具体的业务需求开发自己的功能。