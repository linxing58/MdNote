# geotools中等值面的生成与OL3中的展示_geotools插值_牛老师讲GIS的博客-CSDN博客
**概述：** 

本文讲述如何在geotools中IDW插值生成等值面，并根据给定shp进行裁剪，并生成geojson数据，以及Openlayers3中展示。

**效果：** 

![](https://img-blog.csdn.net/20170829205423960?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvR0lTU2hpWGlTaGVuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

插值数据

![](https://img-blog.csdn.net/20170829205500192?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvR0lTU2hpWGlTaGVuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

裁剪结果

![](https://img-blog.csdn.net/20170829205610197?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvR0lTU2hpWGlTaGVuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

裁剪区域数据

**实现代码：** 

1、geotools

```null
package com.lzugis.geotools;import com.amazonaws.util.json.JSONObject;import com.lzugis.CommonMethod;import com.lzugis.geotools.utils.FeaureUtil;import com.lzugis.geotools.utils.GeoJSONUtil;import com.vividsolutions.jts.geom.Geometry;import org.geotools.data.FeatureSource;import org.geotools.data.shapefile.ShapefileDataStore;import org.geotools.data.simple.SimpleFeatureCollection;import org.geotools.data.simple.SimpleFeatureSource;import org.geotools.feature.FeatureCollection;import org.geotools.feature.FeatureIterator;import org.geotools.geojson.feature.FeatureJSON;import org.opengis.feature.Feature;import org.opengis.feature.simple.SimpleFeature;import wContour.Global.Border;import wContour.Global.PointD;import wContour.Global.PolyLine;import wContour.Global.Polygon;import wContour.Interpolate;import java.io.StringWriter;import java.nio.charset.Charset;public class EquiSurface {public String calEquiSurface(double[][] trainData,String geojsonpogylon = "";double _undefData = -9999.0;SimpleFeatureCollection polygonCollection = null;            List<PolyLine> cPolylineList = new ArrayList<PolyLine>();            List<Polygon> cPolygonList = new ArrayList<Polygon>();double[] _X = new double[width];double[] _Y = new double[height];File file = new File(boundryFile);ShapefileDataStore shpDataStore = null;            shpDataStore = new ShapefileDataStore(file.toURL());Charset charset = Charset.forName("GBK");            shpDataStore.setCharset(charset);String typeName = shpDataStore.getTypeNames()[0];SimpleFeatureSource featureSource = null;            featureSource = shpDataStore.getFeatureSource(typeName);SimpleFeatureCollection fc = featureSource.getFeatures();double minX = fc.getBounds().getMinX();double minY = fc.getBounds().getMinY();double maxX = fc.getBounds().getMaxX();double maxY = fc.getBounds().getMaxY();            Interpolate.CreateGridXY_Num(minX, minY, maxX, maxY, _X, _Y);double[][] _gridData = new double[width][height];int nc = dataInterval.length;            _gridData = Interpolate.Interpolation_IDW_Neighbor(trainData,int[][] S1 = new int[_gridData.length][_gridData[0].length];            List<Border> _borders = Contour.tracingBorders(_gridData, _X, _Y,            cPolylineList = Contour.tracingContourLines(_gridData, _X, _Y, nc,                    dataInterval, _undefData, _borders, S1);            cPolylineList = Contour.smoothLines(cPolylineList);            cPolygonList = Contour.tracingPolygons(_gridData, cPolylineList,            geojsonpogylon = getPolygonGeoJson(cPolygonList);                polygonCollection = GeoJSONUtil.readGeoJsonByString(geojsonpogylon);FeatureSource dc = clipFeatureCollection(fc, polygonCollection);                geojsonpogylon = getPolygonGeoJson(dc.getFeatures());private FeatureSource clipFeatureCollection(FeatureCollection fc,                                                SimpleFeatureCollection gs) {            List<Map<String, Object>> values = new ArrayList<Map<String, Object>>();FeatureIterator contourFeatureIterator = gs.features();FeatureIterator dataFeatureIterator = fc.features();while (dataFeatureIterator.hasNext()) {Feature dataFeature = dataFeatureIterator.next();Geometry dataGeometry = (Geometry) dataFeature.getProperty(while (contourFeatureIterator.hasNext()) {Feature contourFeature = contourFeatureIterator.next();Geometry contourGeometry = (Geometry) contourFeature                            .getProperty("geometry").getValue();double lv = (Double) contourFeature.getProperty("lvalue")double hv = (Double) contourFeature.getProperty("hvalue")if (dataGeometry.intersects(contourGeometry)) {Geometry geo = dataGeometry                                .intersection(contourGeometry);                        Map<String, Object> map = new HashMap<String, Object>();                        map.put("the_geom", geo);            contourFeatureIterator.close();            dataFeatureIterator.close();SimpleFeatureCollection sc = FeaureUtil                    .creatSimpleFeatureByFeilds("crs:4326,the_geom:MultiPolygon,lvalue:double,hvalue:double",            cs = FeaureUtil.creatFeatureSourceByCollection(sc);private String getPolygonGeoJson(FeatureCollection fc) {FeatureJSON fjson = new FeatureJSON();StringBuffer sb = new StringBuffer();            sb.append("{\"type\": \"FeatureCollection\",\"features\": ");FeatureIterator itertor = fc.features();            List<String> list = new ArrayList<String>();while (itertor.hasNext()) {SimpleFeature feature = (SimpleFeature) itertor.next();StringWriter writer = new StringWriter();                fjson.writeFeature(feature, writer);                list.add(writer.toString());            sb.append(list.toString());private String getPolygonGeoJson(List<Polygon> cPolygonList) {String geometry = " { \"type\":\"Feature\",\"geometry\":";String properties = ",\"properties\":{ \"hvalue\":";String head = "{\"type\": \"FeatureCollection\"," + "\"features\": [";if (cPolygonList == null || cPolygonList.size() == 0) {for (Polygon pPolygon : cPolygonList) {                List<Object> ptsTotal = new ArrayList<Object>();                List<Object> pts = new ArrayList<Object>();PolyLine pline = pPolygon.OutLine;for (PointD ptD : pline.PointList) {                    List<Double> pt = new ArrayList<Double>();if (pPolygon.HasHoles()) {for (PolyLine cptLine : pPolygon.HoleLines) {                        List<Object> cpts = new ArrayList<Object>();for (PointD ccptD : cptLine.PointList) {                            List<Double> pt = new ArrayList<Double>();JSONObject js = new JSONObject();                js.put("type", "Polygon");                js.put("coordinates", ptsTotal);double hv = pPolygon.HighValue;double lv = pPolygon.LowValue;if (pPolygon.IsClockWise) {if (!pPolygon.IsHighCenter) {if (!pPolygon.IsHighCenter) {if (!pPolygon.IsClockWise) {if (pPolygon.IsHighCenter) {                geo = geometry + js.toString() + properties + hv                        + ", \"lvalue\":" + lv + "} }" + "," + geo;                geo = geo.substring(0, geo.lastIndexOf(","));public static void main(String[] args) {long start = System.currentTimeMillis();EquiSurface equiSurface = new EquiSurface();CommonMethod cm = new CommonMethod();double[] bounds = {73.4510046356223, 18.1632471876417,134.976797646506, 53.5319431522236};double[][] trainData = new double[3][100];for (int i = 0; i < 100; i++) {double x = bounds[0] + new Random().nextDouble() * (bounds[2] - bounds[0]),                    y = bounds[1] + new Random().nextDouble() * (bounds[3] - bounds[1]),                    v = 0 + new Random().nextDouble() * (45 - 0);double[] dataInterval = new double[]{20, 25, 30, 35, 40, 45};String boundryFile = "D:\\data\\国家基础地理信息系统SHP文件\\国界\\国界\\bou1_4p.shp";int[] size = new int[]{100, 100};String strJson = equiSurface.calEquiSurface(trainData, dataInterval, size, boundryFile, isclip);String strFile = "d:/china1.json";            cm.append2File(strFile, strJson);            System.out.println(strFile + "差值成功, 共耗时" + (System.currentTimeMillis() - start) + "ms");
```

2、openlayers3

```null
<html xmlns="http://www.w3.org/1999/xhtml"><meta http-equiv="Content-Type" content="text/html; charset=utf-8" /><link rel="stylesheet" type="text/css" href="../../../plugin/ol3/css/ol.css"/>box-shadow: 1px 1px 1px #C0C0C0;<script type="text/javascript" src="../../../plugin/ol3/build/ol-debug.js"></script><script type="text/javascript" src="../../../plugin/jquery/jquery-1.8.3.js"></script><script type="text/javascript">var bounds = [73.4510046356223, 18.1632471876417,134.976797646506, 53.5319431522236];var projection = new ol.proj.Projection({			$.get("data/china1.json",null,function(result){var features = (new ol.format.GeoJSON()).readFeatures(result);var vectorSource = new ol.source.Vector();				vectorSource.addFeatures(features);var vector = new ol.layer.Vector({style:function(feature, resolution) {var lvalue = feature.get("lvalue"), color = "0,0,0,0";							color = "245,200,200,255";else if(lvalue>=20&&lvalue<30){							color = "245,183,48,255";else if(lvalue>=25&&lvalue<30){else if(lvalue>=30&&lvalue<35){return new ol.style.Style({stroke: new ol.style.Stroke({fill: new ol.style.Fill({color: "rgba("+color+")",var tiled = new ol.layer.Tile({source: new ol.source.TileWMS({url: 'http://192.168.10.185:8086/geoserver/lzugis/wms',params: {'FORMAT': 'image/png',LAYERS: 'lzugis:capital',controls: ol.control.defaults({				map.getView().fit(bounds, map.getSize());
```

  
```


\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

技术博客

CSDN：http://blog.csdn[.NET](http://lib.csdn.net/base/dotnet ".NET知识库")/gisshixisheng

博客园：http://www.cnblogs.com/lzugis/

在线教程

http://edu.csdn[.Net](http://lib.csdn.net/base/dotnet ".NET知识库")/course/detail/799

Github

https://github.com/lzugis/

联系方式

q       q:1004740957

e-mail:niujp08@qq.com

公众号:lzugis15

Q Q 群:452117357(webgis)

             337469080([Android](http://lib.csdn.net/base/15 "Android知识库"))

 

![](https://img-blog.csdn.net/20161104075032426?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)



```