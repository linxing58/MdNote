# cesium之流动线纹理实现的两种方式_cesium 流动线_逸然健在的博客-CSDN博客
![](https://img-blog.csdnimg.cn/img_convert/d89ed33df3557cc237cb01d21979bb93.png)

直接上代码吧

方法一：采用自定义[shader](https://so.csdn.net/so/search?q=shader&spm=1001.2101.3001.7020) 的实现，利用cesium内置的glsl变量是纹理随着时间按照指定方向进行流动。效果图中科技感的数字流动是呈现沿着线往上流动，这种效果很适合在智慧城市数字孪生的场景中结合其他的三维地物作为装饰。我们可以看到wall的方向跟线的方向流动的方向是不一样的，wall 的流动方向是横着流动，这是着色器中的纹理方向的设置相关，我这里没有把wall的代码放出来。不过，如何更改流动方向，我相信聪明的你应该清楚如何更改了。赶紧去试一试吧。这种方式用的是addPrimiFlowline 方法 

```null
urce1 =  "czm_material czm_getMaterial(czm_materialInput materialInput)\n\
```

方法二：根据[cesium](https://so.csdn.net/so/search?q=cesium&spm=1001.2101.3001.7020) 内置的材质类型实现。具体介绍各位可以去API文档查看，有哪些uniforms和各个属性代表的意思也写的很清楚。示例代码看addPrimitiveFlowAppear方法。

![](https://img-blog.csdnimg.cn/img_convert/26f26abb1ca99b4273b18f3105bce911.png)

要注意:

1、cesium绘制地物有两种方式，一种是通过entity的方式。entity是cesium封装的一种高级接口，很适合上手入门。entity添加的地物可以使用如以下已经封装好的materialProperty 来组合各种材质效果，enetity详细内容参考官方文档，具体使用例子可以参考前面文章[cesium property实现飞行实时姿态仿真](http://mp.weixin.qq.com/s?__biz=MzkwMzMwNTg2NQ==&mid=2247483717&idx=1&sn=3c51d42efbe7703a2108c8c01975633a&chksm=c0990ea3f7ee87b5d47d02d5263bb83eb9031cb4f5afb076c11b16b405e509a7810833e5771b&scene=21#wechat_redirect "cesium property实现飞行实时姿态仿真")中飞行尾部轨迹的实现

![](https://img-blog.csdnimg.cn/img_convert/caa3d6d53c87de3558fa135fed5d7d06.png)

     另一种是primitive的方式，本文中这两个都是通过primitive来绘制几何体。他们的材质跟前面讲的property是不一样的机制，cesium通过primitive提高了渲染的自由度，让精通[GLSL](https://so.csdn.net/so/search?q=GLSL&spm=1001.2101.3001.7020)编程的开发者有更大的发挥舞台。在接口使用上的区别是：1、primitive通过appearance来给material赋值。2、上面列举的materialproperty是不可以在这里使用的，但是内置的几种类型材质可以使用，第二种实现方式实现就是利用内置的类型。

2、第二种实现方式中，通过内置材质类型的uniforms中的time必须是变化才能使其材质产生流动效果，因此为了让其变换，我们将其放到requstAnimation中，进行修改。有没有更好的实现方式呢，我们下期进行探讨。

![](https://img-blog.csdnimg.cn/d34562b226304a77af25223e8da21611.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6YC454S25YGl5Zyo,size_20,color_FFFFFF,t_70,g_se,x_16)