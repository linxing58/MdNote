# 用 Three.js 写了一个涵盖了大部分基础3D功能的综合场景 - 掘金
前言
--

22年1月23日更新：由于知识产权问题，源码在个人GitHub永久下架，代码片段在文章还是有很多的。

作者在实习期间，实现了一个 Web3D 风机综合场景Demo（实习答辩所用，当时未上线）。主要是对 [图扑的风机场景](https://link.juejin.cn/?target=https%3A%2F%2Fwww.hightopo.com%2Fdemo%2Ffan3d-magic%2F "https://www.hightopo.com/demo/fan3d-magic/") 的改进。

可能是因为只是图扑官网的demo，所以可能上线较为仓促，所以图扑的风机场景存在以下问题

1.  首次加载速度慢（主要和3D模型相关）
2.  渲染低效（主要和3D模型相关）
3.  不可场景化（不可改变天气，动画等）
4.  交互性差（基本没有交互功能）

所以我后面对图扑的 demo 进行了如下改进

1.  增加场景化 —— 根据风速变化改变叶片转速、**根据天气预报来模拟风场天气**
2.  提高渲染效率 —— 使用glTF格式模型
3.  增强交互性 —— 增加控件选项，选择权交给用户

  

*   演示视频 —— [风机3D场景演示视频 __ 哔哩哔哩](https://link.juejin.cn/?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1rf4y1F7Am%3Fspm_id_from%3D333.999.0.0 "https://www.bilibili.com/video/BV1rf4y1F7Am?spm_id_from=333.999.0.0")

Web3D的基础知识
----------

3d的基本知识主要是基于计算机图形学的

包括场景Scene、摄像机Camera、渲染器Renderer、几何体Geometry、光照模型Light、纹理贴图Texture、材质Material等

具体的可以参考 [郭隆邦大佬的博客](https://link.juejin.cn/?target=http%3A%2F%2Fwww.yanhuangxueyuan.com%2F "http://www.yanhuangxueyuan.com/")，确实是有比较多的内容，这里就不再展开了

3D模型的选择
-------

3D 模型格式的选择是最简单的一步，但是可能也是对渲染效率最关键的一步。

图扑的 3D格式 选用的是 OBJ+MTL, 这也是他加载速度慢和渲染负载高的重要因素之一，所以我首先从3D模型格式入手，进行第一步改进。

* * *

### 常用Web3D模型的区别

在选择 3D格式 之前，我们需要了解有哪些主流的 3D格式，各自又有什么特点

1.  glTF

glTF 是一个非常全面的格式，几乎支持所有的常见功能，支持颜色、动画、CSG、细节网格工作、纹理、相机、光线、相对定位等重要功能。

glTF 是一个中转格式，不需要像其他格式一样创建 importer、converter，直接可以被相关application使用。

2.  STL

STL仅支持几何图形的简单表达功能，其他类似于颜色、材质、光照、纹理等性质一概没有，所以几乎是一个不可用的 3D 格式

3.  OBJ OBJ 支持某些功能，如简要表达，CSG，颜色和材料。但是没有动画、纹理、相机等重要的3D模型性质。 比如对于OBJ 格式的模型来说，如果需要显示想要的纹理，则要借助额外的mtl文件来显示纹理。
    
4.  FBX
    

更准确的说，FBX模型是一种通用模型格式，而不是像 OBJ 一样的3D模型文件格式，支持所有主要的三维数据元素以及二维、音频和视频媒体元素，还包含动画、材质特性、贴图、骨骼动画、灯光、摄像机等信息。

FBX格式主要支持多边形(Polygons)游戏模型、曲线(Curves)、表面(Surfaces)、点组材质(Point Group Materials）。但是没有提供详细的网格（Detailed Mesh），其他和gltf一样完善，较为适用于游戏。

图表总结

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d7b595d026949d5b20e5f2b403dfba2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

* * *

### 为什么我使用glTF格式

glTF 的底层工作原理还是比较多了，我只是在这里解释一下它作为 Web3D 格式的一些优点，如果需要详细学习 glTF 的同学可以点击 [glTF-Tutorials](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FKhronosGroup%2FglTF-Tutorials "https://github.com/KhronosGroup/glTF-Tutorials") 和 [如何在页面极速渲染3D模型](https://link.juejin.cn/?target=https%3A%2F%2Fisux.tencent.com%2Farticles%2Fisux-optimizing-3d-model.html "https://isux.tencent.com/articles/isux-optimizing-3d-model.html")

1.  免除额外的模型数据转换，提高解析速度

*   glTF的目标是作为一个中转格式，而不是另一个新的3D数据格式
*   使用JSON来描述场景结构，可以方便地被应用程序分析处理。但是3D对象的几何数据和纹理数据通常不被包含在JSON文件中，它们被存储在外部的文件或二进制文件中，JSON文件中只包含了到这些外部文件的链接。
*   3D数据以一种可以被大多数图形API直接使用的方式（以二进制的形式）进行存储，不需要应用程序进行解码或预处理操作。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e47ae450af54663910bfea8179f56a5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

其他格式，比如OBJ在运行相关application使用前，为了读取不同的输入文件格式，所以必须解析场景结构，并且必须将 数值型几何数据转换为图形 API 所需的格式。 所以每个运行时应用程序都必须为其支持的所有文件格式创建很多importer、converter，既占内存，也降低了渲染效率。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/320261dd1a68423085037b4033ae4e9f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

2.  改变动画较为方便

*   图扑的风机场景demo是基于OBJ格式的动画，但是由于 OBJ 没有自带动画，所以在修改动画时会比较麻烦
*   glTF修改动画非常方便，关于动画的详细内容我会放在下文继续讲。

改变glTF 3D模型动画
-------------

图扑的风机叶片转速是固定的，我希望其能场景化，根据后端传来的风速数据进行相应的改变，也就是说风速越快，叶片转的越快，这就涉及到了 改变模型的动画

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c729b9fdac74de995a1c9c17cac003b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

* * *

### 搞清glTF如何形成动画

glTF格式的3D模型是可以自带动画的（这一点和OBJ区别很大，OBJ是不可以自带动画的）

glTF格式动画的底层存放以及动画插值等原理请看 [glTF格式详解(示例：一个简单的动画)](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F65326596 "https://zhuanlan.zhihu.com/p/65326596") 和 [glTF格式详解(动画)](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F65362919 "https://zhuanlan.zhihu.com/p/65362919")

如果不想深究的同学可以跳过以上两个链接，本篇文章只讲在Three.js的loader载入glTF格式的3D模型之后，我们如何改变动画，专注于实现而不展开原理。

* * *

### 改变模型的动画

1.  首先通过 GLTFLoader 载入带有动画的模型

```js
import turbine from '../../assets/3d-gltf-model/turbine.glb';

loadTurbine = () => {
    const loader = new GLTFLoader()
    if (this.scene.getObjectByName('turbine')) {
        let removeTurbine = this.scene.getObjectByName('turbine')
        this.scene.remove(removeTurbine)
    }
    
    loader.load(turbine, object => {
        this.matrixTurbine = object;
        let mesh = object.scene;
        mesh.position.set(0, -2, 0);
        this.scene.add(mesh);
    })
}

```

2.  loader 载入后的 matrixTurbine对象属性

[github](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flebron-li%2Fwind-3d-web "https://github.com/lebron-li/wind-3d-web") 的示例中的glTF模型是自带动画的，所以在load模型之后形成一个对象之后，动画相关的数据就存放在 matrixTurbine对象的属性中

*   matrixTurbine对象 的 animations属性是一个数组，里面存了AnimationsClip，是一组可重复使用的关键帧轨迹，代表了gltf自带的动画。

AnimationsClip 中的 duration 是这组动画持续的时间，tracks数组 存放了该动画时间内的动画帧键值对，是一组关键帧轨道，也就是每一个时间单位所对应的部件位置（因为该 glTF模型 有透明和不透明两种展示模式，所以图中的 tracks数组 有两个 QuaternionKeyframeTrack对象）。不断重复这组关键帧轨迹，就可以得到持续不断的动画

*   QuaternionKeyframeTrack 对象的 times属性 和 values属性

QuaternionKeyframeTrack 对象的 times属性 是一个关键帧的时间数组；QuaternionKeyframeTrack 对象的 values属性 是一个与时间数组中的时间点对应的值数组。

怎么理解呢？

我们先打印这两个属性看一看是什么。我们可以看到，times的长度是values的四分之一

```js
console.log(this.matrixTurbine.animations[0].tracks[1].times)
console.log(this.matrixTurbine.animations[0].tracks[1].values)



```

我们可以想象 values属性 就是一组3D对象需要做的动作，times是这些动作所完成的时刻，比如在上文中，每一个times数组的每一个item就对应了values中的四个item，通过glTF内部的动画插值算法来形成一个完整连续的动画。 具体原理可能又得开一篇文章讲，在此先挖个坑，暂且形象地理解一下。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b72cbeb0a4a14d9c92209ffc8cd5671e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

3.  通过改变 times属性数组 来改变动画的快慢

之前我们提到了animations的duration属性，其就是一轮动画持续的时间，times数组实际上是对该时间的均匀分割。比如duration = 6，此时 `Float32Array(145)` 就是为 \[0, 0.0416666679084301, 0.0833333358168602, 0.125 ...... 5.958333492279053, 6\]。

如果我要改变动画的快慢，只需要根据后端传来的 风速windSpeed 来成比例的修改 times数组 和 duration

```js

this.matrixTurbine.animations[0].tracks[1].times = this.changeArr(object.animations[0].tracks[1].times)
this.matrixTurbine.animations[0].tracks[0].times = this.changeArr(object.animations[0].tracks[0].times) 
this.matrixTurbine.animations[0].duration = 6 / (windSpeed / 5)

function changeArr(arr){
    return arr.map((a) => a / (windSpeed / 5))
}

```

* * *

### 给无动画的gltf模型增加动画

这个例子主要讲的是如何改变有动画的glTF模型，如果遇上没有自带动画的3D模型，可以尝试一下操作：

不是直接使用构造函数实例化 AnimationClip，您可以使用其静态方法之一来创建 AnimationClips：从 JSON (解析)、从变形目标序列 (CreateFromMorphTargetSequence、CreateClipsFromMorphTargetSequences) 或从动画层次结构 (parseAnimation) - 如果您的模型尚未在其几何体的动画数组中保存 AnimationClips。

雷暴天气的模拟
-------

还有一个需求就是希望可以根据后端的天气数据接口渲染天气，我就以雷暴天气作为例子来展开关于纹理、材质和几何体相关的知识。

* * *

### 雷暴天气演示

我们可以看到，不定时会有闪电的出现，而且乌云密布且呈现流动的状态

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c536623de2bb44bab7f4572ecee17274~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

* * *

### 乌云密布的实现

该实现主要和 几何体、纹理、材质相关

1.  基于3D空间坐标，在风机上方新建数个位置随机的平面几何体，同时设置一个非物理的只计算漫反射关注光照朗伯材质

我们先要知道一个基础的3D知识：一个网格(Mesh)由几何体和材质组成，整个3D场景的Object可以近似看作数个Mesh的组合。 一片乌云即可作为一个Mesh，由几何体和材质组成。代码中的 cloud 就是一个由几何体和材质组成的一个Mesh，表示基于以三角形为 [polygon mesh](https://link.juejin.cn/?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FPolygon_mesh "https://en.wikipedia.org/wiki/Polygon_mesh")（多边形网格）的物体的类。

很明显，该Mesh的几何性质可以近似为一个平面，数个 平面几何体PlaneBufferGeometry 被随机平铺开来就形成了上空中的云层，作为 “乌云” 的几何载体。

由于在闪电照亮乌云时，乌云本身并不会出现近似于镜面反射的效果，更接近于漫反射的效果，所以我们可以使用 Three.js 的 MeshLambertMaterial 材质构造函数来创建暗淡的并不光亮的表面，主要反映的是光线的漫反射。

```js

const cloudGeo = new THREE.PlaneBufferGeometry(400, 400);
const cloudMaterial = new THREE.MeshLambertMaterial({
    map: texture,
    transparent: true
});

```

2.  Mesh中的材质性质如果需要起视觉作用，需要和纹理进行结合，也就是常说的“贴图”

通过 `let cloud = new THREE.Mesh(cloudGeo, cloudMaterial)` 这行代码，框架内置的投影方程和映射函数将 Mesh 映射到纹理空间，然后纹理空间检索设置的smoke.png中的颜色值，然后值变换函数对检索见过进行值变换用于改变几何体表面属性，换句话说就是把smoke.png“贴”到了所创建的包含了几何体和材质的Mesh上。

3.  设置多个Mesh的位置

本代码中设置了5个 Mesh，随机地放在了3D风机的上方的空间上，看起来就像“罩住”了风机，读者在自己实现的时候可以借此熟悉Three.js的三维坐标系。

```js
for (let p = 0; p < 5; p++){
    let cloud = new THREE.Mesh(cloudGeo, cloudMaterial); 
    cloud.position.set( Math.random() * 10 + 90, Math.random() * 20 + 15, -Math.random() * 50 - 80 );
}

```

* * *

### 乌云流动的实现

`rotation`:表示物体绕x,y,z轴旋转的弧度。在动画函数中，将所有乌云Mesh绕z轴进行旋转，然后就是使用requestAnimationFrame进行动画的一个套路了，在此不再赘述

```js
function animate() {
    cloudParticles.forEach(p => { 
        p.rotation.z -= 0.002;
    });
}
requestAnimationFrame(animate);

```

* * *

### 雷暴闪电的实现

闪电的实现和光照模型有关，Three.js中的光种类主要包含环境光AmbientLight、 平行光DirectionalLight 和 点光源PointLight。

1.  使用的光照类型

点光源的光线是发散的，无法直接定义它的光线方向，这与闪电的发光特性类似，所以我们使用点光源来模拟闪电。

在每一帧动画渲染时，可能都会在乌云Mesh的上方的随机位置产生一个点光源，光源照到乌云和风机模型，光线会和两者的几何纹理以及顶点法向量进行计算，从而改变大量顶点的颜色，重新进入渲染管线，从而模拟出闪电的效果

```js
wholeFlashGroup.name = 'flash'
const flash = new THREE.PointLight(0xe0ffff, 10000, 0, 2);
flash.position.set(100, 100, -110);
wholeFlashGroup.add(flash);

function animate() {
    if (Math.random() > 0.90 || flash.power > 220) {
        if (flash.power < 100)
            flash.position.set(
                Math.random() * 30 + 80,
                Math.random() * 20 + 10,
                Math.random() * 3 - 100
            );
        flash.power = 50 + Math.random() * 500;
    }
    requestAnimationFrame(animate);
}

```

* * *

### 总结：关于雷暴天气部分的主要代码

```js

loadFlash = () => {
    const wholeFlashGroup = new THREE.Group();
    wholeFlashGroup.name = 'flash'
    const ambient = new THREE.AmbientLight(0x555555);
    wholeFlashGroup.add(ambient);
    const directionalLight = new THREE.DirectionalLight(0xffeedd);
    directionalLight.position.set(0, 0, 1);
    wholeFlashGroup.add(directionalLight);
    const flash = new THREE.PointLight(0xe0ffff, 10000, 0, 2);
    flash.position.set(100, 100, -110);
    wholeFlashGroup.add(flash);
    this.myRef.current.appendChild(renderer.domElement);

    let loader = new THREE.TextureLoader();
    loader.load(smoke, (texture) => {
        const cloudGeo = new THREE.PlaneBufferGeometry(400, 400);
        const cloudMaterial = new THREE.MeshLambertMaterial({
            map: texture,
            transparent: true
        });
        for (let p = 0; p < 5; p++) {
            let cloud = new THREE.Mesh(cloudGeo, cloudMaterial);
            cloud.position.set(
                Math.random() * 10 + 90,
                Math.random() * 20 + 15,
                -Math.random() * 50 - 80
            );
            cloud.rotation.x = 1.16;
            cloud.rotation.y = -0.12;
            cloud.rotation.z = Math.random() * 360;
            cloud.material.opacity = 0.4;
            cloudParticles.push(cloud);
            wholeFlashGroup.add(cloud);
        }
        this.scene.add(wholeFlashGroup)
        animate();
    });

    const animate = () => {
        cloudParticles.forEach(p => { 
            p.rotation.z -= 0.002;
        });
        if (Math.random() > 0.90 || flash.power > 220) {
            if (flash.power < 100)
                flash.position.set(
                    Math.random() * 30 + 80,
                    Math.random() * 20 + 10,
                    Math.random() * 3 - 100
                );
            flash.power = 50 + Math.random() * 500;
        }
        flashAnimatation = requestAnimationFrame(animate);
    }
}

```

glTF 3D模型的交互
------------

### 交互的第一步：鼠标点击获得3D对象

1.  将鼠标的屏幕坐标转换为标准设备坐标

获得3D对象进行交互并不像在2D平面那么简单，核心矛盾在于模型对象在内存中是以3维的形式存在的，但是我们鼠标点击的坐标都是屏幕中基于XY轴的2维坐标

所以首先就要先将鼠标点击的2维屏幕坐标位置变成一个 Three.js 的标准化设备坐标(下文中“注”的反过程)，Three.js 通过标准化设备坐标就可以创建拾取射线来获得3维空间中的对象。

注：OpenGL希望在所有顶点着色器运行后，所有我们可见的顶点都变为标准化设备坐标(Normalized Device Coordinate, NDC)。也就是说，每个方向上的坐标都应该在-1.0到1.0之间，超出这个坐标范围的顶点都将不可见。我们通常会自己设定一个坐标的范围，之后再在顶点着色器中将这些坐标转换为标准化设备坐标。然后将这些标准化设备坐标传入光栅器(Rasterizer)，再将他们转换为屏幕上的二维坐标或像素。

```js


this.mouse.x = (event.clientX / w) * 2 - 1;
this.mouse.y = -(event.clientY / h) * 2 + 1;

```

2.  创建拾取射线获取相交的对象

拾取射线 就像一个穿行在3维空间的一个烤串签子，把所有与它相交的3D对象“串起来”形成一个数组，数组的第一个item一般就是我们点击得到的对象。

通过`raycaster.setFromCamera(mouse, camera)`，即标准化设备坐标的鼠标坐标和camera创建拾取射线

通过`raycaster.intersectObject(equipment, true)`计算与拾取射线相交的对象，返回射线旋转的所有对象，该方法的参数是一个Object3D对象构成的数组，表示射线对象的选择范围，凡是选中的都会以数组的形式返回，如果两个mesh屏幕坐标位置是重合的，那么都会被选中

3.  总结：鼠标点击获得3D模型对象的代码

```js
function onPointerClick (event){
    const [w, h] = [window.innerWidth, window.innerHeight];
    const {mouse, equipment, raycaster} = this;
    this.mouse.x = (event.clientX / w) * 2 - 1;
    this.mouse.y = -(event.clientY / h) * 2 + 1;
    raycaster.setFromCamera(mouse, this.camera);
    const intersects = raycaster.intersectObject(equipment, true);
    if (intersects.length <= 0) {
        return false;
    }
    const selectedObject = intersects[0].object;
    if (selectedObject.isMesh) {
        
    }
}

```

* * *

### 3D模型的交互效果

在这里我主要写其中的一个最重要的交互效果的实现：点击各部件，3D部件外缘轮廓发光，并显示 label 可展示其详情

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1257d1bcff584e2cb183260f61c116ce~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

1.  后期处理过程链EffectComposer

交互的效果其实多种多样，但是都需要创建产生最终视觉效果的后期处理过程链 EffectComposer，先渲染原场景，再叠加额外的交互渲染效果

*   new EffectComposer( renderer : WebGLRenderer, renderTarget : WebGLRenderTarget ) 用于在three.js中实现后期处理效果。EffectComposer 类管理了产生最终视觉效果的后期处理过程链。 后期处理过程根据它们添加/插入的顺序来执行，最后一个过程会被自动渲染到屏幕上。

```js
let compose = new EffectComposer(this.renderer);
compose.addPass(firstPass);  
compose.addPass(secondPass); 

```

2.  3D部件边缘轮廓光

为了给3D模型增加轮廓光，我使用了`three-outlinepass`库，首先调用RenderPass渲染原来的场景后，再渲染增加轮廓光的outlinePass

*   new RenderPass(scene, camera)，一个 RenderPass 就是渲染管线的单次运行。一个 RenderPass 将图像渲染到内存中的帧缓冲附件中。在渲染过程开始时，每个附件需要在块状内存中初始化而且在渲染结束时可能需要写回到内存中。
    
*   关于 OutlinePass 的学习，可以参考 [three.js examples (OutlinePass)](https://link.juejin.cn/?target=https%3A%2F%2Fthreejs.org%2Fexamples%2F%3Fq%3Dout%23webgl_postprocessing_outline "https://threejs.org/examples/?q=out#webgl_postprocessing_outline")这个demo进行学习，这里不再展开。
    

```js

function outline (selectedObjects, color = 0x15c5e8) {
    const [w, h] = [window.innerWidth, window.innerHeight];
    let compose = new EffectComposer(this.renderer);
    let renderPass = new RenderPass(this.scene, this.camera);
    let outlinePass = new OutlinePass(
        new THREE.Vector2(w, h),
        this.scene,
        this.camera,
        selectedObjects
    );
    outlinePass.renderToScreen = true;
    outlinePass.selectedObjects = selectedObjects;
    compose.addPass(renderPass);  
    compose.addPass(outlinePass); 
    const params = {
        edgeStrength: 3,
        edgeGlow: 0,
        edgeThickness: 20,
        pulsePeriod: 1,
        usePatternTexture: false
    };
    outlinePass.edgeStrength = params.edgeStrength;
    outlinePass.edgeGlow = params.edgeGlow;
    outlinePass.visibleEdgeColor.set(color);
    outlinePass.hiddenEdgeColor.set(color);
    compose.render(this.scene, this.camera);
    this.compose = compose
}

```

结束语
---

这篇文章没有过多的原理上的展开，因为3D 方面需要补充的原理太多了，可能会打乱文章的连续性，所以相关学习资料在文中以链接的形式呈现，请读者自行查阅。

我一直觉得3D和VR将会成为前端一个非常重要的方向，希望这篇文章能够帮助你