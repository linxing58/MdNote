# threejs gltf 修改materail材质颜色与贴图 贴图不生效_tcmmx1993的博客-CSDN博客_threejs修改材质颜色
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

异常：可修改颜色但不能加贴图，图片路径无误

实际使用中的部分关键代码

````null
var texture = new THREE.TextureLoader().load( "texture/Bricks052_1K_Color.jpg" );        texture.encoding = THREE.sRGBEncoding;        texture.needsUpdate = true;        texture.wrapS = THREE.RepeatWrapping;        texture.wrapT = THREE.RepeatWrapping;        texture.repeat.set( 4, 4 );var tmaterial = new THREE.MeshStandardMaterial({ map: texture });loader.load('model/000.glb',( gltf )=> {            gltf.scene.children[0].material = tmaterial            gltf.scene.children[0].material.needsUpdate  = true;            gltf.scene.traverse( function ( child ) {                child.material.emissive =  child.material.color;                child.material.emissiveMap = child.material.map ;const senc = gltf.scene || gltf.scene[0];          }, undefined, function ( error ) {console.error('加载错误：'+ error );```

代码经使用盒模型测试替换效果是无误的

但是gltf模型只能改变颜色却无法显示贴图，经过一番苦苦搜索发现问题应该是模型本身的设置问题

导出的原模型不论有无贴图都须添加UVmap设置才能在threejs中操作,这里以Blender软件界面为例

![](https://img-blog.csdnimg.cn/20210325095807923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjbW14MTk5Mw==,size_16,color_FFFFFF,t_70) 
 [https://blog.csdn.net/tcmmx1993/article/details/115197506](https://blog.csdn.net/tcmmx1993/article/details/115197506)
````
