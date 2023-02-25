# CesiumJS 2022^ 源码解读[5] - 着色器相关的封装设计 - 四季留歌 - 博客园
* * *

本篇涉及到的所有接口在公开文档中均无，需要下载 GitHub 上的源码，自己创建私有类的文档。

```null
npm run generateDocumentation -- --private
yarn generateDocumentation -- --private
pnpm generateDocumentation -- --private

``` 

本篇当然不会涉及着色器算法讲解。

任何一个有追求的 WebGL 3D 库都会封装 WebGL 原生接口。CesiumJS 从内部封测到现在，已经有十年了，WebGL 自 2011 年发布以来也有 11 年了，这期间小修小补不可避免。

更何况 CesiumJS 是一个 JavaScript 的地理 3D 框架，它在源代码设计上具备两大特征：

*   面向对象
*   模块化

关于模块化策略，CesiumJS 在 1.63 版本已经从 `require.js` 切换到原生 `es-module` 格式了。而 WebGL 是一种使用全局状态的指令式风格接口，改为面向对象风格就必须做封装。ThreeJS 是通用 Web3D 库中做 WebGL 封装的代表作品。

封装有另外的好处，就是底层 WebGL 接口在这十多年中的变化，可以在封装后屏蔽掉这些变化，上层应用调用封装后的 API 方式基本不变。

1.1. 缓冲对象封装
-----------

CesiumJS 封装了 `WebGLBuffer` 以及 WebGL 2.0 才正式支持（1.0 中用扩展）的 VAO，分别封装成了 `Buffer` 类和 `VertexArray` 类。

`Buffer` 类比较简单，提供了简单工厂模式的静态创建方法：

 ```null

Buffer.createVertexBuffer = function (options) {
  

  return new Buffer({
    context: options.context,
    bufferTarget: WebGLConstants.ARRAY_BUFFER,
    typedArray: options.typedArray,
    sizeInBytes: options.sizeInBytes,
    usage: options.usage,
  });
};


Buffer.createIndexBuffer = function (options) {
  
};

``` 

`Buffer` 对象在实例化时，就会创建 `WebGLBuffer` 并将类型数组上载：

 ```null

const buffer = gl.createBuffer();
gl.bindBuffer(bufferTarget, buffer);
gl.bufferData(bufferTarget, hasArray ? typedArray : sizeInBytes, usage);
gl.bindBuffer(bufferTarget, null);

``` 

除了这两个用于创建的静态方法，还有一些拷贝缓冲对象的方法，就不一一列举了。

注意一点：**`Buffer` 对象不保存原始顶点类型数组数据。** 这一点是出于节约 JavaScript 内存考虑。

而顶点数组对象 `VertexArray`，封装的则是 OpenGL 系中的一个数据模型 `VertexArrayObject`，在 WebGL 中是意图节约设置多个顶点缓冲到全局状态对象的性能损耗。

创建 CesiumJS 的顶点数组对象也很简单，只需按 WebGL 的顶点属性（Vertex Attribute）的格式去装配 `Buffer` 对象即可：

```null
const positionBuffer = Buffer.createVertexBuffer({
  context: context,
  sizeInBytes: 12,
  usage: BufferUsage.STATIC_DRAW
})
const normalBuffer = Buffer.createVertexBuffer({
  context: context,
  sizeInBytes: 12,
  usage: BufferUsage.STATIC_DRAW
})
const attributes = [
  {
    index: 0,
    vertexBuffer: positionBuffer,
    componentsPerAttribute: 3,
    componentDatatype: ComponentDatatype.FLOAT
  },
  {
    index: 1,
    vertexBuffer: normalBuffer,
    componentsPerAttribute: 3,
    componentDatatype: ComponentDatatype.FLOAT
  }
]
const va = new VertexArray({
  context: context,
  attributes: attributes
})

``` 

你如果在对着上述代码练习，你肯定没法成功创建，并发现一个问题：没有 `context` 参数传递给 `Buffer` 或 `VertexArray`，因为 `context` 对象（类型 `Context`）是 WebGL 渲染上下文对象等底层接口的的封装对象，没有它无法创建 `WebGLBuffer` 等原始接口对象。

所以，`Buffer`、`VertexArray` 并不是孤立的 API，必须与其它封装一起搭配来用，它们两个至少要依赖 `Context` 对象才行，在 1.4 中会介绍如何使用 `Context` 类封装 WebGL 底层接口并如何访问 `Context` 对象的。

