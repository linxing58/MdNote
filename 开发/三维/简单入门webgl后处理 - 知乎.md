# 简单入门webgl后处理 - 知乎
大家好，在之前聊过 shader 后，相信大家对 webgl 的原生和底层渲染机制有了一些的了解，但是只是看概念，似乎很难加深印象。今天我们接着更进一步，聊下后处理。

首先，我们来看看啥是后处理。threejs 的官方示例里面，有个章节就叫后处理，里面有许多的后处理示例。其实后处理主要集中在一些效果展示上，例如下面的：

**bloom 泛光**

![](https://pic4.zhimg.com/v2-6280ed92265787f877a50db914a5de87_b.jpg)

**godray 体积光**

![](https://pic2.zhimg.com/v2-5e2a1b7220b6bd2ceb3719cb64fc4b65_b.jpg)

**outline 描边**

![](https://pic1.zhimg.com/v2-e0e520c264b24380a0942d04eef1b884_b.jpg)

## 原理与基本概念

在讲解源代码之前，我们还是先从基础理论开始讲起，否则对于很多新手同学，可能理解代码有点费劲。我们先从概念理解入手，一步步的深入到代码，看看如何在 threejs 中简单的应用后处理技术。

### 什么是后处理

后处理也叫 PostProcessing，是图形渲染或者游戏引擎中非常常见的一种技术。它的实现方式通常是在普通的场景渲染结束后对结果再执行进一步的处理, 一般是通过绘制一个铺满屏幕的四方形（quad），并将渲染的 buffer 作为 texture 传入，调用 shader 计算，将计算结果写入到另一个 buffer 中，最终显示在屏幕上。每个计算流程和普通的模型场景渲染一样, 不同之处在于在 vertex shader 中通常只是简单的拷贝, 主要的逻辑计算写在 fragement shader 中。一般我们将一组特定的功能代码组织在一起配合 shader 处理，这样一次处理我们叫一个 pass，然后将多个 pass 一起组合在一起组成了整个后处理的渲染流水线。

### FBO 与 RTT

这里，我们需要着重讲一些知识点，就是 FBO（FrameBuffer Object）和 RTT（Render to Texture）。这些知识点对于理解 webgl 的渲染，以及后处理流程都是非常重要的。

FrameBuffer 就像是一个 webgl 绘制的容器一样，平时我们默认绘制都是将 3d 场景绘制在了默认的窗口中输出（此时绑定 framebuffer 为 null），而当我们指定一个 FrameBuffer 为当前绘制容器，再绘制时则会将对象绘制于指定的 FrameBuffer 中，而不是直接绘制到屏幕。

framebuffer 对象是一个可以包含颜色（color）、透明度（alpha），深度（depth）、模板（stencil）等缓冲（buffer）的集合。并且提供 2 个方法`gl.framebufferTexture2D`,`gl.framebufferRenderbuffer`用来绑定 texture 或者 renderbuffer 数据到指定的附加点（attachment）。

![](https://pic3.zhimg.com/v2-547f097ad6c2d97b8c300f788b4dd232_b.jpg)

相关组合有以下几种 **[官网解释](https://link.zhihu.com/?target=https%3A//www.khronos.org/registry/webgl/specs/latest/1.0/%23FBO_ATTACHMENTS)**

1. 仅 texture（COLOR_ATTACHMENT0 = RGBA/UNSIGNED_BYTE）

2. texture（COLOR_ATTACHMENT0 = RGBA/UNSIGNED_BYTE） + renderbuffer（DEPTH_ATTACHMENT = DEPTH_COMPONENT16）

3. texture（COLOR_ATTACHMENT0 = RGBA/UNSIGNED_BYTE） + renderbuffer（DEPTH_STENCIL_ATTACHMENT = DEPTH_STENCIL）

更多 framebuffer 的信息可以查看 Webgl 官方介绍 **[地址](https://link.zhihu.com/?target=https%3A//www.khronos.org/registry/webgl/specs/latest/1.0/%235.14.6)**

当我们使用 framebuffer 对象渲染当前场景到一个 texture 上的时候，我们实际上就已经在使用 rtt 技术了。

## threejs 中如何实现一个后处理

当理解完基本的底层概念后，我们来看看，如何在 threejs 中应用一个简单的后处理实现天空盒的模糊效果。

首先我们创建一个普通的场景结构，包含一个 renderer，一个相机，已经一个 scene。这部分代码和一般场景没有什么区别，这里不做赘述。

```text
const renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true,
    canvas: document.getElementById("renderCanvas"),
}); 
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(0, 200, 300);
camera.lookAt(new THREE.Vector3(0, 300, 0));
// controls
const controls = new OrbitControls(camera, renderer.domElement);
```

然后我们初始化一个`EffectComposer`对象的实例，这个对象的定义实现代码在 `examples\jsm\postprocessing` 中。一个 composer 对象其实是一个后处理流程的调度器或者也可以叫 manager。它包含一个 addPass 方法，用来添加一个后处理 pass。然后它有一个叫做 render 的方法，用来渲染整个后处理流程。

```text
const composer = new EffectComposer(renderer);
```

我们的后处理具有 5 个 pass，一个渲染背景色，一个渲染天空盒，一个对天空盒做模糊处理，一个渲染主场景, 最后一个 pass 将内容输出到屏幕。

```text
// 清屏背景色pass
const clearPass = new ClearPass(0xff0000, 1);
composer.addPass(clearPass);
// 天空盒pass
const cubeTexturePassP = new CubeTexturePass(camera);
composer.addPass(cubeTexturePassP);
// 模糊处理pass
const blurPass = new BlurPass(
    1,
    new THREE.Vector2(renderer.domElement.width, renderer.domElement.height)
);
composer.addPass(blurPass);
// 场景渲染
const renderPass = new RenderPass(scene, camera);
renderPass.clear = false;
composer.addPass(renderPass);
// copy内容到屏幕输出
const copyPass = new ShaderPass(CopyShader);
composer.addPass(copyPass);
```

当添加完 pass 后，我们修改主渲染循环，将普通渲染场景的 renderer.render() 方法换成 composer.render()。

```text
function animate() {
    requestAnimationFrame(animate);
    // 设置模糊
    blurPass.blur = params.blur;
    // 设置明暗
    renderer.toneMappingExposure = params.exposure;
    // 设置天空盒透明度
    cubeTexturePassP.opacity = params.cubeTexturePassOpacity;
    // 渲染
    composer.render();
}
```

配合 dat.gui 组件，实时修改参数，我们可以控制天空盒的模糊度。

```text
var gui = new GUI();
gui.add(params, "exposure", 0, 2, 0.01);
gui.add(params, "blur", 0, 100);
gui.add(params, "cubeTexturePassOpacity", 0, 1);
gui.open();
```

![](https://pic1.zhimg.com/v2-0590e7b36c8337cf26422279d5161014_b.gif)

[demo](https://link.zhihu.com/?target=https%3A//github.com/code945/threejs-tutorial/blob/master/src/scripts/postprocess.js)

[代码在这里](https://link.zhihu.com/?target=https%3A//github.com/code945/threejs-tutorial/blob/master/src/scripts/postprocess.js)

后处理的核心，是在每个 pass 的 shader 中，这部分才是整个后处理的难点和算法核心。在熟悉流程后，其实大家只需要专心攻克 shader 中的图像算法就好了，其他流程其实只是辅助。在了解完整个流程后，我们来看看 threejs 的后处理实现的背后都做了哪些事情。

**EffectComposer**

首先是我们来看看`EffectComposer`，它内部到底在做些什么。从源码中我们可以看到，一个 composer 中会初始化两个`WebGLRenderTarget` (FBO)，一个叫 writebuffer，另一个叫 readbuffer。在渲染执行的时候，composer 会遍历内部所有的 pass，然后执行每个 pass 的渲染，每个 pass 都会传入 `writeBuffer` 和 `readBuffer` ，一般每个 pass 渲染会读取 readbuffer 中的数据，并写入 writebuffer。在每次 pass 的渲染后，可以根据 pass 的 `needsSwap` 属性来互换这两个 buffer，从而形成一个读取写入的流式处理。当处理到最后一个 pass 的时候，一般会把当前的渲染的 fbo 设置为 null，也就是渲染到屏幕。所以经过最后一次渲染后，我们的渲染结果最终会被输出到屏幕。

**pass**

pass 对象是 threejs 为我们提供的一个 pass 的基类，里面其实非常简单只有几个简单属性。

```text
// 是否启用
    this.enabled = true;
    //  是否交换buffer
    this.needsSwap = true;
    //  是否清屏
    this.clear = false;
    //  是否渲染到屏幕
    this.renderToScreen = false;
```

**renderpass**

renderpass 一般用来渲染场景，把 scene 中的场景渲染到某个 buffer 中，注意这个 renderpass 如果是最后一个的话，会将内容直接渲染到屏幕输出，所以如果想叠加之前的处理结果，是需要额外加一个 copypass 的。

**shaderpass**

shaderpass 的作用是使用一个 shader，处理平面图形。一般来说 shaderpass 的输入是一些之前的处理后 rendertarget 的 texture，输出也是通过一个正交相机对着的平面输出到 buffer 中。所以这个时候，一些深度计算和测试都会有些问题，大家在处理的时候需要额外注意。

**我们的实现 blurpass 对象**

理解完其他的概念后，我们不难写出一个属于自己的 blurpass。继承了 pass 基类后，在 blurpass 中我们主要是做了两次的高斯径向模糊。具体高斯模糊实现的代码在 shander 中

```text
varying vec2 vUv;
uniform sampler2D colorTexture;
uniform vec2 texSize;
uniform vec2 direction;
uniform float kernelRadius;

float gaussianPdf(in float x, in float sigma) {
    return 0.39894 * exp( -0.5 * x * x/( sigma * sigma))/sigma;
}
void main() {
    vec2 invSize = 1.0 / texSize;
    float weightSum = gaussianPdf(0.0, kernelRadius);
    vec4 diffuseSum = texture2D( colorTexture, vUv) * weightSum;
    vec2 delta = direction * invSize * kernelRadius/float(MAX_RADIUS);
    vec2 uvOffset = delta;
    for( int i = 1; i <= MAX_RADIUS; i ++ ) {
        float w = gaussianPdf(uvOffset.x, kernelRadius);
        vec4 sample1 = texture2D( colorTexture, vUv + uvOffset);
        vec4 sample2 = texture2D( colorTexture, vUv - uvOffset);
        diffuseSum += ((sample1 + sample2) * w);
        weightSum += (2.0 * w);
        uvOffset += delta;
    }
    gl_FragColor = diffuseSum/weightSum;
}"
```

关于高斯模糊的原理，可以看这篇 [http://www.ruanyifeng.com/blog/2012/11/gaussian_blur.html](https://link.zhihu.com/?target=http%3A//www.ruanyifeng.com/blog/2012/11/gaussian_blur.html) 阮一峰的科普文。

以上便是对后处理的总结，总得来说是一些科普和概念总结，并没有什么高深的技术，旨在帮助大家理解整个流程和使用中常见的问题。如果有兴趣的同学，可以继续研究，探索下后处理里面可能会遇到的，比如后处理之后的抗锯齿、后处理的时候深度贴图处理等等，这里只是抛砖引玉，希望对大家入门后处理有所帮助。 
 [https://zhuanlan.zhihu.com/p/151254866](https://zhuanlan.zhihu.com/p/151254866)
