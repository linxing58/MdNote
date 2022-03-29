# WebGL浅入浅出
豆皮粉儿们，又见面了，今天这一期，由字节跳动数据平台的 “Kakashi” 小哥哥，给我们带来 webgl 的入门学习。那到底什么是 WebGL 呢？WebGL 有什么用？如何快速入门呢？让我们跟随小哥哥快速入坑吧！

![](https://pic2.zhimg.com/v2-d09dea4d8c7b1667b0f8b5d072ecff65_b.jpg)

图片

> 本文作者：Kakashi

前言

本篇文章预计阅读耗时【15min】

将较为全面基础的带你学习 如何使用 WebGL（偏重） && 相关原理（简略）。

经过本次学习 你将可以从 0 到 1 搭建自己的 webgl 简易渲染库

## 一 、背景

### 1.1 为什么要用 WebGL？

先热身一下吧，看个问题：如果你有一堆作业没有做完，可以选择帮手替你完成，你会选择？

![](https://pic2.zhimg.com/v2-072853aa983c97c453a199c67f331401_b.jpg)

图片

A. 1 个老教授  
B. 9 个小学生

• 选 A 的同学，你真的忍心让老教授帮忙写语文填空？• 选 B 的同学，你真的放心让小学生帮你解线性代数？所以我们要看看 “作业” 内容究竟是什么，有多少作业量。• 如果题目是 1+2 ，9 个小学生 = 人海战术免费劳动力，没有技术含量，纯粹体力活，效率会很高。• 如果题目是线性代数 ，老教授 >> 99 个小学生。回到我们的主题，类似的，由于设计目标的不同，CPU 和 GPU 也是大不相同的。它们分别针对了两种不同的应用场景：•CPU 需要很强的通用性来处理各种不同的数据类型，同时又要逻辑判断又会引入大量的分支跳转和中断的处理。这些都使得 CPU 的内部结构异常复杂。•GPU 面对的则是类型高度统一的、相互无依赖的大规模数据和不需要被打断的纯净的计算环境。

![](https://pic2.zhimg.com/v2-5446b96b67c29deab78ba586172c00f5_b.jpg)

图片

（图片来自 nVidia CUDA 文档。其中绿色的是计算单元，橙红色的是存储单元，橙黄色的是控制单元。） GPU 在概念上适用于高度并行计算，因为 GPU 可以通过计算隐藏内存访问延迟，而不是通过大数据缓存和流控制来避免内存访问延迟。简而言之，**CPU 基于低延时的设计，GPU 是基于大的吞吐量设计**。

### 1.2 什么是 WebGL？

WebGL 是一种 3D 绘图标准，这种绘图技术标准允许把 JavaScript 和 OpenGL ES 2.0 结合在一起，通过增加 OpenGL ES 2.0 的一个 JavaScript 绑定，**WebGL 可以为 HTML5 Canvas 提供硬件 3D 加速渲染（部分计算 GPU）**，这样 Web 开发人员就可以借助系统显卡来在浏览器里更流畅地展示 3D 场景和模型了，还能创建复杂的导航和数据视觉化。显然，WebGL 技术标准免去了开发网页专用渲染插件的麻烦，可被用于创建具有复杂 3D 结构的网站页面，甚至可以用来设计 3D 网页游戏等等。总结一下，**WebGL 的本质 —— JavaScript 操作 OpenGL 接口**。

![](https://pic3.zhimg.com/v2-837a35848cf0d034dcbfa563d52cd472_b.jpg)

图片

```text
当然，WebGL只是绑定了一层，内部的一些核心内容，如着色器，材质，灯光等都是需要借助GLSL ES语法来操作的。
```

### 1.2.1 WebGL / OpenGL / ES X.0 之间的关系？

![](https://pic3.zhimg.com/v2-9b0990503bd5a79646c7d72818819546_b.jpg)

图片

•OpenGL（Open Graphics Library） 一种图形应用程序编程接口规范 (Application Programming Interface，API)。它是一种可以对图形硬件设备特性进行访问的软件库，OpenGL 被设计为一个现代化的、硬件无关的接口，因此我们可以在不考虑计算机操作系统或窗口系统的前提下，在多种不同的图形硬件系统上，完全通过软件的方式实现 OpenGL 的接口。•OpenGL ES（embedded system） OpenGL ES 是 OpenGL 的一个特殊版本，主要针对嵌入式设备使用，专用于嵌入式计算机、智能手机、家用游戏机等设备。OpenGL ES 2003-2004 年被首次提出来，其中两次重要升级分别在 2007 年（OpenGL ES 2.0）和 2012 年（OpenGL ES 3.0），WebGL 就是基于 OpenGL ES 2.0 的。

* * *

## 二、如何使用 WebGL

### 2.1 入门示例，不再是 hello world!

![](https://pic1.zhimg.com/v2-ed9807d9e96feb7898945317b1d91550_b.jpg)

运行结果 & 流程示意：

![](https://pic4.zhimg.com/v2-ed48f3dab043c6c8f50906b5174a098f_b.jpg)

结论：

1.WebGL 需要依赖 canvas 这个载体获取对应的绘图上下文 2.WebGL API 基于 OpenGL ES，函数命名相对应 3.WebGL 基于多缓冲区模型

### 2.2 绘制一个点

如果类似 canvas 2d 的话，你可能以为 WebGL 绘制一个点：

![](https://pic2.zhimg.com/v2-8b2581bbff20bd7d197b7c4b9b9b54f9_b.jpg)

类似这样就可以了，不幸的是：WebGL 依赖于一种着色器（shader）的绘图机制，复杂而强大，着色器是 WebGL 的核心机制。WebGL 中着色器程序可以写成字符串变量：

![](https://pic1.zhimg.com/v2-72c9f001fa7b60a911d3ce9d5932de5c_b.jpg)

图片

当然也支持通过 ajax 请求读取 glsl 文件，又或者参考 MDN GLSL 示例，写在 script 标签中。接下来我们初始化着色器（包括加在着色器代码，创建程序，链接，编译等一套流程）：

![](https://pic4.zhimg.com/v2-8c5fe7c42f12fbaf43638ee4842d1afb_b.jpg)

图片

绘制点：

![](https://pic2.zhimg.com/v2-a87843ec368ad6617f92f20db634fb61_b.jpg)

运行结果 & 流程示意：

![](https://pic1.zhimg.com/v2-b6f7ff0bb86cbe944172d41036e41694_b.jpg)

（思考一下，为什么画在了屏幕右上角？悄悄提示：坐标系统）

结论：

1.WebGL 需要两种着色器，使用 GLSL ES 语言来编写（需要学习成本）。

顶点着色器（Vertex shader）：描述顶点特性（位置，尺寸）。

片元着色器（Fragment shader）：描述 片元（像素）处理过程。

2\. 着色器运行在 WebGL 系统中，而不是 JS 程序。

3.WebGL 位置使用向量，使用右手坐标系。

### 2.3 绘制多个动态的点

上一节绘制出了一个点，但是点的位置、颜色之类属性是在代码中写死的，这种 “硬编码” 便于理解，但是缺乏可拓展性，怎么动态的设置点的信息呢？

我们的目标，将位置信息通过 Javascript 动态传给顶点着色器，需要经过下面的流程：

1\. 在顶点着色器，声明 attribute 变量。2. 将 attribute 变量赋值给 gl_Positon 变量。3. 向 attribute 变量传输数据。

![](https://pic1.zhimg.com/v2-d92026d27fb1ae5679a12a8569a94994_b.jpg)

图片

我们需要在第三步来绘制多个点，举个例子：一共绘制 20 个点，采取的策略是：

• 每次绘制一个点，绘制 20 次？• 绘制 1 次，一次性绘制 20 个点？当然选择后者。我们需要借助 缓冲区对象，一次性传入多个顶点数据。

### 2.4 绘制一个箭头

查看 Canvas 的绘图 API，我们会发现它能画直线、矩形、圆、弧线、贝塞尔曲线。于是，我们看了看 WebGL 绘图 API，发现：

![](https://pic2.zhimg.com/v2-759208b9bfdd18148fab366893e8877d_b.jpg)

mode 指定绘制图元的方式，可能值如下：•gl.POINTS: 绘制一系列点。•gl.LINE_STRIP: 绘制一个线条。即，绘制一系列线段，上一点连接下一点。•gl.LINE_LOOP: 绘制一个线圈。即，绘制一系列线段，上一点连接下一点，并且最后一点与第一个点相连。•gl.LINES: 绘制一系列单独线段。每两个点作为端点，线段之间不连接。•gl.TRIANGLE_STRIP：绘制一个三角带。•gl.TRIANGLE_FAN：绘制一个三角扇。•gl.TRIANGLES: 绘制一系列三角形。每三个点作为顶点。

![](https://pic4.zhimg.com/v2-3a0e47324bd15943c6f74cf53c527687_b.jpg)

```text
const VERTEXRS = [V0,V1,V2,V3,V4,V5];
```

![](https://pic3.zhimg.com/v2-de4e4efe1fccba04cf578ab7747f03b2_b.jpg)

如图所示，WebGL 可只能绘制三种图元：点、线、三角形。不过所有其他图元，从圆到球体、从立方体到复杂三维模型，都可以由一个个三角面片组成。实际上，我们可以用三角形绘制出一切你想绘制的东西。那么绘制一个箭头的思路可以有很多，举两个例子：

![](https://pic4.zhimg.com/v2-fdfa9460f388464a9810dd9c66a2c69b_b.jpg)

不过不同的绘制方案直接是有区别的，会影响到法线，进而影响面片方向（正面还是背面）的判断，这些就不深入介绍啦。

结论：WebGL 只能绘制点，线，三角形，但是万物可以由三角形组成（原则：数量越少越好）。

### 2.5 绘制一个正方体

### 2.5.1 关于三维世界

二维图形和三维图形，最明显的区别是：三维图形有深度，也就是 z 轴。我们需要考虑两个角度：

1\. 观察方向：观察者在什么位置，在看场景的哪一部分？

![](https://pic2.zhimg.com/v2-c837434a9f4890d74610c412e870bed5_b.jpg)

图片

观察者的位置称为视点（eye point），从视点出发沿观察方向的射线 叫做视线（viewing direction），通过视图矩阵确定观察方向：

![](https://pic1.zhimg.com/v2-b5f7032bf7cb9d4c80f584557ea94e14_b.jpg)

1\. 可视空间：观察者能看到多远？（能看多少？是否需要远小近大）

• 正摄投影：长方体可视空间，也称盒状空间：

![](https://pic1.zhimg.com/v2-b5275807639ec0893a629ea6c4192fa4_b.jpg)

正射投影可以通过投影矩阵确定可视空间：

```text
Matrix4.setOrtho(left, right, bottom, top, near, far);
```

• 透视投影：四棱锥 / 金字塔可视空间：

![](https://pic2.zhimg.com/v2-a4c6844b5490bb83b28a317aca5e03e1_b.jpg)

透视投影可以通过投影矩阵确定可视空间：

![](https://pic4.zhimg.com/v2-b7affc72611fe21c2d8545d55e12bf4f_b.png)

### 2.5.2 关于正方体

通过之前学习，我们联想到，两个三角形可以拼成一个矩形平面，作为正方体一面。一个正方体也就是 6\*2 = 12 个三角形，每个三角形 3 个顶点，也就是 12 \*3 = 36 个顶点。我们的顶点着色器传入 36 个顶点就可以了。但是用正常逻辑一想，正方体实际上不是只有 8 个顶点吗？显然我们定义 36 个是有重复的，并没有被多个三角形公用。如你所愿，WebGL 提供了一个通过 gl.drawElements 函数的复用方案：立方体被拆成 6 个面，每个面都由两个三角形组成，每个三角形都有三个顶点，和顶点列表中的三个点相关联。三角形列表中的数字表示该三角形的三个顶点在顶点列表中的索引值。如下图所示：

![](https://pic2.zhimg.com/v2-2e0aa0dfccd99adfc45110bc969481c9_b.jpg)

结论：WebGL 绘制 3D 较为复杂，需要依赖视图矩阵和投影矩阵实现 3d 视觉效果，但是建模部分还是依赖于最小图元 - 三角形。

### 2.6 其他疑惑

• 怎么样平移，旋转，缩放？A：线性代数上场啦。举个例子，平移需要将平移矩阵\[Tx，Ty，Tz]传入顶点着色器，逐顶点操作，加到对应顶点坐标分量上，再赋值给 gl_positon。旋转、缩放类似。• 怎么样写文字？A：参考链接，或者可以用 tiny-sdf 库 ，文字即图片的方案。• 怎么实现动画？A：WebGL 不提供动画接口，主要依赖 requestAnimation 方法。• 怎么实现渐变色？A：借用顶点着色器缓冲区对象，通过 varying 变量， 从顶点着色器向片元着色器传输数据。• 怎么实现背景图片？A：纹理映射（Texture Mapping）。在片元着色器将准备好的纹理（Image 转换）取出赋值给片元。• 怎么画圆？A：把正方形削成圆形。片元着色器内可以通过坐标判断放弃绘制一些点。

* * *

## 三、WebGL 的流程原理

简单说来，WebGL 绘制过程包括以下三步：

1、获取顶点坐标

![](https://pic4.zhimg.com/v2-000411dd1961d859c89b673e0183d177_b.jpg)

2、图元装配（即画出一个个三角形）

![](https://pic2.zhimg.com/v2-a18cb608a1ac8bc86e2606bde8bbd061_b.jpg)

3、光栅化（生成片元，即一个个像素点）

![](https://pic4.zhimg.com/v2-c61d6a03a92cb06468d99c83de5bf81f_b.jpg)

完整链路：

![](https://pic1.zhimg.com/v2-256114531292e66fb84207111c3009fc_b.jpg)

* * *

## 四、WebGL 的现状

看到下图，笔者不禁一把辛酸泪。

![](https://pic1.zhimg.com/v2-76ed114f1eb9bda441dd583dc8442c30_b.jpg)

图片

对于 WebGL 而言，最大的问题就是**兼容性**。

### 4.1 浏览器支不支持？（入门难度：安装最新 chrome）

WebGL 普及程度相对较高。2.0 版本 Safari 似乎正在接入，或许对 WebGL2.0 的推广有所助力。

![](https://pic4.zhimg.com/v2-223b7c4847b028fb54514bf6154af3a7_b.jpg)

### 4.2 显卡支不支持？（地狱难度：换显卡)

![](https://pic1.zhimg.com/v2-d38c840dbb0da7d4078c0dab0392e7a4_b.jpg)

图片

### 4.3 显卡和浏览器都 ok 的话，设置合不合理？（难以评估难度：对非开发者极不友好）

•Case1：浏览器设置开启硬件加速。打开 chrome 设置（地址栏输入[chrome://settings/](https://link.zhihu.com/?target=chrome%3A//settings/)），『显示高级设置』，找到『系统』，在『使用硬件加速模式』选项前打钩，重启浏览器。•Case2：浏览器设置禁用 webgl WebGL：--disable-webgl 打开 [chrome://flags](https://link.zhihu.com/?target=chrome%3A//flags/) 面板（将这行字输入地址栏），ctrl/command + f 搜索『--disable-webgl』并启用该选项。重启浏览器。•Case3：浏览器设置显卡黑名单 Hardware accelerated：WebGL 已启用，并获得了显卡支持 Software only, hardware acceleration unavailable：WebGL 已启用，但没有显卡支持，只有软件渲染支持 Unavailable：WebGL 既没有显卡支持也没有软件支持。

![](https://pic2.zhimg.com/v2-65851c560bb5cc182eb3d36494f8651d_b.jpg)

图片

WebGL 应用一般需要显卡加速，如果你的浏览器显示 WebGL 未获得显卡支持（即显示为后面两项），如上图 webgl2 Unavailable，则两种可能：

1\. 你电脑的显卡可能进入了 chrome 的黑名单。2. 你的显卡不行（\_） 有一些显卡和显卡驱动因为 bug 太多会导致浏览器崩溃甚至系统崩溃，所以很多浏览器都有一个显卡黑名单，对这些有问题的显卡和驱动，浏览器将不启用硬件加速。可以在 [chrome://flags](https://link.zhihu.com/?target=chrome%3A//flags/) 中开启 --ignore-gpu-blacklist 来无视这个黑名单（不推荐，可能会引发浏览器崩溃或者系统崩溃）

* * *

## 附录

一些常用的库：

•Three （基于原生 WebGL 封装运行的三维引擎，在所有 WebGL 引擎中，Three.js 是国内文资料最多、使用最广泛的三维引擎）•stats.js （JavaScript 性能监控器，可以测试 webgl 代码的渲染性能）•dat.GUI（google 开发人员创建的，是一个轻量级的图形用户界面库（GUI 组件），使用这个库可以很容易地创建出能够改变代码变量的界面组件）•ShaderToy（一个酷炫有趣的网站，学习 shader 编程）•rainbow\[1]（一个简易的 webgl 渲染库，包含本次全部 demo⭐️）

* * *

## 参考资料

• 图解 WebGL&Three.js 工作原理：[https://www.cnblogs.com/wanbo/p/6754066.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/wanbo/p/6754066.html)•《WebGL 编程指南》：【必看】

### References

`[1]` rainbow: _[https://github.com/liangyuqi/rainbow](https://link.zhihu.com/?target=https%3A//github.com/liangyuqi/rainbow)_

教程资源

我们为大家精心准备了 2 份开源 webgl 在线电子教程，公众号后台回复 \*\*“webgl” \*\*即可获取。

The End

![](https://pic3.zhimg.com/v2-ad62d7a2f0db827c8ee16ee3b418b076_b.gif) 
 [https://zhuanlan.zhihu.com/p/337460642](https://zhuanlan.zhihu.com/p/337460642)
