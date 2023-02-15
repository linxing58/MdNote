# Three.js 进阶之旅：Shader着色器基础图案-旷野之息神庙铁球 🔮 - 掘金
> 本文为稀土掘金技术社区首发签约文章，14天内禁止转载，14天后未获授权禁止转载，侵权必究！

摘要
--

专栏上一篇文章[《Three.js 进阶之旅：Shader着色器入门》](https://juejin.cn/post/7158032481302609950 "https://juejin.cn/post/7158032481302609950")主要介绍了着色器的基本原理和使用语法知识点，本文内容主要是 `Three.js` 着色器的实践和应用，将在上篇文章的基础上通过 `50个` 着色器简易图案例子，一步步理解使用着色器创建图案的步骤和技巧。最后，将使用绘制的着色器图案，构建一个类似于《**塞尔达传说：旷野之息**》游戏中的**神庙铁球**效果。

通过本文的阅读，你将学习到的知识点包括：使用着色器绘制基础图案、应用着色器作为网格模型的材质纹理、学会后期渲染的基本流程，并使用辉光效果创建发光物体等。本文内容篇幅较长，阅读和学习全部内容可能需要花费较长时间，**建议收藏**起来以备后续查看，`收藏=学会` `🤣`。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dad66f3d893e46839b3c461697bb254c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

效果
--

本文代码实现内容将使用着色器绘制 `50个` 基本图案，下图所示是使用着色器绘制的最后一个图案效果，然后使用这种图案作为材质构建了一个类似神庙铁球的**发光球体** `🔮`。在代码中解开每个图案的注释，即可在浏览器查看使用着色器材质的效果。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4592de09585e4372b96e3a09b2b0e56e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

打开以下链接中的任意一个，在线预览效果，大屏访问效果更佳。

*   `👁‍🗨` 在线预览地址1：[dragonir.github.io/3d/#/shader…](https://link.juejin.cn/?target=https%3A%2F%2Fdragonir.github.io%2F3d%2F%23%2FshaderPattern "https://dragonir.github.io/3d/#/shaderPattern")
*   `👁‍🗨` 在线预览地址2：[3d-eosin.vercel.app/#/shaderPat…](https://link.juejin.cn/?target=https%3A%2F%2F3d-eosin.vercel.app%2F%23%2FshaderPattern "https://3d-eosin.vercel.app/#/shaderPattern")

本专栏系列代码托管在 `Github` 仓库[【threejs-odessey】](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")，**后续所有目录也都将在此仓库中更新**。

> `🔗` 代码仓库地址：[git@github.com:dragonir/threejs-ode…](https://link.juejin.cn/?target=)

《塞尔达传说：旷野之息》 林克在神庙中移动铁球游戏画面

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7364bdbd30124e3eb0e85984c7a1e56f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3927e91462d4f59b913d148babdbfb1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

码上掘金
----

实现
--

下面我们开始使用着色器进行基础图案的绘制，每个图案的绘制过程与前后图案相关联，因此强烈建议**按从上到下的顺序**阅读学习。本文 `50个` 图案绘制流程翻译并整理于[threejs-journey](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs-journey.com%2F "https://threejs-journey.com/")，若需要学习英文原版，可前往查看。

### 50个基本图案

#### 初始化

```js
const testGeometry = new THREE.PlaneBufferGeometry(6, 6, 1, 1);
const testMaterial = new THREE.ShaderMaterial({
  side: THREE.DoubleSide
});
const testMesh = new THREE.Mesh(testGeometry, testMaterial);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b8ac4ab319c43e8afb15268919b6b55~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案1

要将此值从顶点着色器发送到片元着色器，我们需要一个 `varying` 。我们将它称为 `vUv`，并将 `uv` 赋值给它：

**vertex.glsl**

```glsl
varying vec2 vUv;

void main() {
  gl_Position =projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  vUv = uv;
}

```

在片元着色器中，我们可以使用相同的 `varying` 声明来获取此 `vUv`。此时，我们可以在片元着色器中使用 `vUv` 来进行访问 `uv`，它的值从左下角的 `0, 0` 变化到到右上角的 `1, 1`。

**fragment.glsl**

```glsl
varying vec2 vUv;

void main() {
  gl_FragColor = vec4(vUv, 1.0, 1.0);
}

```

这种颜色的图案是最容易得到的，我们只需要在 `gl_FragColor` 中使用 `vUv` 和值为 `1.0` 的蓝色值即可。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce016297bffb4b83b65b8d8a8312c513~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案2

将蓝色值设置为 `0.0`。

```glsl
gl_FragColor = vec4(vUv, 0.0, 1.0);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9cc0fc1464a4f3c940866a6072bd86b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案3

将 `gl_FragColor` 的前三个值都设置为 `vUv` 的 `x` 值，就能得到如下所示的左右方向的渐变效果。

```glsl
float strength = vUv.x;
gl_FragColor = vec4(vec3(strength), 1.0);
// 等价于
gl_FragColor = vec4(vUv.x, vUv.x, vUv.x, 1.0);

```

从现在开始，我们绘制这样图案时，可以创建一个名为 `strength` 的浮点变量，这样接不用分别设置 `r`、`g` 和 `b` 上的值。后面的图形，我们将专注于 `strength` 变量的修改。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cc0120e2b5b417883da22fbc5526712~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案4

当 `gl_FragColor` 的前三个值都使用 `vUv.y` 时，可以实现从上到下的渐变效果。

```glsl
float strength = vUv.y;
gl_FragColor = vec4(vec3(strength), 1.0);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56a820474b9e4ebf9ba085bf9ec88c82~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案5

使用 `1.0 - vUv.y` 时可以实现相反的效果。

```glsl
float strength = 1.0 - vUv.y;

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/855b19b12f484986b80d3b94c3820645~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案6

为了实现下图所示的挤压渐变效果，我们只需将值相乘即可。此时渐变强度会迅速跃升至 `1`，由于我们无法显示比白色更亮的颜色，因此其余渐变将保持白色。

```glsl
float strength = vUv.y * 10.0;

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5dbb7ebf32d4896ab64cbd293b00d1f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案7

现在我们来实现这种重复的渐变效果，此时需要用到**模运算**，模运算返回两数之间的余数，如：

*   `0.8` 模 `1.0` 值为 `0.8`
*   `1.2` 模 `1.0` 值为 `0.2`

在多数语言中，我们通常使用 `%` 运算符进行模运算，但是在 `GLSL` 中需要使用 `mod()` 方法。

```glsl
float strength = mod(vUv.y * 10.0, 1.0);

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f0adb6897af4f0593426d7e153b3c5e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案8

这个图案看起来是基于上一个图案，但是图案颜色值要么是 `0.0` 要么是 `1.0`，而不是渐变色。在 `GLSL` 中可以使用 `if` 条件判断语句来实现这一效果，但是应该避免使用条件判断语句以避免性能问题。我们可以使用 `step(edge, value)` 方法来实现这个功能，它接收两个参数：`edge` 表示一个临界值，第二个参数 `value` 是传入的参数，当传入参数小于临界值时，该方法返回 `0.0`，当传入参数大于临界值时，该方法返回 `1.0`。

```glsl
float strength = mod(vUv.y * 10.0, 1.0);
strength = step(0.5, strength);

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d76c6dd0f1ff49dc9a9db2c6e24eb9b7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案9

当提高 `step` 方法的临界值时，可以得到如下的图案。

```glsl
float strength = mod(vUv.y * 10.0, 1.0);
strength = step(0.8, strength);

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/799e0954b82042a7adb31dad5efba7c5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案10

当我们把上个示例中 `vUv` 的 `y` 值 替换为 `x` 值时，生成的图案是相同的，只是方向变了。

```glsl
float strength = mod(vUv.x * 10.0, 1.0);
strength = step(0.8, strength);

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/130e4ea2c68d4413b592c551c19a6973~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案11

如果将 `x轴` 的结果**相加**到 `y轴` 上，可以得到两者**结合**的效果。

```glsl
float strength = step(0.8, mod(vUv.x * 10.0, 1.0));
strength += step(0.8, mod(vUv.y * 10.0, 1.0));

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe49b7e79ef24f63a167252dcbf7a052~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案12

如果将 `x轴` 与 `y轴` 的结果**相乘**，则会得到它们的**交集**。

```glsl
float strength = step(0.8, mod(vUv.x * 10.0, 1.0));
strength *= step(0.8, mod(vUv.y * 10.0, 1.0));

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3376d2ef427746f182b1d1edc590877b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案13

将 `x轴` 上 `step` 方法中的边界值改成 `0.4`。

```glsl
float strength = step(0.4, mod(vUv.x * 10.0, 1.0));
strength *= step(0.8, mod(vUv.y * 10.0, 1.0));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9593e41546e743c5a9499960c60846d2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案14

这个图案是根据上个图案生成的，我们在 `x轴` 和 `y轴` 上都创建条形图。

```glsl
float barX = step(0.4, mod(vUv.x * 10.0, 1.0)) * step(0.8, mod(vUv.y * 10.0, 1.0));
float barY = step(0.8, mod(vUv.x * 10.0, 1.0)) * step(0.4, mod(vUv.y * 10.0, 1.0));
float strength = barX + barY;

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/813e8a782f814e97be781763a6ed545a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案15

在 `图案14` 代码的基础上，`x轴` 和 `y轴条形图上都设置一个小偏移，可以得到如下结果。

```glsl
float barX = step(0.4, mod(vUv.x * 10.0 - 0.2, 1.0)) * step(0.8, mod(vUv.y * 10.0, 1.0));
float barY = step(0.8, mod(vUv.x * 10.0, 1.0)) * step(0.4, mod(vUv.y * 10.0 - 0.2, 1.0));
float strength = barX + barY;

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f5f7f05c9b740b98a5862b84837f949~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案16

来看看另一个例子，为了得到下面的结果，我们需要给 `vUv.x`设置一个偏移量，使它的值保持在 `-0.5` 到 `0.5` 之间。然后我们需要它的值始终是正数，也就是它的值从 `0.5` 变化到 `0.0` 再变化到 `0.5`。为了实现这一功能，我们可以使用 `abs(...)` 方法。

```glsl
float strength = abs(vUv.x - 0.5);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/574d80c9010a4552b31cc22e4083d282~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案17

这个图案是在上个图案的基础之上，在 `y轴` 上也设置了一个变量。这种组合方式不是单纯使用两者的相加结果，而是使用了 `x轴` 和 `y轴` 图案的最小值，这一功能可以使用 `min(...)` 方法来实现。

```glsl
float strength = min(abs(vUv.x - 0.5), abs(vUv.y - 0.5));

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6feae97269f468497a2537e1bbbbac2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案18

通过 `max(...)` 方法使用最大值，可以得到如下的图案。

```glsl
float strength = max(abs(vUv.x - 0.5), abs(vUv.y - 0.5));

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32145c1008d54d4a959c4fdd35f317f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案19

对上个图案使用 `step(...)` 方法，就能实现由渐变色变为纯色。

```glsl
float strength = step(0.2, max(abs(vUv.x - 0.5), abs(vUv.y - 0.5)));

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36ca228566d844e7b175b6b404ac370c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案20

这个图案是一个正方形图案与另一个更小的反色图案的乘积。

```glsl
float strength = step(0.2, max(abs(vUv.x - 0.5), abs(vUv.y - 0.5)));
strength *= 1.0 - step(0.25, max(abs(vUv.x - 0.5), abs(vUv.y - 0.5)));

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d23810386104f0e858a6fa1e199a4c9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案21

对于这个图案，我们对 `vUv.x` 乘以 `10.0`，然后使用 `floor(...)` 方法向下取整，然后对它除以 `10.0`，就能得到处于 `0.0` 和 `1.0` 范围内的值。

```glsl
float strength = floor(vUv.x * 10.0) / 10.0;

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dabb4009dcc64af986bc8d07ac3623e8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案22

像前面一样，我们可以通过相乘的方式实现不同轴图案的结合。

```glsl
float strength = floor(vUv.x * 10.0) / 10.0 * floor(vUv.y * 10.0) / 10.0;

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d20e405c7b7946c0b94f02e695a224be~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案23

这个图案是一个噪点图，得到这种图案是很复杂的，因为在 `GLSL` 中没有内置的随机函数。获取随机数诀窍是尽量生成一个不可预测的值以至于它看起来像是随机的。一个比较通用的生成随机数方法可以向下面这样使用：

```glsl
float random(vec2 st) {
  return fract(sin(dot(st.xy, vec2(12.9898,78.233))) * 43758.5453123);
}

```

我们为该函数提供了一个 `vec2`，并得到一个伪随机值。在 `GLSL` 代码中，我们可以把这个函数写在 `main` 函数的外面，然后可以像下面这样传入 `vUv` 使用它，生成随机噪点图案。

```glsl
float strength = random(vUv);

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0963435be9734eec81c693a017bef373~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> `📌` 如果你想了解更多关于随机函数的内容，可以访问[The Book of Shaders](https://link.juejin.cn/?target=https%3A%2F%2Fthebookofshaders.com%2F10%2F "https://thebookofshaders.com/10/")，要小心使用随机函数，如果传入错误的值可能会在随机图案中生成明显的不随机的形状。

#### 图案24

这个图案是 `图案22` 和 `图案23` 的结合，首先我们使用 `图案22` 的方法创建一个 `vec2` 二维坐标 名为 `gridUv`，然后用它生成随机图案。

```glsl
vec2 gridUv = vec2(floor(vUv.x * 10.0) / 10.0, floor(vUv.y * 10.0) / 10.0);
float strength = random(gridUv);

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b7a6bb87788462bb4d612517ead50c4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案25

在上个图案的基础上，我们在创建 `gridUv` 时，将 `vUv.x` 想加到 `vUv.y` 上，可以得到这种倾斜效果。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f814f6e0241b47479fabc156e94ff15b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案26

在这种图案中，离左下角越远，亮度越大，是一种以左下角为中心点的径向变化效果。

它实际上使用的是 `vUv` 的长度。`vUv` 的值等于 `0.0, 0.0`，因此左下角的长度值是 `0.0`，离这个角落越远，长度值就越大。在 `GLSL` 中我们可以使用 `length(...)` 方法来获取向量（`vec2`，`vec3`，或 `vec4`）的长度。

```glsl
float strength = length(vUv);

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93bf0ca8787640b2b555b94985147b98~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案27

相反，我们将获得vUv与平面中心之间的距离。

因为平面的 `UV` 值从 `0.0, 0.0` 变化到 `1.0, 1.0`，因此中心点是 `0.5, 0.5`。我们将创建一个对应于中心点的 `vec2`，并使用 `distance(...)` 方法来获取与 `vUv` 的距离。

```glsl
float strength = distance(vUv, vec2(0.5));

```

> 当仅使用一个参数创建向量时，它会同时传递到每个属性。在我们的例子中创建 vec2(0.5)时，它的x值和y值都是0.5。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2581676ca9214a84a47e041242d894c5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案28

使用 `1.0` 减去上个图案的结果，可以得到这个图案。

```glsl
float strength = 1.0 - distance(vUv, vec2(0.5));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d4f3707296941898ddabc4f70175a47~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案29

在创建光照透镜光晕效果时，使用这个图案会非常方便。我们可以通过使用一个很小的数除以前面计算的距离结果来得到这个图案。

```glsl
float strength = 0.015 / (distance(vUv, vec2(0.5)));

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d75d502430df4917869861941aa03cd2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案30

这个图案在上述图案基础上在 `y轴` 上对 `UV` 进行挤压和位移来实现。

```glsl
float strength = 0.15 / (distance(vec2(vUv.x, (vUv.y - 0.5) * 5.0 + 0.5), vec2(0.5)));

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6e3c9e6892d4baf81d85634c1c0020c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案31

这个图案使用两个同类型的的运算式**相乘**生成，但是第二个运算式是基于 `x轴` 对 `UV` 进行挤压和位移来实现。

```glsl
float strength = 0.15 / (distance(vec2(vUv.x, (vUv.y - 0.5) * 5.0 + 0.5), vec2(0.5)));
strength *= 0.15 / (distance(vec2(vUv.y, (vUv.x - 0.5) * 5.0 + 0.5), vec2(0.5)));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b9a72baa2914851b1d803815e8433ac~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案32

这个图案是上个图案旋转生成，生成这个图案是比较费力的。我们需要从中心点旋转 `vUv` 坐标系，在 `GLSL` 中旋转功能可以通过 `cos(...)` 和 `sin(...)` 方法来实现。为了便于后续调用，我们可以在 `main` 函数前面添加一个如下的 `roteta` 方法：

```glsl
vec2 rotate(vec2 uv, float rotation, vec2 mid) {
  return vec2(
    cos(rotation) * (uv.x - mid.x) + sin(rotation) * (uv.y - mid.y) + mid.x,
    cos(rotation) * (uv.y - mid.y) - sin(rotation) * (uv.x - mid.x) + mid.y
  );
}

```

我们就可以使用这个方法来创建一个新的旋转后的 `UV`，是示例中命名为 `totatedUV`。从 `图案31` 到 `图案32`，我们需要旋转八分之一个圆的角度，遗憾的是在 `GLSL` 中是无法直接访问 `π` 的，因此我们需要在代码顶部创建一个 `π` 的常量。

```glsl
#define PI 3.1415926535897932384626433832795

```

然后我们使用 `PI` 值作为 `rotate(...)` 方法的第二个参数，创建旋转八分之一个圆的新 `UV`。最后我们将上个示例中的 `vUv` 替换为新的 `rotatedUV`。

```glsl
vec2 rotatedUv = rotate(vUv, PI * 0.25, vec2(0.5));
float strength = 0.15 / (distance(vec2(rotatedUv.x, (rotatedUv.y - 0.5) * 5.0 + 0.5), vec2(0.5)));
strength *= 0.15 / (distance(vec2(rotatedUv.y, (rotatedUv.x - 0.5) * 5.0 + 0.5), vec2(0.5)));

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33150a2c8b714fa0b23f4027779996f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案33

绘制下面这个图案，我们需要在 `step(...)` 方法中使用 `distance(...)` 方法来生成一个偏移量来控制中心圆的半径。

```glsl
float strength = step(0.5, distance(vUv, vec2(0.5)) + 0.25);

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6988bb392414a19bdf7fdd9c9d088fa~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案34

这个图案和上个图案非常接近，但是我们使用 `abs(...)` 方法来保证是正数。

```glsl
float strength = step(0.02, abs(distance(vUv, vec2(0.5)) - 0.25));

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee4719c3798f49baa829224db1ca37b1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案35

如果将前面两个方法结合起来，可以生成一个圆环。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d952673783ff4a85b00765f250cd363c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案36

使用 `1.0` **减去** 上个图案的值，可以生成反色效果。

```glsl
float strength = 1.0 - step(0.01, abs(distance(vUv, vec2(0.5)) - 0.25));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c18e2ff05d8a4a3e96fbff45a5f429cd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案37

这个图案也是基于前一个图案，但是它有一个**波浪状的变形**效果。为了实现这一效果，我们创建一个新的 `UV` 变量 `wavedUV`，然后使用 `sin(...)` 方法给 `y` 值增加一个基于 `x轴` 的值。然后使用新的 `wavedUV` 代替 `vUv`。

```glsl
vec2 wavedUv = vec2(
  vUv.x,
  vUv.y + sin(vUv.x * 30.0) * 0.1
);
float strength = 1.0 - step(0.01, abs(distance(wavedUv, vec2(0.5)) - 0.25));

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e37c4d2a09941ef8ab6e30221130a1f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案38

这个图案，我们在 `x` 轴上也添加波动效果。

```glsl
vec2 wavedUv = vec2(
  vUv.x + sin(vUv.y * 30.0) * 0.1,
  vUv.y + sin(vUv.x * 30.0) * 0.1
);
float strength = 1.0 - step(0.01, abs(distance(wavedUv, vec2(0.5)) - 0.25));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cfca2086ed2438eae196c2cd1cda27d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案39

如果我们增加 `sin(...)` 的频率，最终会生成如下的奇特效果。

```glsl
vec2 wavedUv = vec2(
  vUv.x + sin(vUv.y * 100.0) * 0.1,
  vUv.y + sin(vUv.x * 100.0) * 0.1
);
float strength = 1.0 - step(0.01, abs(distance(wavedUv, vec2(0.5)) - 0.25));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1ce1bfc9d594c00b9961d6688f2ccaf~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案40

这个图案实际上是 `vUv` 的角度。要从 `2D` 坐标获得一个角度，我们可以使用 `atan(...)` 方法：

```glsl
float angle = atan(vUv.x, vUv.y);
float strength = angle;

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df4736ebb8694d90b70c382394aa023e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案41

如果给 `vUv` 设置 `0.5` 的偏移量获取角度，得到的是一个围绕中心的角度。

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5);
float strength = angle;

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d8bc81847134deea54f81eb16d6127b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案42

这个图案显示了围绕中心点的完整角度，角度值从 `0.0` 变化到 `1.0`。此时 `atan(...)` 返回一个 `-π` 到 `+π` 之间的值。首先我们使用 `PI * 2`进行整除，我们得到一个介于 `-0.5` 到 `0.5` 之间的数，我们需要加上 `0.5`。

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5);
angle /= PI * 2.0;
angle += 0.5;
float strength = angle;

```

简写为：

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5) / (PI * 2.0) + 0.5;

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7924447ff6364523b1c275cb8a78fecf~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案43

对于上个图案的结果，我们进行模运算可以得到如下所示的图案。

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5) / (PI * 2.0) + 0.5;
float strength = mod(angle * 20.0, 1.0);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bec5f3990bf4b069d1c86ebacf55fb7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案44

使用 `sin(...)` 方法。

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5) / (PI * 2.0) + 0.5;
float strength = sin(angle * 100.0);

```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f62480b00ea74e6385cc075d848e77a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案45

我们可以使用前面的值来定义我们之前绘制的圆的半径：

```glsl
float angle = atan(vUv.x - 0.5, vUv.y - 0.5) / (PI * 2.0) + 0.5;
float radius = 0.25 + sin(angle * 100.0) * 0.02;
float strength = 1.0 - step(0.01, abs(distance(vUv, vec2(0.5)) - radius));

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f351d825b8414520a3611fd486e7db56~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案46

这个图案称为**柏林噪声**，你之前可能已经听说过它。柏林噪声有助于生成自然的形状，如云 `☁`、水 `💧`、火 `🔥`、地形 `🏔` 等，它同时也可以用在动画的生成中，如可以实现在风中飘动的草 `🍀`、雪花 `❄` 等动画效果。

柏林噪声的生成算法有很多种，它们有不同的生成结果，或是应用在不同的域中（`2D`、`3D` 甚至 `4D`）,它们中有些是通过重复自身生成的，有的则具有更高的性能优势。这个[Github仓库地址](https://link.juejin.cn/?target=https%3A%2F%2Fgist.github.com%2Fpatriciogonzalezvivo%2F670c22f3966e662d2f83 "https://gist.github.com/patriciogonzalezvivo/670c22f3966e662d2f83")中列举了在 `GLSL` 中比较受欢迎的一些柏林噪声效果。

下面这个**经典柏林噪声算法** 是 **Stefan Gustavson** 开发的，我们传入一个 `vec2` 可以得到一个 `float` 值的 `2D` 噪声，如果**直接使用他的这个算法会报错**。

```glsl
//  Classic Perlin 2D Noise
//  by Stefan Gustavson

vec2 fade(vec2 t) {
  return t*t*t*(t*(t*6.0-15.0)+10.0);
}

float cnoise(vec2 P) {
  vec4 Pi = floor(P.xyxy) + vec4(0.0, 0.0, 1.0, 1.0);
  vec4 Pf = fract(P.xyxy) - vec4(0.0, 0.0, 1.0, 1.0);
  Pi = mod(Pi, 289.0); // To avoid truncation effects in permutation
  vec4 ix = Pi.xzxz;
  vec4 iy = Pi.yyww;
  vec4 fx = Pf.xzxz;
  vec4 fy = Pf.yyww;
  vec4 i = permute(permute(ix) + iy);
  vec4 gx = 2.0 * fract(i * 0.0243902439) - 1.0; // 1/41 = 0.024...
  vec4 gy = abs(gx) - 0.5;
  vec4 tx = floor(gx + 0.5);
  gx = gx - tx;
  vec2 g00 = vec2(gx.x,gy.x);
  vec2 g10 = vec2(gx.y,gy.y);
  vec2 g01 = vec2(gx.z,gy.z);
  vec2 g11 = vec2(gx.w,gy.w);
  vec4 norm = 1.79284291400159 - 0.85373472095314 * vec4(dot(g00, g00), dot(g01, g01), dot(g10, g10), dot(g11, g11));
  g00 *= norm.x;
  g01 *= norm.y;
  g10 *= norm.z;
  g11 *= norm.w;
  float n00 = dot(g00, vec2(fx.x, fy.x));
  float n10 = dot(g10, vec2(fx.y, fy.y));
  float n01 = dot(g01, vec2(fx.z, fy.z));
  float n11 = dot(g11, vec2(fx.w, fy.w));
  vec2 fade_xy = fade(Pf.xy);
  vec2 n_x = mix(vec2(n00, n01), vec2(n10, n11), fade_xy.x);
  float n_xy = mix(n_x.x, n_x.y, fade_xy.y);
  return 2.3 * n_xy;
}

```

原因是缺失了 `permute` 方法，我们需要在调用之前补充这个方法：

```glsl
vec4 permute(vec4 x) {
  return mod(((x*34.0)+1.0)*x, 289.0);
}

```

现在我们可以使用 `vUv` 来调用 `cnoise` 方法创建噪声了。

```glsl
float strength = cnoise(vUv);

```

结果是几乎看不到噪声效果，此时我们可以对 `vUv` 乘以 `10.0`，就能生成如下所示的噪声图案了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b54109472d84da787096253d2e57114~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案47

这个图案也是使用同一个噪声效果，不过调用了 `step(...)` 方法。我们可以使用这种噪声图案创建一头牛 `🐄`，可以尝试给一个牛模型贴上这种材质试试。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17074a0be7f8403dae4b1ee0e4ae7164~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案48

这个图案，我们对噪声值使用 `abs(...)` 方法，然后使用 `1.0` 减去这个值。可以使用这种图案创建闪电 `⚡`，水的波纹 `💧` 等。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/245ca8f1d1a24268b60a09ad7ab1ab47~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案49

在噪声上使用 `sin(...)` 方法，可以得到如下的图案。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9187432890d457e8cadf9a4f739f421~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 图案50

最后一个，我们结合 `sin(...)` 方法和 `step(...)` 方法，可以创建出这样的图案。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b88e5425dfb84d1194a94df647986eae~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 使用颜色

上面所有的示例几乎都是黑白色的，刚开始创建的那个图案 我们使用 `gl_FragColor = vec4(vUv, 1.0, 1.0)` 就能创建出好看是渐变色，现在我们也可以使用这种渐变色 `🌈`，来替换上面例子中的黑白色。

混合颜色

为了实现这一效果，我们将使用 `mix(...)` 方法，它接收 `3` 个参数：

*   第一个参数可以是一个 `float`, 一个 `vec2`, 一个 `vec3`, 或者是一个 `vec4`.
*   第二个参数应该是与第一个类型相同的参数
*   第三个参数必须是一个 `float` 浮点值，它将决定接受更多的第一个输入或更多的第二个输入。如果我们使用 `0.0`，则返回的值将是第一个输入。如果我们使用 `1.0`，则返回值将是第二个。如果我们使用 `0.5`，则该值将是两个输入之间的混合。您也可以低于 `0.0` 或高于 `1.0`，这些值将被排除。

对于第50个图案，我们可以像这样应用渐变色颜色。

```glsl
float strength = step(0.9, sin(cnoise(vUv * 10.0) * 20.0));
// Final color
vec3 blackColor = vec3(0.0);
vec3 uvColor = vec3(vUv, 1.0);
vec3 mixedColor = mix(blackColor, uvColor, strength);
gl_FragColor = vec4(mixedColor, 1.0);

```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a1637397aa64848a53f38404301add3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 神庙铁球

首先我们来使用上面创建的图案来生成一个神庙铁球 `🔮`，首先我们创建一个球体添加到场景中，并为其应用上面绘制的着色器作为材质：从代码中可以看出，这次我们使用了**二十面缓冲体** `THREE.IcosahedronBufferGeometry` 来创建球体而不是用**基础球体几何体** `THREE.SphereGeometry`，其实并没有特殊意义，只是为了让大家了解一下 `Three.js` 中各种几何体的用法 `😂`。

```js

const material = new ShaderMaterial({
  side: DoubleSide,
  vertexShader: vertexShader,
  fragmentShader: fragmentShader
});

const Icosahedron = new Mesh(new IcosahedronBufferGeometry(1, 64), material);
scene.add(Icosahedron);

```

在**片元着色器**中在最后一个图案的基础上调一下 `uvColor` 的参数，使其更加接近塞尔达旷野之息游戏中的神庙铁球颜色。

```glsl
vec3 uvColor = vec3(vUv, 0.2);

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/982cb95781db452889a3b6168f82f42b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

现在看起来有图案纹理了，但是还是看起来不够立体，也不像真实的的铁球，我们需要进一步优化一下。为了使铁球看起来更真实，我们按以下方式可以给场景添加**辉光效果**。首先我们需要引入 `EffectComposer`、`UnrealBloomPass`、`RenderPass`，使用它们来添加后期渲染效果。添加后期渲染效果的大致步骤是：

*   创建 `EffectComposer` 效果组合器对象，在该对象上添加后期处理通道。
*   配置 `EffectComposer` 对象，使它可以渲染场景，并应用后期处理。
*   在渲染循环中，使用 `EffectComposer` 来渲染场景、应用通道和输出结果。

**需要注意的是，要将 `renderer.autoClear` 设置为 `false` 以及在页面重绘方法中调用 `composer` 的 `render` 方法，不然后期效果不会生效**。

```js
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";

renderer.autoClear = false;


const renderScene = new RenderPass(scene, camera);
const bloomPass = new UnrealBloomPass(new Vector2(window.innerWidth, window.innerHeight), 1.5, .4, .85);
bloomPass.threshold = 0;
bloomPass.strength = 2.2;
bloomPass.radius = .2;
const bloomComposer = new EffectComposer(renderer);
bloomComposer.renderToScreen = true;
bloomComposer.addPass(renderScene);
bloomComposer.addPass(bloomPass);


const tick = () => {
  Icosahedron && (Icosahedron.rotation.y += .01);
  Icosahedron && (Icosahedron.rotation.x += .005);
  bloomComposer && bloomComposer.render();
  
  controls.update();
  
  renderer.render(scene, camera);
  
  window.requestAnimationFrame(tick);
}
tick();

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48bb51233fb340d08d9143de468e624c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

> `🔗` 源码地址：[github.com/dragonir/th…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdragonir%2Fthreejs-odessey "https://github.com/dragonir/threejs-odessey")

> `📌` 后期渲染知识也是比较有难度的，由于本文主题是着色器图案且篇幅有限，无法在本文简短的示例中说清楚，大家只需了解如何使用**辉光效果**即可，后续将专门整理一篇 `Three.js` 后期处理知识的文章，感兴趣的同学也可自行搜索学习。

总结
--

本文中主要包含的知识点包括：

*   使用着色器绘制 `50` 个基础图案
*   应用着色器作为网格模型的材质纹理
*   学会后期渲染的基本流程，并使用辉光效果创建发光物体

> 想了解其他前端知识或其他未在本文中详细描述的**Web 3D**开发技术相关知识，可阅读我往期的文章。如果有疑问可以在评论中**留言**，如果觉得文章对你有帮助，不要忘了**一键三连哦 👍**。

最后，期待《**塞尔达传说2：王国之泪**》早日发行！！！ `(☆▽☆)`

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d10e160c3eca4721b48433fc7a62d353~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

附录
--

*   \[1\]. [🌴 Three.js 打造缤纷夏日3D梦中情岛](https://juejin.cn/post/7102215670477094925 "https://juejin.cn/post/7102215670477094925")
*   \[2\]. [🔥 Three.js 实现炫酷的赛博朋克风格3D数字地球大屏](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7124116814937718797 "https://juejin.cn/post/7124116814937718797")
*   \[3\]. [🐼 Three.js 实现2022冬奥主题3D趣味页面，含冰墩墩](https://juejin.cn/post/7060292943608807460 "https://juejin.cn/post/7060292943608807460")
*   \[4\]. [🦊 Three.js 实现3D开放世界小游戏：阿狸的多元宇宙](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fpost%2F7081429595689320478 "https://juejin.cn/post/7081429595689320478")
*   \[5\]. [🏆 掘金1000粉！使用Three.js实现一个创意纪念页面](https://juejin.cn/post/7143039765725020167 "https://juejin.cn/post/7143039765725020167")
*   `...`
*   [【Three.js 进阶之旅】系列专栏访问 👈](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fcolumn%2F7140122697622618119 "https://juejin.cn/column/7140122697622618119")
*   [更多往期【3D】专栏访问 👈](https://juejin.cn/column/7049923956257587213 "https://juejin.cn/column/7049923956257587213")
*   [更多往期【前端】专栏访问 👈](https://link.juejin.cn/?target=https%3A%2F%2Fjuejin.cn%2Fcolumn%2F7021076460089638926 "https://juejin.cn/column/7021076460089638926")

参考
--

*   \[1\]. [three.js journey](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs-journey.com%2F "https://threejs-journey.com/")
*   \[2\]. [threejs.org](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs.org "https://threejs.org")
*   \[3\]. 《Three.js 开发指南——基于WebGL和HTML5在网页上渲染3D图形和动画》