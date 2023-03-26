# cesium + kriging.js动态生成克里金图_耀耀是个憨憨吖～的博客-CSDN博客
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

kriging.js [GIThub](https://so.csdn.net/so/search?q=GIThub&spm=1001.2101.3001.7020)地址 :: [https://github.com/oeo4b/kriging.js/blob/master/kriging.js](https://www.csdn.net/).

js文件下载下来直接引用就可以

直接贴代码

HTML半部分:

```javascript

<script src="js/kriging.js" type="text/javascript"></script>

<canvas id="canvasMap" style="display:none;"></canvas>

```

js部分:

```javascript
function loadkriging() {
	
    var canvas = document.getElementById("canvasMap");
    canvas.width = 2000;
    canvas.height = 2000;
    var n = stationData.length;
    var t = [];
    var x = [];
    var y = [];
    for (var i = 0; i < n; i++) {
        if (aqTypeCurrent == 'aqi'){
            t.push(stationData[i].aqi); 
        }
        
        x.push(stationData[i].lon);
        y.push(stationData[i].lat); 
    }

    var variogram = kriging.train(t, x, y, "exponential", 0, 100);
    var grid = kriging.grid(data_scoure, variogram, 0.0008);

	
    var colors = [
        '#68b0dc',
        '#85bcd2',
        '#a9c9c6',
        '#bdd5bd',
        '#d4ddb5',
        '#e4eaa9',
        '#fafb97',
        '#f4e58d',
        '#f8cb72',
        '#f1b079',
        '#fc9b5e',
        '#f5875e',
        '#ef604f',
    ];
    kriging.plot(canvas, grid, [114.03797, 114.89431], [36.36971, 36.90616], colors);
    
}

function returnImgae() {
    var mycanvas = document.getElementById("canvasMap");
    return mycanvas.toDataURL("image/png");
}

loadkriging();

viewer.imageryLayers.addImageryProvider(
	new Cesium.SingleTileImageryProvider({
      url:returnImgae(),
      rectangle: new Cesium.Rectangle(
          Cesium.Math.toRadians(114.03797),
          Cesium.Math.toRadians(36.36971),
          Cesium.Math.toRadians(114.89431),
          Cesium.Math.toRadians(36.90616)
      )
  }));




```

效果: ![](https://img-blog.csdnimg.cn/20210602102015350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lhdG9nYW1pVG9oa2E=,size_16,color_FFFFFF,t_70)