很少有需要直接创建 `Buffer`、`VertexArray` 的时候，使用这两个接口，就意味着你获得的数据符合 `VBO` 格式，其它人类阅读友好型的数据格式必须转换为 VBO 格式才能直接用这俩类。如果你需要使用第 2 节中提及的指令对象，这两个类就能派上用场了。

1.2. 纹理与采样参数封装
--------------

纹理是 WebGL 中一个非常复杂的话题。

先说采用参数吧，在 WebGL 1.0 时还没有原生的采样器 API，到 2.0 才推出的 `WebGLSampler` 接口。所以，CesiumJS 封装了一个简单的 `Sampler` 类：

```null
function Sampler(options) {
  
  this._wrapS = wrapS;
  this._wrapT = wrapT;
  this._minificationFilter = minificationFilter;
  this._magnificationFilter = magnificationFilter;
  this._maximumAnisotropy = maximumAnisotropy;
}

``` 

其实就是把 WebGL 1.0 中的纹理采样参数做成了一个对象，没什么难的。

纹理类 `Texture` 则是对 `WebGLTexture` 的封装，它不仅封装了 `WebGLTexture`，还封装了数据上载的功能，只需安心地把贴图数据传入即可。

同 `Buffer`、`VertexArray`，`Texture` 也要 `context` 参数。

```null
import {
  Texture, Sampler,
} from 'cesium'

new Texture({
  context: context,
  width: 1920,
  height: 936,
  source: new Float32Array([]), 
  
  sampler: new Sampler()
})

``` 

你可以在 `ImageryLayer.js` 模块中找到创建影像瓦片纹理的代码：

```null
ImageryLayer.prototype._createTextureWebGL = function (context, imagery) {
  
  return new Texture({
    context: context,
    source: image,
    pixelFormat: this._imageryProvider.hasAlphaChannel
      ? PixelFormat.RGBA
      : PixelFormat.RGB,
    sampler: sampler,
  });
}

``` 

除了创建纹理，CesiumJS 还提供了纹理的拷贝工具函数，譬如从帧缓冲对象中拷贝出一个纹理：

```null
Texture.fromFramebuffer = function () {  }

Texture.prototype.copyFromFramebuffer = function () {  }

``` 

或者创建 mipmap：

```null
Texture.prototype.generateMipmap = function () {  }

``` 

1.3. 着色器封装
----------

众所周知，WebGL 的着色器相关 API 是 `WebGLShader` 和 `WebGLProgram`，顶点着色器和片元着色器共同构成一个着色器程序对象。在一帧的渲染中，由多个通道构成，每个通道在触发 `draw` 动作之前，通常要切换着色器程序，以达到不同的计算效果。

CesiumJS 的渲染远远复杂于通用 Web3D，意味着有大量着色器程序。对象多了，就要管理。CesiumJS 封装了有关底层 API 的同时，还设计了缓存机制。

CesiumJS 使用 `ShaderSource` 类来管理着色器代码文本，使用 `ShaderProgram` 类来管理 `WebGLProgram` 和 `WebGLShader`，使用 `ShaderCache` 类来缓存 `ShaderProgram`，再使用 `ShaderFunction`、`ShaderStruct`、`ShaderDestination` 来辅助 `ShaderSource` 处理着色器代码文本中的 glsl 函数、结构体成员、宏定义。

此外，还有一个 `ShaderBuilder` 类来辅助 `ShaderProgram` 的创建。

这一堆私有类与前面 `Buffer`、`VertexArray`、`Texture` 一样，并不能单独使用，通常是与第 2 节中的各种指令对象一起用。

下面给出一个例子，它使用 `ShaderProgram` 的静态方法 `fromCache` 创建着色器程序对象，这个方法会创建对象的同时并缓存到 `ShaderCache` 对象中，有兴趣的可以自行查看缓存的代码。

```null
const vertexShaderText = `attribute vec3 position;
 void main() {
   gl_Position = czm_projection * czm_view * czm_model * vec4(position, 1.0);
 }`
const fragmentShaderText = `uniform vec3 u_color;
 void main(){
   gl_FragColor = vec4(u_color, 1.0);
 }`
 
const program = ShaderProgram.fromCache({
  context: context,
  vertexShaderSource: vertexShaderText,
  fragmentShaderSource: fragmentShaderText,
  attributeLocations: {
    "position": 0,
  },
})

``` 

