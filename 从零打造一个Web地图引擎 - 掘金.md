# 从零打造一个Web地图引擎 - 掘金
说到地图，大家一定很熟悉，平时应该都使用过百度地图、高德地图、腾讯地图等，如果涉及到地图相关的开发需求，也有很多选择，比如前面的几个地图都会提供一套`js API`，此外也有一些开源地图框架可以使用，比如`OpenLayers`、`Leaflet`等。

那么大家有没有想过这些地图是怎么渲染出来的呢，为什么根据一个经纬度就能显示对应的地图呢，不知道没关系，本文会带各位从零实现一个简单的地图引擎，来帮助大家了解`GIS`基础知识及`Web`地图的实现原理。

首先我们去高德地图上选个经纬度，作为我们后期的地图中心点，打开[高德坐标拾取](https://link.juejin.cn/?target=https%3A%2F%2Flbs.amap.com%2Ftools%2Fpicker "https&#x3A;//lbs.amap.com/tools/picker")工具，随便选择一个点：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/241915e1b4454cec967c01eec9536626~tplv-k3u1fbpfcp-watermark.awebp?)

笔者选择了杭州的雷峰塔，经纬度为：`[120.148732,30.231006]`。

地图瓦片我们使用高德的在线瓦片，地址如下：

    https://webrd0{1-4}.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scale=1&style=8
    复制代码

目前各大地图厂商的瓦片服务遵循的规则是有不同的：

> 谷歌 XYZ 规范：谷歌地图、OpenStreetMap、高德地图、geoq、天地图，坐标原点在左上角
>
> TMS 规范：腾讯地图，坐标原点在左下角
>
> WMTS 规范：原点在左上角，瓦片不是正方形，而是矩形，这个应该是官方标准
>
> 百度地图比较特立独行，投影、分辨率、坐标系都跟其他厂商不一样，原点在经纬度都为 0 的位置，也就是中间，向右为 X 正方向，向上为 Y 正方向

谷歌和`TMS`的瓦片行列号区别如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cba011ccc26459fae002e4e554cea15~tplv-k3u1fbpfcp-watermark.awebp?)

虽然规范不同，但原理基本是一致的，都是把地球投影成一个巨大的正方形世界平面图，然后按照四叉树进行分层切割，比如第一层，只有一张瓦片，显示整个世界的信息，所以基本只能看到洲和海的名称和边界线，第二层，切割成四张瓦片，显示信息稍微多了一点，以此类推，就像一个金字塔一样，底层分辨率最高，显示的细节最多，瓦片数也最多，顶层分辨率最低，显示的信息很少，瓦片数量相对也最少：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf82643211c148efbc4a8009f8f61158~tplv-k3u1fbpfcp-watermark.awebp?)

每一层的瓦片数量计算公式：

    Math.pow(Math.pow(2, n), 2)// 行*列：2^n * 2^n
    复制代码

十八层就需要`68719476736`张瓦片，所以一套地图瓦片整体数量是非常庞大的。

瓦片切好以后，通过行列号和缩放层级来保存，所以可以看到瓦片地址中有三个变量：`x`、`y`、`z`

    x：行号
    y：列号
    z：分辨率，一般为0-18
    复制代码

通过这三个变量就可以定位到一张瓦片，比如下面这个地址，行号为`109280`，列号为`53979`，缩放层级为`17`：

    https://webrd01.is.autonavi.com/appmaptile?x=109280&y=53979&z=17&lang=zh_cn&size=1&scale=1&style=8
    复制代码

对应的瓦片为：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46b5ff5847ad4e05b701e31daf6a5546~tplv-k3u1fbpfcp-watermark.awebp)

高德地图使用的是`GCJ-02 坐标系`，也称火星坐标系，由中国国家测绘局在 02 年发布，是在 GPS 坐标（`WGS-84`坐标系）基础上经加密后而来，也就是增加了非线性的偏移，让你摸不准真实位置，为了国家安全，国内地图服务商都需要使用`GCJ-02 坐标系`。

