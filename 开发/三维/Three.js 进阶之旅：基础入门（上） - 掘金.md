# Three.js 进阶之旅：基础入门（上） - 掘金
> 本文为稀土掘金技术社区首发签约文章，14天内禁止转载，14天后未获授权禁止转载，侵权必究！

背景
--

### 写在前面

本文是**Three.js 进阶之旅**系列专栏的首篇文章，本专栏的主要内容具体规划如下：前两篇将简要介绍 `Three.js` 开发环境搭建以及Three.js 的一些基础概念和必备知识，如果读者已经有一定的 `Three.js` 3D项目开发基础，可以直接跳过这两章内容。后续文章会通过一个个趣味的 `3D` 页面实例，逐步讲解 `Three.js` 相关性能优化、着色器、后期渲染、物理特性等应用中知识，期间也会穿插介绍一些 `3D` 建模、压缩工具和技巧。本专栏适用于有一定 `JavaScript` 和 `CSS` 编程基础的同学，相信阅读完此专栏，一定会对 `Three.js` 有进一步的理解。

### 一些约定

为了规范化文章写作，在本系列文章写作制定了一些约定，大家可以根据以下符号以及文字样式格式迅速找到自己感兴趣的内容，后续文章都将沿用此格式。

文章目录结构：

*   **摘要**：本文包含重要内容的概括性描述。
*   **效果**：根据本文内容构建的页面效果。
*   **实现**：具体实现步骤及涉及到的知识点。
*   **总结**：本文涉及的知识点列表清单。
*   **附录**：本文的外部引用及其一些附加内容。

文章内容标注：

*   强调内容：**粗体文本**
*   知识点：`💡 知识点`
*   注意点：`📌 注意`

正文
--

### Three.js 简介

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ef6932c776549c5a5d69ff027e734f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

随着 `HTML5` 标准的颁布，以及主流浏览器的功能日益强大，直接在浏览器中展示三维图像和动画已经变得越来越容易。`WebGL` 技术为在浏览器中创建丰富的三维图形提供了丰富且强大的接口，它们不仅可以创建二维图形和应用，还可以充分利用 `GPU`，创建漂亮的、高性能的三维应用。

但是直接使用 `WebGL` 编程非常复杂，需要了解 `WebGL` 的内部细节，学习复杂的着色器语法和足够多的数学和图形学方面的专业知识才能学好 `WebGL`。

`Three.js` 就是一款基于原生 `WebGL` 封装运行在浏览器中的 `3D` 引擎，可以用它创建各种三维场景，包括了摄影机、光影、材质等各种对象。使用 `Three.js` 不必学习 `WebGL` 的详细细节，就能轻松创建出漂亮的三维图形。

### 开发环境

`Three.js` 支持在 `<script>`标签从直接从本地文件目录或 `CDN` 及静态主机引入，也支持使用 \`NPM 下载安装并在项目页面导入。

```html
<script src="https://cdn.skypack.dev/three@<version>"></script>

```

要安装 `three` 的 `npm` 模块，需要在项目文件夹里打开终端窗口，并运行以下命令，包将会被下载并安装，然后就可以将它导入我们的代码了。

```cmd
npm install --save three

```

**引入方式1**：导入整个 `Three.js` 核心库

```js
import * as THREE from 'three';
const scene = new THREE.Scene();

```

**引入方式2**：按需导入你所需要的内容

```js
import { Scene } from 'three';
const scene = new Scene();

