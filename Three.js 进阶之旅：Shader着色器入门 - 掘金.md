# Three.js 进阶之旅：Shader着色器入门 - 掘金
> 本文为稀土掘金技术社区首发签约文章，14天内禁止转载，14天后未获授权禁止转载，侵权必究！

摘要
--

本文内容主要介绍 `Three.js` 中的着色器知识，通过讲解什么是着色器、着色器的分类、`GLSL` 语言的核心语法要点、`Three.js` 中的两种着色器材质的 `RawShaderMaterial` 和 `ShaderMaterial` 的区别和用法等基本知识，深入理解着色器，并使用它创建出有趣的三维图形。

本文篇幅较长，涉及到的知识点也比较广，内容可能相对枯燥，有些地方需要耐心思考。我相信通过纵览全文，掌握全文的核心要点，一定会获益匪浅，着色器**入门者**建议收藏起来定期复习`🤣`。

效果
--

随着本文内容一步步深入，最终将使用着色器构建一个如下所示的波动旗帜 `🚩` 效果，通过滑动调整页面右上方的 `dat.GUI` 控制器，可以调整 `x轴` 和 `y轴` 上的波动幅度。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81fbaeb4c6240ebbc0c6c4058c49acb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

打开以下链接中的任意一个，在线预览效果，大屏访问效果更佳。