`WGS-84`坐标系是国际通用的标准，`EPSG`编号为`EPSG:4326`，通常 GPS 设备获取到的原始经纬度和国外的地图厂商使用的都是`WGS-84`坐标系。

这两种坐标系都是地理坐标系，球面坐标，单位为`度`，这种坐标方便在地球上定位，但是不方便展示和进行面积距离计算，我们印象中的地图都是平面的，所以就有了另外一种平面坐标系，平面坐标系是通过投影的方式从地理坐标系中转换过来，所以也称为投影坐标系，通常单位为`米`，投影坐标系根据投影方式的不同存在多种，在`Web`开发的场景里通常使用的是`Web 墨卡托投影`，编号为`EPSG:3857`，它基于`墨卡托投影`，把`WGS-84`坐标系投影成正方形，这是通过舍弃了南北`85.051129 纬度`以上的地区实现的，因为它是正方形，所以一个大的正方形可以很方便的被分割为更小的正方形。

上一节里我们简单介绍了一下坐标系，按照`Web`地图的标准，我们的地图引擎也选择支持`EPSG:3857`投影，但是我们通过高德工具获取到的是火星坐标系的经纬度坐标，所以第一步要把经纬度坐标转换为`Web 墨卡托`投影坐标，这里为了简单，先直接把火星坐标当做`WGS-84`坐标，后面再来看这个问题。

转换方法网上一搜就有：

```js

const angleToRad = (angle) => {
    return angle * (Math.PI / 180)
}


const radToAngle = (rad) => {
    return rad * (180 / Math.PI)
}


const EARTH_RAD = 6378137


const lngLat2Mercator = (lng, lat) => {
    
    let x = angleToRad(lng) * EARTH_RAD; 
    
    let rad = angleToRad(lat)
    
    let sin = Math.sin(rad)
    let y = EARTH_RAD / 2 * Math.log((1 + sin) / (1 - sin))
    return [x, y]
}


const mercatorTolnglat = (x, y) => {
    let lng = radToAngle(x) / EARTH_RAD
    let lat = radToAngle((2 * Math.atan(Math.exp(y / EARTH_RAD)) - (Math.PI / 2)))
    return [lng, lat]
}
```

`3857`坐标有了，它的单位是`米`，那么怎么转换成瓦片的行列号呢，这就涉及到`分辨率`的概念了，即地图上一像素代表实际多少米，分辨率如果能从地图厂商的文档里获取是最好的，如果找不到，也可以简单计算一下（如果使用计算出来的也不行，那就只能求助搜索引擎了），我们知道地球半径是`6378137`米，`3857`坐标系把地球当做正圆球体来处理，所以可以算出地球周长，投影是贴着地球赤道的：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08c3b179e8594829bc3682c0f5d1ae1f~tplv-k3u1fbpfcp-watermark.awebp?)

所以投影成正方形的世界平面图后的边长代表的就是地球的周长，前面我们也知道了每一层级的瓦片数量的计算方式，而一张瓦片的大小一般是`256*256`像素，所以用地球周长除以展开后的世界平面图的边长就知道了地图上每像素代表实际多少米：

```js

const EARTH_PERIMETER = 2 * Math.PI * EARTH_RAD

const TILE_SIZE = 256


const getResolution = (n) => {
    const tileNums = Math.pow(2, n)
    const tileTotalPx = tileNums * TILE_SIZE
    return EARTH_PERIMETER / tileTotalPx
}
```

地球周长算出来是`40075016.68557849`，可以看到`OpenLayers`就是这么计算的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ceb3a5b09d644ff0a03188ecff1a5eca~tplv-k3u1fbpfcp-watermark.awebp?)

`3857`坐标的单位是`米`，那么把坐标除以分辨率就可以得到对应的像素坐标，再除以`256`，就可以得到瓦片的行列号：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4b4032c30114b9ea9fe0a9bba2122f8~tplv-k3u1fbpfcp-watermark.awebp?)

函数如下：

