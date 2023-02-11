# GLSL内置变量详解 - 莫小龙 - 博客园
GLSL内置变量详解
----------

顶点属性：
-----

![](https://common.cnblogs.com/images/copycode.gif)

attribute vec4 gl_Color;              // 顶点颜色
attribute vec4 gl_SecondaryColor;     // 辅助顶点颜色
attribute vec3 gl_Normal;             // 顶点法线
attribute vec4 gl_Vertex;             // 顶点物体空间坐标（未变换）
attribute vec4 gl_MultiTexCoord\[0-N\]; // 顶点纹理坐标（N = gl_MaxTextureCoords）
attribute float gl_FogCoord;          // 顶点雾坐标

![](https://common.cnblogs.com/images/copycode.gif)

用法示例：

//OpenGL将光源的方向保存在视点空间坐标系内，因此需要把法线也变换到视点空间。
vec4 normal = gl\_NormalMatrix * gl\_Normal;

vec4 ecPosition = gl\_ModelViewMatrix * gl\_Vertex;   //求取顶点变换到相机空间的位置

vec2 Texcoord = gl_MultiTexCoord0.xy;     //用于向片元着色器传递纹理坐标
uniform sampler2D decal;                  //片元着色器中的纹理采样器
vec4 color = texture2D(decal, Texcoord);  //获取纹理颜色

一致变量：
-----

内置的一致变量的值最终是通过OpenGL的函数传递过去的。当用户没有使用着色器(即可编程管线)编写程序的时候，程序走固定管线完成绘制，但，一旦用户使用了着色器编写GLSL程序，就必须将固定管线的那部分功能用GLSL程序模仿出来！！！   
e.g. 当使用着色器实现其他功能的时候，灯光效果、雾效等也都必须有相应的GLSL实现代码。

![](https://common.cnblogs.com/images/copycode.gif)

/*矩阵状态*/ uniform mat4 gl_ModelViewMatrix; // 模型视图变换矩阵
uniform mat4 gl_ProjectMatrix;                  // 投影矩阵
uniform mat4 gl_ModelViewProjectMatrix;         // 模型视图投影变换矩阵（ftransform()）
uniform mat3 gl_NormalMatrix;                   // 法向量变换到视空间矩阵
uniform mat4 gl\_TextureMatrix\[gl\_MatTextureCoords\];     // 各纹理变换矩阵

/*法线专用缩放因子*/ uniform float gl_NormalScale;   //示例：normal = noraml * gl_NormalScale;

/*窗口坐标中深度范围*/ struct gl_DepthRangeParameters
{ float near;  //*剪切*面
    float far;   //远剪切*面
    float diff;  //进剪切*面到远剪切*面的距离差
};
uniform gl\_DepthRangeParameters gl\_DepthRange; /*裁剪*面*/ uniform vec4 gl\_ClipPlane\[gl\_MaxClipPlanes\]; /*点属性*/ struct gl_PointParameters
{ float size; float sizeMin; float sizeMax; float fadeThresholdSize; float distanceConstantAttenuation; float distanceLinearAttenuation; float distanceQuadraticAttenuation;
};
uniform gl\_PointParameters gl\_Point; /*材质，对应于OSG中的Material类，下述参数由该类的函数传入*/ struct gl_MaterialParameters
{
    vec4 emission; //自身发出的光
    vec4 ambient;       //对环境光的反射能力
    vec4 diffuse;       //对散射光的反射能力
    vec4 specular;      //对镜面光的反射能力
    float shininess;    //镜面指数，示例：float spec = pow(max(dot(eyeVec, reflectVec), 0.0), shininess); 
                        //镜面指数越大，镜面反射光越强，散射程度越小，高光点就越集中。
};
uniform gl\_MaterialParameters gl\_FrontMaterial; // 正面材质
uniform gl\_MaterialParameters gl\_BackMaterial;        // 反面材质

/*光源性质，对应于OSG中的Light类，下述参数由该类的函数传入*/ struct gl_LightSourceParameters
{
    vec4 ambient; //环境光的颜色
    vec4 diffuse;                //散射光的颜色，反映了在场景中光源对RGBA各成分的散射能力
    vec4 specular;               //镜面反射光的颜色，它直接影响着材质上高光的颜色，通常为白色或灰色
    vec4 position;               //光源在场景中的放置位置
    vec4 halfVector;             //半角向量，示例：halfVector = normalize(单位入射向量 + 单位观察者向量);
    vec3 spotDirection;          //聚光灯方向，表示圆锥体的轴
    float spotExponent;          //聚光指数，表示从圆锥的中心轴向外表面变化时光强度的衰减
    float spotCutoff;            //聚光灯的切角，取值范围为\[0.0, 90.0\]和180，光锥的角度为为此角的2倍
    float spotCosCutoff;         //为上个量的cos值，取值范围为\[1.0, 0.0\]和-1.0
    float constantAttenuation;   //常量衰减因子
    float linearAttenuation;     //线性衰减因子
    float quadraticAttenuation;  //二次衰减因子
    //示例：attenuation = 1.0 / (gl_LightSource\[0\].constantAttenuation + 
    //gl\_LightSource\[0\].linearAttenuation * d + gl\_LightSource\[0\].quadraticAttenuation * d * d);
};
uniform gl\_LightSourceParameters gl\_LightSource\[gl_MaxLights\];

struct gl_LightModelParameters
{
    vec4 ambient; //整个场景的环境光的RGBA强度
};
uniform gl\_LightModelParameters gl\_LightModel; /*光照和材质的派生状态*/ struct gl_LightModelProducts
{
    vec4 sceneColor; //等于gl\_FrontMaterial.emission + gl\_FrontMaterial.ambient * gl_LightModel.ambient;
};
uniform gl\_LightModelProducts gl\_FrontLightModelProduct; 
uniform gl\_LightModelProducts gl\_BackLightModelProduct;

struct gl_LightProducts
{
    vec4 ambient; //等于gl\_FrontMaterial.ambient  * gl\_LightSource\[0\].ambient
    vec4 diffuse;    //等于gl\_FrontMaterial.diffuse  * gl\_LightSource\[0\].diffuse
    vec4 specular;   //等于gl\_FrontMaterial.specular * gl\_LightSource\[0\].specular
};
uniform gl\_LightProducts gl\_FrontLightProduct\[gl_MaxLights\];
uniform gl\_LightProducts gl\_BackLightProduct\[gl_MaxLights\]; /*纹理环境和生成*/ unifrom vec4 gl\_TextureEnvColor\[gl\_MaxTextureImageUnits\];
unifrom vec4 gl\_EyePlaneS\[gl\_MaxTextureCoords\];　
unifrom vec4 gl\_EyePlaneT\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_EyePlaneR\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_EyePlaneQ\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_ObjectPlaneS\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_ObjectPlaneT\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_ObjectPlaneR\[gl\_MaxTextureCoords\];
unifrom vec4 gl\_ObjectPlaneQ\[gl\_MaxTextureCoords\]; /*雾参数，对应于OSG的Fog类，下述参数由该类的函数传入*/ struct gl_FogParameters
{
    vec4  color; float density; float start; float end; float scale;  // 1/(end-start)
};
uniform gl\_FogParameters gl\_Fog;

![](https://common.cnblogs.com/images/copycode.gif)

易变变量
----

varying vec4 gl_Color;
varying vec4 gl_SecondaryColor;
varying vec4 gl\_TexCoord\[gl\_MaxTextureCoords\];
varying float gl_FogFragCoord;

原文：https://blog.csdn.net/u014587123/article/details/80626652

顶点属性
----

```null
attribute vec4 gl_Color;              
attribute vec4 gl_SecondaryColor;     
attribute vec3 gl_Normal;             
attribute vec4 gl_Vertex;             
attribute vec4 gl_MultiTexCoord[0-N]; 
attribute float gl_FogCoord;          
```

用法示例：

```null

vec4 normal = gl_NormalMatrix * gl_Normal;
```

```null
vec4 ecPosition = gl_ModelViewMatrix * gl_Vertex;   
```

```null
vec2 Texcoord = gl_MultiTexCoord0.xy;     
uniform sampler2D decal;                  
vec4 color = texture2D(decal, Texcoord);  
```

一致变量
----

内置的一致变量的值最终是通过OpenGL的函数传递过去的。当用户没有使用着色器(即可编程管线)编写程序的时候，程序走固定管线完成绘制，但，一旦用户使用了着色器编写GLSL程序，就必须将固定管线的那部分功能用GLSL程序模仿出来！！！   
e.g. 当使用着色器实现其他功能的时候，灯光效果、雾效等也都必须有相应的GLSL实现代码。

```null

uniform mat4 gl_ModelViewMatrix;                
uniform mat4 gl_ProjectMatrix;                  
uniform mat4 gl_ModelViewProjectMatrix;         
uniform mat3 gl_NormalMatrix;                   
uniform mat4 gl_TextureMatrix[gl_MatTextureCoords];     


uniform float gl_NormalScale;   


struct gl_DepthRangeParameters
{
    float near;  
    float far;   
    float diff;  
};
uniform gl_DepthRangeParameters gl_DepthRange;


uniform vec4 gl_ClipPlane[gl_MaxClipPlanes];


struct gl_PointParameters
{
    float size;
    float sizeMin;
    float sizeMax;
    float fadeThresholdSize;
    float distanceConstantAttenuation;
    float distanceLinearAttenuation;
    float distanceQuadraticAttenuation;
};
uniform gl_PointParameters gl_Point;


struct gl_MaterialParameters
{
    vec4 emission;      
    vec4 ambient;       
    vec4 diffuse;       
    vec4 specular;      
    float shininess;    
                        
};
uniform gl_MaterialParameters gl_FrontMaterial;       
uniform gl_MaterialParameters gl_BackMaterial;        


struct gl_LightSourceParameters
{
    vec4 ambient;                
    vec4 diffuse;                
    vec4 specular;               
    vec4 position;               
    vec4 halfVector;             
    vec3 spotDirection;          
    float spotExponent;          
    float spotCutoff;            
    float spotCosCutoff;         
    float constantAttenuation;   
    float linearAttenuation;     
    float quadraticAttenuation;  
    
    
};
uniform gl_LightSourceParameters gl_LightSource[gl_MaxLights];

struct gl_LightModelParameters
{
    vec4 ambient;    
};
uniform gl_LightModelParameters gl_LightModel;


struct gl_LightModelProducts
{
    vec4 sceneColor; 
};
uniform gl_LightModelProducts gl_FrontLightModelProduct; 
uniform gl_LightModelProducts gl_BackLightModelProduct;

struct gl_LightProducts
{
    vec4 ambient;    
    vec4 diffuse;    
    vec4 specular;   
};
uniform gl_LightProducts gl_FrontLightProduct[gl_MaxLights];
uniform gl_LightProducts gl_BackLightProduct[gl_MaxLights];


unifrom vec4 gl_TextureEnvColor[gl_MaxTextureImageUnits];
unifrom vec4 gl_EyePlaneS[gl_MaxTextureCoords];　
unifrom vec4 gl_EyePlaneT[gl_MaxTextureCoords];
unifrom vec4 gl_EyePlaneR[gl_MaxTextureCoords];
unifrom vec4 gl_EyePlaneQ[gl_MaxTextureCoords];
unifrom vec4 gl_ObjectPlaneS[gl_MaxTextureCoords];
unifrom vec4 gl_ObjectPlaneT[gl_MaxTextureCoords];
unifrom vec4 gl_ObjectPlaneR[gl_MaxTextureCoords];
unifrom vec4 gl_ObjectPlaneQ[gl_MaxTextureCoords];


struct gl_FogParameters
{
    vec4  color;
    float density;
    float start;
    float end;
    float scale;  
};
uniform gl_FogParameters gl_Fog;
```

易变变量
----

```null
varying vec4 gl_Color;
varying vec4 gl_SecondaryColor;
varying vec4 gl_TexCoord[gl_MaxTextureCoords];
varying float gl_FogFragCoord;
```