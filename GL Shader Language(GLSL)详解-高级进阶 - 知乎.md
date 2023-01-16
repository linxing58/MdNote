# GL Shader Language(GLSL)详解-高级进阶 - 知乎
新一期的教程又来啦，在上一期我们给大家讲解了GL Shader Language(GLSL)的一些基本概念和语法知识，在这期我们给大家带来着色器语言的进阶版教程，有了上一期的铺垫，相信能够让大家由浅入深的理解GL Shader Language(GLSL)。

**一、数据精度**
----------

**1.默认精度**

顶点着色器中默认精度

precision highp float;

precision highp int;

precision lowp sampler2D;

precision lowp samplerCube;

像素着色器中默认精度

precision mediump int;

precision lowp sampler2D;

precision lowp samplerCube;

float 没有默认精度

**2.精度值最低范围**

![](https://pic3.zhimg.com/v2-0c898e66c5582ba192be45378a8d79d2_b.jpg)

**​3.** **推荐使用精度**

vertex position : highp

texture coordinate : midump

colors : lowp

**二、优化策略**
----------

**1.延迟vector计算**

例如：

// 不好的用法

highp float fValue1, fValue2;

highp vec4 vector1, vector2;

vector2= (vector1* fValue1) * fValue2;

// 优化后的用法

highp float fValue1, fValue2;

highp vec4 vector1, vector2;

vector2= vector1 * (fValue1 * fValue2);

**2.** **去冗余计算, vector整体计算**

例如：

// 优化的用法

highp vec4 vector0;

highp vec4 vector1;

highp vec4 vector2;

vector2.xz = vector0* vector1;

**3.避免使用分支语句（if语句和个别for语句）**

下面是分支语句的性能排序

a) 最佳：编译确定的常量

b) 可接受: uniform变量

c) 可能很差: 在shader内计算的变量。

解决方案：将各个分支作为单独的shader（可能会增加一点工作量以及复杂度）。‍‍

**4.使用glsl_optimizer 优化工具进行优化**

glsl_optimizer 是一个免费开源的glsl优化器。可以生成GPU无关的shader优化代码。可以进行非常多的优化项目，比如 函数内联，死代码删除，常量折叠，常量传递，数学优化等等。‍‍

**5.只计算需要计算的东西**

尽量减少无用的顶点数据,比如贴图坐标, 如果有Object使用2组有的使用1组, 那么不要将他们放在一个vertex buffer中, 这样可以减少传输的数据量;避免过多的顶点计算,比如过多的光源, 过于复杂的光照计算(复杂的光照模型);

避免顶点着色器指令数量太多或者分支过多, 尽量减少顶点着色器的长度和复杂程度;‍

**6.尽量在 VS 中计算**

通常，需要渲染的像素比顶点数多，而顶点数又比物体数多很多。所以如果可以，尽量将运算从片段着色器移到顶点着色器，或直接通过script 来设置某些固定值；‍‍

**7.浮点数精度相关：** 

float: 最高精度，通常32位.

half: 中等精度，通常16位，-60000到60000。

fixed: 最低精度，通常11位，-2.0到2.0，1/256的精度。

尽量使用低精度。对于color和unit length vectors，使用fixed，其他情况，根据取值范围尽量使用half，实在不够则使用float 。

在移动平台，关键是在片段着色器中尽可能多的使用低精度数据。另外，对于多数移动GPU，在低精度和高精度之间转换是非常耗的，在fixed上做swizzle操作也是很费事的。‍

**三、注意事项**
----------

1\. GLSL变量类型和外部传值方法需一一对应，如果GLSL里面是vec3类型的变量，则必须使用glUniform3fv()传值，使用其它的，如glUniform4fv，就会出错。

2\. 浮点类型的变量直接和整型的进行算数操作在win32上没有问题，但是在移动平台必须进行强制类型转换后才能使用，否则编译失败。同理，如果一个常量是整形的，例如1，那么直接跟一个浮点型的变量进行算数操作时，会报错，需要将“1”改成“1.0”。

3\. Uniform类型的变量不能初始化，否则在win32下不会编译出错，但是在移动平台上会编译失败。

4\. GLSL里有三种内置精度类型：precision lowp、mediump和highp。OpenGLES 2.0下，不论是浮点数还是整数，lowp、mediump、highp分别用8、10、16位实现，而在OpenGLES 3.0中分别用8、16、32位来实现。OpenGLES 2.0下面，片段着色器里支持highp关键字，但是并不要求硬件一定支持，如果硬件不支持，实际运行中会用mediump代替。例如华为Honor的Che2-TL00（Mali-450 MP, glVendor: ARM, glVersion: OpenGLES 2.0）。谨慎使用精度，既能保证达到想要的效果，性能又高。但是有些效果要求高精度浮点值，例如有的效果希望使用循环纹理，要求传入片段着色器的坐标值很大，只支持低精度的硬件就会出现瑕疵了。变通的方法就是将计算移到顶点着色器里。

5\. 顶点着色器里不能修改attribute类型的值，要先将值赋值给一个临时变量，直接对临时变量进行操作。

6\. 片段着色器里不能修改varying类型的变量，要先将值赋值给一个临时变量，直接对临时变量进行操作。‍

好啦，综上所述GL Shader Language(GLSL)的进阶版教程到此结束啦，感兴趣的小伙伴可以关注我们的公众号了解更多。