```js

const getTileRowAndCol = (x, y, z) => {
    let resolution = getResolution(z)
    let row = Math.floor(x / resolution / TILE_SIZE)
    let col = Math.floor(y / resolution / TILE_SIZE)
    return [row, col]
}
```

接下来我们把层级固定为`17`，那么分辨率`resolution`就是`1.194328566955879`，雷峰塔的经纬度转成`3857`的坐标为：`[13374895.665697495, 3533278.205310311]`，使用上面的函数计算出来行列号为：`[43744, 11556]`，我们把这几个数据代入瓦片的地址里进行访问：

    https://webrd01.is.autonavi.com/appmaptile?x=43744&y=11556&z=17&lang=zh_cn&size=1&scale=1&style=8
    复制代码

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/125aec7dce294fbc98c4d2c4f96ef7cf~tplv-k3u1fbpfcp-watermark.awebp?)

一片空白，这是为啥呢，其实是因为原点不一样，`4326`和`3857`坐标系的原点在赤道和本初子午线相交点，非洲边上的海里，而瓦片的原点在左上角：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3431c302f67c49c2af620f83ac1c4032~tplv-k3u1fbpfcp-watermark.awebp?)

再来看下图会更容易理解：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36cc5f47d58343949d011b6ca541d8a6~tplv-k3u1fbpfcp-watermark.awebp?)

`3857`坐标系的原点相当于在世界平面图的中间，向右为`x`轴正方向，向上为`y`轴正方向，而瓦片地图的原点在左上角，所以我们需要根据图上【绿色虚线】的距离计算出【橙色实线】的距离，这也很简单，水平坐标就是水平绿色虚线的长度加上世界平面图的一半，垂直坐标就是世界平面图的一半减去垂直绿色虚线的长度，世界平面图的一半也就是地球周长的一半，修改`getTileRowAndCol`函数：

```js
const getTileRowAndCol = (x, y, z) => {
  x += EARTH_PERIMETER / 2     
  y = EARTH_PERIMETER / 2 - y  
  let resolution = getResolution(z)
  let row = Math.floor(x / resolution / TILE_SIZE)
  let col = Math.floor(y / resolution / TILE_SIZE)
  return [row, col]
}
```

这次计算出来的瓦片行列号为`[109280, 53979]`，代入瓦片地址：

    https://webrd01.is.autonavi.com/appmaptile?x=109280&y=53979&z=17&lang=zh_cn&size=1&scale=1&style=8
    复制代码

结果如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb031151883146458f775f00a29b99b5~tplv-k3u1fbpfcp-watermark.awebp?)

可以看到雷峰塔出来了。

我们现在能根据一个经纬度找到对应的瓦片，但是这还不够，我们的目标是要能在浏览器上显示出来，这就需要解决两个问题，一个是加载多少块瓦片，二是计算每一块瓦片的显示位置。

渲染瓦片我们使用`canvas`画布，模板如下：

```html
<template>
  <div class="map" ref="map">
    <canvas ref="canvas"></canvas>
  </div>
</template>
```

地图画布容器`map`的大小我们很容易获取：

```js

let { width, height } = this.$refs.map.getBoundingClientRect()
this.width = width
this.height = height

let canvas = this.$refs.canvas
canvas.width = width
canvas.height = height

this.ctx = canvas.getContext('2d')
```

地图中心点我们设在画布中间，另外中心点的经纬度`center`和缩放层级`zoom`因为都是我们自己设定的，所以也是已知的，那么我们可以计算出中心坐标对应的瓦片：

```js

let centerTile = getTileRowAndCol(
    ...lngLat2Mercator(...this.center),
    this.zoom
)
```

缩放层级还是设为`17`，中心点还是使用雷峰塔的经纬度，那么对应的瓦片行列号前面我们已经计算过了，为`[109280, 53979]`。

中心坐标对应的瓦片行列号知道了，那么该瓦片左上角在世界平面图中的像素位置我们也就知道了：

```js

let centerTilePos = [centerTile[0] * TILE_SIZE, centerTile[1] * TILE_SIZE]
```

