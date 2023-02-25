# Three特效-道路流光特效_我想当喝水人的博客-CSDN博客_threejs 流光
使用 Three.js 来创建[智慧城市](https://so.csdn.net/so/search?q=%E6%99%BA%E6%85%A7%E5%9F%8E%E5%B8%82&spm=1001.2101.3001.7020)场景中的道路流光动画，主要原理是使用贴图动画，效果图：  
![](https://img-blog.csdnimg.cn/20210109140534330.gif#pic_center)

贴图素材：  
![](https://img-blog.csdnimg.cn/20210109142059972.png#pic_center)

1.  创建道路顶点数组

```csharp

  let points = [new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(10, 0, 0),
    new THREE.Vector3(10, 0, 10),
    new THREE.Vector3(0, 0, 10)
  ]

```

2.  使用 CatmullRomCurve3 生成曲线  
    CatmullRomCurve3（点：数组，闭合：布尔值，curveType：字符串，张力：浮点）  
    points – Vector3 点数组  
    closed –该曲线是否闭合，替换为假  
    curveType –曲线的类型，交替变量向心。  
    张力–曲线的张力，交替为 0.5。

```csharp
let curve = new THREE.CatmullRomCurve3(points) 

```

3.  使用 TubeGeometry 创建管道  
    TubeGeometry（路径：曲线，管状段：整数，半径：浮点，径向段：整数，封闭：布尔）  
    tubularSegments —整数 - 组成该管道的分段数，相互之间的距离为 64。radius —浮点 - 管道的 截面，其大小为 1。radialSegments—整数 -- 路径 - 曲线 - 一个由基类曲线继承而来的路径。close —布尔型管道的交替是否闭合，替换为 false。

```csharp

let tubeGeometry = new THREE.TubeGeometry(curve, 80, 0.1)

```

4.  创建材质并在动画函数中使用贴图位移

```csharp
  let texture = new THREE.TextureLoader().load("line.png")
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping; 
  texture.repeat.set(1, 1)
  texture.needsUpdate = true

  let material = new THREE.MeshBasicMaterial({
    map: texture,
    side: THREE.BackSide,
    transparent: true
  })


```

5.  创建 mesh

```csharp
let mesh = new THREE.Mesh(tubeGeometry, material);

scene.add(mesh)


```

```javascript
let texture = new THREE.TextureLoader().load("line.png")
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping; 
  texture.repeat.set(1, 1)
  texture.needsUpdate = true

  let material = new THREE.MeshBasicMaterial({
    map: texture,
    side: THREE.BackSide,
    transparent: true
  })

  
  let points = [new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(10, 0, 0),
    new THREE.Vector3(10, 0, 10),
    new THREE.Vector3(0, 0, 10)
  ]

  
  let curve = new THREE.CatmullRomCurve3(points) 

  
  let tubeGeometry = new THREE.TubeGeometry(curve, 80, 0.1)
  
  let mesh = new THREE.Mesh(tubeGeometry, material);

  scene.add(mesh)

  function animate() {
	
    if(texture) texture.offset.x -= 0.01
    requestAnimationFrame(animate)
  }

  animate()

```

 [https://blog.csdn.net/qq_39503511/article/details/112391060](https://blog.csdn.net/qq_39503511/article/details/112391060)