完整例子可以找我之前的写的关于使用 `DrawCommand` 绘制三角形的文章。

1.4. 上下文对象与渲染通道
---------------

WebGL 底层接口的封装，基本上都在 `Context` 类中。最核心的就是渲染上下文（`WebGLRenderingContext`、`WebGL2RenderingContext`）对象了，除此之外，`Context` 上还有一些重要的渲染相关的功能和成员变量：

*   一系列 WebGL 2.0 中才支持的、WebGL 1.0 中用扩展才支持的特性
*   压缩纹理的支持信息
*   `UniformState` 对象
*   `PassState` 对象
*   `RenderState` 对象
*   参与帧渲染的功能，譬如 draw、readPixels 等
*   创建拾取用的 PickId
*   操作、校验 `Framebuffer` 对象

通常，通过 `Scene` 对象上的 `FrameState` 对象，即可访问到 `Context` 对象。

WebGL 渲染上下文对象暴露的常量很多，CesiumJS 把渲染上下文上的常量以及可能会用到的常量都封装到 `WebGLConstants.js` 导出的对象中了。

还有一个东西要特别说明，就是通道，WebGL 是没有通道 API 的，而一帧之内切换着色器进行多道绘制过程是很常见的事情，每一道触发 `draw` 的行为，叫做通道。

CesiumJS 把高层级三维对象的渲染行为做了打包，封装成了三类指令对象，在第 2 节中会讲；这些指令对象是有先后优先级的，CesiumJS 把这些优先级描述为通道，使用 `Pass.js` 导出的枚举来定义，目前指令对象有 10 个优先级：

```null
const Pass = {
  ENVIRONMENT: 0,
  COMPUTE: 1,
  GLOBE: 2,
  TERRAIN_CLASSIFICATION: 3,
  CESIUM_3D_TILE: 4,
  CESIUM_3D_TILE_CLASSIFICATION: 5,
  CESIUM_3D_TILE_CLASSIFICATION_IGNORE_SHOW: 6,
  OPAQUE: 7,
  TRANSLUCENT: 8,
  OVERLAY: 9,
  NUMBER_OF_PASSES: 10,
};

``` 

`NUMBER_OF_PASSES` 成员代表当前有 10 个优先级。

而在帧状态对象上，也有一个 `passes` 成员：

 ```null

this.passes = {
  render: false,
  pick: false,
  depth: false,
  postProcess: false,
  offscreen: false,
};

``` 

这 5 个布尔值就控制着渲染用的是哪个通道。

指令对象的通道状态值，加上帧状态对象上的通道状态，共同构成了 CesiumJS 庞大的抽象模型中的“通道”概念。

> 其实我认为这样设计会导致 Scene 单帧渲染时有大量的 if 判断、排序处理，显得有些冗余，能像 WebGPU 这种新 API 一样提供通道编码器或许会简化通道的概念。

1.5. 统一值（uniform）封装
-------------------

统一值，即 WebGL 中的 `Uniform`，不熟悉的读者需要自己先学习 WebGL 相关概念。

每一帧，有大量的状态值是和上一帧不一样的，也就是需要随时更新进着色器中。CesiumJS 为此做出了封装，这种频繁变动的统一值被封装进了 `AutomaticUniforms` 对象中了，每一个成员都是 `AutomaticUniform` 类实例：

 ```null

function AutomaticUniform(options) {
  this._size = options.size;
  this._datatype = options.datatype;
  this.getValue = options.getValue;
}

``` 

从默认导出的 `AutomaticUniforms` 对象中拿一个成员来看：

```null
czm_projection: new AutomaticUniform({
  size: 1,
  datatype: WebGLConstants.FLOAT_MAT4,
  getValue: function (uniformState) {
    return uniformState.projection;
  },
})

``` 

这个统一值是摄像机的投影矩阵，它的取值函数需要一个 `uniformState` 参数，也就是实时地从统一值状态对象（类型 `UniformState`）上获取的。

`Context` 对象拥有一个只读的 `UniformState` getter，指向一个私有的成员。当 `Scene` 在执行帧状态上的指令列表时，会调用 `Context` 的绘制函数，进一步地会调用 `Context.js` 模块内的 `continueDraw` 函数，它就会执行着色器程序对象的 `_setUniforms` 方法：