计算出来为`[27975680, 13818624]`。这个坐标怎么转换到屏幕上呢，请看下图：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e115185348d443d99a081a3bd65140a~tplv-k3u1fbpfcp-watermark.awebp?)

中心经纬度的瓦片我们计算出来了，瓦片左上角的像素坐标也知道了，然后我们再计算出中心经纬度本身对应的像素坐标，那么和瓦片左上角的差值就可以计算出来，最后我们把画布的原点移动到画布中间（画布默认原点为左上角，x 轴正方向向右，y 轴正方向向下），也就是把中心经纬度作为坐标原点，那么中心瓦片的显示位置就是这个差值。

补充一下将经纬度转换成像素的方法：

```js

const getPxFromLngLat = (lng, lat, z) => {
  let [_x, _y] = lngLat2Mercator(lng, lat)
  
  _x += EARTH_PERIMETER / 2
  _y = EARTH_PERIMETER / 2 - _y
  let resolution = resolutions[z]
  
  let x = Math.floor(_x / resolution)
  let y = Math.floor(_y / resolution)
  return [x, y]
}
```

计算中心经纬度对应的像素坐标：

```js

let centerPos = getPxFromLngLat(...this.center, this.zoom)
```

计算差值：

```js

let offset = [
    centerPos[0] - centerTilePos[0],
    centerPos[1] - centerTilePos[1]
]
```

最后通过`canvas`来把中心瓦片渲染出来：

```js

this.ctx.translate(this.width / 2, this.height / 2)

let img = new Image()

img.src = getTileUrl(...centerTile, this.zoom)
img.onload = () => {
    
    this.ctx.drawImage(img, -offset[0], -offset[1])
}
```

这里先来看看`getTileUrl`方法的实现：

```js

const getTileUrl = (x, y, z) => {
  let domainIndexList = [1, 2, 3, 4]
  let domainIndex =
    domainIndexList[Math.floor(Math.random() * domainIndexList.length)]
  return `https://webrd0${domainIndex}.is.autonavi.com/appmaptile?x=${x}&y=${y}&z=${z}&lang=zh_cn&size=1&scale=1&style=8`
}
```

这里随机了四个子域：`webrd01`、`webrd02`、`webrd03`、`webrd04`，这是因为浏览器对于同一域名同时请求的资源是有数量限制的，而当地图层级变大后需要加载的瓦片数量会比较多，那么均匀分散到各个子域下去请求可以更快的渲染出所有瓦片，减少排队等待时间，基本所有地图厂商的瓦片服务地址都支持多个子域。

为了方便看到中心点的位置，我们再额外渲染两条中心辅助线，效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/feea95376f604dd8874acabc7c4a1b01~tplv-k3u1fbpfcp-watermark.awebp?)

可以看到中心点确实是雷峰塔，当然这只是渲染了中心瓦片，我们要的是瓦片铺满整个画布，对于其他瓦片我们都可以根据中心瓦片计算出来，比如中心瓦片左边的一块，它的计算如下：

```js

let leftTile = [centerTile[0] - 1, centerTile[1]]

let leftTilePos = [
    offset[0] - TILE_SIZE * 1,
    offset[1]
]
```

所以我们只要计算出中心瓦片四个方向各需要几块瓦片，然后用一个双重循环即可计算出画布需要的所有瓦片，计算需要的瓦片数量很简单，请看下图：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c015e197c69498babe2cd034beadf6a~tplv-k3u1fbpfcp-watermark.awebp?)

画布宽高的一半减去中心瓦片占据的空间即可得到该方向剩余的空间，然后除以瓦片的尺寸就知道需要几块瓦片了：

```js

let rowMinNum = Math.ceil((this.width / 2 - offset[0]) / TILE_SIZE)
let colMinNum = Math.ceil((this.height / 2 - offset[1]) / TILE_SIZE)
let rowMaxNum = Math.ceil((this.width / 2 - (TILE_SIZE - offset[0])) / TILE_SIZE)
let colMaxNum = Math.ceil((this.height / 2 - (TILE_SIZE - offset[1])) / TILE_SIZE)
```

我们把中心瓦片作为原点，坐标为`[0, 0]`，来个双重循环扫描一遍即可渲染出所有瓦片：

```js

