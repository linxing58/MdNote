# Cesium实战(七十三)纹理叠加 | cesium中文网
![](http://cesium.xin/wordpress/wp-content/uploads/2021/09/pexels-photo-19213361.jpg)

有朋友想做纹理叠加的效果，他参考的效果是这个

![](http://develop.cesium.xin/image/multimaterial/01.jpg)

其实参考的内容有一些：

1.  沙盒示例 Materials 里面的 BumpMap water 都有效，但是要深入源码
2.  \[Cesium 实战 (二十三)primitive 动态纹理][http://cesium.xin/wordpress/archives/cesium-primitive-dynamic.html](http://cesium.xin/wordpress/archives/cesium-primitive-dynamic.html)

这里我的实现很简陋，抛砖引玉，简单来说就这几步

1.  构建形状
2.  把两个纹理传入着色器
3.  在着色器里把两个颜色相加，然后显示

纹理是我在 photoshop 中随手画的两个 png 文件\~~

![](http://develop.cesium.xin/image/multimaterial/01.png)
![](http://develop.cesium.xin/image/multimaterial/02.png)

代码如下：

```null
var viewer = new Cesium.Viewer('cesiumContainer', {
    imageryProvider: new Cesium.TileMapServiceImageryProvider({
        url: Cesium.buildModuleUrl("Assets/Textures/NaturalEarthII"),
    })
});

let scene = viewer.scene;
scene.primitives.add(
    new Cesium.Primitive({
    geometryInstances: new Cesium.GeometryInstance({
        geometry: new Cesium.RectangleGeometry({
        rectangle: Cesium.Rectangle.fromDegrees(
            -80.0,
            20.0,
            -60.0,
            40.0
        ),
        vertexFormat: Cesium.EllipsoidSurfaceAppearance.VERTEX_FORMAT,
        }),
    }),
    appearance: new Cesium.EllipsoidSurfaceAppearance({
        material:new Cesium.Material({
            fabric: {
            uniforms: {
                image1: "./01.png",
                image2: "./02.png",
            },
            source:
            `
                czm_material czm_getMaterial(czm_materialInput materialInput) { 
                    czm_material material = czm_getDefaultMaterial(materialInput); 
                    
                    float time = czm_frameNumber * 0.001;
                    vec2 st = materialInput.st;
                    
                    vec4 color1 = texture2D(image1, vec2(fract(st.s-time),st.t)); 
                    
                    vec4 color2 = texture2D(image2, st); 
                    
                    vec4 color = mix(color1,color2,0.5);
                    material.diffuse = color.rgb; 
                    material.alpha = color.a; 
                    return material; 
                }
                `
            },
        })
    }),
    })
);
```

效果如下：

视频播放器

ps: 图片是用带柔边的画笔画的，所以边缘有一段透明的边框~ 
 [http://cesium.xin/wordpress/archives/cesium-multimaterial.html](http://cesium.xin/wordpress/archives/cesium-multimaterial.html)