```

本文及后续文章都将使用 `NPM` 安装导入的形式，并使用 `Webpack` 进行本地热加载更新和打包。本专栏系列项目已经配置好开发环境，大家只需克隆仓库安装即可，不必从头开始配置。项目托管在 `Github` 仓库[【threejs-odessey】](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")，**后续所有专栏项目目录都将在这个仓库中更新**。

> `🔗` 代码仓库地址：[git@github.com:dragonir/threejs-ode…](https://link.juejin.cn/?target=)

先来看看仓库目录的基本结构

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30867f5f2158493e9dc65f8bdbb75f7a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

其中 `bundler` 目录是 `Webpack` 的配置文件目录；`dist` 目录是打包之后的文件，`src` 就是主要开发目录，后续所有的开发工作都将在此文件夹完成，`index.html` 是主页面，页面逻辑相关代码位于 `script.js`、`style.css` 是全局样式表；`static` 用于存放一些静态资源，如开发所需的图面，模型等。后续随着页面功能的扩展以及对性能优化的考虑，会对目录进行按模块拆分，**当前目录只适应于交互比较简单的单页面应用**。

### 运行环境

本专栏目录下的所有示例都将使用原生 `JavaScript` 开发，因此可以很容易移植到其他如`React`、`Vue`、`Anaular`、`jQuery` 等项目中，大家克隆代码后直接在项目中运行 `npm install` 及 `npm run dev` 即可，页面会自动在默认浏览器中打开，对代码内容修改也会实时反映到浏览器。

### 部署环境

项目开发完成后，为了给他人展示页面，我们可以运行 `npm run build`，然后将 `dist` 文件夹中的打包文件进行部署，可以选择部署在 `Github Page`、`Vercel`、`Codepen`、`码上掘金` 等免费静态页面服务上，也可以部署到自己的服务器上。（`Vercel`是一个很好用的前端可持续继承部署服务，可以关联 `Github`，静态页面可以免费部署并能生成预览链接）。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1a1ebb5c56e428f9c774c02ff2c7bb6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

> `📌` 由于项目采用纯原生 `JavaScript` 开发，因此可以直接移移植到**码上掘金**部署预览。当然，直接使用码上掘金开发也是不错的。

入门示例
----

`Three.js` 项目整体开发流程可以按照以下步骤进行，其中有些步骤可以并行进行。后续本专栏所有项目也都将按照这个流程进行开发。

*   **容器构建**：添加需要渲染 `3D` 内容的容器和基本页面结构。
*   **引入资源**：导入`Three.js`、以及开发页面功能所需要的其他库、静态资源等。
*   **场景初始化**：定义一些全局变量如渲染尺寸、容器等；初始化渲染器；初始化场景；处理页面缩放事件监听处理等。
*   **逻辑开发**：按需求开发业务功能，并将其添加到场景中。
*   **动画更新**：更新渲染器和相机，并根据业务需求，更新其他网格模型的动画。
*   **性能优化**：离开页面时释放 `GPU` 资源、清除定时器和动画等。
*   **修饰优化**：使用 `CSS`、图片等元素装饰界面，提升页面视觉效果。

### 容器构建

配置完成开发环境，现在马上可以尝试开发自己的第一个 `Three.js` `3D` 页面。首先 `index.html` 中构建基本如下的页面结构，如不需要显示额外的元素，则只需要在 `body` 中添加一个 `canvas` 元素即可。

```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>01 - Introduction</title>
</head>
<body>
  <canvas class="webgl"></canvas>
</body>
</html>

```

### 引入资源

在顶部引 `样式表` 和 `Three.js`，`Three.js`可以像示例中一样全量引入，也可以这样 `import { Scene } from 'three'` 这样按需引入以减少文件体积，提高加载速率。

```js
import './style.css';
import * as THREE from 'three';

```

### 场景初始化

定义好渲染尺寸的常量，我们需要覆盖满整个窗口，所以宽高的取值是 `window.innerWidth` 和 `window.innerHeight`。如果需要渲染到具体 `DOM` 中，那么宽高就取 `DOM` 的宽高。然后初始化渲染器、场景和相机。

```js

const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}


const canvas = document.querySelector('canvas.webgl');
const renderer = new THREE.WebGLRenderer({ canvas: canvas });
renderer.setSize(sizes.width, sizes.height);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));


const scene = new THREE.Scene();


const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100);
camera.position.z = 3;
scene.add(camera);

```

#### `知识点 💡` 渲染器 WebGLRenderer

`WebGLRenderer` 用 `WebGL` 渲染出场景。通过 `new THREE.WebGLRenderer` 初始化渲染器，并将 `canvas` 容器作为参数传给它。通过调用 `setSize` 方法设置渲染器的尺寸；调用 `setPixelRatio` 设置 `canvas` 的像素比为当前设备的屏幕像素比，避免高分屏下出现模糊情况。

#### `知识点 💡` 场景 Scene

`Scene` 是场景对象，所有的网格对象、灯光、动画等都需要放在场景中，使用 `new THREE.Scene` 初始化场景，下面是场景的一些常用属性和方法。

*   `fog`：设置场景的雾化效果,可以渲染出一层雾气，隐层远处的的物体。
*   `overrideMaterial`：强制场景中所有物体使用相同材质。
*   `autoUpdate`：设置是否自动更新。
*   `background`：设置场景背景，默认为黑色。
*   `children`：所有对象的列表。
*   `add()`：向场景中添加对象。
*   `remove()`：从场景中移除对象。
*   `getChildByName()`：根据名字直接返回这个对象。
*   `traverse()`：传入一个回调函数访问所有的对象。

#### `知识点 💡` 透视相机 PerspectiveCamera

为了在场景中显示物体，就必须给场景添加相机，相机类型可以分为**正交相机**和**透视相机**，本例中使用的是透视相机 `PerspectiveCamera`，可以像下面这样使用透视相机。

**构造函数**：

```js
PerspectiveCamera(fov, aspect, near, far)

```

*   `fov`：表示视场，就是能够看到的角度范围，人的眼睛大约能够看到 `180度` 的视场，视角大小设置要根据具体应用，一般游戏会设置 `60~90` 度，默认值 `45`。
*   `aspect`：表示渲染窗口的长宽比，如果一个网页上只有一个全屏的 `canvas` 画布且画布上只有一个窗口，那么 `aspect` 的值就是网页窗口客户区的宽高比 `window.innerWidth/window.innerHeight`。
*   `near`：属性表示的是从距离相机多远的位置开始渲染，一般情况会设置一个很小的值。 默认值 `0.1`。
*   `far`：属性表示的是距离相机多远的位置截止渲染，如果设置的值偏小，会有部分场景看不到，默认值 `1000`。

### 页面缩放适配

为了防止画布渲染内容在浏览器尺寸发生变化时产生变形和移位，可以对 `resize` 事件进行监听，当页面发生变化时，同时更新渲染器的尺寸和相机视角比例，并调用 `.updateProjectionMatrix()` 手动更新相机对象的投影矩阵属性。

```js