*   `👁‍🗨` 在线预览地址1：[dragonir.github.io/3d/#/flag](https://link.juejin.cn/?target=https%3A%2F%2Fdragonir.github.io%2F3d%2F%23%2Fflag "https://dragonir.github.io/3d/#/flag")
*   `👁‍🗨` 在线预览地址2：[3d-eosin.vercel.app/#/flag](https://link.juejin.cn/?target=https%3A%2F%2F3d-eosin.vercel.app%2F%23%2Fflag "https://3d-eosin.vercel.app/#/flag")

本专栏系列代码托管在 `Github` 仓库[【threejs-odessey】](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")，**后续所有目录也都将在此仓库中更新**。

> `🔗` 代码仓库地址：[git@github.com:dragonir/threejs-ode…](https://link.juejin.cn/?target=)

码上掘金
----

正文
--

### Shader着色器简介

着色器是 `WebGL` 的重要组件之一，它是一种使用 `GLSL` 语言编写的运行在 `GPU` 上的程序。顾名思义，着色器用于定位几何体的每个顶点，并为几何体的每个可见像素进行着色 `🎨`。着色器是屏幕上呈现画面之前的最后一步，用它可以实现对先前渲染结果进行修改，如颜色、位置等，也可以对先前渲染的结果做后处理，实现高级的渲染效果。

例如，对于相同场景、相同光照、相同模型等条件下，对这个模型分别使用不同的着色器，就会呈现出完全不同的渲染效果：使用 `plastic shader` 的模型渲染出塑料质感，而使用了 `toon shader` 的模型则看起来是二维卡通效果。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5a37a1dbf99455b911aa463d850df11~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 为什么要使用着色器

虽然 `Three.js` 已经内置了非常多的材质，但是在实际开发中很难满足我们的需求，比如在数字孪生系统的开发中，我们经常需要添加一些炫酷的**飞线效果**、**雷达效果**等 `✨`，它们是无法直接使用 `Three.js` 来生成，此时就需要我们创建自己的着色器。而且出于性能的考虑，我们也可以使用自己的着色器材质代替像 `MeshStandardMaterial` 这样的材质非常精细涉及大量代码和计算的材质，以便于提升页面性能。

### 着色器的类型

#### 顶点着色器Vertex Shader

`Vertex Shader` 用于定位几何体的顶点，它的工作原理是发送顶点位置、网格变换（`position`、旋`rotation`和 `scale` 等）、摄像机信息（`position`、`rotation`、`fov` 等）。`GPU` 将按照 `Vertex Shader` 中的指令处理这些信息，然后将顶点投影到 `2D` 空间中渲染成 `Canvas`。

当使用 `Vertex Shader` 时，它的代码将作用于几何体的每个顶点。在每个顶点之间，有些数据会发生变化，这类数据称为 `attribute`；有些数据在顶点之间永远不会变化，称这种数据为 `uniform`。`Vertex Shader` 会首先触发，当顶点被放置，`GPU` 知道几何体的哪些像素可见，然后执行 `Fragment Shader`。

*   `attribute`：使用顶点数组封装每个顶点的数据，一般用于每个顶点都各不相同的变量，如顶点的位置。
*   `uniform`：顶点着色器使用的常量数据，不能被修改，一般用于对同一组顶点组成的单个 `3D` 物体中所有顶点都相同的变量，如当前光源的位置。

#### 片元着色器Fragment Shader

`Fragment Shader` 在 `Vertex Shader` 之后执行，它的作用是为几何体的每个可见像素进行着色。我们可以通过`uniforms` 将数据发送给它，也可以将 `Vertex Shader` 中的数据发送给它，我们将这种从 `Vertex Shader` 发送到 `Fragment Shader` 的数据称为 `varying`。

`Fragment Shader` 中最直接的指令就是可以使用相同的颜色为所有像素进行着色。如果只设置了颜色属性，就相当于得到了与 `MeshBasicMaterial` 等价的材质。如果我们将光照的位置发送给 `Fragment Shader`，然后根据像素收到光照影响的多少来给像素上色，此时就能得到与 `MeshPhongMaterial` 效果等价的材质。

*   `varying`: 从顶点着色器发送到片元着色器中的插值计算数据

> `📌` 以下内容示例流程翻译、并整理于[《three.js journey》](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs-journey.com%2F "https://threejs-journey.com/") shader 相关课程，如果对英文原版感兴趣可前往查看。

### 原始着色器材质RawShaderMaterial

在 `Three.js` 中可以渲染着色器的材质有两种：`RawShaderMaterial` 和 `ShaderMaterial`，它们之间的区别是 `ShaderMaterial` 会自动将一些初始化着色器的参数添加到代码中（内置 `attributes` 和 `uniforms`），而 `RawShaderMaterial` 则什么都不会添加。

我们先来看看如何使用 `RawShaderMaterial` 材质，首先我们创建一个平面，然后和创建其他材质一样，通过 `new THREE.RawShaderMaterial` 初始化原始着色器材质，并给它添加两个参数 `vertexShader` 和 `fragmentShader` 代表材质的**顶点着色器**和**片元着色器**。

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: '',
  fragmentShader: ''
})

```

然后开始编写材质的顶点着色器和片元着色器，分别添加如下的代码。

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: `
    uniform mat4 projectionMatrix;
    uniform mat4 viewMatrix;
    uniform mat4 modelMatrix;

    attribute vec3 position;

    void main() {
      gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    precision mediump float;

    void main(){
      gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
    }
  `
});

```

此时可以得到一个**红色**的平面，说明我们编写的第一个着色器运行成功了 `🎉`。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6053ce1ab2eb472fb9bdc1148a8948bd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 分离两种着色器

在实际开发中，着色器比较复杂，代码量比较多，如果直接放在材质中的话会增加代码阅读困难量。我们可以将着色器代码单独拆分出来，分别存放在 `vertex.glsl` 和 `fragment.glsl` 文件中，然后在代码中像下面这样引入即可。这样做还有一个好处就是可以安装代码编辑器的 `GLSL` 高亮语法插件，提高编程效率。

```js
import testVertexShader from './shaders/test/vertex.glsl';
import testFragmentShader from './shaders/test/fragment.glsl';

const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader
});

```

此时查看页面，得到的结果还是一样的。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca1bae7507474b0284e115210e375195~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 属性

材质的一些通用属性在 `RawShaderMaterial` 同样是适用的，比如 `wireframe`、`side`、`transparent`、`flatShading` 等都可以生效，对上面材质开启 `wireframe` 属性，可以得到如下图所示的平面的网格模型。

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  wireframe: true
});

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc969074096f4cfdb7c89139d72b65a7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> `📌` 但是需要注意的是，像map、alphaMap、opacity、color等属性在着色器材质中会失效，我们需要在着色器代码中自己实现。

### GLSL 语言

在 `Three.js` 中，需要使用 `GLSL` 语言来编写着色器，全称是 `OpenGL Shading Language`，意为 `OpenGL` 中的着色语言。它的语法类似于 `C语言`，在开始编写着色器之前，我们先了解一些它的基本语法。

*   **日志**：由于着色器语言是针对每个顶点和每个片元执行的，日志记录是没有意义的，因此编写 `GLSL` 时没有控制台。
*   **缩进**：代码缩进格式**没有严格要求**，只要易读美观就行。
*   **分号**：和 `C语言` 一样，编写 `GLSL` 语言时，任何指令的结尾都**必须添加分号**，丢失分号就会导致代码无法运行。
*   **类型**：和 `C语言` 一样， `GLSL` 是一种强类型语言，**不同类型的变量不能混用**，否则会报错。

#### 变量

在 `GLSL` 中有很多变量类型，编写着色器时，我们需要根据需要选择合适类型的变量。

##### 整型

用以定义整数。

```glsl
int foo = 123;
int bar = - 1;

```

##### 浮点类型

浮点数就是小数，可以是正数也可以是负数，必须提供小数点 `.`。

```glsl
float foo = - 0.123;
float bar = 1.0;

```

##### 布尔类型

用于表示值得真假。

```glsl
bool foo = true;
bool bar = false;

```

##### 二维向量vec2

如果我们需要存储具有 `x` 和 `y` 属性这样具有2个坐标的值时，可以使用 `vec2`。需要注意的是，直接使用 `vec2 foo = vec2()` 这样未添加参数的空值会报错，应该像下面这样提供完整的参数：

```glsl
vec2 foo = vec2(1.0, 2.0);

```

创建 `vec2` 后修改属性值：

```glsl
vec2 foo = vec2(0.0);
foo.x = 1.0;
foo.y = 2.0;

```

进行浮点数与 `vec2` 相乘等操作运算时，结果将同时作用于 `x` 和 `y`：

```glsl
vec2 foo = vec2(1.0, 2.0);
foo *= 2.0;

```

##### 三维向量vec3

与 `vec2` 类似，`vec3` 用于表示具有 `x`、`y`、`z` 三个坐标的值，可以用它非常方便的表示**三维空间坐标**。

```glsl
vec3 foo = vec3(0.0);
vec3 bar = vec3(1.0, 2.0, 3.0);
bar.z = 4.0;

```

`RGB` 颜色也同样适合使用 `vec3` 表示：

```glsl
vec3 color = vec3(0.0);
color.r = 0.5;
color.b = 1.0;

```

可以使用 `vec2` 来创建 `vec3`：

```glsl
vec2 foo = vec2(1.0, 2.0);
vec3 bar = vec3(foo, 3.0);

```

也可以使用 `vec3` 来创建 `vec2`，其中 `bar` 的值为 `1.0, 2.0`，`baz` 的 值为 `2.0, 1.0`：

```glsl
vec3 foo = vec3(1.0, 2.0, 3.0);
vec2 bar = foo.xy;
vec2 baz = foo.yx;

```

##### 四维向量vec4

与前面几个类似，`vec4` 用于表示四维向量，四个值命名为 `x, y, z, w` 或 `r, g, b, a`，向量之间同样能进行相互转换：

```glsl
vec4 foo = vec4(1.0, 2.0, 3.0, 4.0);
vec4 bar = vec4(foo.zw, vec2(5.0, 6.0));

```

除上述之外，还有一些其它类型的变量，如 `mat2`、`mat3`、`mat4`、`sampler2D` 等将在后续学习中介绍。

*   在着色器内，一般命名以 `gl_` 开头的变量是着色器的内置变量。
*   `webgl_` 和 `_webgl` 是着色器保留字，自定义变量不能以 `webgl_` 或 `_webgl` 开头。
*   变量声明一般包含 `<存储限定符> <数据类型> <变量名称>`，以 `attribute vec4 a_Position` 为例，`attribute` 表示存储限定符，`vec` 是数据类型，`a_Position` 为变量名。

#### 函数

在 `GLSL` 中定义函数，必须以返回值的类型开头，如果没有返回值，则可以使用 `void`。定义函数的参数时，也必须提供参数类型。

```glsl
// 有返回值
float loremIpsum() {
  float a = 1.0;
  float b = 2.0;
  return a + b;
}
// 无返回值
void justDoingStuff() {
  float a = 1.0;
  float b = 2.0;
}
// 定义参数类型
float add(float a, float b) {
  return a + b;
}

```

##### 内置函数

`GLSL` 内置了很多使用的函数，下面列举了一些比较常用的：

*   **运算函数**
    *   `abs(x)`：取 `x` 的绝对值
    *   `radians(x)`：角度转弧度
    *   `degrees(x)`：弧度转角度
    *   `sin(x)`：正弦函数，传入值为弧度。还有 `cos` 余弦函数、`tan` 正切函数、`asin` 反正弦、`acos`反余弦、`atan` 反正切等
    *   `pow(x,y)`：`x^y`
    *   `exp(x)`：`e^x`
    *   `exp2(x)`：`2^x`
    *   `log(x)`：`logex`
    *   `log2(x)`：`log2x`
    *   `sqrt(x)`：`x√`
    *   `inversesqr(x)`：`1x√`
    *   `sign(x)`：`x>0` 返回 `1.0`，`x<0` 返回 `-1.0`，否则返回 `0.0`
    *   `ceil(x)`：返回大于或者等于 `x` 的整数
    *   `floor(x)`：返回小于或者等于 `x` 的整数
    *   `fract(x)`：返回 `x-floor(x)` 的值
    *   `mod(x,y)`：取模求余数
    *   `min(x,y)`：获取 `x`、`y` 中小的那个
    *   `max(x,y)`：获取 `x`、`y` 中大的那个
    *   `mix(x,y,a)`：返回 `x∗(1−a)+y∗a`
    *   `step(x,a)`：`x<a`返回 `0.0`，否则返回 `1.0`。
    *   `smoothstep(x,y,a)`：`a<x` 返回 `0.0`，`a>y` 返回 `1.0`，否则返回 `0.0-1.0` 之间平滑的 `Hermite` 插值。
    *   `dFdx(p)`：`p` 在 `x` 方向上的偏导数
    *   `dFdy(p)`：`p` 在 `y` 方向上的偏导数
    *   `fwidth(p)`：`p` 在 `x` 和 `y` 方向上的偏导数的绝对值之和
*   **几何函数**
    *   `length(x)`：计算向量 `x` 的长度
    *   `distance(x, y)`：返回向量 `xy` 之间的距离
    *   `dot(x,y)`：返回向量 `xy` 的点积
    *   `cross(x,y)`：返回向量 `xy` 的差积
    *   `normalize(x)`：返回与 `x` 向量方向相同，长度为 `1` 的向量
*   **矩阵函数**
    *   `matrixCompMult(x,y)`：将矩阵相乘
    *   `lessThan(x,y)`：返回向量 `xy` 的各个分量执行 `x<y` 的结果
    *   lessThanEqual(x,y)：返回向量 `xy` 的各个分量执行 `x<=y` 的结果，类似的有类似的有 `greaterThanEqual`
    *   `any(bvec x)`：`x` 有一个元素为 `true`，则为 `true`
    *   `all(bvec x)`：`x` 所有元素为 `true`，则返回 `true`，否则返回 `false`
    *   `not(bvec x)`：`x` 所有分量执行逻辑非运算

> `🔗` 如果想了解更多GLSL的内置函数，可以到这个网站查询：[Kronos Group OpenGL reference pages](https://link.juejin.cn/?target=https%3A%2F%2Fwww.khronos.org%2Fregistry%2FOpenGL-Refpages%2Fgl4%2Fhtml%2Findexflat.php "https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/indexflat.php")

### 理解顶点着色器Vertex Shader

接下来讲解着色器里代码的具体内容。

**顶点着色器**的作用是将几何体的每个顶点放置在 `2D` 渲染空间上，即顶点着色器将 `3D` 顶点坐标转换为 `2D` `canvas` 坐标。

#### main函数

它将被自动调用，并且不会返回任何内容。

```glsl
void main() {}

```

#### gl_Position

`gl_Position` 是一个内置变量，我们只需要给它重新赋值就能使用，它将会包含屏幕上的顶点的位置。下面 `main` 函数中就是用于给它设置合适的值。执行这段指令后，将得到一个 `vec4`，意味着我们可以直接在 `gl_Position` 变量上使用其`x`、`y`、`z` 和 `w` 属性。

```glsl
void main() {
  gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0);
  gl_Position.x += 0.5;
  gl_Position.y += 0.5;
}

```

平面向右上角发生了位移，但是需要注意的是，我们并没有像在 `Three.js` 中一样将平面在三维空间中进行了移动，我们只是在二维空间中移动了平面的投影。就像你在桌子上画了一幅具有透视效果的画，然后把它向桌子右上角移动，但是你的画中的透视效果并没有发生变化。

`gl_Position` 的作用是在 `2D` 空间上定位 `📍` 顶点，既然是 `2D` 空间，为什么需要使用一个四维向量表示呢？实际上是这些坐标并不是精确的在 `2D` 空间，而是位于被称为 `Clip Space` 需要四个维度的**裁切空间**。裁切空间是指在 `-1` 到 `+1` 范围内所有 `x`、`y`、`z` `3`个方向上的空间，第四个值 `w` 用于表示透视。就像把所有东西都放在 `3D` 盒子中一样，任何超出范围的内容都将被裁切。`gl_Position` 这些内容的这些内容都是自动完成的，我们只需明白其大概原理即可。

#### 位置属性Position attributes

相同的代码将应用于几何体的每一个顶点，属性变量 `attribute` 是在顶点之间唯一会发生改变的变量。相同的顶点着色器 `Vertex Shader` 将应用于每一个顶点，`position` 属性将包含具体顶点的 `x, y, z` 坐标值。我们可以使用如下代码获取顶点位置：

```glsl
attribute vec3 position;

```

因为 `gl_Position` 是 `vec4` 类型，可以使用以下方法将 `vec3` 转化成 `vec4`：

```glsl
gl_Position = /* ... */ vec4(position, 1.0);

```

#### 矩阵限定变量Matrices uniforms

每个矩阵将转换 `position`，直到我们获得最终的裁切空间坐标。下面是 `3` 个矩阵，因为在几何体所有顶点中它们的值都是相同的，我们可以通过 `uniform` 来获取它们。

```glsl
uniform mat4 projectionMatrix;
uniform mat4 viewMatrix;
uniform mat4 modelMatrix;

```

下面将对每个矩阵做出一些变换：

*   `modelMatrix`：将进行网格相关的变换，如缩放、旋转、移动等操作变换都将作用于 `position`。
*   `viewMatrix`：将进行相机相关的变换，如我们向左移动相机，顶点应该在右边、如果我们朝着网格方向移动相机，顶点会变大等。
*   `projectionMatrix`：会将我们的坐标转化为裁切空间坐标。

为了使用矩阵，我们需要将其相乘，如果想让一个 `mat4` 作为变量，则该变量类型必须是 `vec4`。我们也可以将多个矩阵相乘：

```glsl
gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0);

```

实际上还可以使用更短的写法来让 `viewMatrix` 和 `modelMatrix` 组合成一个 `projectionMatrix`，虽然代码少了，但我们可控制的步骤也少了。

```glsl
uniform mat4 projectionMatrix;
uniform mat4 modelViewMatrix;
attribute vec3 position;

void main(){
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}

```

实际中我们会选择更长的写法，以便于更好地理解及对 `position` 进行更多的控制。

```glsl
uniform mat4 projectionMatrix;
uniform mat4 viewMatrix;
uniform mat4 modelMatrix;
attribute vec3 position;

void main(){
  vec4 modelPosition = modelMatrix * vec4(position, 1.0);
  vec4 viewPosition = viewMatrix * modelPosition;
  vec4 projectedPosition = projectionMatrix * viewPosition;~~~~
  gl_Position = projectedPosition;
}

```

上面两种写法都是等价的，使用下面这种时，我们可以更方便地进行控制，比如可以通过调整 `modelPosition` 的值来对整个模型进行移动，通过以下代码，就能向上移动模型：

```glsl
void main() {
  vec4 modelPosition = modelMatrix * vec4(position, 1.0);
  modelPosition.y += 1.0;
  // ...
}

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/271638193fe04f56adaef64756bc535e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

我们还可以做一些更有趣的操作，比如将平面变换为**波浪形状**：

```glsl
void main() {
  vec4 modelPosition = modelMatrix * vec4(position, 1.0);
  modelPosition.z += sin(modelPosition.x * 10.0) * 0.1;
  // ...
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a689a90d2584939a8129bfee8a0e5b6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 理解片元着色器Fragment Shader

**片元着色器**的代码将应用于几何体的每个可见像素，这就是片元着色器在顶点着色器之后运行的原因，它的代码比顶点着色器更易于管理。

#### 主函数main

同样，片元着色器中也有一个主函数：

```glsl
void main() {}

```

#### 精度Precision

在顶部有一条这样的指令，我们用它来决定浮点数的精度，有以下几种值供选择：

*   `highp`：会影响性能，在有些机器上可能无法运行；
*   `mediump`：常用的类型；
*   `lowp`：可能会由于精度问题产生错误。

```glsl
precision mediump float;

```

我们现在示例使用的是 `RawShaderMaterial` 原始着色器材质才需要设置精度，在着色器材质 `ShaderMaterial`中会自动处理。

> 在顶点着色器中也可以是指精度，但是这是非必须的。

#### gl_FragColor

`gl_FragColor` 和 `gl_Position` 类似，但它用于颜色。它也一样是已经被内置声明了的，我们只需要在`main` 函数中重新给它赋值。它是一个 `vec4`，前三个值是红色、绿色、蓝色通道 `(r, g, b)`，第四个值是透明度 `alpha` `(a)`。`gl_FragColor` 的每个值的取值范围是 `0.0` 到 `1.0`，如果我们设置的值高于它们，也不会产生报错。

下面这段代码将生成一个**紫色**的几何体

```glsl
gl_FragColor = vec4(0.5, 0.0, 1.0, 1.0);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ce60555035f4a04b586686c8859372b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

为了 `alpha` 透明度值可以生效，我们需要在材质中将 `transparent` 属性设置为 `true`：

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  transparent: true
})

```

### 属性Attributes

`Attributes` 是每个顶点之间变化的值，我们之前已经有一个命名为 `position` 的属性变量，它是每个顶点在坐标轴中的 `vec3` 值。我们将为每个顶点添加一个随机值，并根据这个值在 `z` 轴上移动该顶点。在 `JavaScript` 代码中我们可以像下面这个直接给 `BufferGeometry` 添加 `attribute` 属性。然后再创建一个 `32位` 的浮点类型数组 `Float32Array`，为了知道几何体中有多少个顶点，现在可以通过 `attributes` 属性获取。最后在 `BufferAttribute` 中使用该数组，并将它添加到几何体的属性中。

*   `setAttribute`：第一个参数是需要设置的 `attribute` **属性名称**，然后在着色器中可以使用该名字，属性名命名时最好加一个 `a` 前缀方便区分。
*   `BufferAttribute`：第一个参数是**数据数组**；第二个参数表示组成一个属性的值的数量，如我们要发送一个 `(x, y, z)` 构成位置，则需要使用 `3`，示例中每个顶点的随机数只有 `1个`，因此这个参数使用 `1`。

```js
const geometry = new THREE.PlaneBufferGeometry(1, 1, 32, 32)
const count = geometry.attributes.position.count
const randoms = new Float32Array(count)

for(let i = 0; i < count; i++) {
  randoms[i] = Math.random()
}

geometry.setAttribute('aRandom', new THREE.BufferAttribute(randoms, 1))

```

现在，我们可以在**顶点着色器**中获取该属性，并使用它移动顶点，可以得到一个如下图所示的一个由**随机尖峰**构成的平面。

```glsl
attribute float aRandom;

void main(){
  // ...
  modelPosition.z += aRandom * 0.1;
  // ...
}

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59315c111b714f908f413206964d2d41~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 限定变量Varyings

现在我们若想在**片元着色器**中想使用 `aRandom` 属性给片元着色，是无法直接使用 `attribute` 属性变量的。此时，实现这个功能的方法就是将这个值从**顶点着色器发送到片元着色器**，称这种变量为 `varying`。我们需要在两种着色器中都做如下的操作：

在**顶点着色器**中，我们需要在 `main` 函数之前创建 `varying`，将其命名为以 `v` 作为前缀的变量名 `vRandom`，然后在 `main` 函数中给它赋值：

```glsl
varying float vRandom;

void main() {
  // ...
  vRandom = aRandom;
}

```

在**片元着色器**中，使用相同的方法声明，然后在 `main` 函数中使用它，可以得到如下的染色效果：

```glsl
precision mediump float;
varying float vRandom;

void main() {
  gl_FragColor = vec4(0.5, vRandom, 1.0, 1.0);
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38e958d81aa548e8ba17ce10e7717963~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> `📌` varying的一个有趣之处是，顶点之间的值是线性插值的，如GPU在两个顶点之间绘制一个片元，一个顶点的varying是1.0，另一个顶点的varying是0.0，则该片元值将为0.5。这个特性可以实现平滑的渐变效果。

现在我们先移除上面所有的效果，恢复到纯紫色的平面。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e2e08a8dabb4fdeb057d9aa9368b2cd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 统一变量Uniforms

`uniform` 用于将数据从 `JavaScript` 发送到 `着色器`。如果我们使用同一个着色器但是参数不同时就可以使用 `uniform`，使用期间参数还可以改变。在**顶点着色器** 和 `片元着色器` 中都可以使用 `uniform`，它的值在每个顶点和每个片元中的数据都是相同的。实际上在我们的代码中已经有 `projectionMatrix`、`viewMatrix`、`modelMatrix` 等 `uniform`，`Three.js` 内置创建了它们。

现在，我们来创建自己的 `uniform`。为了将统一变量添加到材质中，需要使用 `uniforms` 属性。我们将创建一个波动的平面，并使用变量来控制波浪的频率。下面用于控制 **频率** 的变量命名为 `uFrequency`，特地加了一个 `u` 字符作为前缀来标识是 `uniform` 变量，方便在着色器中和其他参数区分开来，但这不是强制的。

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  uniforms: {
    uFrequency: { value: 10 }
  }
})

```

然后，可以在着色器代码中获取 `uniform` 值，并在 `main` 函数中使用它：

```glsl
uniform mat4 projectionMatrix;
uniform mat4 viewMatrix;
uniform mat4 modelMatrix;
uniform float uFrequency;

attribute vec3 position;

void main() {
    // ...
    modelPosition.z += sin(modelPosition.x * uFrequency) * 0.1;
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78f68b7804b348f78866b29cb76e4328~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

显示结果和前面的相同，但是现在我们可以在 `JavaScript` 来控制频率了。我们可以把频率 `frequency` 改成 `vec2` 来控制水平和垂直方向的波动，在 `Three.js` 中可以使用二维向量 `THREE.Vector2`：

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  uniforms: {
    uFrequency: { value: new THREE.Vector2(10, 5) }
  }
})

```

然后在着色器中，将 `uFrequency` 的类型从 `float` 改为 `vec2` 并在 `z轴` 同时应用 `uFrequency` 的 `x值` 和 `y值`，此时我们的模型网格就会同时产生在**水平**和**垂直**方向的波动：

```glsl
// ...
uniform vec2 uFrequency;

void main() {
  // ...
  modelPosition.z += sin(modelPosition.x * uFrequency.x) * 0.1;
  modelPosition.z += sin(modelPosition.y * uFrequency.y) * 0.1;
  // ...
}

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3da9e414ee2496f83df7c38950c2f6e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

[《Three.js 进阶之旅：神奇的粒子系统-迷失太空 👨‍🚀》](https://juejin.cn/post/7155278132806123557 "https://juejin.cn/post/7155278132806123557")一文中我们已经了解过 `dat.GUI` 的用法，现在我们可以使用它动态修改 `uFrequency` 的值在页面上实时查看不同参数生成的波动效果。

```js
gui.add(material.uniforms.uFrequency.value, 'x').min(0).max(20).step(0.01).name('frequencyX');
gui.add(material.uniforms.uFrequency.value, 'y').min(0).max(20).step(0.01).name('frequencyY');

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbadceb33e0e4981ad8bcf92e71bde68~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

让我们再新加一个 `uniform` 来让平面像在风中飘动的旗帜。我们将使用统一变量 `uTime` 向着色器发送一个时间值，然后在 `sin(...)` 函数中使用它：

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  uniforms: {
    uFrequency: { value: new THREE.Vector2(10, 5) },
    uTime: { value: 0 }
  }
})

```

不要忘了在 `tick` 页面重绘函数中更新 `uTime`，我们使用 `getElapsedTime` 来获取已经花费了多少时间:

```js
const tick = () => {
  const elapsedTime = clock.getElapsedTime();
  material.uniforms.uTime.value = elapsedTime;
  
}

```

然后在着色器中获取 `uTime` 并在 `sin(...)` 函数中使用它，我们的平面就会看起来像一个在风中飘动的旗帜 `🚩`：

```glsl
// ...
uniform float uTime;

void main() {
  modelPosition.z += sin(modelPosition.x * uFrequency.x + uTime) * 0.1;
  modelPosition.z += sin(modelPosition.y * uFrequency.y + uTime) * 0.1;
  // ...
}

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3d62258a7dc44a3af6c443c63d00f2c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

我们也可以将 `uTime` 之前的 `+` 改为 `-` 来修改波动的方向。

```glsl
modelPosition.z += sin(modelPosition.x * uFrequency.x - uTime) * 0.1;
modelPosition.z += sin(modelPosition.y * uFrequency.y - uTime) * 0.1;

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ffac009ffbf4d699f22b0f9d294216d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> `📌` 注意，使用uTime时如果直接使用JavaScript的Date.now()，会发现不起作用，因为它的数值对于着色器而言太过庞大，我们不能发送太小或太大的统一变量值。

虽然现在网格模型具有波动效果，但是它仍然是一个平面网格构成，我们可以修改它的属性来使它看起来更像个旗子 `🚩`。我们可以修改它的大小比例：

```js
const mesh = new THREE.Mesh(geometry, material);
mesh.scale.y = 2 / 3;

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75f4916ccb74452db3a55cdcc0365c74~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在片元着色器中也可以使用 `uniform` 统一变量，我们添加一个 `uColor` 作为颜色变量：

```js
const material = new THREE.RawShaderMaterial({
  vertexShader: testVertexShader,
  fragmentShader: testFragmentShader,
  uniforms: {
    uFrequency: { value: new THREE.Vector2(10, 5) },
    uTime: { value: 0 },
    uColor: { value: new THREE.Color('orange') }
  }
})

```

然后在片元着色器中获取颜色变量，并将它作为 `gl_FragColor` 的值，你会看到平面将变成设定的颜色效果：

```glsl
precision mediump float;
uniform vec3 uColor;

void main() {
  gl_FragColor = vec4(uColor, 1.0);
}

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c423a48582304e31994e60cc04e7b0a6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 纹理Textures

`Textures` 知识比较复杂，在之前的文章中已经介绍过使用 `THREE.TextureLoader` 加载纹理，下面我们给着色器材质添加一个图片纹理，并使用 `uTexture` 统一变量传递给着色器：

```js
const material = new THREE.RawShaderMaterial({
  
  uniforms: {
    
    uTexture: { value: textureLoader.load('/textures/flag.png') }
  }
})

```

然后在着色器中，为了使纹理的颜色应用于每个可见片元上，我们需要使用 `texture2D(...)`，它接收两个参数，第一个是需要应用的纹理即 `uTexture`，第二个是纹理上拾取颜色的坐标系，这个坐标系其实就是前面讨论的 `UV坐标`，它的作用是将纹理坐标投射到几何体上。我们用于创建几何体的 `PlaneBufferGeometry` 会自动生成这个坐标，我们可以通过 `geometry.attributes.uv` 来查看它。`texture2D(...)` 的返回结果是一个由 `r, g, b, a` 构成的 `vec4`。

因为 `uv` 是一个 `attribute` 属性，因此需要在**顶点着色器**中需要这样获取它，我们需要在片元着色器中使用它，因此还需要通过 `varying` 发送到片元着色器，并在 `main` 函数中更新它：

```glsl
// ...
attribute vec2 uv;
varying vec2 vUv;

void main() {
  // ...
  vUv = uv;
}

```

现在，我们可以在**片元着色器**中获取 `vUv` 变量，并在 `texture2D(...)`方法中使用它：

```glsl
precision mediump float;

uniform vec3 uColor;
uniform sampler2D uTexture;

varying vec2 vUv;

void main() {
  vec4 textureColor = texture2D(uTexture, vUv);
  gl_FragColor = textureColor;
}

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00a8132012cd4978996ffa8818cd2a72~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 颜色变化

现在虽然有了图片贴图，但是旗子 `🚩` 的明暗颜色变化还不太明显，下面我们将为它添加一些阴影变化。

首先在**顶点着色器**中，我们将把风的高程存储 `elevation` 变量中，然后通过 `varying` 发送到**片元着色器**：

```glsl
varying float vElevation;

void main() {
  // ...
  float elevation = sin(modelPosition.x * uFrequency.x - uTime) * 0.1;
  elevation += sin(modelPosition.y * uFrequency.y - uTime) * 0.1;
  modelPosition.z += elevation;
  // ...
  vElevation = elevation;
}

```

然后在**片元着色器**中获取 `vElevation`，用它来改变 `textureColor` 的 `r, g, b`属性：

```glsl
// ...
varying float vElevation;

void main() {
  vec4 textureColor = texture2D(uTexture, vUv);
  textureColor.rgb *= vElevation * 2.0 + 0.5;
  gl_FragColor = textureColor;
}

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42809664ea7e459383fefe98b3557655~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 着色器材质ShaderMaterial

上面所有内容，为了深入理解着色器的原理，我们使用的是 `RawShaderMaterial`，接下来我们使用**更简单**的 `ShaderMaterial` 来重构上面完成的所有功能。 `ShaderMaterial` 和 `RawShaderMaterial` 的工作原理其实是一样的，只不过其内置 `attributes` 和 `uniforms`，`精度` 也会自动设置。我们只需按下面流程稍加修改代码即可。

在 `JavaScript` 代码中将材质换为 `THREE.ShaderMaterial`。

```js
const material = new THREE.ShaderMaterial({});

```

然后**删除**着色器中以下属性和定义：

*   `uniform mat4 projectionMatrix;`
*   `uniform mat4 viewMatrix;`
*   `uniform mat4 modelMatrix;`
*   `attribute vec3 position;`
*   `attribute vec2 uv;`
*   `precision mediump float;`

### 其他

*   查错：因为着色器是对每个片元执行，因此没有日志记录，出错的话很难查找，如果我们忘写了分号，`Three.js` 会将整个着色器代码打印出来并会提示出错的行号；
*   调试：调试数值的一种方法是可以在 `gl_FragColor` 中使用它，虽然不够精确，但是可以看到颜色变化；
*   `GLSLify`：一个 `node module` 模块，可以改对 `glsl` 文件的处理，通过 `glslify` 我们可以像模块一样导入和导出 `glsl` 代码。你可以使用 `glslify-loader` 并将其加到 `webpack` 配置中。
*   拓展阅读：[The Book of Shaders](https://link.juejin.cn/?target=https%3A%2F%2Fthebookofshaders.com%2F "https://thebookofshaders.com/")、[ShaderToy](https://link.juejin.cn/?target=https%3A%2F%2Fwww.shadertoy.com%2F "https://www.shadertoy.com/")

> `🔗` 源码地址：[github.com/dragonir/th…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")

总结
--

本文中主要包含的知识点包括：

*   了解什么是着色器
*   了解为什么要使用着色器
*   `GLSL` 语言的基本语法规则
*   理解 `Vertex Shader` 顶点着色器
*   理解 `Fragment Shader` 片元着色器
*   掌握 `Attributes`、`Varyings`、`Uniforms`的区别和用法
*   着色器在两种着色器材质 `RawShaderMaterial` 和 `ShanderMaterial` 中的使用方法
*   使用着色器设置颜色和纹理等

> 想了解其他前端知识或其他未在本文中详细描述的**Web 3D**开发技术相关知识，可阅读我往期的文章。如果有疑问可以在评论中**留言**，如果觉得文章对你有帮助，不要忘了**一键三连哦 👍**。

附录
--

*   \[1\]. [🌴 Three.js 打造缤纷夏日3D梦中情岛](https://juejin.cn/post/7102215670477094925 "https://juejin.cn/post/7102215670477094925")
*   \[2\]. [🔥 Three.js 实现炫酷的赛博朋克风格3D数字地球大屏](https://juejin.cn/post/7124116814937718797 "https://juejin.cn/post/7124116814937718797")
*   \[3\]. [🐼 Three.js 实现2022冬奥主题3D趣味页面，含冰墩墩](https://juejin.cn/post/7060292943608807460 "https://juejin.cn/post/7060292943608807460")
*   \[4\]. [🦊 Three.js 实现3D开放世界小游戏：阿狸的多元宇宙](https://juejin.cn/post/7081429595689320478 "https://juejin.cn/post/7081429595689320478")
*   \[5\]. [🏆 掘金1000粉！使用Three.js实现一个创意纪念页面](https://juejin.cn/post/7143039765725020167 "https://juejin.cn/post/7143039765725020167")
*   `...`
*   [【Three.js 进阶之旅】系列专栏访问 👈](https://juejin.cn/column/7140122697622618119 "https://juejin.cn/column/7140122697622618119")
*   [更多往期【3D】专栏访问 👈](https://juejin.cn/column/7049923956257587213 "https://juejin.cn/column/7049923956257587213")
*   [更多往期【前端】专栏访问 👈](https://juejin.cn/column/7021076460089638926 "https://juejin.cn/column/7021076460089638926")

参考
--

*   \[1\]. [three.js journey](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs-journey.com%2F "https://threejs-journey.com/")
*   \[2\]. [threejs.org](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs.org "https://threejs.org")
*   \[3\]. 《Three.js 开发指南——基于WebGL和HTML5在网页上渲染3D图形和动画》