for (let i = -rowMinNum; i <= rowMaxNum; i++) {
    for (let j = -colMinNum; j <= colMaxNum; j++) {
        
        let img = new Image()
        img.src = getTileUrl(
            centerTile[0] + i,
            centerTile[1] + j,
            this.zoom
        )
        img.onload = () => {
            
            this.ctx.drawImage(
                img, 
                i * TILE_SIZE - offset[0], 
                j * TILE_SIZE - offset[1]
            )
        }
    }
}
```

效果如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3f54767d0d184c8291b761fef2fac83b~tplv-k3u1fbpfcp-watermark.awebp?)

很完美。

拖动可以这么考虑，前面已经实现了渲染指定经纬度的瓦片，当我们按住进行拖动时，可以知道鼠标滑动的距离，然后把该距离，也就是像素转换成经纬度的数值，最后我们再更新当前中心点的经纬度，并清空画布，调用之前的方法重新渲染，不停重绘造成是在移动的视觉假象。

监听鼠标相关事件：

```html
<canvas ref="canvas" @mousedown="onMousedown"></canvas>
```

```js
export default {
    data(){
        return {
            isMousedown: false
        }
    },
    mounted() {
        window.addEventListener("mousemove", this.onMousemove);
        window.addEventListener("mouseup", this.onMouseup);
    },
    methods: {
        
        onMousedown(e) {
            if (e.which === 1) {
                this.isMousedown = true;
            }
        },

        
        onMousemove(e) {
            if (!this.isMousedown) {
                return;
            }
            
        },

        
        onMouseup() {
            this.isMousedown = false;
        }
    }
}
```

在`onMousemove`方法里计算拖动后的中心经纬度及重新渲染画布：

```js

let mx = e.movementX * resolutions[this.zoom];
let my = e.movementY * resolutions[this.zoom];

let [x, y] = lngLat2Mercator(...this.center);

center = mercatorToLngLat(x - mx, my + y);
```

`movementX`和`movementY`属性能获取本次和上一次鼠标事件中的移动值，兼容性不是很好，不过自己计算该值也很简单，详细请移步[MDN](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FMouseEvent%2FmovementX "https&#x3A;//developer.mozilla.org/zh-CN/docs/Web/API/MouseEvent/movementX")。乘以当前分辨率把`像素`换算成`米`，然后把当前中心点经纬度也转成`3857`的`米`坐标，偏移本次移动的距离，最后再转回`4326`的经纬度坐标作为更新后的中心点即可。

为什么`x`是减，`y`是加呢，很简单，我们鼠标向右和向下移动时距离是正的，相应的地图会向右或向下移动，`4326`坐标系向右和向上为正方向，那么地图向右移动时，中心点显然是相对来说是向左移了，因为向右为正方向，所以中心点经度方向就是减少了，所以是减去移动的距离，而地图向下移动，中心点相对来说是向上移了，因为向上为正方向，所以中心点纬度方向就是增加了，所以加上移动的距离。

更新完中心经纬度，然后清空画布重新绘制：

```js

this.clear();

this.renderTiles();
```

效果如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ca1acb1ec2348f9bf1b6aaf4c58735c~tplv-k3u1fbpfcp-watermark.awebp?)

可以看到已经凌乱了，这是为啥呢，其实是因为图片加载是一个异步的过程，我们鼠标移动过程中，会不断的计算出要加载的瓦片进行加载，但是可能上一批瓦片还没加载完成，鼠标已经移动到新的位置了，又计算出一批新的瓦片进行加载，此时上一批瓦片可能加载完成并渲染出来了，但是这些瓦片有些可能已经被移除画布，不需要显示，有些可能还在画布内，但是使用的还是之前的位置，渲染出来也是不对的，同时新的一批瓦片可能也加载完成并渲染出来，自然导致了最终显示的错乱。

知道原因就简单了，首先我们加个缓存对象，因为在拖动过程中，很多瓦片只是位置变了，不需要重新加载，同一个瓦片加载一次，后续只更新它的位置即可；另外再设置一个对象来记录当前画布上应该显示的瓦片，防止不应该出现的瓦片渲染出来：

```js
{
    
    tileCache: {},
    
    currentTileCache: {}
}
```

因为需要记录瓦片的位置、加载状态等信息，我们创建一个瓦片类：

```js

