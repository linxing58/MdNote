# 用shader写一个发光材质 - 知乎
在绘制三维图形的时候，不论是三维建模软件还是 Three.js 都提供了很多现成的材质，可供开发者选择，如 Lambert 材质、Phong 材质、法向材质等。然鹅在做开发时，这些材质往往不满足设计需求，比如发光材质就是设计常用的材质，那么如何写出一个透明发光材质呢？

![](https://pic3.zhimg.com/v2-2b52223bd76d7572ec9c433274b697ba_b.jpg)

分析透明发光材质的特点，它是在三维图形结构的边缘处颜色不透明度越高，向内不透明度逐渐降低，其着色跟视角方向强相关。

**敲黑板~** 这是一个经典的**菲涅尔反射**问题。菲尼尔反射描述了一种光学现象，即当光纤照射到物体表面上时，部分光发生反射，部分光发生折射或散射，反射的光和入射的光存在一定比例关系，这个关系可用菲涅尔等式计算。常见的例子如站在湖边，脚边的湖面的水是透明的，可以看到水底的石头，远处的湖面是看不到水下物体的，只能看到水面反射的结果。

![](https://pic2.zhimg.com/v2-a88e03e9ff3ecf3a2c2d9e161a3ed099_b.jpg)

常用的菲涅尔近似等式有：

![](https://www.zhihu.com/equation?tex=F_%7Bschlick%7D%28v%2C+n%29+%3D+F_%7B0%7D+%2B+%281+-+F_%7B0%7D%29%281+-+v%5Ccdot+n%29)

其中 ![](https://www.zhihu.com/equation?tex=F_%7B0%7D)
 是一个反射系数，用来控制反射的强度， ![](https://www.zhihu.com/equation?tex=v)
 是视角方向， ![](https://www.zhihu.com/equation?tex=n)
 是法线方向。

我们用的是另一个广泛应用的等式：

![](https://www.zhihu.com/equation?tex=F_%7BEmpricial%7D%28v%2C+n%29+%3D+max%280%2C+min%281%2C+bias+%2B+scale+%5Ctimes+%281+-+v%5Ccdot+n%29%5E%7Bpower%7D%29%29)

其中 ![](https://www.zhihu.com/equation?tex=bias)
 、 ![](https://www.zhihu.com/equation?tex=scale)
 和 ![](https://www.zhihu.com/equation?tex=power)
 是控制项。

**看图说话**：

![](https://pic3.zhimg.com/v2-65f213d5522491eac122d66ececaba4a_b.jpg)
![](https://pic2.zhimg.com/v2-b297983e1e0fccc2a0255c11f54b2c2d_b.jpg)

令归一化的法线方向为 ![](https://www.zhihu.com/equation?tex=n)
 ，视角方向为 ![](https://www.zhihu.com/equation?tex=v)
 ，那么有表面的透明度与 ![](https://www.zhihu.com/equation?tex=n%5Ccdot+v)
 成反比。当 ![](https://www.zhihu.com/equation?tex=%7C+n%5Ccdot+v+%7C)
 接近 0 时，越不透明，![](https://www.zhihu.com/equation?tex=%7C+n%5Ccdot+v+%7C)
接近 1 时，越透明。

简化后的代码有：

```text
    void main() 
    {
      float a = pow( bias + scale * abs(dot(vNormal, vPositionNormal)), power );
      gl_FragColor = vec4( glowColor, a );
    }
```

其中 bias 值决定了颜色最亮值的位置，power 决定了透明度变化速度及方向。

下图为 bias 取 1.0，power 取 2.0 的效果，scale 取 - 1.0。

![](https://pic1.zhimg.com/v2-e3e4b823376653f03f7633016485d57c_b.jpg)

同理，下图为 bias 取 0，power 取 2.0，scale 取 1.0。

![](https://pic3.zhimg.com/v2-a59d1cf6acabb9c55efd5bcb07e01e06_b.jpg)

此处注意，vNormal 的 normalMatrix 为模型矩阵的逆矩阵的转置。

代码示例请戳：

[https://codepen.io/mysisi/details/xYNWNZ/](https://link.zhihu.com/?target=https%3A//codepen.io/mysisi/details/xYNWNZ/)

参考：

\[1] Unity Shader 入门精要，冯乐乐.

\[2] [Everything has Fresnel](https://link.zhihu.com/?target=http%3A//filmicworlds.com/blog/everything-has-fresnel/)

![](https://pic2.zhimg.com/v2-c502375f73b42074cfd33631e2495aad_b.jpg)

请大家持续关注我们的公众号

我们会不断地分享更多有趣的干货~

笔芯~ 
 [https://zhuanlan.zhihu.com/p/38548428](https://zhuanlan.zhihu.com/p/38548428)