```null
shaderProgram._setUniforms(
  uniformMap,
  context._us,
  context.validateShaderProgram
);

``` 

这个函数就能把指令对象传下来的自定义 `uniformMap`，以及 `AutomaticUniforms` 给设置到 `ShaderProgram` 内置的 `WebGLProgram` 上，也就是完成着色器内统一值的设置。

1.6. 渲染容器封装
-----------

渲染容器主要就是指帧缓冲对象、渲染缓冲对象。

渲染缓冲对象，CesiumJS 封装为 `Renderbuffer` 类，是对 `WebGLRenderbuffer` 的一个非常简单的封装，不细说了，但是要单独提一点，若启用了 msaa，会调用相关的绑定函数：

 ```null


function Renderbuffer(options) {
  
  const gl = context._gl;

  
  this._renderbuffer = this._gl.createRenderbuffer();

  gl.bindRenderbuffer(gl.RENDERBUFFER, this._renderbuffer);
  if (numSamples > 1) {
    gl.renderbufferStorageMultisample(
      gl.RENDERBUFFER,
      numSamples,
      format,
      width,
      height
    );
  } else {
    gl.renderbufferStorage(gl.RENDERBUFFER, format, width, height);
  }
  gl.bindRenderbuffer(gl.RENDERBUFFER, null);
}

``` 

接下来说帧缓冲的封装。

普通的帧缓冲，也就是常规的 `WebGLFramebuffer` 被封装到 `Framebuffer` 类里了，它有几个数组成员，用于保存帧缓冲用到的颜色附件、深度模板附件的容器纹理、容器渲染缓冲。

```null
function Framebuffer(options) {
  const context = options.context;
  
  Check.defined("options.context", context);
  

  const gl = context._gl;
  const maximumColorAttachments = ContextLimits.maximumColorAttachments;

  this._gl = gl;
  this._framebuffer = gl.createFramebuffer();

  this._colorTextures = [];
  this._colorRenderbuffers = [];
  this._activeColorAttachments = [];

  this._depthTexture = undefined;
  this._depthRenderbuffer = undefined;
  this._stencilRenderbuffer = undefined;
  this._depthStencilTexture = undefined;
  this._depthStencilRenderbuffer = undefined;
  
  
}

``` 

用也很简单，调用原型链上绑定相关的方法即可。CesiumJS 支持 MRT，所以有一个对应的 `bindDraw` 方法：

```null
Framebuffer.prototype.bindDraw = function () {
  const gl = this._gl;
  gl.bindFramebuffer(gl.DRAW_FRAMEBUFFER, this._framebuffer);
};

``` 

msaa 则用到了 `MultisampleFramebuffer` 这个类；CesiumJS 还设计了 `FramebufferManager` 类来管理帧缓冲对象，在后处理、OIT、拾取、Scene 的帧缓冲管理等模块中均有使用。

CesiumJS 并不会直接处理地理三维对象，而是在各种更新的流程控制函数中，由每个三维对象去生成一种叫做“指令”的对象，送入帧状态对象的相关渲染队列中。

这些指令对象的就屏蔽了各种高层级的“人类友好”型三维数据对象的差异，`Context` 能方便统一地处理它们携带的数据资源（缓冲、纹理）和行为（着色器）。

这些指令对象分成三类：

*   绘图指令（绘制指令），`DrawCommand` 类，负责渲染绘图
*   清屏指令，`ClearCommand` 类，负责清空绘图区域
*   通用计算指令，`ComputeCommand` 类，用 WebGL 来进行 GPU 并行计算

下面进行简单讲解。

2.1. 绘图指令（绘制指令）
---------------

也就是 `DrawCommand` 类，位于 `Renderer/DrawCommand.js` 模块。

绘图指令，在 `Scene` 对象的每一帧更新过程中，由各种高级三维对象生成，并添加到帧状态对象中，等待渲染。

我曾经写过一篇 `DrawCommand` 画最简单的三角形的文，如果你点进我的用户文章列表找不到，你可能看到本文的盗版了：）

简而言之，创建 `DrawCommand` 需要数据（`VertexArray`、uniformMap、`RenderState`），也需要行为（`ShaderProgram`）。创建 `DrawCommand` 这些辅料，大多数都需要 `Context` 对象。

它的执行过程如下：

