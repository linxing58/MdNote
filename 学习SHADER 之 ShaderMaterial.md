# 学习SHADER 之 ShaderMaterial 
> 根据自己的理解，翻译了 ShaderMaterial，供大家学习使用

## 简介：

material 用自己写的着色器来展现，Shader 是一个 glsl 语言写的小程序，运行在 GPU 上。

## 用到它的场景：

-   实现一个效果不用已写好的 Material
-   把许多对象合并为一个 Geometry 或 BufferGeometry 以提高性能

## 以下是 ShaderMaterial 的使用注意事项：

-   ShaderMaterial 只会用于 WebGLRenderer，vertexShader 和 fragmentShader 属性必须使用 Webgl 编译运行
-   对于 THREE 的 r72 版本，不能直接用 ShaderMaterial 的属性。必须把 Geometry 替换为 BufferGeometry，用 BufferAttribute d 的实例自定义属性
-   对于 THREE 的 r77 版本，WebGlRenderTarget 或 WebGLRenderTargetCube 不能用于 uniforms 中。 他们的 texture 属性必须被替换
-   编写 attributes 和 uniforms 会被传递到 shader 中，如果你不想 webGLProgram 添加其他东西到你的 shader 中，用 RawShaderMaterial 这个类替代。
-   你可以使用 #pragma unroll_loop 指令，用 for 循环处理 GLSL 材质，必须符合标准：循环必须规范， 循环的变量必须是 i ，循环使用特定的空白格式

```text
#pragma unroll_loop
for ( int i = 0; i < 10; i ++ ) {
	// ...
}
```

## 例子：

```text
var material = new THREE.ShaderMaterial( {
	uniforms: {
		time: { value: 1.0 },
		resolution: { value: new THREE.Vector2() }
	},
	vertexShader: document.getElementById( 'vertexShader' ).textContent,
	fragmentShader: document.getElementById( 'fragmentShader' ).textContent


} );     
```

## 顶点着色器和片段着色器

可以给每个材质指定两种类型的 shader：

-   顶点着色器先运行，它接收属性值（attributes），然后计算并修改每个顶点的位置，并且传递附加的数据（varyings）给片元着色器。
-   片元着色器后运行，它接收顶点着色器的数据，并给每个单独的片元（像素）赋值，渲染到屏幕中。

Shader 中有三种类型的数据： uniforms, attributes 和 varyings。

-   Uniforms 是提供给所有的顶点接收的变量 —— 光照（lighting）雾（fog）和阴影地图（shadow maps）这些数据都可存储在 uniforms 中。Uniforms 可以被 顶点着色器和片元着色器接收。
-   Attributes 是每个顶点提供的变量 —— 比如：attributes 中可以存储 顶点的坐标，面法向量和顶点颜色值。Attributes 只能用于顶点着色器中。
-   Varyings 是从顶点着色器传递到片元着色器的变量。对于每个片元（像素），每个 varying 的值会被直接传入相邻顶点中。

请注意：在着色器中，Uniforms、Attributes 像常量一样，想修改他们的值，只能将值传递给缓冲区来修改。

## 内置 Attributes 和 Uniforms

