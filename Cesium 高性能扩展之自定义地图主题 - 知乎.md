# Cesium 高性能扩展之自定义地图主题 - 知乎
项目中经常需要各种炫酷的效果，普通的地图样式无法满足，我们会到处找各种深色主题（深蓝色、暗黑等等）的地图数据源，或者使用地图服务重新配图，总之还是比较麻烦的。那么有没有一种方法，可以在前端直接实现各种主题的效果呢？

本文介绍一种 Cesium 影像图层后处理技术，实现各种各样的地图主题样式。通过本文您可能有如下收获：

-   加深对 Cesium 影像加载（`ImageryLayer`和`ImageryProvider`）的理解；
-   加深对`DrawCommand`的理解；
-   了解`Cesium 实现 RTT`的基本流程。

![](https://pic3.zhimg.com/v2-0556bbf463c97c5b2c4bb3dd3ec6907e_b.jpg)

### 1、原理

基本原理就是图像处理，即：在地形瓦片渲染之前，对请求到的瓦片图像进行加工处理（如生成灰度图等） 。

这里介绍两种实现方式的原理，对比之下才体现出第二个方式的 “高性能”：

-   基于 Canvas 的图像处理：完全在 CPU 中实现，逐个像素处理；
-   基于 WebGL 的图像处理：使用 GPU 并行加速。

两种方式共同的处理流程大体相同，流程如下：

-   输入图像；
-   读取图像颜色；
-   扫描每个像素，应用具体的处理算法；
-   将新颜色写入新图像；
-   返回新新图像。

对于第一种方式，研究`Cesium.ImageryProvider`就会发现相对容易实现，通过重写其`requestImage`，实现拦截图像（image），插入图像处理流程，再返回加工后的图像。这个过程也比较容易理解。

但是第二种方式，就相对隐蔽一些，需要研究源码：通过重写`Cesium.ImageryLayer`的`_createTextureWebGL`方法，拦截瓦片的纹理贴图（texture）, 这里就从处理图像变成了处理纹理，再返回加工后的纹理。这个方式除了比较难想到之外，还需要熟悉 Cesium 实现`渲染到纹理`的技术，即 `RTT` 技术，这也是这个方式高性能的基础。

### 2、加载天地图

我们使用天地图的矢量数据作为数据源，这也是可以放心使用的公开数据，使用前请先去其官网申请 appKey。  
加载天地图的代码虽然很基础，但是入门的时候可能经常找半天，所以还是贴出来吧。

```js
function createTdtImageryProvider(params) {
  var tileMatrixSet = "w";
  var host = params.host || "http://t{s}.tianditu.com/";
  var subdomains = ["0", "1", "2", "3", "4", "5", "6", "7"];

  if (host[host.length - 1] == "/") {
    host = host.substr(0, host.length - 1);
  }
  var url =
    host +
    "/" +
    params.layer +
    "_" +
    tileMatrixSet +
    "/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=" +
    params.layer +
    "&tileMatrixSet=" +
    tileMatrixSet +
    "&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles";
  url += "&tk=" + params.appKey;

  let provider = new Cesium.WebMapTileServiceImageryProvider({
    url: url,
    layer: params.layer,
    style: "default",
    subdomains: subdomains,
    tileMatrixSetID: tileMatrixSet,
    maximumLevel: params.maximumLevel || 18,
    minimumLevel: params.minimumLevel,
  });

  return provider;
}
var imageryProvider = createTdtImageryProvider({
  layer: "vec",
  appKey: "你的天地图AppKey",
});
//接下来我们就要对这个图层进行处理
var layer = viewer.imageryLayers.addImageryProvider(imageryProvider);

```

### 3、基于 Canvas 的图像处理

下面通过 canvas 技术实现天地图官网上的地图样式示例，来演示基于 Canvas 的图像处理基本流程。不多说，上代码，看注释。

```js
class ImageryTheme {
  constructor(options) {
    this.params = this.params;
  }
  //背景混合
  _drawBg(image) {
    const width = image.width,
      height = image.height;
    var cv = document.createElement("canvas");
    cv.width = width;
    cv.height = height;
    var ctx = cv.getContext("2d");
    ctx.fillStyle = this.params.bgColor;
    ctx.fillRect(0, 0, width, height);
    ctx.drawImage(image, 0, 0);
    return cv;
  }
  //处理瓦片图像
  processImage(image, layer) {
    const { width, height } = image;
    //新图像
    var cv = document.createElement("canvas");
    cv.width = width;
    cv.height = height;
    var ctx = cv.getContext("2d");
    //将原始图像绘制到新图像中，这是因为输入的图像可能是Image、Canvas或者ImageBitmap
    ctx.drawImage(image, 0, 0, width, height);
    //读取图像颜色
    const imgData = ctx.getImageData(0, 0, width, height),
      params = this.params,
      bgColor = params.bgColor,
      alpha = params.alpha;
    //扫描，对每个像素进行加工处理
    for (let i = 0; i < imgData.data.length; i += 4) {
      const r = imgData.data[i],
        g = imgData.data[i + 1],
        b = imgData.data[i + 2],
        a = imgData.data[i + 3];
      //计算灰度
      var grayVal = (r + g + b) / 3;
      //灰度反转
      if (params.invert) {
        grayVal = 255 - grayVal;
      }
      //将灰度替换掉原始的颜色
      imgData.data[i] = grayVal;
      imgData.data[i + 1] = grayVal;
      imgData.data[i + 2] = grayVal;
      //设置一个前景透明度，以便和背景混合
      imgData.data[i + 3] = a * alpha;
    }
    //将处理后的像素写入新图像
    ctx.putImageData(imgData, 0, 0);

    if (bgColor && alpha > 0) {
      //与背景混合，并返回新图像
      return this._drawBg(cv);
    } else {
      //直接返回新图像
      return cv;
    }
  }
}

```

### 应用主题的代码：

```js
//创建主题，设置相关参数
const theme = new ImageryTheme({
  bgColor: "black",
  alpha: 0.5,
  invert: true,
});
//重写 imageryProvider.requestImage 方法
const requestImage = layer.imageryProvider.requestImage;
imageryProvider.requestImage = function (x, y, level, request) {
  var promise = requestImage.bind(imageryProvider)(x, y, level, request);
  if (promise) {
    promise = promise.then((image) => {
      var imageProcessed = theme.processImage(image, layer);
      return imageProcessed || image;
    });
  }
  return promise;
};

```

### 4、基于 WebGL 的图像处理

下面我们着重介绍第二种方式的实现过程。

### 4.1、Cesium 实现 RTT

渲染到纹理（`Render To Texture`，RTT），是是一种离屏渲染技术，在 Cesium 中其实并不陌生，各种好看的炫酷的后期特效，就是基于这个技术的，甚至可以这么说，在开启某个后期处理流程后，整个 Cesium 场景都是先渲染到纹理的，再渲染到屏幕上的。

通过前面几期文章，可以了解到 Cesium 渲染器的核心类`DrawCommand`，我们要实现`RTT`离不开它。

假设`DrawCommand`已经创建，那么 RTT 的关键步骤就两个：

-   创建颜色纹理（或者渲染缓冲区）；
-   创建深度纹理（或者深度缓冲区），这一步不是必须的；
-   创建帧缓冲区，绑定颜色、深度纹理或者渲染（深度）缓冲区；
-   `DrawCommand`绑定帧缓冲区；
-   执行`DrawCommand`；
-   `DrawCommand`解除绑定帧缓冲区。

本文我们处理图像的过程和 Cesium 后期处理是一样的，`Cesium.Context`提供了一个方法让我们可以快捷创建一个可以用于处理二维图像的`DrawCommand`:

```js
const fs = "这里写你的图像处理glsl代码";
var drawCommand = context.createViewportQuadCommand(fs, {
  //绑定帧缓冲区对象
  framebuffer: framebuffer,
  //绑定渲染状态对象
  renderState: renderState,
  //这里向shader传递uniform参数，
  //和DrawCommand系列文章的中写法一样
  uniformMap: uniformMap,
});
//执行drawCommand
drawCommand.execute(context);

```

### 4.2、定义主题类

这里通过 shader 实现和天地图官网上的样式示例类似的主题样式。

### 4.2.1、处理背景

```js
uniform sampler2D u_map;
uniform vec4 u_bgColor;
    
varying vec2 v_textureCoordinates;
void main(){
    vec2 uv = v_textureCoordinates;
    vec4 bgColor = u_bgColor;
    vec4 color = texture2D( u_map , uv );
    if( color.a > 0. ){
        gl_FragColor = bgColor;
    }
}

```

### 4.2.2、处理瓦片图像

```js
uniform sampler2D u_map;
uniform bool u_invert;
uniform float u_alpha;

varying vec2 v_textureCoordinates;
void main(){
    vec2 uv = v_textureCoordinates;
    vec4 color = texture2D( u_map , uv );
    //计算灰度
    float grayVal =( color.r + color.g + color.b ) / 3.;、
    //灰度反转
    if(u_invert){
        grayVal = 1. - grayVal;
    }
    //应用前景透明度，以便和背景混合
    float alpha = color.a * u_alpha;
    gl_FragColor=vec4( vec3( grayVal ) , alpha );
}

```

### 4.2.3、主题类

```js
class ImageryThemeGL {
  /**
   *
   * @param {{
   *  name: string
   *  bgColor?: Cesium.Color
   *  alpha?: number
   *  invert?: boolean
   * }} options
   */
  constructor(options) {
    super(options);
    const params = this.params,
      { defaultValue, Color } = Cesium;
    params.bgColor = defaultValue(params.bgColor, Color.TRANSPARENT);
    params.alpha = defaultValue(params.alpha, 1);
    params.invert = defaultValue(params.invert, false);
    params.preMultiplyAlpha = defaultValue(params.preMultiplyAlpha, true);

    var scope = this;
    this.shaders = {
      bgFS:'这里贴背景处理的shader代码',
      mainFS: `这里贴处理瓦片图像的shader代码`,
      uniformMap: {
        u_invert() {
          return params.invert;
        },
        u_alpha() {
          return params.alpha;
        },
        u_bgColor() {
          return params.bgColor;
        },
      },
    };
  }

  /**
   *
   * @param {Cesium.Context} context
   * @param {Cesium.Texture} texture
   * @returns {Cesium.Texture}
   */
  processTexture(context, texture) {
    const { params, shaders } = this;
    const sampler = texture.sampler;
    const {
      Texture,
      PixelFormat,
      PixelDatatype,
      WebGLConstants,
      Framebuffer,
      RenderState,
      BlendingState,
      Color,
      defined,
    } = Cesium;

    //创建处理后的瓦片贴图，作为RTT的目标纹理
    var textureProcessed = new Texture({
      context: context,
      pixelFormat: PixelFormat.RGBA,
      pixelDatatype: PixelDatatype.UNSIGNED_BYTE,
      sampler: sampler,
      width: texture.width,
      height: texture.height,
      flipY: texture.flipY,
      target: WebGLConstants.TEXTURE_2D,
      preMultiplyAlpha: params.preMultiplyAlpha,
    });
    var framebuffer = new Framebuffer({
      context: context,
      colorTextures: [textureProcessed],
      destroyAttachments: false,
    });
    const renderState = RenderState.fromCache({
      depthTest: {
        enabled: false,
      },
      blending: params.preMultiplyAlpha
        ? BlendingState.PRE_MULTIPLIED_ALPHA_BLEND
        : BlendingState.ALPHA_BLEND,
      viewport: {
        x: 0,
        y: 0,
        width: texture.width,
        height: texture.height,
      },
    });

    //传递原始瓦片贴图
    shaders.uniformMap.u_map = function () {
      return texture;
    };

    //画背景
    if (!Color.equals(params.bgColor, Color.TRANSPARENT)) {
      var bgCommand = context.createViewportQuadCommand(shaders.bgFS, {
        framebuffer: framebuffer,
        renderState: renderState,
        uniformMap: shaders.uniformMap,
      });
      bgCommand.execute(context);
    }

    //瓦片加工
    var mainCommand = context.createViewportQuadCommand(shaders.mainFS, {
      framebuffer: framebuffer,
      renderState: renderState,
      uniformMap: shaders.uniformMap,
    });
    mainCommand.execute(context);

    framebuffer.destroy();
    texture.destroy();

    return textureProcessed;
  }
}

```

### 4.2.4、应用主题

最后重写 layer.\_createTextureWebGL 方法，应用主题样式。

```js
//创建主题，设置相关参数
const theme=new ImageryThemeGL({
    bgColor:Cesium.Color.BLACK,
    alpha:0.5,
    invert:true,
    preMultiplyAlpha:true
})
//重写 layer._createTextureWebGL 方法
var _createTextureWebGL = layer._createTextureWebGL;
layer._createTextureWebGL = function (context, imagery) {
  var texture = _createTextureWebGL.bind(this)(context, imagery);
  var textureProcessed = scope.processTexture(context, texture);
  return textureProcessed || texture;
};

```

### 5、示例主题效果

![](https://pic3.zhimg.com/v2-0556bbf463c97c5b2c4bb3dd3ec6907e_b.jpg)
![](https://pic4.zhimg.com/v2-cb3711c44b23533ccfe8022b21f5397f_b.jpg)
![](https://pic1.zhimg.com/v2-704d959d8046ba5029f56be8183331cc_b.jpg) 
 [https://zhuanlan.zhihu.com/p/480835644](https://zhuanlan.zhihu.com/p/480835644)