class Tile {
  constructor(opt = {}) {
    
    this.ctx = ctx
    
    this.row = row
    this.col = col
    
    this.zoom = zoom
    
    this.x = x
    this.y = y
    
    this.shouldRender = shouldRender
    
    this.url = ''
    
    this.cacheKey = this.row + '_' + this.col + '_' + this.zoom
    
    this.img = null
    
    this.loaded = false

    this.createUrl()
    this.load()
  }
    
  
  createUrl() {
    this.url = getTileUrl(this.row, this.col, this.zoom)
  }

  
  load() {
    this.img = new Image()
    this.img.src = this.url
    this.img.onload = () => {
      this.loaded = true
      this.render()
    }
  }

  
  render() {
    if (!this.loaded || !this.shouldRender(this.cacheKey)) {
      return
    }
    this.ctx.drawImage(this.img, this.x, this.y)
  }
    
  
  updatePos(x, y) {
    this.x = x
    this.y = y
    return this
  }
}
```

然后修改之前的双重循环渲染瓦片的逻辑：

```js
this.currentTileCache = {}
for (let i = -rowMinNum; i <= rowMaxNum; i++) {
    for (let j = -colMinNum; j <= colMaxNum; j++) {
        
        let row = centerTile[0] + i
        let col = centerTile[1] + j
        
        let x = i * TILE_SIZE - offset[0]
        let y = j * TILE_SIZE - offset[1]
        
        let cacheKey = row + '_' + col + '_' + this.zoom
        
        this.currentTileCache[cacheKey] = true
        
        if (this.tileCache[cacheKey]) {
            
            this.tileCache[cacheKey].updatePos(x, y).render()
        } else {
            
            this.tileCache[cacheKey] = new Tile({
                ctx: this.ctx,
                row,
                col,
                zoom: this.zoom,
                x,
                y,
                
                shouldRender: (key) => {
                    return this.currentTileCache[key]
                },
            })
        }
    }
}
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47545910fab64c17a393ded3b1b36e28~tplv-k3u1fbpfcp-watermark.awebp?)

可以看到，拖动已经正常了，当然，上述实现还是很粗糙的，需要优化的地方很多，比如：

1\. 一般会先排个序，优先加载中心瓦片

2\. 缓存的瓦片越来越多肯定也会影响性能，所以还需要一些清除策略

这些问题有兴趣的可以自行思考。

拖动是实时更新中心点经纬度，那么缩放自然更新缩放层级就行了：

```js
export default {
    data() {
        return {
            
            minZoom: 3,
            maxZoom: 18,
            
            zoomTimer: null
        }
    },
    mounted() {
        window.addEventListener('wheel', this.onMousewheel)
    },
    methods: {
        
        onMousewheel(e) {
            if (e.deltaY > 0) {
                
                if (this.zoom > this.minZoom) this.zoom--
            } else {
                
                if (this.zoom < this.maxZoom) this.zoom++
            }
            
            this.zoomTimer = setTimeout(() => {
                this.clear()
                this.renderTiles()
            }, 300)
        }
    }
}
```

效果如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88596e9c348b4477958f42a43cd8494a~tplv-k3u1fbpfcp-watermark.awebp?)

功能是有了，不过效果很一般，因为我们平常使用的地图缩放都是有一个放大或缩小的过渡动画，而这个是直接空白然后重新渲染，不仔细看都不知道是放大还是缩小。

所以我们不妨加个过渡效果，当我们鼠标滚动后，先将画布放大或缩小，动画结束后再根据最终的缩放值来渲染需要的瓦片。