window.addEventListener('resize', () => {
  sizes.width = window.innerWidth;
  sizes.height = window.innerHeight;
  
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  
  camera.aspect = sizes.width / sizes.height;
  camera.updateProjectionMatrix();
});

```

### 逻辑开发

要创建可加载显示在场景中的内置三维模型，需要添加网格 `Mesh`，并为它创建几何体 `Geometry` 和 材质 `Material`。本示例中添加了一个 `Three.js` 内置的立方体网格模型。

```js
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshBasicMaterial({ color: 0x03c03c });
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

```

#### `知识点 💡` 立方体 BoxGeometry

`BoxGeometry` 是四边形的原始几何类，来创建立方体或者不规则四边形。

**构造函数**：

```js
BoxGeometry(width : Float, height : Float, depth : Float, widthSegments : Integer, heightSegments : Integer, depthSegments : Integer)

```

*   `width`：`X轴` 的宽度，默认为 `1`。
*   `height`：`Y轴` 的高度，默认为 `1`。
*   `depth`：`Z轴` 的深度，默认为 `1`。
*   `widthSegments`：可选，宽度的分段数，默认是 `1`。
*   `heightSegments`：可选，高度的分段数，默认是 `1`。
*   `depthSegments`：可选，深度的分段数，默认是 `1`。

#### `知识点 💡` 基础网格材质 MeshBasicMaterial

基础网格材质是一种一个以简单着色方式来绘制几何体的材质，它不受光照的影响。

**构造函数**：

```js
MeshBasicMaterial(parameters: Object)

```

*   `parameters`：可选，用于定义材质外观的对象，具有一个或多个属性如 `color`、`map` 等。

### 动画更新

完成上面的内容，如果你在浏览器中直接运行，会发现页面不会加载任何东西，因为还没有进行真正的渲染。为此，我们需要调用 `requestAnimationFrame` 在页面重回动画中更新渲染内容。在这里创建了一个 `tick` `动画方法，它的功能是使渲染器可以在每次屏幕刷新时对场景进行绘制的循环，在大多数屏幕上，requestAnimationFrame` 刷新率一般为 `60次/秒`。

```js

const tick = () => {
  
  renderer.render(scene, camera);
  
  mesh && (mesh.rotation.y += .02);
  mesh && (mesh.rotation.x += .02);
  
  window.requestAnimationFrame(tick);
}
tick();

```

### 运行效果

到此，入门示例全部开发完毕了，在浏览器中打开可以看到如下的 `3D` 界面。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14c6bb320c514aee8a0d956de12c4f20~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 提示

因为 `Scene` 默认场景背景是黑色的，而浏览器在常规下默认的背景色是白色的，在 `Mac` 上用鼠标拖拽屏幕时可能会出现露出底部白色的问题，我们可以通过设置以下样式解决此问题。

```css
* {
  margin: 0;
  padding: 0;
}
html, body {
  overflow: hidden;
}
.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}

```

> `🔗` 源码仓库地址：[github.com/dragonir/th…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")

总结
--

本文中主要包含的知识点包括：

*   `Three.js` 的安装和引用
*   `Three.js` 初始化项目搭建
*   开发基于 `Three.js` 项目的基本步骤
*   渲染器 `WebGLRenderer`
*   场景 `Scene`
*   透视相机 `PerspectiveCamera`
*   页面缩放适配
*   立方体 `BoxGeometry`
*   基础网格材质 `MeshBasicMaterial`
*   `requestAnimationFrame` 动画更新

> 想了解其他前端知识或其他未在本文中详细描述的**Web 3D**开发技术相关知识，可阅读我往期的文章。如果有疑问可以在评论中**留言**，如果觉得文章对你有帮助，不要忘了**一键三连哦 👍**。

附录
--

*   [【Three.js 进阶之旅】系列专栏访问 👈](https://juejin.cn/column/7140122697622618119 "https://juejin.cn/column/7140122697622618119")
*   [更多往期【3D】专栏访问 👈](https://juejin.cn/column/7049923956257587213 "https://juejin.cn/column/7049923956257587213")
*   \[1\]. [🌴 Three.js 打造缤纷夏日3D梦中情岛](https://juejin.cn/post/7102215670477094925 "https://juejin.cn/post/7102215670477094925")
*   \[2\]. [🔥 Three.js 实现炫酷的赛博朋克风格3D数字地球大屏](https://juejin.cn/post/7124116814937718797 "https://juejin.cn/post/7124116814937718797")
*   \[3\]. [🐼 Three.js 实现2022冬奥主题3D趣味页面，含冰墩墩](https://juejin.cn/post/7060292943608807460 "https://juejin.cn/post/7060292943608807460")
*   \[4\]. [🏆 掘金1000粉！使用Three.js实现一个创意纪念页面](https://juejin.cn/post/7143039765725020167 "https://juejin.cn/post/7143039765725020167")
*   `...`