```null
DrawCommand.prototype.execute
  Context.prototype.draw
    fn beginDraw
    fn continueDraw

``` 

关于 `Scene` 渲染一帧的文章中已经提过如何执行这些绘制指令，是 `Scene` 原型链上的 `updateAndExecuteCommands` 方法出发，一路走到 `executeCommand` 函数，最终调用各种指令对象的执行方法。

上面的简易逻辑流程中，`beginDraw` 这个模块内的函数，负责绑定帧缓冲对象和渲染状态，并绑定 `ShaderProgram` 到 WebGL 全局状态上：

```null
function beginDraw() {
  
  bindFramebuffer(context, framebuffer);
  applyRenderState(context, renderState, passState, false);
  shaderProgram._bind();
  
}

``` 

紧接着，`continueDraw` 函数会向 WebGL 全局状态设置（更新）统一值：

 ```null

shaderProgram._setUniforms(
  uniformMap,
  context._us,
  context.validateShaderProgram
);

``` 

然后就是走 WebGL 的常规绘制流程了，绑定 `VertexArray`，判断是否用了索引缓冲，岔开逻辑分别绘制顶点数据：

 ```null

va._bind();
const indexBuffer = va.indexBuffer;

if (defined(indexBuffer)) {
  
} else {
  count = defaultValue(count, va.numberOfVertices);
  if (instanceCount === 0) {
    context._gl.drawArrays(primitiveType, offset, count);
  } else {
    context.glDrawArraysInstanced(
      primitiveType,
      offset,
      count,
      instanceCount
    );
  }
}

va._unBind();

``` 

代码 `va._bind();` 是在绑定各种顶点数据。

2.2. 清屏指令
---------

清屏指令，与 WebGL 的 `clear` 方法目的是一样的，即清空当前帧缓冲（或 canvas）的颜色部分、深度模板部分，并填充特定的值，封装成了 `ClearCommand` 类。

清屏指令就比较简单了，它的执行与绘制指令是一个过程，都在 `Scene.js` 模块下的 `executeCommand` 函数中：

 ```null

if (command instanceof ClearCommand) {
  command.execute(context, passState);
  return;
}

``` 

可以看到它一旦被执行，就不继续执行后面的关于绘制指令的代码，直接 return 了。

紧接着会执行 `Context` 原型链上的 `clear` 方法，它也会绑定帧缓冲、设置渲染状态，最后调用 `gl.clear` 方法，将设定的待清除后要填充的颜色、深度、模板值刷上去，过程比较简单，就不贴源码了。

`Scene` 对象上有几个清屏指令成员对象，在渲染流程中由 `updateAndClearFramebuffers` 函数执行颜色清屏指令，而模板与深度的清屏指令则由 `executeCommands` 函数执行：

 ```null


function executeCommandsInViewport() {
  
  if (firstViewport) {
    if (defined(backgroundColor)) {
      updateAndClearFramebuffers(scene, passState, backgroundColor);
    }
    
  }
  executeCommands(scene, passState);  
}

function updateAndClearFramebuffers(scene, passState, clearColor) {
  
  const clear = scene._clearColorCommand;
  Color.clone(clearColor, clear.color);
  clear.execute(context, passState);
  
}

function executeCommands(scene, passState) {
  
  const clearDepth = scene._depthClearCommand;
  const clearStencil = scene._stencilClearCommand;
  
  
  for (let i = 0; i < numFrustums; ++i) {
    
    clearDepth.execute(context, passState);
    if (context.stencilBuffer) {
      clearStencil.execute(context, passState);
    }
    
  }
}

``` 

当然，有 `ClearCommand` 的对象不仅仅只有 `Scene`，其它地方也有，你可以在源码中全局搜索 `new ClearCommand` 关键词。

2.3. 通用计算指令
-----------

早期的 WebGL 1.0 对 GPU 通用计算（GPGPU）支持得并不是很好，想让 GPU 模拟普通的并行计算，需要把数据编码成纹理，经由渲染管线完成纹理采样、计算后，再输出到帧缓冲对象上，再使用 WebGL 读像素的接口函数把结果读取出来。

WebGL 2.0 的计算着色器是姗姗来迟。

CesiumJS 最开始使用了 `ComputeCommand` 来区别作用于渲染任务的 `DrawCommand`。

CesiumJS 源码中使用了计算指令的地方不多，最经典的就是影像图层的重投影方法中：

