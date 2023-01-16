# cesium之shader 内置变量、常量、函数 － 小专栏
### 内置uniform

内置uniform主要置于AutomaticUniforms类里面，该类私有未开放文档。

*   czm_backgroundColor

> 代表当前场景背景颜色的自动GLSL制服。
> 
> ##### 例：
> 
> ```null
> 统一vec4 czm_backgroundColor;
> vec4 AdjustColorForContrast（vec4颜色）
> {
>  如果（czm_backgroundColor.rgb == color.rgb）
>  {
>      color.rgb = vec3（1.0）-color.rgb;
>  }
>  返回颜色；
> }
> 
> ```

*   czm_brdfLut

> 包含BRDF查找纹理的自动GLSL制服，用于基于图像的照明计算。
> 
> ##### 例：
> 
> ```null
> 统一采样器2D czm_brdfLut;
> 浮点粗糙度＝ 0.5；
> float NdotV =点（法线，视图）；
> vec2 brdfLut = texture2D（czm_brdfLut，vec2（NdotV，1.0-粗糙度））。rg;
> 
> ```

*   [czm_currentFrustum](https://cesiumjs.org/releases/b28/Documentation/czm_currentFrustum.html?classFilter=&show=glsl)

> 包含相机定义的平截头体的近距离（`x`）和远距离（）的自动GLSL制服`y`。这是用于多视锥渲染的单个视锥。
> 
> ##### 例：
> 
> ```null
> 统一vec2 czm_currentFrustum;
> 浮动frustumLength = czm_currentFrustum.y-czm_currentFrustum.x;
> 
> ```

*   [czm_encodedCameraPositionMCHigh](https://cesiumjs.org/releases/b28/Documentation/czm_encodedCameraPositionMCHigh.html?classFilter=&show=glsl)

> 自动GLSL制服，代表模型坐标中摄像机位置的高位。如[Precisions，Precisions中](http://blogs.agi.com/insight3d/index.php/2008/09/03/precisions-precisions/)所述，它用于GPU RTE以消除渲染时的抖动伪影 。
> 
> ##### 例：
> 
> ```null
> 统一vec3 czm_encodedCameraPositionMCHigh;
> 
> ```

*   [czm_encodedCameraPositionMCLow](https://cesiumjs.org/releases/b28/Documentation/czm_encodedCameraPositionMCLow.html?classFilter=&show=glsl)

> 自动GLSL制服，表示模型坐标中摄像机位置的低位。如[Precisions，Precisions中](http://blogs.agi.com/insight3d/index.php/2008/09/03/precisions-precisions/)所述，它用于GPU RTE以消除渲染时的抖动伪影 。
> 
> ##### 例：
> 
> ```null
> 统一vec3 czm_encodedCameraPositionMCLow;
> 
> ```

*   [czm_entireFrustum](https://cesiumjs.org/releases/b28/Documentation/czm_entireFrustum.html?classFilter=&show=glsl)

> 包含相机定义的平截头体的近距离（`x`）和远距离（）的自动GLSL制服`y`。这是可能的最大视锥，而不是用于多视锥渲染的单个视锥。
> 
> ##### 例：
> 
> ```null
> 统一vec2 czm_entireFrustum;
> 浮动frustumLength = czm_entireFrustum.y-czm_entireFrustum.x;
> 
> ```

*   czm_environmentMap

> 包含场景内使用的环境贴图的自动GLSL制服。
> 
> ##### 例：
> 
> ```null
> 统一采样器多维数据集czm_environmentMap;
> 浮动反射=反射（视图，正常）；
> vec4 ReflectionColor = textureCube（czm_environmentMap，反映）;
> 
> ```

*   [czm_eyeHeight2D](https://cesiumjs.org/releases/b28/Documentation/czm_eyeHeight2D.html?classFilter=&show=glsl)

> 自动GLSL制服，其中包含以米为单位的2D场景中眼睛（相机）的高度（`x`）和高度的平方（`y`）。

*   czm_fogDensity

> 自动GLSL均匀标量，用于根据距相机的距离将颜色与雾色混合。

*   [czm_frameNumber](https://cesiumjs.org/releases/b28/Documentation/czm_frameNumber.html?classFilter=&show=glsl)

> 表示帧号的自动GLSL制服。该制服每帧自动增加。

*   czm_frustumPlanes

> 到视锥平面的距离。顶部，底部，左侧和右侧的距离分别是x，y，z和w分量。

*   czm_geometricToleranceOverMeter

> 自动GLSL均匀标量，表示每米的几何公差

*   czm_imagerySplitPosition

> 自动GLSL制服，表示在使用分割器渲染图像图层时要使用的分割器位置。这将是相对于画布的像素坐标。

*   [czm_infiniteProjection](https://cesiumjs.org/releases/b28/Documentation/czm_infiniteProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个4x4投影变换矩阵，其远平面为无穷大，它将眼睛坐标转换为剪辑坐标。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。无限远平面用于阴影体积和具有代理几何体的GPU射线投射等算法中，以确保三角形不会被远平面夹住。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_infiniteProjection;
> gl_Position = czm_infiniteProjection * eyePosition;
> 
> ```

*   [czm_inverseModel](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModel.html?classFilter=&show=glsl)

> 自动GLSL制服，代表将世界坐标转换为模型坐标的4x4模型转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModel;
> vec4 modelPosition = czm_inverseModel * worldPosition;
> 
> ```

*   [czm_inverseModelView](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelView.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从眼睛坐标转换为模型坐标的4x4转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelView;
> vec4 modelPosition = czm_inverseModelView * eyePosition;
> 
> ```

*   [czm_inverseModelView3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从眼睛坐标转换为3D模型坐标的4x4转换矩阵。在3D模式下，这与[czm_inverseModelView](https://infoearth.yuque.com/space/kf6g27/czm_inverseModelView.html)相同 ，但是在2D和Columbus View中，它表示模型逆视图矩阵，就好像相机在3D模式下处于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelView3D;
> vec4 modelPosition = czm_inverseModelView3D * eyePosition;
> 
> ```

*   [czm_inverseModelViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将剪辑坐标转换为模型坐标的4x4逆模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelViewProjection;
> vec4 modelPosition = czm_inverseModelViewProjection * clipPosition;
> 
> ```

*   [czm_inverseNormal](https://cesiumjs.org/releases/b28/Documentation/czm_inverseNormal.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个3x3法向变换矩阵，该矩阵将眼坐标中的法向矢量变换为模型坐标。这与[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)提供的转换相反 。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseNormal;
> vec3 normalMC = czm_inverseNormal * normalEC;
> 
> ```

*   [czm_inverseNormal3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseNormal3D.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个3x3法线变换矩阵，该矩阵将眼坐标中的法向矢量变换为3D模型坐标。这与[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)提供的转换相反 。在3D模式下，这与[czm_inverseNormal](https://infoearth.yuque.com/space/kf6g27/czm_inverseNormal.html)相同 ，但是在2D和Columbus View中，它表示逆法线变换矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseNormal3D;
> vec3 normalMC = czm_inverseNormal3D * normalEC;
> 
> ```

*   [czm_inverseProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示从剪辑坐标转换为眼睛坐标的4x4反投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseProjection;
> vec4 eyePosition = czm_inverseProjection * clipPosition;
> 
> ```

*   [czm_inverseView](https://cesiumjs.org/releases/b28/Documentation/czm_inverseView.html?classFilter=&show=glsl)

> 自动GLSL制服，表示从眼睛坐标转换为世界坐标的4x4转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseView;
> vec4 worldPosition = czm_inverseView * eyePosition;
> 
> ```

*   [czm_inverseView3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从3D眼睛坐标转换为世界坐标的4x4转换矩阵。在3D模式下，这与[czm_inverseView](https://infoearth.yuque.com/space/kf6g27/czm_inverseView.html)相同 ，但是在2D和Columbus View中，它表示反视图矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseView3D;
> vec4 worldPosition = czm_inverseView3D * eyePosition;
> 
> ```

*   [czm_inverseViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将剪辑坐标转换为世界坐标的4x4视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseViewProjection;
> vec4 worldPosition = czm_inverseViewProjection * clipPosition;
> 
> ```

*   [czm_inverseViewRotation](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewRotation.html?classFilter=&show=glsl)

> 表示3x3旋转矩阵的自动GLSL制服，该矩阵将向量从眼睛坐标转换为世界坐标。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseViewRotation;
> vec4 worldVector = czm_inverseViewRotation * eyeVector;
> 
> ```

*   [czm_inverseViewRotation3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewRotation3D.html?classFilter=&show=glsl)

> 表示3x3旋转矩阵的自动GLSL制服，该矩阵将向量从3D眼睛坐标转换为世界坐标。在3D模式下，这与[czm_inverseViewRotation](https://infoearth.yuque.com/space/kf6g27/czm_inverseViewRotation.html)相同 ，但是在2D和Columbus View中，它表示反视图矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseViewRotation3D;
> vec4 worldVector = czm_inverseViewRotation3D * eyeVector;
> 
> ```

*   czm_invertClassificationColor

> 自动GLSL制服，将成为未分类3D瓷砖的突出显示颜色。

*   czm_log2FarPlusOne

> 自动GLSL制服，包含远距离+ 1.0的log2。在反转对数深度计算时使用。

*   czm_log2NearDistance

> 包含近距离log2的自动GLSL制服。在片段着色器中写入日志深度时使用。

*   czm_minimumDisableDepthTestDistance

> 自动GLSL制服，表示到摄像机的距离，在该距离上禁用广告牌，标签和点的深度测试，以例如防止剪切地形。设置为零时，应始终应用深度测试。如果小于零，则永远不要应用深度测试。

*   [czm_model](https://cesiumjs.org/releases/b28/Documentation/czm_model.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为世界坐标的4x4模型转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_model;
> vec4 worldPosition = czm_model * modelPosition;
> 
> ```

*   [czm_modelView](https://cesiumjs.org/releases/b28/Documentation/czm_modelView.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为眼睛坐标的4x4模型视图转换矩阵。
> 
> 位置应使用转换为眼睛坐标 `czm_modelView` ，法线应使用[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)转换 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelView;
> vec4 eyePosition = czm_modelView * modelPosition;
> vec4 eyePosition = czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelView3D](https://cesiumjs.org/releases/b28/Documentation/czm_modelView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将3D模型坐标转换为眼睛坐标的4x4模型视图转换矩阵。在3D模式下，这与[czm_modelView](https://infoearth.yuque.com/space/kf6g27/czm_modelView.html)相同 ，但是在2D和Columbus View中，它表示模型视图矩阵，就好像照相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> 位置应使用转换为眼睛坐标 `czm_modelView3D` ，法线应使用[czm_normal3D](https://infoearth.yuque.com/space/kf6g27/czm_normal3D.html)转换 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelView3D;
> vec4 eyePosition = czm_modelView3D * modelPosition;
> vec4 eyePosition = czm_view3D * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewInfiniteProjection](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewInfiniteProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为剪辑坐标的4x4模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。投影矩阵将远平面放置在无穷远处。这在诸如阴影体积和具有代理几何体的GPU射线投射之类的算法中很有用，以确保三角形不会被远端平面修剪。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewInfiniteProjection;
> vec4 gl_Position = czm_modelViewInfiniteProjection * modelPosition;
> gl_Position = czm_infiniteProjection * czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为剪辑坐标的4x4模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewProjection;
> vec4 gl_Position = czm_modelViewProjection * modelPosition;
> gl_Position = czm_projection * czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewProjectionRelativeToEye](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewProjectionRelativeToEye.html?classFilter=&show=glsl)

> 自动GLSL制服，表示4x4模型-视图-投影转换矩阵，该矩阵将相对于眼睛的模型坐标转换为裁剪坐标。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。与czm_translateRelativeToEye结合使用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewProjectionRelativeToEye;
> vec3 positionHigh属性；
> 属性vec3 positionLow;
> 无效main（）
> {
> vec4 p = czm_translateRelativeToEye（positionHigh，positionLow）;
> gl_Position = czm_modelViewProjectionRelativeToEye * p;
> }
> 
> ```

*   [czm_modelViewRelativeToEye](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewRelativeToEye.html?classFilter=&show=glsl)

> 自动GLSL制服，代表4x4模型-视图转换矩阵，该矩阵将相对于眼睛的模型坐标转换为眼睛坐标。与czm_translateRelativeToEye结合使用。

### 内置uniform

内置uniform主要置于AutomaticUniforms类里面，该类私有未开放文档。

*   czm_backgroundColor

> 代表当前场景背景颜色的自动GLSL制服。
> 
> ##### 例：
> 
> ```null
> 统一vec4 czm_backgroundColor;
> vec4 AdjustColorForContrast（vec4颜色）
> {
>  如果（czm_backgroundColor.rgb == color.rgb）
>  {
>      color.rgb = vec3（1.0）-color.rgb;
>  }
>  返回颜色；
> }
> 
> ```

*   czm_brdfLut

> 包含BRDF查找纹理的自动GLSL制服，用于基于图像的照明计算。
> 
> ##### 例：
> 
> ```null
> 统一采样器2D czm_brdfLut;
> 浮点粗糙度＝ 0.5；
> float NdotV =点（法线，视图）；
> vec2 brdfLut = texture2D（czm_brdfLut，vec2（NdotV，1.0-粗糙度））。rg;
> 
> ```

*   [czm_currentFrustum](https://cesiumjs.org/releases/b28/Documentation/czm_currentFrustum.html?classFilter=&show=glsl)

> 包含相机定义的平截头体的近距离（`x`）和远距离（）的自动GLSL制服`y`。这是用于多视锥渲染的单个视锥。
> 
> ##### 例：
> 
> ```null
> 统一vec2 czm_currentFrustum;
> 浮动frustumLength = czm_currentFrustum.y-czm_currentFrustum.x;
> 
> ```

*   [czm_encodedCameraPositionMCHigh](https://cesiumjs.org/releases/b28/Documentation/czm_encodedCameraPositionMCHigh.html?classFilter=&show=glsl)

> 自动GLSL制服，代表模型坐标中摄像机位置的高位。如[Precisions，Precisions中](http://blogs.agi.com/insight3d/index.php/2008/09/03/precisions-precisions/)所述，它用于GPU RTE以消除渲染时的抖动伪影 。
> 
> ##### 例：
> 
> ```null
> 统一vec3 czm_encodedCameraPositionMCHigh;
> 
> ```

*   [czm_encodedCameraPositionMCLow](https://cesiumjs.org/releases/b28/Documentation/czm_encodedCameraPositionMCLow.html?classFilter=&show=glsl)

> 自动GLSL制服，表示模型坐标中摄像机位置的低位。如[Precisions，Precisions中](http://blogs.agi.com/insight3d/index.php/2008/09/03/precisions-precisions/)所述，它用于GPU RTE以消除渲染时的抖动伪影 。
> 
> ##### 例：
> 
> ```null
> 统一vec3 czm_encodedCameraPositionMCLow;
> 
> ```

*   [czm_entireFrustum](https://cesiumjs.org/releases/b28/Documentation/czm_entireFrustum.html?classFilter=&show=glsl)

> 包含相机定义的平截头体的近距离（`x`）和远距离（）的自动GLSL制服`y`。这是可能的最大视锥，而不是用于多视锥渲染的单个视锥。
> 
> ##### 例：
> 
> ```null
> 统一vec2 czm_entireFrustum;
> 浮动frustumLength = czm_entireFrustum.y-czm_entireFrustum.x;
> 
> ```

*   czm_environmentMap

> 包含场景内使用的环境贴图的自动GLSL制服。
> 
> ##### 例：
> 
> ```null
> 统一采样器多维数据集czm_environmentMap;
> 浮动反射=反射（视图，正常）；
> vec4 ReflectionColor = textureCube（czm_environmentMap，反映）;
> 
> ```

*   [czm_eyeHeight2D](https://cesiumjs.org/releases/b28/Documentation/czm_eyeHeight2D.html?classFilter=&show=glsl)

> 自动GLSL制服，其中包含以米为单位的2D场景中眼睛（相机）的高度（`x`）和高度的平方（`y`）。

*   czm_fogDensity

> 自动GLSL均匀标量，用于根据距相机的距离将颜色与雾色混合。

*   [czm_frameNumber](https://cesiumjs.org/releases/b28/Documentation/czm_frameNumber.html?classFilter=&show=glsl)

> 表示帧号的自动GLSL制服。该制服每帧自动增加。

*   czm_frustumPlanes

> 到视锥平面的距离。顶部，底部，左侧和右侧的距离分别是x，y，z和w分量。

*   czm_geometricToleranceOverMeter

> 自动GLSL均匀标量，表示每米的几何公差

*   czm_imagerySplitPosition

> 自动GLSL制服，表示在使用分割器渲染图像图层时要使用的分割器位置。这将是相对于画布的像素坐标。

*   [czm_infiniteProjection](https://cesiumjs.org/releases/b28/Documentation/czm_infiniteProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个4x4投影变换矩阵，其远平面为无穷大，它将眼睛坐标转换为剪辑坐标。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。无限远平面用于阴影体积和具有代理几何体的GPU射线投射等算法中，以确保三角形不会被远平面夹住。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_infiniteProjection;
> gl_Position = czm_infiniteProjection * eyePosition;
> 
> ```

*   [czm_inverseModel](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModel.html?classFilter=&show=glsl)

> 自动GLSL制服，代表将世界坐标转换为模型坐标的4x4模型转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModel;
> vec4 modelPosition = czm_inverseModel * worldPosition;
> 
> ```

*   [czm_inverseModelView](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelView.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从眼睛坐标转换为模型坐标的4x4转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelView;
> vec4 modelPosition = czm_inverseModelView * eyePosition;
> 
> ```

*   [czm_inverseModelView3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从眼睛坐标转换为3D模型坐标的4x4转换矩阵。在3D模式下，这与[czm_inverseModelView](https://infoearth.yuque.com/space/kf6g27/czm_inverseModelView.html)相同 ，但是在2D和Columbus View中，它表示模型逆视图矩阵，就好像相机在3D模式下处于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelView3D;
> vec4 modelPosition = czm_inverseModelView3D * eyePosition;
> 
> ```

*   [czm_inverseModelViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseModelViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将剪辑坐标转换为模型坐标的4x4逆模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseModelViewProjection;
> vec4 modelPosition = czm_inverseModelViewProjection * clipPosition;
> 
> ```

*   [czm_inverseNormal](https://cesiumjs.org/releases/b28/Documentation/czm_inverseNormal.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个3x3法向变换矩阵，该矩阵将眼坐标中的法向矢量变换为模型坐标。这与[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)提供的转换相反 。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseNormal;
> vec3 normalMC = czm_inverseNormal * normalEC;
> 
> ```

*   [czm_inverseNormal3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseNormal3D.html?classFilter=&show=glsl)

> 自动GLSL制服，表示一个3x3法线变换矩阵，该矩阵将眼坐标中的法向矢量变换为3D模型坐标。这与[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)提供的转换相反 。在3D模式下，这与[czm_inverseNormal](https://infoearth.yuque.com/space/kf6g27/czm_inverseNormal.html)相同 ，但是在2D和Columbus View中，它表示逆法线变换矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseNormal3D;
> vec3 normalMC = czm_inverseNormal3D * normalEC;
> 
> ```

*   [czm_inverseProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示从剪辑坐标转换为眼睛坐标的4x4反投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseProjection;
> vec4 eyePosition = czm_inverseProjection * clipPosition;
> 
> ```

*   [czm_inverseView](https://cesiumjs.org/releases/b28/Documentation/czm_inverseView.html?classFilter=&show=glsl)

> 自动GLSL制服，表示从眼睛坐标转换为世界坐标的4x4转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseView;
> vec4 worldPosition = czm_inverseView * eyePosition;
> 
> ```

*   [czm_inverseView3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，代表从3D眼睛坐标转换为世界坐标的4x4转换矩阵。在3D模式下，这与[czm_inverseView](https://infoearth.yuque.com/space/kf6g27/czm_inverseView.html)相同 ，但是在2D和Columbus View中，它表示反视图矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseView3D;
> vec4 worldPosition = czm_inverseView3D * eyePosition;
> 
> ```

*   [czm_inverseViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将剪辑坐标转换为世界坐标的4x4视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_inverseViewProjection;
> vec4 worldPosition = czm_inverseViewProjection * clipPosition;
> 
> ```

*   [czm_inverseViewRotation](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewRotation.html?classFilter=&show=glsl)

> 表示3x3旋转矩阵的自动GLSL制服，该矩阵将向量从眼睛坐标转换为世界坐标。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseViewRotation;
> vec4 worldVector = czm_inverseViewRotation * eyeVector;
> 
> ```

*   [czm_inverseViewRotation3D](https://cesiumjs.org/releases/b28/Documentation/czm_inverseViewRotation3D.html?classFilter=&show=glsl)

> 表示3x3旋转矩阵的自动GLSL制服，该矩阵将向量从3D眼睛坐标转换为世界坐标。在3D模式下，这与[czm_inverseViewRotation](https://infoearth.yuque.com/space/kf6g27/czm_inverseViewRotation.html)相同 ，但是在2D和Columbus View中，它表示反视图矩阵，就好像相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> ##### 例：
> 
> ```null
> 统一的mat3 czm_inverseViewRotation3D;
> vec4 worldVector = czm_inverseViewRotation3D * eyeVector;
> 
> ```

*   czm_invertClassificationColor

> 自动GLSL制服，将成为未分类3D瓷砖的突出显示颜色。

*   czm_log2FarPlusOne

> 自动GLSL制服，包含远距离+ 1.0的log2。在反转对数深度计算时使用。

*   czm_log2NearDistance

> 包含近距离log2的自动GLSL制服。在片段着色器中写入日志深度时使用。

*   czm_minimumDisableDepthTestDistance

> 自动GLSL制服，表示到摄像机的距离，在该距离上禁用广告牌，标签和点的深度测试，以例如防止剪切地形。设置为零时，应始终应用深度测试。如果小于零，则永远不要应用深度测试。

*   [czm_model](https://cesiumjs.org/releases/b28/Documentation/czm_model.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为世界坐标的4x4模型转换矩阵。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_model;
> vec4 worldPosition = czm_model * modelPosition;
> 
> ```

*   [czm_modelView](https://cesiumjs.org/releases/b28/Documentation/czm_modelView.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为眼睛坐标的4x4模型视图转换矩阵。
> 
> 位置应使用转换为眼睛坐标 `czm_modelView` ，法线应使用[czm_normal](https://infoearth.yuque.com/space/kf6g27/czm_normal.html)转换 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelView;
> vec4 eyePosition = czm_modelView * modelPosition;
> vec4 eyePosition = czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelView3D](https://cesiumjs.org/releases/b28/Documentation/czm_modelView3D.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将3D模型坐标转换为眼睛坐标的4x4模型视图转换矩阵。在3D模式下，这与[czm_modelView](https://infoearth.yuque.com/space/kf6g27/czm_modelView.html)相同 ，但是在2D和Columbus View中，它表示模型视图矩阵，就好像照相机在3D模式下位于等效位置一样。这与以3D照亮的相同方式照亮2D和Columbus View很有用。
> 
> 位置应使用转换为眼睛坐标 `czm_modelView3D` ，法线应使用[czm_normal3D](https://infoearth.yuque.com/space/kf6g27/czm_normal3D.html)转换 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelView3D;
> vec4 eyePosition = czm_modelView3D * modelPosition;
> vec4 eyePosition = czm_view3D * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewInfiniteProjection](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewInfiniteProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为剪辑坐标的4x4模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。投影矩阵将远平面放置在无穷远处。这在诸如阴影体积和具有代理几何体的GPU射线投射之类的算法中很有用，以确保三角形不会被远端平面修剪。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewInfiniteProjection;
> vec4 gl_Position = czm_modelViewInfiniteProjection * modelPosition;
> gl_Position = czm_infiniteProjection * czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewProjection](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewProjection.html?classFilter=&show=glsl)

> 自动GLSL制服，表示将模型坐标转换为剪辑坐标的4x4模型-视图-投影转换矩阵。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewProjection;
> vec4 gl_Position = czm_modelViewProjection * modelPosition;
> gl_Position = czm_projection * czm_view * czm_model * modelPosition;
> 
> ```

*   [czm_modelViewProjectionRelativeToEye](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewProjectionRelativeToEye.html?classFilter=&show=glsl)

> 自动GLSL制服，表示4x4模型-视图-投影转换矩阵，该矩阵将相对于眼睛的模型坐标转换为裁剪坐标。剪辑坐标是顶点着色器`gl_Position` 输出的坐标系 。与czm_translateRelativeToEye结合使用。
> 
> ##### 例：
> 
> ```null
> 统一的mat4 czm_modelViewProjectionRelativeToEye;
> vec3 positionHigh属性；
> 属性vec3 positionLow;
> 无效main（）
> {
> vec4 p = czm_translateRelativeToEye（positionHigh，positionLow）;
> gl_Position = czm_modelViewProjectionRelativeToEye * p;
> }
> 
> ```

*   [czm_modelViewRelativeToEye](https://cesiumjs.org/releases/b28/Documentation/czm_modelViewRelativeToEye.html?classFilter=&show=glsl)

> 自动GLSL制服，代表4x4模型-视图转换矩阵，该矩阵将相对于眼睛的模型坐标转换为眼睛坐标。与czm_translateRelativeToEye结合使用。