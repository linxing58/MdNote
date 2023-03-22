# Cesium结合kriging.js制作降雨等值面_北外滩的博客-CSDN博客
因工作需要使用cesium制作降雨等值面，所以在参考众多博客后。终于是成功实现了降雨等值面。  
参考博客：  
[https://blog.csdn.net/weixin_44265800/article/details/103106321](https://blog.csdn.net/weixin_44265800/article/details/103106321)  
[https://www.jianshu.com/p/c8473ac0b08a](https://www.jianshu.com/p/c8473ac0b08a)  
[https://zhuanlan.zhihu.com/p/73769138](https://zhuanlan.zhihu.com/p/73769138)  
[https://blog.csdn.net/jessezappy/article/details/108891162](https://blog.csdn.net/jessezappy/article/details/108891162)

实现效果图
-----

![](https://img-blog.csdnimg.cn/49162c85cf884595ad35db52a33c7b9c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Zi_5Zi_MDEwNA==,size_20,color_FFFFFF,t_70,g_se,x_16)

使用克里金插值
-------

1.kriging.js  
克里金法有开源的克里金插值算法实现，可以将kriging.js直接导入项目进行使用。  
克里金项目的GitHub地址：[https://github.com/oeo4b/kriging.js](https://github.com/oeo4b/kriging.js)  
2.使用kriging.js  
相关代码如下：

```javascript
import kriging from "@/utils/kriging.js";
colors: [
        { min: 0, max: 5, color: "#A9F08E" }, 
        { min: 5, max: 10, color: "#72D66B" }, 
        { min: 10, max: 25, color: "#3DB83D" }, 
        { min: 25, max: 50, color: "#61B7FC" }, 
        { min: 50, max: 100, color: "#0001FE" }, 
        { min: 100, max: 250, color: "#FD00FA" }, 
        { min: 250, max: 1000, color: "#7F013E" }, 
],
map.drawKriging = function(viewer,values,colors){
    
    var KrigingRain = viewer.entities.getById('KrigingRain');
    viewer.entities.remove(KrigingRain);
    var lngs = [];
    var lats = [];
    var siteValue = [];
    var coords = [];
    var ex = [];
    ex = [LYBJJSON.features[0].geometry.coordinates[0]];
    for(var i=0; i < sites.length; i++){
      if(sites[i].type == "1"){
        for(var j=0; j < values.length; j++){
          if(sites[i].id == values[j].code && sites[i].state == "normal"){
            sites[i].label.text = values[j].value+"";
            var ellipsoid=viewer.scene.globe.ellipsoid;
            var cartesian3=new Cesium.Cartesian3(sites[i]._position._value.x,sites[i]._position._value.y,sites[i]._position._value.z);
            var cartographic=ellipsoid.cartesianToCartographic(cartesian3);
            var lat=Cesium.Math.toDegrees(cartographic.latitude);
            var lng=Cesium.Math.toDegrees(cartographic.longitude);
            
            siteValue.push(values[j].value);
            lngs.push(lng);
            lats.push(lat);
          }
        }
      }
    }
    for(let item of LYBJJSON.features[0].geometry.coordinates[0]){
      coords.push(...item)
    }
    if (siteValue.length > 2) {
        const polygon = new Cesium.PolygonGeometry ({
            polygonHierarchy: new Cesium.PolygonHierarchy (
                Cesium.Cartesian3.fromDegreesArray ( coords )
            )
        });
        let extent = Cesium.PolygonGeometry.computeRectangle ({
            polygonHierarchy: new Cesium.PolygonHierarchy (
                Cesium.Cartesian3.fromDegreesArray ( coords )
            )   
        });
        let minx = Cesium.Math.toDegrees ( extent.west );
        let miny = Cesium.Math.toDegrees ( extent.south );
        let maxx = Cesium.Math.toDegrees ( extent.east );
        let maxy = Cesium.Math.toDegrees ( extent.north );
        let canvas = null;
        function getCanvas() {
            
            let variogram = kriging.train ( siteValue, lngs, lats, 'exponential', 0, 100 );
            
            let grid = kriging.grid ( ex, variogram, (maxy - miny) / 500 );
            canvas = document.createElement ( 'canvas' );
            canvas.width = 1000;
            canvas.height = 1000;
            canvas.style.display = 'block';
            canvas.getContext ( '2d' ).globalAlpha = 1;
            
            kriging.plot ( canvas, grid, [minx, maxx], [miny, maxy], colors );
        }
        getCanvas ();
        if (canvas != null) {
          KrigingObj = viewer.entities.add ({
            id: "KrigingRain",
            polygon: {
              show: ShowName == "drawKriging"?true:false,
              clampToGround: true,
              hierarchy: {
                  positions: Cesium.Cartesian3.fromDegreesArray ( coords )
              },
              material: new Cesium.ImageMaterialProperty ({
                  image: canvas
              })
            }
        });
      }
    }
};

```

3.计算使用数据

```javascript
var lngs = [];
var lats = [];
var siteValue = [];
var coords = [];
var ex = [];

```

4.更改kriging.js的plot方法

```
`kriging.plot = function(canvas, grid, xlim, ylim, colors) {
    
    let ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    let range = [xlim[1]-xlim[0], ylim[1]-ylim[0], grid.zlim[1]-grid.zlim[0]];
    let i, j, x, y, z;
    let n = grid.length;
    let m = grid[0].length;
    let wx = Math.ceil(grid.width*canvas.width/(xlim[1]-xlim[0]));
    let wy = Math.ceil(grid.width*canvas.height/(ylim[1]-ylim[0]));
    for(i=0;i<n;i++){
        for(j=0;j<m;j++) {
            if(grid[i][j]==undefined) continue;
            x = canvas.width*(i*grid.width+grid.xlim[0]-xlim[0])/range[0];
            y = canvas.height*(1-(j*grid.width+grid.ylim[0]-ylim[0])/range[1]);
            z = (grid[i][j]-grid.zlim[0])/range[2];
            if(z<0.0) z = 0.0;
            if(z>1.0) z = 1.0;
            ctx.fillStyle = kriging.getColor(colors,grid[i][j]);
            ctx.fillRect(Math.round(x-wx/2), Math.round(y-wy/2), wx, wy);
        }
    }
};

kriging.getColor = function (colors, z) {
    var l = colors.length;
    for (var i = 0; i < l; i++) {
        if (z >= colors[i].min && z < colors[i].max) return colors[i].color;
    }
    if (z < 0) {
    	return colors[0].color;
    }
};` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34


```

kriging.js使用方法解析
----------------

**kriging.train ( siteValue, lngs, lats, ‘exponential’, 0, 100 );**  
使用gaussian、exponential或spherical模型对数据集进行训练，返回的是一个variogram对象。  
0对应高斯过程的方差参数，一般设置为 0。  
100对应方差函数的先验值，默认设置为100。

**kriging.grid ( ex, variogram, (maxy - miny) / 500 );**  
使用train返回的variogram对象使ex描述的地理位置内的格网元素具备不一样的预测值。  
(maxy - miny) / 500的500是返回的网格数量,越大越细处理越慢。

**kriging.plot ( canvas, grid, \[minx, maxx\], \[miny, maxy\], colors );**  
将得到的格网grid渲染至canvas上。  
这里我们更改了plot方法的色彩赋值，更改后就可以根据数据范围赋予自定义的色彩了。