```null
ImageryLayer.prototype._reprojectTexture = function () {
  
  const computeCommand = new ComputeCommand({
    persists: true,
    owner: this,
    preExecute: function (command) {
      reprojectToGeographic(command, context, texture, imagery.rectangle);
    },
    postExecute: function (outputTexture) {
      imagery.texture = outputTexture;
      that._finalizeReprojectTexture(context, outputTexture);
      imagery.state = ImageryState.READY;
      imagery.releaseReference();
    },
    canceled: function () {
      imagery.state = ImageryState.TEXTURE_LOADED;
      imagery.releaseReference();
    },
  });
  
}

``` 

执行它的“管家”，不是 `Context` 而是 `ComputeEngine` 类。

```null
ComputeCommand.prototype.execute = function (computeEngine) {
  computeEngine.execute(this);
};

``` 

当然，去看 `ComputeEngine` 类的构造函数，它也只不过是对 `Context` 的一个封装，用到了装饰器模式：

```null
function ComputeEngine(context) {
  this._context = context;
}

``` 

查看 `ComputeEngine.prototype.execute` 的核心执行部分，其实它也是用 `DrawCommand` 和 `ClearCommand`，在独立的 `Framebuffer` 上执行传入的 `ShaderProgram`。

```null
ComputeEngine.prototype.execute = function (computeCommand) {
  
  const outputTexture = computeCommand.outputTexture;
  const width = outputTexture.width;
  const height = outputTexture.height;

  const context = this._context;
  const vertexArray = defined(computeCommand.vertexArray)
    ? computeCommand.vertexArray
    : context.getViewportQuadVertexArray();
  const shaderProgram = defined(computeCommand.shaderProgram)
    ? computeCommand.shaderProgram
    : createViewportQuadShader(context, computeCommand.fragmentShaderSource);
  
  const framebuffer = createFramebuffer(context, outputTexture);
  const renderState = createRenderState(width, height);
  const uniformMap = computeCommand.uniformMap;

  
  const clearCommand = clearCommandScratch;
  clearCommand.framebuffer = framebuffer;
  clearCommand.renderState = renderState;
  clearCommand.execute(context);

  
  const drawCommand = drawCommandScratch;
  drawCommand.vertexArray = vertexArray;
  drawCommand.renderState = renderState;
  drawCommand.shaderProgram = shaderProgram;
  drawCommand.uniformMap = uniformMap;
  drawCommand.framebuffer = framebuffer;
  drawCommand.execute(context);

  framebuffer.destroy();
  
}

``` 

具体的计算着色、纹理编码原理就不介绍了，属于着色器原理，本文（本系列文章）更多是介绍架构设计细节。

CesiumJS 留有一些公开的 API，允许开发者写自己的着色过程。

在 Cesium 团队大力开发下一代 3DTiles 和模型类新架构之前，这部分能力比较弱，只有一个 `Fabric` 材质规范能写写现有几何对象的材质效果，且文档较少。

随着下一代 3DTiles 与新的模型类实验性启用后，带来了自由度更高的 `CustomShader API`，不仅仅有齐全的文档，而且给到开发者最大的自由去修改图形渲染。

3.1. 早期 Fabric 材质规范中的自定义着色器
---------------------------

写 `Primitive API` 时，有这么一个字段：

```null
new Primitive({
  
  appearance: new MaterialAppearance({
    material: Material.fromType('Color'),
    faceForward: true
  })
})

``` 

这个 `MaterialAppearance` 是 `Appearance` 类的一个子类，除了上述这两个属性之外，还可以自己传递顶点着色器代码。

但是，通常不会直接向 `Appearance` 的派生子类们提供顶点着色器、片元着色器，因为外观对象所需的着色器有额外的要求，通常是创建 `Material` 时写材质 glsl 函数：

```null
const fabricMaterial = new Material({
  fabric: {
    uniforms: {
      my_var: 0.5,
    },
    source: `czm_material czm_getMaterial(czm_materialInput input) {
      czm_material material = czm_getDefaultMaterial(input);
      material.diffuse = vec3(materialInput.st, 0.0);
      material.alpha = my_var;
      return material;
    }`
  }
})

``` 

然后把这个遵循了 `Fabric` 材质规范的材质对象，传递给外观对象：

```null
new MaterialAppearance({
  material: fabricMaterial
})

``` 