画布默认缩放值为`1`，放大则在此基础上乘以`2`倍，缩小则除以`2`，然后动画到目标值，动画期间设置画布的缩放值及清空画布，重新绘制画布上的已有瓦片，达到放大或缩小的视觉效果，动画结束后再调用`renderTiles`重新渲染最终缩放值需要的瓦片。

```js

import { animate } from 'popmotion'

export default {
    data() {
        return {
            lastZoom: 0,
            scale: 1,
            scaleTmp: 1,
            playback: null,
        }
    },
    methods: {
        
        onMousewheel(e) {
            if (e.deltaY > 0) {
                
                if (this.zoom > this.minZoom) this.zoom--
            } else {
                
                if (this.zoom < this.maxZoom) this.zoom++
            }
            
            if (this.lastZoom === this.zoom) {
                return
            }
            this.lastZoom = this.zoom
            
            this.scale *= e.deltaY > 0 ? 0.5 : 2
            
            if (this.playback) {
                this.playback.stop()
            }
            
            this.playback = animate({
                from: this.scaleTmp,
                to: this.scale,
                onUpdate: (latest) => {
                    
                    this.scaleTmp = latest
                    
                    
                    
                    this.ctx.save()
                    this.clear()
                    this.ctx.scale(latest, latest)
                    
                    Object.keys(this.currentTileCache).forEach((tile) => {
                        this.tileCache[tile].render()
                    })
                    
                    this.ctx.restore()
                },
                onComplete: () => {
                    
                    this.scale = 1
                    this.scaleTmp = 1
                    
                    this.renderTiles()
                },
            })
        }
    }
}
```

效果如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c804e8573fb42a7af5ebb11195d1823~tplv-k3u1fbpfcp-watermark.awebp?)

虽然效果还是一般，不过至少能看出来是在放大还是缩小。

前面还遗留了一个小问题，即我们把高德工具上选出的经纬度直接当做`4326`经纬度，前面也讲过，它们之间是存在偏移的，比如手机`GPS`获取到的经纬度一般都是`84`坐标，直接在高德地图显示，会发现和你实际位置不一样，所以就需要进行一个转换，有一些工具可以帮你做些事情，比如[Gcoord](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fhujiulong%2Fgcoord "https&#x3A;//github.com/hujiulong/gcoord")、[coordtransform](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwandergis%2Fcoordtransform "https&#x3A;//github.com/wandergis/coordtransform")等。

上述效果看着比较一般，其实只要在上面的基础上稍微加一点瓦片的淡出动画，效果就会好很多，目前一般都是使用`canvas`来渲染`2D`地图，如果自己实现动画不太方便，也有一些强大的`canvas`库可以选择，笔者最后使用[Konva.js](https://link.juejin.cn/?target=https%3A%2F%2Fkonvajs.org%2F "https&#x3A;//konvajs.org/")库重做了一版，加入了瓦片淡出动画，最终效果如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/725ab970c4ea411db86822bb74c797dc~tplv-k3u1fbpfcp-watermark.awebp?)

另外只要搞清楚各个地图的瓦片规则，就能稍加修改支持更多的地图瓦片：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c66f520fca824bc1aa12fc572b71e479~tplv-k3u1fbpfcp-watermark.awebp?)

具体实现限于篇幅不再展开，有兴趣的可以阅读本文源码。

本文详细的介绍了一个简单的`web`地图开发过程，上述实现原理仅是笔者的个人思路，不代表`openlayers`等框架的原理，因为笔者也是`GIS`的初学者，所以难免会有问题，或更好的实现，欢迎指出。

在线`demo`：[wanglin2.github.io/web_map_dem…](https://link.juejin.cn/?target=https%3A%2F%2Fwanglin2.github.io%2Fweb_map_demo%2F "https&#x3A;//wanglin2.github.io/web_map_demo/")

完整源码：[github.com/wanglin2/we…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwanglin2%2Fweb_map_demo "https&#x3A;//github.com/wanglin2/web_map_demo") 
 [https://juejin.cn/post/7054729902871805966](https://juejin.cn/post/7054729902871805966)
