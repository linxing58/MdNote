# Three特效-扩散波_我想当喝水人的博客-CSDN博客_three 扩散效果
使用 Three.js 来创建[智慧城市](https://so.csdn.net/so/search?q=%E6%99%BA%E6%85%A7%E5%9F%8E%E5%B8%82&spm=1001.2101.3001.7020)场景中的扩散波动画，主要原理是使用控制 mesh 的缩放，效果图：  
![](https://img-blog.csdnimg.cn/20210109144245657.gif#pic_center)

贴图素材：  
![](https://img-blog.csdnimg.cn/2021010914430314.png#pic_center)

1.  创建圆柱几何体  
    CylinderGeometry(radiusTop : Float, radiusBottom : Float, height : Float, radialSegments : Integer, heightSegments : Integer, openEnded : Boolean, thetaStart : Float, thetaLength : Float)  
    radiusTop — 圆柱的顶部半径，默认值是 1。  
    radiusBottom — 圆柱的底部半径，默认值是 1。  
    height — 圆柱的高度，默认值是 1。  
    radialSegments — 圆柱侧面周围的分段数，默认为 8。  
    heightSegments — 圆柱侧面沿着其高度的分段数，默认值为 1。  
    openEnded — 一个 Boolean 值，指明该圆锥的底面是开放的还是封顶的。默认值为 false，即其底面默认是封顶的。  
    thetaStart — 第一个分段的起始角度，默认为 0。（three o’clock position）  
    thetaLength — 圆柱底面圆扇区的中心角，通常被称为 “θ”（西塔）。默认值是 2\*Pi，这使其成为一个完整的圆柱。

```csharp
let geometry = new THREE.CylinderGeometry(4, 4, 2, 64);

```

2.  创建多材质并生成 mesh

```csharp
let texture = new THREE.TextureLoader().load("zhu.png")
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping; 
  texture.repeat.set(1, 1)
  texture.needsUpdate = true

let materials = [
    new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.DoubleSide,
      transparent: true
    }),
    new THREE.MeshBasicMaterial({
      transparent: true,
      opacity: 0,
      side: THREE.DoubleSide
    }),
    new THREE.MeshBasicMaterial({
      transparent: true,
      opacity: 0,
      side: THREE.DoubleSide
    })
  ]
 
 let mesh = new THREE.Mesh(geometry, materials)

 scene.add(mesh)
  

```

```javascript
let texture = new THREE.TextureLoader().load("ball.png")
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping; 
  texture.repeat.set(1, 1)
  texture.needsUpdate = true


  let geometry = new THREE.CylinderGeometry(4, 4, 2, 64);
  let materials = [
    new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.DoubleSide,
      transparent: true
    }),
    new THREE.MeshBasicMaterial({
      transparent: true,
      opacity: 0,
      side: THREE.DoubleSide
    }),
    new THREE.MeshBasicMaterial({
      transparent: true,
      opacity: 0,
      side: THREE.DoubleSide
    })
  ]
  let mesh = new THREE.Mesh(geometry, materials)

  scene.add(mesh)

  let s = 0;
  let p = 1;
  function animate() {
	
    s += 0.01;
    p -= 0.005;
    if (s > 2) {
      s = 0;
      p = 1;
    }
    mesh.scale.set(1 + s, 1, 1 + s);
    mesh.material[0].opacity = p;
    
    requestAnimationFrame(animate)
  }

  animate()

```

 [https://blog.csdn.net/qq_39503511/article/details/112391607](https://blog.csdn.net/qq_39503511/article/details/112391607)