`Fabric` 材质规范这里不多介绍，以后有机会再开一文吧，简单的说传递一个 JavaScript 对象给 `Material` 的 `fabric` 成员变量即可，这个对象可以自定义一种材质，可以具备 uniformMap，并在 glsl 代码中使用一个函数，返回一个 `czm_material` 结构体作为材质。

虽然可以创建 glsl 结构体作为材质，但是它仅仅只能作用于片元的部分着色过程。

`Appearance API` 作用的是 `Primitive API` 生成的图元对象，外观对象支持直接传递 Primitive 所需的两大着色器代码，但是也是有限制的，一些 `vertex attritbute`、`varying` 是必须存在的，而且还得自己处理渲染管线的转换，这方面资料较少。

通过下列代码，你可以输出最简单的 `MaterialAppearance` 对象生成的内置默认两大着色器代码，方便自己修改：

```null
const appearance = new Cesium.MaterialAppearance({
  material: new Cesium.Material({}),
})

const vs = appearance.vertexShaderSource
const fs = appearance.fragmentShaderSource
const fsWithFabricMaterial = appearance.getFragmentShaderSource()




``` 

3.2. 后处理中的自定义着色器
----------------

CesiumJS 其实内置了一大堆常见的后处理器，常见的辉光（Bloom）、快速抗锯齿（FXAA）等都有，参考 `PostProcessStageLibrary` 这个类导出的静态字段即可。

> 虽然这些后处理都是最基本的、对整个 FBO 进行的，效果一般。

你可以访问 `scene.postProcessStages` 访问后处理器的容器，譬如启用快速抗锯齿是这样启用的：

```null
viewer.scene.postProcessStages.fxaa.enabled = true

``` 

> 内置的环境光遮蔽（AO）或辉光（Bloom）必定在所有后处理器之前执行，FXAA 必定位于所有后处理器之后执行。这三个阶段也是 CesiumJS 默认创建的后处理器。

你可以自己创建单独的后处理阶段（`PostProcessStage`）或合成后处理阶段（`PostProcessStageComposite`）作为一个后处理器传入 `PostProcessStageCollection` 容器中，如果不使用官方提供的其它常见后处理算法，那么你也可以自己写着色器。

官方文档写道：

> 每个后处理阶段的输入纹理，是 Scene 渲染的纹理，或前一后处理阶段的输出纹理。

参考 `PostProcessStage` 类的文档，官方也提供了两个例子：

 ```null

const fs = `uniform sampler2D colorTexture;
varying vec2 v_textureCoordinates;
uniform float scale;
uniform vec3 offset;
void main() {
  vec4 color = texture2D(colorTexture, v_textureCoordinates);
  gl_FragColor = vec4(color.rgb * scale + offset, 1.0);
}`
scene.postProcessStages.add(new Cesium.PostProcessStage({
  fragmentShader: fs,
  uniforms: {
    scale: 1.1,
    offset: function() {
      return new Cesium.Cartesian3(0.1, 0.2, 0.3);
    }
  }
}))

``` 

后处理还支持拾取对象的判断，也就是例子2，修改拾取对象的颜色：

```null
const fs = `uniform sampler2D colorTexture;
varying vec2 v_textureCoordinates;
uniform vec4 highlight;
void main() {
  vec4 color = texture2D(colorTexture, v_textureCoordinates);
  if (czm_selected()) {
    vec3 highlighted = 
      highlight.a * highlight.rgb + (1.0 - highlight.a) * color.rgb;
    gl_FragColor = vec4(highlighted, 1.0);
  } else {
    gl_FragColor = color;
  }
}`
const stage = scene.postProcessStages.add(new Cesium.PostProcessStage({
  fragmentShader: fs,
  uniforms: {
    highlight: function() {
      return new Cesium.Color(1.0, 0.0, 0.0, 0.5);
    }
  }
}))
stage.selected = [cesium3DTileFeature]

``` 

`PostProcessStage` 的 `selected` 是一个 js 数组，支持设定 `Cesium3DTileFeature`、`Label`、`Billboard` 等具备 `pickId` 访问器，或 `Model`、`Cesium3DTilePointFeature` 等具备 `pickIds` 访问器的类为“选中对象”，并在更新过程（`PostProcessStage.prototype.update`）中创建一张选择纹理。