WebGLRenderer 在默认情况下，为着色器提供了许多的 Attributes 和 Uniforms； 在编译着色器的时候，这些定义的变量将被 WebGLProgram 预先添加到您的 fragmentShader 和 vertexShader 中，不用自己去定义。请参阅 [WebGLProgram](https://link.zhihu.com/?target=https%3A//threejs.org/docs/%23api/renderers/webgl/WebGLProgram) 了解更多。

有些 Uniforms 或 Attributes （比如附加的光照和雾）需要设置一些属性，以便让 WebGLRender 把值复制给 GPU。如果在 shader 中用这些属性，就要设置这些值。

如果你不想让 WebGLProgram 给你的 shader 代码加任何东西，用 RawShaderMaterial 这个类吧。

## 自定义 Attributes 和 Uniforms

Attributes 和 Uniforms 都必须在 glsl 着色器代码中进行声明（在 vertexShader 或 fragmentShader 中）自定义的 Uniforms 必须在 ShaderMaterial 的 Uniforms 属性中声明，而任何的自定义 Attributes 必须通过 BufferGeometry 实例定义。 请注意，Varyings 只需要在着色器代码中声明，不需要在材质中声明。

要声明一个自定义的属性，请参阅 BufferGeometry 了解概述，以及 BufferAttribute 以了解它的 API

当创建属性的时候，每个您创建的类型数组都应该是数据类型大小的背书。比如，如果您的 Attributes 是 THREE.Vector3 类型，并且在 BufferGeometry 中有 3000 个顶点，就必须创建 长度为 3000 \* 3 的 类型数组值。以下列出每个数据类型大小的表格以供参考。

属性大小

GLSL typeJavaScript typeSizefloatNumber1vec2THREE.Vector22vec3THREE.Vector33vec3THREE.Color3vec4THREE.Vector44

请注意，Attributes 在缓冲区的值在变化的时候不会自动更新，要更新 Attributes，请在几何体的 BufferAttribute 上将 needUpdate 设置为 true （详情参阅 BufferGeometry）

要声明一个自定义的 Uniform， 请使用 Uniforms 的属性

```text
uniforms: {
    time: { value: 1.0 },
    resolution: { value: new THREE.Vector2() }
}
```

推荐您根据 Object3D.onBeforeRender 的 对象 和 照相机来更新自定义的 Uniform 值，因为材质可以在网格中共享，照相机和场景中的矩阵世界会随着 WebGLRender.render 更新，并且某些效果（eg: VREffect）会用他们自己的摄像机来渲染场景。

## 构造函数

## ShaderMaterial（参数：对象）

参数 ： 可选的，一个对象中包含一个或多个定义材质外观的属性。任何材质的特性都能从这输入（包含所有 [Material](https://link.zhihu.com/?target=https%3A//threejs.org/docs/%23api/materials/Material) 中的属性）

## 属性

## [Material](https://link.zhihu.com/?target=https%3A//threejs.org/docs/%23api/materials/Material) 基类中的共同属性见链接

## .clipping : Boolean

定义材质是否支持裁剪 clipping ，是否在渲染器中执行 clippingPlanes 的 uniform。默认不执行

## .defaultAttributeValues : Object

如果材质有的 attributes 可是被渲染的几何体没有，这些默认值会被传给着色器。这就避免了缓冲区数据丢失时发生报错。

```text
this.defaultAttributeValues = {
    'color': [ 1, 1, 1 ],
    'uv': [ 0, 0 ],
    'uv2': [ 0, 0 ]
};
```

.defines : Object

* * *

在顶点着色器和片元着色器的 GLSL 代码中，用 #define 指令定义自定义常量，每个键值对是一个指令

每个键值对都会产生另一个指令

```text
defines: {
	FOO: 15,
	BAR: true
}
```

他们会转换为 GLSL 中一行行代码

```text
#define FOO 15
#define BAR true
```

## .extensions : Object

它包含以下属性

```text
this.extensions = {
  derivatives: false, // set to use derivatives
  fragDepth: false, // set to use fragment depth values
  drawBuffers: false, // set to use draw buffers
  shaderTextureLOD: false // set to use shader texture LOD
};
```

## .fog : Boolean

定义是否让材质的颜色收到全局的雾的作用。开启的话，会将雾的 Uniforms 传给着色器，默认关闭此选项。

## .fragmentShader : String

片段着色器的 GLSL 代码。它可以从 DOM 中提取，可以作为字符串传递，也可以用 AJAX 加载。

## .index0AttributeName : String

设置它的话，会调用 [gl.bindAttribLocation](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/bindAttribLocation) 把通用的顶点索引绑定到属性的变量上。默认是 undefined.

## .isShaderMaterial : Boolean

用这个方法检查它或者它的派生类是否是着色器材质。默认是 true

你不能改变这个值，因为它用于内部优化

## .lights : Boolean

定义这个材质是否使用光照； 如果设为 true 就将 uniform 的光照数据传给着色器。默认为 false。

## .linewidth : Float

控制线框的厚度。默认值是 1。

由于 ANGLE 层的限制，在 Windows 平台上，无论设置值如何，线宽始终为 1。

## .morphTargets : Boolean

定义材质是否使用 morphTargets，设为 true 后 morphTargets 属性就会应用到这个着色器了。

## .morphNormals : boolean

定义材质是否使用 morphNormals，设为 true 后 morphTargets 属性就会从 Geometry 传递到这个着色器了，默认是 false。

## .program : WebGLProgram

WebGLRenderer 生成的关于本材质的编译的着色器程序。这个属性一般不用访问到。

## .flatShading : Boolean

定义材质是否以平面着色呈现。默认为 false.

## .skinning : Boolean

定义这个材质是否使用是 skinning； 如果设为 true 就将 uniform 的皮肤数据传给着色器。默认为 false。

## .uniforms : Object

他是一个对象的形式

```text
{ "uniform1": { value: 1.0 }, "uniform2": { value: 2 } }
```

指定传给着色器的 Uniforms； key 是 Uniform 的名字。值像下面这样定义。

这里，value 指的 Uniforms 的值，Names 必须和 GLSL 代码中定义的 Uniforms 的名字匹配。请注意，每一帧都会刷新 Uniforms，所以更新 Uniforms 的值会立刻更新 GLSL 代码中的值。

## .vertexColors : Number

通过定义颜色的属性的填充方法来定义顶点的着色方法。比如 THREE.NoColors, THREE.FaceColors and THREE.VertexColors.

默认是 THREE.NoColors

## .vertexShader : String

片段着色器的 GLSL 代码。它可以从 DOM 中提取，可以作为字符串传递，也可以用 AJAX 加载。

## .wireframe : Boolean

几何图形会被渲染为线框（用 GL_LINES 而不是 GL_TRIANGLES）默认值是 false（即呈现为平面多边形）

## .wireframeLinewidth : Float

控制线框的厚度。默认值是 1。

由于 ANGLE 层的限制，在 Windows 平台上，无论设置值如何，线宽始终为 1。 
 [https://zhuanlan.zhihu.com/p/36931159](https://zhuanlan.zhihu.com/p/36931159)
