# Three.js纹理贴图
颜色贴图

.map

normalMap

法线贴图

.bumpMap

凹凸贴图

envMap

环境贴图

纹理贴图

高光贴图

.specularMap

lightMap

光照贴图(阴影贴图)

金属度贴图

metalnessMap

粗糙度贴图

roughnessMap

发光贴图

emissiveMap

![](https://cdn.nlark.com/yuque/0/2021/png/344249/1636512539434-8c652562-b789-4966-beca-5a92fcf0f662.png?x-oss-process=image%2Fresize%2Cw_533%2Climit_0)

  
创建纹理贴图  
通过纹理贴图加载器TextureLoader的load()方法加载一张图片可以返回一个纹理对象Texture，纹理对象Texture可以作为模型材质颜色贴图.map属性的值。  
材质的颜色贴图属性.map设置后，模型会从纹理贴图上采集像素值，这时候一般来说不需要再设置材质颜色.color。.map贴图之所以称之为颜色贴图就是因为网格模型会获得颜色贴图的颜色值RGB。  
​​纹理对象Texture  
​

执行load方法加载纹理图片成功后返回一个纹理对象Texture

.loado方法

回调函数函数参数:纹理对象Texture

TextureLoader纹理加载器对象

卓

异步加载

加载后调用render方法,或者一直循环渲染

顶点uV坐标数据

几何体Geometry

网格模型Mesh

材质的纹理贴图属性map-值:纹理对象texture

材质Material

纹理对象Texture兑?

image属性:image对象,html的img素

创建纹理贴图

加载一张图片,返回一个image对象,可以用来生成纹理

图片加载器ImageLoader

兑

内部使用一文件加载器FileLoader

图片宽高最好是2的次方-比如512x512,64X64....

纹理贴图

支持常见的jpg,png等格式

3D美术:创建模型和纹理贴图纹理贴图如何映射在几何体上

协作:如何映射

程序员:解析3D美术导出的资源

纹理贴图如何映射在几何体上,一般美术导出的楼型会设置好,程序员只需要解析显示就可以

![](https://cdn.nlark.com/yuque/0/2021/png/344249/1636556472192-be7713c4-bfb7-42cc-9cad-14773bba1e4f.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

  
几何体顶点纹理坐标UV  

顶点位置坐标数据

position

顶点颜色数据

color

顶点法线方向向量数据

normal

顶点数据

图片映射

顶点纹理坐标数据

和\]顶点位置,颜色,法向量等数据一一对应

faceVertexUvs\[o\]

UV

Geometry点

faceVertexUvs\[1\]

.attributes.uv

BufferGeometry点

.attributes.uv2

![](https://cdn.nlark.com/yuque/0/2021/png/344249/1636556591251-ed01cc2f-5958-449b-8ed8-d3363e0afcf1.png)

  
纹理UV坐标  
一张纹理贴图图像的坐标，选择一张图片，比如以图片左下角为坐标原点，右上角为坐标(1.0,1.0)，图片上所有位置纵横坐标都介于0.0~1.0之间。  
只要把图片的每个纹理坐标和模型的顶点位置建立一对一的关系，就可以实现图像到模型的映射。  

统狐械科

y2(80,80,

y3(0,80,

uv21.1

41y300.11

三角区域像素1

盘

HOH

三角面2

三角面1

uvoo,o

三角区域像素2

1

11y

10,00

1(80,0,

![](https://cdn.nlark.com/yuque/0/2021/png/344249/1636590914374-e7990681-25fe-456b-a9bf-06680c7aa9f6.png)

  

UVMaPp

3-DModEL

Texture

\-(u,y

p-(X.x3)

![](https://cdn.nlark.com/yuque/0/2021/png/344249/1636590938031-7896076f-e5d4-4fc2-9c5e-219fb9b4c193.png)

  
两组UV坐标  
几何体有两组UV坐标，第一组组用于.map、.normalMap、.specularMap等贴图的映射，第二组用于阴影贴图.lightMap的映射。  
修改纹理坐标  
​原来几何体平面默认是两个三角形构成，把细分数设置为4，三角形数量变为16个。  
​Geometry自定义顶点UV坐标  
通过几何体Geometry自定义了一个由两个三角形组成的矩形几何体，并且通过几何体的.faceVertexUvs\[0\]属性设置了每个顶点对应的第一组UV坐标。  
BufferGeometry自定义顶点UV坐标  
加载一个包含UV坐标的模型文件  
数组材质、材质索引.materialIndex  
数组材质  
材质索引属性  
三角形面Face3可以设置材质索引属性.materialIndex,Face3.materialIndex指向数组材质中的材质对象，表达的意思是数组材质中哪一个元素用于渲染该三角形面Face3。  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
纹理对象Texture阵列、偏移、旋转  
阵列  
偏移  
不设置阵列纹理贴图，只设置偏移  
阵列纹理贴图的同时，进行偏移设置  
纹理旋转  
纹理动画  
纹理动画比较简单，必须要在渲染函数中render()一直执行texture.offset.x -= 0.06动态改变纹理对象Texture的偏移属性.offset就可以。  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
canvas画布、视频作为纹理贴图  
通过Three.js两个类CanvasTexture和VideoTexture可以分别实现把Canvas画布、视频作为纹理贴图使用。  
Canvas画布作为Three.js纹理贴图(CanvasTexture)  
Canvas画布可以通过2D API绘制各种各样的几何形状，可以通过Canvas绘制一个轮廓后然后作为Three.js网格模型、精灵模型等模型对象的纹理贴图。  
把绘制了几何图案的canvas元素作为构造函数CanvasTexture的参数创建一个canvas纹理贴图。  
Canvas画布加载图片  
如果作为纹理贴图使用的Canvas画布加载了图片，注意在图片加载完成的时候更新Threejs相关模型的纹理贴图。如果不更新纹理，你会发现canvas画布上的图片无法现在是Threejs模型的纹理上。  
视频作为Three.js纹理贴图(VideoTexture)  
视频本质上就是一帧帧图片流构成，把视频作为Threejs模型的纹理贴图使用，就是从视频中提取一帧一帧的图片作为模型的纹理贴图，然后不停的更新的纹理贴图就可以产生视频播放的效果。  
VideoTexture.js封装了一个update函数，Threejs每次执行渲染方法进行渲染场景中的时候，都会执行VideoTexture封装的update函数，执行update函数中代码this.needsUpdate = true;读取视频流最新一帧图片来更新Threejs模型纹理贴图。  
凹凸贴图bumpMap和法线贴图.normalMap  
一个复杂的曲面模型，往往模型顶点数量比较多，模型文件比较大，为了降低模型文件大小，法线贴图.normalMap算法自然就产生了，复杂的三维模型3D美术可以通过减面操作把精模简化为简模，然后把精模表面的复杂几何信息映射到法线贴图.normalMap上。  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
法线贴图  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
法线贴图:地球案例

![](https://www.yuque.com/u191126/oe15yv/image.png)

  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
凹凸贴图  
凹凸贴图和法线贴图功能相似，只是没有法线贴图表达的几何体表面信息更丰富。凹凸贴图是用图片像素的灰度值表示几何表面的高低深度，如果模型定义了法线贴图，就没有必要在使用凹凸贴图。  

![](https://www.yuque.com/u191126/oe15yv/image.png)

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
光照贴图添加阴影(·lightMap)  
Geometry属性.faceVertexUvs  
一般几何体有两套UV坐标，对于Geometry类型几何体而言  
Geometry.faceVertexUvs\[0\]包含的纹理坐标用于颜色贴图map、法线贴图normalMap等,Geometry.faceVertexUvs\[1\]包含的第二套纹理贴图用于光照阴影贴图  
一般通过Threejs几何体API创建的几何体默认只有一组纹理坐标Geometry.faceVertexUvs\[0\]，所以为了设置光照阴影贴图，需要给另一组纹理坐标赋值Geometry.faceVertexUvs\[1\] = Geometry.faceVertexUvs\[0\];  
BufferGeometry属性.uv和.uv2  
一般通过Threejs加载外部模型，解析三维模型数据得到的几何体类型是缓冲类型几何体BufferGeometry，对于BufferGeometry而言两套纹理坐标分别通过.uv和.uv2属性表示。  
高光贴图(.specularMap)  
高光网格材质MeshPhongMaterial具有高光属性.specular,如果一个网格模型Mesh都是相同的材质并且表面粗糙度相同,或者说网格模型外表面所有不同区域的镜面反射能力相同，可以直接设置材质的高光属性.specular。如果一个网格模型表示一个人，那么人的不同部位高光程度是不同的，不可能直接通过.specular属性来描述，在这种情况通过高光贴图.specularMap的RGB值来描述不同区域镜面反射的能力，.specularMap和颜色贴图.Map一样和通过UV坐标映射到模型表面。高光贴图.specularMap不同区域像素值不同，表示网格模型不同区域的高光值不同。  
高光贴图属性.specularMap和高光属性.specular是对应的，也就是说只有高光网格材质对象MeshPhongMaterial才具备高光贴图属性.specularMap。  
环境贴图(.envMap)  
Three.js环境贴图.envMap字面意思就是三维模型周边环境，比如你渲染一个立方体，立方体放在一个屋子里面，屋子里面的周边环境肯定影响立方体的渲染效果，目的是为了渲染该立方体而不是立方体周围环境，为了更方便所以没必要创建立方体周边环境所有物体的网格模型，可以通过图片来表达立方体周边的环境。  
创建一个立方体盒子作为天空盒使用，然后把一个环境中上下左右前后六张视图图片作为立方体盒子的纹理贴图使用。  
加环境贴图的6张纹理贴图，可以通过CubeTextureLoader类趋势线。  
高光网格材质MeshPhongMaterial和物理PBR材质MeshStandardMaterial通常会使用环境贴图.envMap来实现更好的渲染效果。一个材质对应的是普通次时代模型，一个材质对应的是PBR模型。  
数据纹理对象DataTexture  
Three.js数据纹理对象DataTexture简单地说就是通过程序创建纹理贴图的每一个像素值。  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  

![](https://www.yuque.com/u191126/oe15yv/image.png)

  
像素值包含RGB三个分量的图片格式有.jpg、.BMP等格式，通过WebGL原生API加载解析这些类型格式的图片需要设置gl.RGB，对于Threejs而言对WebGL进行封装了，gl.RGB对应的设置是THREE.RGBFormat  
像素值包含RGBA四个分量的图片格式有.PNG等格式，通过WebGL原生API加载解析这些类型格式的图片需要设置gl.RGBA，对于Threejs而言对WebGL进行封装了，gl.RGBA对应的设置是THREE.RGBAFormat.