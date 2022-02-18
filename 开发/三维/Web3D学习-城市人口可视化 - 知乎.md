# Web3D学习-城市人口可视化 - 知乎
上一篇文章中我们进行了城市要素可视化的探索学习

的 demo 中有做了个简易的城市**人口分布可视化**

![](https://pic4.zhimg.com/v2-a48efa1de5916145be6872dac1bebd07_b.jpg)

这篇文章我们将专门的对城市人口数据进行可视化探索，包含了**城市人口的热区，人口迁徙，人口密度等。** 

**i. 和以前的 demo 相比，这个 demo 我们加入一些新的可视化效果 / 组件**

> **建筑物材质使用了 ShaderMaterial**

这个 ShaderMaterial 材质可以使建筑物表面动起来，周期性的更新颜色和条纹, 更能装逼

![](https://pic3.zhimg.com/v2-3be8b26eb6f486e7942607d98a07dbb6_b.jpg)

视屏演示:

相关 shader 代码

```html

 <script id="fragment_shader3" type="x-shader/x-fragment">
        uniform float time;
        varying vec2 vUv;
        void main( void ) {
            vec2 position = vUv;
            float color = 0.0;
            color += sin( position.x * cos( time / 15.0 ) * 80.0 ) + cos( position.y * cos( time / 15.0 ) * 10.0 );
            color += sin( position.y * sin( time / 10.0 ) * 40.0 ) + cos( position.x * sin( time / 25.0 ) * 40.0 );
            color += sin( position.x * sin( time / 5.0 ) * 10.0 ) + sin( position.y * sin( time / 35.0 ) * 80.0 );
            color *= sin( time / 10.0 ) * 0.5;
            gl_FragColor = vec4( vec3( color, color * 0.5, sin( color + time / 3.0 ) * 0.75 ), 1.0 );
        }
    </script>


  <script id="vertexShader" type="x-shader/x-vertex">
        varying vec2 vUv;
        void main()
        {
            vUv = uv;
            vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
            gl_Position = projectionMatrix * mvPosition;
        }
    </script>

```

```js
  const uniforms = {
            time: { value: 1.0 }
   };
  const material = new THREE.ShaderMaterial({
                uniforms: uniforms,
                vertexShader: document.getElementById('vertexShader').textContent,
                fragmentShader: document.getElementById('fragment_shader3').textContent
  });

 //动画函数中
   uniforms.time.value = timestamp / 2000;

```

参考地址 :

> **半球形的发光的罩子，用于覆盖或者标注作用**

![](https://pic3.zhimg.com/v2-84b8e3045ffb0a246ed799f02333f972_b.jpg)

```js
function getGeoemtry(radius = 2000) {
    return new THREE.SphereBufferGeometry(radius, 50, 50, 0, Math.PI);
}

function getMaterial(color = 0x00ffff) {
    const customMaterial = new THREE.ShaderMaterial({
        uniforms:
        {
            "s": { type: "f", value: -1.0 },
            "b": { type: "f", value: 1.0 },
            "p": { type: "f", value: 2.0 },
            glowColor: { type: "c", value: new THREE.Color(color) }
        },
        vertexShader: GlowShader.vertexShader,
        fragmentShader: GlowShader.fragmentShader,
        side: THREE.FrontSide,
        blending: THREE.AdditiveBlending,
        transparent: true
    });
    return customMaterial;
}

```

材质参考

> **加入 TubeTripArcLine**

TubeTripArcLine 是一个可以周期性动态旅行的弧线，该 demo 中用来表示其他城市的人口向上海迁徙的情况

![](https://pic3.zhimg.com/v2-7d92192cd2a3fef5ed811a7e1077646e_b.jpg)

其实现思路为根据起始点计算两点距离，然后在两点的中心点特定高度取一个点，从而形成一个三个点组成的点的数组，然后利用 THREE.CatmullRomCurve3 计算平滑的曲线的点的点集

```js
  const ellipse = new THREE.CatmullRomCurve3([v, vh, v1], false, 'catmullrom');
  const points = ellipse.getPoints(numbers);


   const CatmullRomCurve3 = new THREE.CatmullRomCurve3(points);
   //管道
   const geometry = new THREE.TubeBufferGeometry(CatmullRomCurve3, len, radius, segment, false);

```

然后在动画函数中动态的更新 geometry 的相关参数即可

```js
        const { params } = self;
        const { index, layer, positions, geometries, radius } = params;
        const ps = positions.slice(0, index);
        if (ps.length > 1) {
            let geometry = geometries[index];
            if (!geometry) {
                geometry = getTubeGeometry(ps, layer, radius);
                self.params.geometries[index] = geometry;
            }
            self.geometry = geometry;
            self.geometry.attributes.position.needsUpdate = true;
            self.geometry.attributes.normal.needsUpdate = true;
            self.geometry.attributes.uv.needsUpdate = true;
        }
        if (index === positions.length - 1) {
            self.params.index = -1;
        }
        self.params.index++;

```

> 用来表示人口密度的柱子（bar）

![](https://pic2.zhimg.com/v2-df5fc5b1d46aa52b53ec0e130275a995_b.jpg)

柱子的实现比较简单了, 根据不同的值动态的设置柱子的高度和颜色即可

数据来源 :

```js
function getGeometry(property) {
    const { h, radialSegments, radius } = property;
    return new THREE.CylinderBufferGeometry(radius, radius, h, radialSegments);
}

  function getColor(count) {
                var colors = [
                    '#2c7bb6',
                    '#abd9e9',
                    '#ffffbf',
                    '#fdae61',
                    '#d7191c'
                ];
                if (count < 100) return colors[0];
                if (count >= 100 && count < 1000) return colors[1];
                if (count >= 1000 && count < 2000) return colors[2];
                if (count >= 2000 && count < 5000) return colors[3];
                return colors[4];
   }

   function getMaterial(count, color) {
                if (!color) {
                    color = getColor(count);
                }
                return new THREE.MeshLambertMaterial({ color: color || 'red' });
   }

```

> 3d heatmap,3d 热区 (数据纯属虚构)

![](https://pic2.zhimg.com/v2-af2b8a2824395ebdc429b3e5cffc4c59_b.jpg)

其实现方法参考了

在热区绘制了这里我么有使用 heatmap.js 而是使用了

实现的核心思路为：找出所给数据的外接矩形，从而确定 PlaneBufferGeometry 的长和宽，和其中心点，将数据利用 simpleheat 绘制在 canvas 上，然后 canvas 作为 PlaneBufferGeometry 材质的纹理即可

```js
  //假设canvas 的大小为1024x1024  
    const { canvasheight, canvaswidth, minmercator, maxmercator } = params;
    const minx = minmercator[0], miny = minmercator[1], maxx = maxmercator[0], maxy = maxmercator[1];
    const xaverage = canvaswidth / (maxx - minx);
    const yaverage = canvasheight / (maxy - miny);
    data.forEach(element => {
       //根据经纬度计算点数据在canvas上的平面坐标，会接下来绘制热区做铺垫
        const { mercator } = element;
        const x = (mercator[0] - minx) * xaverage;
        const y = canvasheight - (mercator[1] - miny) * yaverage;
        element.xy = [x, y];
    });

```

> 补间动画（tween.js）

从视频中可以看到 柱子和 3d 热区都做了一个简单的过度动画，这里使用了

实现思路为在动画回调函数中改变 Mesh 的 scale 值，例如 3d heatmap 的动画实现

```js
      if (delay > 0) {
            const SCALE = 0.0001;
            this.scale.set(scaleX * 2, SCALE, scaleY * 2);
            const params = { scale: 0 };
            const tween = this.tween = new TWEEN.Tween(params) // Create a new tween that modifies 'coords'.
                .to({ scale: 1 }, delay) // Move to (300, 200) in 1 second.
                .easing(TWEEN.Easing.Bounce.Out) // Use an easing function to make the animation smooth.
                .onUpdate(() => { // Called after tween.js updates 'coords'.
                    const scale = params.scale;
                    if (scale > 0) {
                        this.scale.set(scaleX * 2, scale, scaleY * 2);
                    }
                }).onComplete(() => {
                    TWEEN.remove(tween);
                });
        } else {
            this.scale.set(scaleX * 2, 1, scaleY * 2);
        }

```

**ii. 相关数据来源** 
 [https://zhuanlan.zhihu.com/p/57825543](https://zhuanlan.zhihu.com/p/57825543)
