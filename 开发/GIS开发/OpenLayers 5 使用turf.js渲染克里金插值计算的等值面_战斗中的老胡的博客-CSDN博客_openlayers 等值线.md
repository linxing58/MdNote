# OpenLayers 5 使用turf.js渲染克里金插值计算的等值面_战斗中的老胡的博客-CSDN博客_openlayers 等值线
之前[Trojx](http://www.trojx.me/2018/10/19/openlayers-kriging/)同学实现了一个[克里金插值](https://so.csdn.net/so/search?q=%E5%85%8B%E9%87%8C%E9%87%91%E6%8F%92%E5%80%BC&spm=1001.2101.3001.7020)渲染等值面的例子，使用的是[kriging.js](https://github.com/oeo4b/kriging.js)内置的绘图函数，然后在 openlayer 中利用 ImageCanvas 渲染。

这个方法比较简便，但是有一些性能上的问题：

-   网格切分比较小的时候，总体的网格数增加，每次重新渲染都要将整个网格数组遍历一次，交互体验不是很好；
-   网格是以方块的形式呈现的，视觉效果不如曲线和多边形。

我使用了 turf.js 来处理网格数据，生成等值面，提升了交互性能，效果也更好看一点：

![](https://img-blog.csdnimg.cn/20191014020422466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L251ZHRjYWRldA==,size_16,color_FFFFFF,t_70)

下面是源码：

````null
<link rel="stylesheet" href="./include/ol.css" type="text/css" /><script src="./include/ol.js"></script><script src="./include/kriging.js"></script><script src='./include/turf.js'></script><div id="map" class="map"></div>mapCenter: [121.483101, 31.227036],krigingModel: 'spherical',colors: ["#006837", "#1a9850", "#66bd63", "#a6d96a", "#d9ef8b", "#ffffbf","#fee08b", "#fdae61", "#f46d43", "#d73027", "#a50026"],let baseLayer = new ol.layer.Tile({source: new ol.source.OSM()center: params.mapCenter,let WFSVectorSource = new ol.source.Vector();let WFSVectorLayer = new ol.layer.Vector(        map.addLayer(WFSVectorLayer);let select = new ol.interaction.Select();        map.addInteraction(select);let dragBox = new ol.interaction.DragBox({condition: ol.events.condition.platformModifierKeyOnly        map.addInteraction(dragBox);for (let i = 0; i < 15; i++) {let feature = new ol.Feature({geometry: new ol.geom.Point(                        params.mapCenter[0] + Math.random() * 0.01 - .005,                        params.mapCenter[1] + Math.random() * 0.01 - .005value: Math.round(Math.random() * params.maxValue)            feature.setStyle(new ol.style.Style({image: new ol.style.Circle({fill: new ol.style.Fill({ color: "#00F" })WFSVectorSource.addFeature(feature);let selectedFeatures = select.getFeatures();        dragBox.on('boxend', () => {let extent = dragBox.getGeometry().getExtent();WFSVectorSource.forEachFeatureIntersectingExtent(extent, (feature) => {                selectedFeatures.push(feature);        dragBox.on('boxstart', () => {            selectedFeatures.clear();const gridFeatureCollection = function (grid, xlim, ylim) {var range =grid.zlim[1] - grid.zlim[0];for (j = 0; j < m ; j++) {                    x = (i) * grid.width + grid.xlim[0];                    y = (j) * grid.width + grid.ylim[0];                    z = (grid[i][j] - grid.zlim[0]) / range;                    pointArray.push(turf.point([x, y], { value: z }));const drawKriging = (extent) => {let values = [], lngs = [], lats = [];            selectedFeatures.forEach(feature => {                values.push(feature.values_.value);                lngs.push(feature.values_.geometry.flatCoordinates[0]);                lats.push(feature.values_.geometry.flatCoordinates[1]);let variogram = kriging.train(                    [extent[0], extent[1]], [extent[0], extent[3]],                    [extent[2], extent[3]], [extent[2], extent[1]]let grid = kriging.grid(polygons, variogram, (extent[2] - extent[0]) / 500);let dragboxExtent = extent;if (vectorLayer !== null) {                    map.removeLayer(vectorLayer);var vectorSource = new ol.source.Vector();                vectorLayer = new ol.layer.Vector(style: function (feature) {var style = new ol.style.Style({fill: new ol.style.Fill({color: params.colors[parseFloat(feature.get('value').split('-')[1]) * 10]let fc = gridFeatureCollection(grid,                    [extent[0], extent[2]], [extent[1], extent[3]]);var collection = turf.featureCollection(fc);var breaks = [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0];var isobands = turf.isobands(collection, breaks, { zProperty: 'value' });return turf.area(b)-turf.area(a);                isobands.features.sort(sortArea)var polyFeatures = new ol.format.GeoJSON().readFeatures(isobands, {featureProjection: 'EPSG:4326'                vectorSource.addFeatures(polyFeatures);                map.addLayer(vectorLayer);let extent = [params.mapCenter[0] - .005, params.mapCenter[1] - .005, params.mapCenter[0] + .005, params.mapCenter[1] + .005];WFSVectorSource.forEachFeatureIntersectingExtent(extent, (feature) => {            selectedFeatures.push(feature);```

再次对

[Trojx](https://www.jianshu.com/u/6b5ce1b9566c)

表示敬意，感谢他之前创造性的工作。 
 [https://blog.csdn.net/nudtcadet/article/details/102540613](https://blog.csdn.net/nudtcadet/article/details/102540613)
````