有关后处理的资料以后可能会专门写应用篇来介绍。

3.3. 新架构带来的 CustomShader API
----------------------------

这个伴随着 `ModelExperimental` 这个新架构（2022年5月，此架构正在启动对原 `Model` 类相关架构的替换）的更新而随时可能会更新。

目前，`CustomShader API` 仅支持在 `Cesium3DTileset`、`ModelExperimental` 两个类上使用。传入的自定义着色器作用在每个瓦片或模型上。

举例：

```null
import {
  CustomShader, UniformType, TextureUniform, VaryingType
} from 'cesium'
const customShader = new CustomShader({
  uniforms: {
    u_colorIndex: {
      type: UniformType.FLOAT,
      value: 1.0
    },
    u_normalMap: {
      type: UniformType.SAMPLER_2D,
      value: new TextureUniform({
        url: "http://example.com/normal.png"
      })
    }
  },
  varyings: {
    v_selectedColor: VaryingType.VEC3
  },
  vertexShaderText: `void vertexMain(
    VertexInput vsInput,
    inout czm_modelVertexOutput vsOutput
  ) {
    v_selectedColor = mix(
      vsInput.attributes.color_0,
      vsInput.attributes.color_1, u_colorIndex
    );
    vsOutput.positionMC += 0.1 * vsInput.attributes.normal;
  }`,
  fragmentShaderText: `void fragmentMain(
    FragmentInput fsInput,
    inout czm_modelMaterial material
  ) {
    material.normal = texture2D(
      u_normalMap, fsInput.attributes.texCoord_0
    );
    material.diffuse = v_selectedColor;
  }`
})

``` 

相关规范文档可以在源代码根目录下 `Documentation/CustomShaderGuide/README.md` 文件中查阅。

你可以设定在渲染管线中需要的 uniformMap，指定两个着色器之间的交换值（varyings），并且能在两大着色器中访问 CesiumJS 封装好的两个结构体 `VertexInput`、`FragmentInput`，它们为你提供了尽可能详尽的值，譬如：

*   顶点属性（Vertex Attributes）
*   要素/批次ID（FeatureID/BatchID）
*   3DTiles 1.1 规范中的属性元数据（Metadata）

顶点属性中提供了尽可能详尽的顶点信息，常见的：顶点坐标、法线、纹理坐标、颜色等均有附带，而且附带了各种坐标系下的值。譬如，你可以在顶点着色器中这样访问相机坐标系下的顶点坐标：

```null
void vertexMain(
  VertexInput vsInput,
  inout czm_modelVertexOutput vsOutput
) {
  vsOutput.positionMC = czm_projection * (
    vec4(vsInput.attributes.positionEC) + vec4(.0, .0, 10.0, 0.0)
  );
}

``` 

上述代码将观察坐标的 z 值提高了 10 个单位，最后乘上内置的投影矩阵，作为输出模型坐标。

> CesiumJS 提供的所有内置 glsl 函数、常量、自动变量均支持在 CustomShader API 中使用。

这无疑给想修改模型形状、模型效果的开发者们提供了极大的便利。

个人觉得在 WebGL 封装这部分，在本文已经讲得可以了，更高层级的应用封装，例如 OIT、GPUPick、各种对象的着色器和指令生成过程，均基于这篇文章的内容，均发生在 `Scene` 的渲染流程中。

最后再强调一下，本文仅是对架构方面的介绍，而不是着色器算法的详解，着色器算法可谓 CesiumJS 的头脑风暴区，一篇文章容不下。

具备渲染架构、WebGL 封装基础后，接下来就该看最经典的模型（glTF）、3DTiles 的渲染架构设计了，旧版本的模型架构正在被新架构替换中，所以之后直接以新架构为基础讲解。

\_\_EOF\_\_

[![](https://blog-static.cnblogs.com/files/onsummer/avatar.gif)
](https://blog-static.cnblogs.com/files/onsummer/avatar.gif)

*   **本文作者：**  [四季留歌](https://www.cnblogs.com/onsummer)
*   **本文链接：**  [https://www.cnblogs.com/onsummer/p/16272537.html](https://www.cnblogs.com/onsummer/p/16272537.html)
*   **关于博主：**  评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/onsummer)我。
*   **版权声明：**  本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-nd/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
*   **声援博主：**  如果您觉得文章对您有帮助，可以点击文章右下角**【推荐】** 一下。