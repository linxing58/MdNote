# GL Shader Language（GLSL）详解-基础语法 - 知乎
在上一周我们给大家讲解了美颜类算法的Shader，今天我们的教程是跟着色器语言相关，在接下来的文章中我们将会分两节详细讲解着色器语言GL Shader Language（GLSL）的一些基本概念和语法知识。这期教程可以说是干货满满的一期了，希望能够帮助大家了解着色器语言。

在可编程管线中，必须使用GLSL编写顶点着色器和片源着色器，而GLSL的语法和C语言有很多相似之处，本篇文章将介绍GLSL的基础语法。更多细节和说明可以查阅官方文档 [http://www.opengl.org/sdk/docs/man/](https://link.zhihu.com/?target=http%3A//www.opengl.org/sdk/docs/man/) [http://www.opengl.org/registry/doc/GLSLangSpec.Full.1.20.8.pdf](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttp%253A%252F%252Fwww.opengl.org%252Fregistry%252Fdoc%252FGLSLangSpec.Full.1.20.8.pdf)

在VSCode中可以创建后缀为.glsl的文件编写着色器代码，并通过ShaderToy进行实时预览调试。

![](https://pic3.zhimg.com/v2-b6984e890f465aadc6b3bff9fbe10b46_b.jpg)

**一、变量‍**

GLSL的命名规范建议使用驼峰式，命名规则和C语言类似。 GLSL的变量名称可以使用字母，数字以及下划线，不能以数字开头， gl_作为GLSL保留前缀只能用于内部变量。还有一些GLSL内置函数名称是不能够作为变量的名称。

1.1基本类型

下表是GLSL的基本类型:  

| 数据类型 | 描述 |
| void | 跟C语言的void类似，表示空类型。作为函数的返回类型，表示这个函数不返回值。 |
| bool | 布尔类型，可以是true 和false，以及可以产生布尔型的表达式。 |
| int | 整型 代表至少包含16位的有符号的整数。可以是十进制的，十六进制的，八进制的。 |
| float | 浮点型 |
| bvec2 | 包含2个布尔成分的向量 |
| bvec3 | 包含3个布尔成分的向量 |
| bvec4 | 包含4个布尔成分的向量 |
| ivec2 | 包含2个整型成分的向量 |
| ivec3 | 包含3个整型成分的向量 |
| ivec4 | 包含4个整型成分的向量 |
| mat2 或者 mat2x2 | 2×2的浮点数矩阵类型 |
| mat3或者mat3x3 | 3×3的浮点数矩阵类型 |
| mat4x4 | 4×4的浮点矩阵 |
| mat2x3 | 2列3行的浮点矩阵（OpenGL的矩阵是列主顺序的） |
| mat2x4 | 2列4行的浮点矩阵 |
| mat3x2 | 3列2行的浮点矩阵 |
| mat3x4 | 3列4行的浮点矩阵 |
| mat4x2 | 4列2行的浮点矩阵 |
| mat4x3 | 4列3行的浮点矩阵 |
| sampler1D | 用于内建的纹理函数中引用指定的1D纹理的句柄。只可以作为一致变量或者函数参数使用 |
| sampler2D | 二维纹理句柄 |
| sampler3D | 三维纹理句柄 |
| samplerCube | cube map纹理句柄 |
| sampler1DShadow | 一维深度纹理句柄 |
| sampler2DShadow | 二维深度纹理句柄 |

1.2内置变量  

定点着色器可用的内置变量如下表：

| 名称 | 类型 | 描述 |
| gl_Color | vec4 | 输入属性-表示顶点的主颜色 |
| gl_SecondaryColor | vec4 | 输入属性-表示顶点的辅助颜色 |
| gl_Normal | vec3 | 输入属性-表示顶点的法线值 |
| gl_Vertex | vec4 | 输入属性-表示物体空间的顶点位置 |
| gl_MultiTexCoordn | vec4 | 输入属性-表示顶点的第n个纹理的坐标 |
| gl_FogCoord | float | 输入属性-表示顶点的雾坐标 |
| gl_Position | vec4 | 输出属性-变换后的顶点的位置，用于后面的固定的裁剪等操作。所有的顶点着色器都必须写这个值。 |
| gl_ClipVertex | vec4 | 输出坐标，用于用户裁剪平面的裁剪 |
| gl_PointSize | float | 点的大小 |
| gl_FrontColor | vec4 | 正面的主颜色的varying输出 |
| gl_BackColor | vec4 | 背面主颜色的varying输出 |
| gl_FrontSecondaryColor | vec4 | 正面的辅助颜色的varying输出 |
| gl_BackSecondaryColor | vec4 | 背面的辅助颜色的varying输出 |
| gl_TexCoord\[\] | vec4 | 纹理坐标的数组varying输出 |
| gl_FogFragCoord | float | 雾坐标的varying输出 |

片段着色器的内置变量如下表：

| 名称 | 类型 | 描述 |
| gl_Color | vec4 | 包含主颜色的插值只读输入 |
| gl_SecondaryColor | vec4 | 包含辅助颜色的插值只读输入 |
| gl_TexCoord\[\] | vec4 | 包含纹理坐标数组的插值只读输入 |
| gl_FogFragCoord | float | 包含雾坐标的插值只读输入 |
| gl_FragCoord | vec4 | 只读输入，窗口的x,y,z和1/w |
| gl_FrontFacing | bool | 只读输入，如果是窗口正面图元的一部分，则这个值为true |
| gl_PointCoord | vec2 | 点精灵的二维空间坐标范围在(0.0, 0.0)到(1.0, 1.0)之间，仅用于点图元和点精灵开启的情况下。 |
| gl_FragData\[\] | vec4 | 使用glDrawBuffers输出的数据数组。不能与gl_FragColor结合使用。 |
| gl_FragColor | vec4 | 输出的颜色用于随后的像素操作 |
| gl_FragDepth | float | 输出的深度用于随后的像素操作，如果这个值没有被写，则使用固定功能管线的深度值代替 |

1.3修饰符

变量的声明可以使用如下的修饰符：

| 修饰符 | 描述 |
| const | 常量值必须在声明时初始化。它是只读的不可修改的。 |
| attribute | 表示只读的顶点数据，只用在顶点着色器中。数据来自当前的顶点状态或者顶点数组。它必须是全局范围声明的，不能在函数内部。一个attribute可以是浮点数类型的标量，向量，或者矩阵。不可以是数组或者结构体 |
| uniform | 一致变量。在着色器执行期间一致变量的值是不变的。与const常量不同的是，这个值在编译时期是未知的是由着色器外部初始化的。一致变量在顶点着色器和片段着色器之间是共享的。它也只能在全局范围进行声明。 |
| varying | 顶点着色器的输出。例如颜色或者纹理坐标，（插值后的数据）作为片段着色器的只读输入数据。必须是全局范围声明的全局变量。可以是浮点数类型的标量，向量，矩阵。不能是数组或者结构体。 |
| centorid varying | 在没有多重采样的情况下，与varying是一样的意思。在多重采样时，centorid varying在光栅化的图形内部进行求值而不是在片段中心的固定位置求值。 |
| invariant | (不变量)用于表示顶点着色器的输出和任何匹配片段着色器的输入，在不同的着色器中计算产生的值必须是一致的。所有的数据流和控制流，写入一个invariant变量的是一致的。编译器为了保证结果是完全一致的，需要放弃那些可能会导致不一致值的潜在的优化。除非必要，不要使用这个修饰符。在多通道渲染中避免z-fighting可能会使用到。 |
| in | 用在函数的参数中，表示这个参数是输入的，在函数中改变这个值，并不会影响对调用的函数产生副作用。（相当于C语言的传值），这个是函数参数默认的修饰符 |
| out | 用在函数的参数中，表示该参数是输出参数，值是会改变的。 |
| inout | 用在函数的参数，表示这个参数即是输入参数也是输出参数。 |

1.4 数组‍  

GLSL中只能使用一维数组。数组的类型可以是一切基本类型或者结构体。声明方式如下：

vec4 transMatrix\[4\];

vec4 affineMatrix\[4\] = {0, 1, 2, 3};

vec4 rotateMatrix = affineMatrix;

1.5 结构体‍  

结构体的定义方式和C语言类似，可以组合基本类型和数组来形成用户自定义的类型, 区别是GLSL的结构体不支持嵌套定义，只有预先声明的结构体可以嵌套其中，参考代码如下：

struct rotateMatrix {

float x;

float y;

float z;

float coeff\[8\];

}

struct positionInfo {

vec2 coord;

float value;

rotateMatrix matrix;

}

**二、表达式**

2.1操作符  

GLSL语言的操作符与C语言相似。如下表（操作符的优先级从高到低排列）

| 操作符 | 描述 |
| () | 用于表达式组合，函数调用，构造 |
| \[\] | 数组下标，向量或矩阵的选择器 |
| . | 结构体和向量的成员选择 |
| \+\+ – | 前缀或后缀的自增自减操作符 |
| \+ – ! | 一元操作符，表示正 负 逻辑非 |
| \* / | 乘 除操作符 |
| \+ - | 二元操作符 表示加 减操作 |
| <\> <= >= == != | 小于，大于，小于等于， 大于等于，等于，不等于 判断符 |
| && |  |
| ^^ | 逻辑与 ，或， 异或 |
| ?: | 条件判断符 |
| = += –= *= /= | 赋值操作符 |
| , | 表示序列 |

位操作符(&,|,^,~, <<, >\> ,&=, |=, ^=, <<=, >>=)是GLSL保留的操作符，将来可能会被使用。还有求模操作（%，%=)也是保留的。

2.2 运算符‍

下表显示glsl用到的运算符，如下表（操作符的优先级从高到低排列）：

| 运算符 | 说明 | 结合性 |
| () | 聚组:a*(b+c) | N/A |
| \[\] () . \+\+ -- | 数组下标__\[\],方法参数__fun(arg1,arg2,arg3),属性访问a.b,自增/减后缀a++ a-- | L - R |
| \+\+ \-\- \+ \- ! | 自增/减前缀++a --a,正负号(一般正号不写)a ,-a,取反!false | R - L |
| \* / | 乘除数学运算 | L - R |
| \+ - | 加减数学运算 | L - R |
| < \> <= >= | 关系运算符 | L - R |
| == != | 相等性运算符 | L - R |
| && | 逻辑与 | L - R |
| ^^ | 逻辑排他或(用处基本等于!=) | L - R |
| II | 逻辑或 | L - R |
| ? : | 三目运算符 | L - R |
| = += -= *= /= | 赋值与复合赋值 | L - R |
| , | 顺序分配运算 | L - R |

2.3 数组访问‍  

和C语言一样， 数组的下标从0开始。取值范围是\[0, sizeOfArray - 1\]。如果越界了，会提示编译失败。

**三、控制流‍**  

3.1 循环‍

和C语言一样，GLSL语言可以使用for, while, do/while的循环方式，语法和C语言一样，参考下面代码：

for (int s = 0; s < 7; s++) {  
vec2 r;  
r = vec2(cos(uv.y * i0 - i4 + time / i1), sin(uv.x * i0 - i4 + time / i1)) / i2;  
r += vec2(-r.y, r.x) * 0.3;  
uv.xy += r;

i0 *= 1.93;  
i1 *= 1.15;  
i2 *= 1.7;  
i4 += 0.05 + 0.1 * time * i1;  
}

3.2 控制语句‍  

GLSL语言可使用if/else语句进行逻辑控制，语法和C语言一致

**四、函数**

4.1自定义函数

自定义函数规则和C语言差不多，每个shader中必须有一个main函数。参数的修饰符(in, out, inout, const等）是可选的。下面代码示例：

#pragma glslify: snoise = require('glsl-noise/simplex/2d')

float noise(in vec2 pt) {

return snoise(pt) * 0.5 + 0.5;

}

// GLSL的函数是支持重载的。函数可以同名但其参数类型或者参数个数不同即可

float noise(in vec2 pt, in float value) {

return snoise(pt) * 0.5 + value;

}

  
void main () {

float r = noise(gl_FragCoord.xy * 0.01);

float g = noise(gl_FragCoord.xy * 0.01 + 100.0);

float b = noise(gl_FragCoord.xy * 0.01 + 300.0);

gl_FragColor = vec4(r, g, b, 1);

}

4.2内置函数  

**角和三角函数**

| 语法 | 说明 |
| genType radians (genType degrees) | 角度转弧度（degrees to radians） |
| genType degrees (genType radians) | 弧度转角度（radians to degrees） |
| genType sin (genType angle) | 三角函数-正弦sine |
| genType cos (genType angle) | 三角函数-余弦cosine |
| genType tan (genType angle) | 三角函数-正切tangent |
| genType asin (genType x) | 反三角函数-反正弦arc sine |
| genType atan (genType y, genType x) | 反三角函数-反余弦arc cosine |
| genType atan (genType y\_over\_x) | 反三角函数-反正切arc tangent |

**指数函数**

| 语法 | 说明 |
| genType pow (genType x, genType y) | x的y次方，\\(x^y\\)。如果x<0，则结果是undefined；如果x=0并且y<=0，则结果是undefined |
| genType exp (genType x) | x的自然指数，\\(e^x\\) |
| genType log (genType x) | x的自然对数，\\(\\log_ex\\)，即\\(\\ln{x}\\)x<=0时结果是undefined |
| genType exp2 (genType x) | 2的x次方，\\(2^x\\) |
| genType log2 (genType x) | 以2为底，x的自然对数，\\(log_2x\\)，x<=0时结果是undefined |
| genType sqrt (genType x) | 对x进行开根号，\\(\\sqrt{x}\\)，x<0时结果是undefined |
| genType inversesqrt (genType x) | sqrt的倒数，\\(\\frac{1}{\\sqrt x}\\)，x<=0时结果是undefined |

**常用函数**

| 语法 | 说明 |
| genType abs (genType x) | x的绝对值 |
| genType sign (genType x) | 判断x是正数、负数，还是零 |
| genType floor (genType x) | 返回不大于x的最大整数 |
| genType ceil (genType x) | 返回不小于x的最小整数 |
| genType fract (genType x) | 返回x的小数部分，即x-floor(x) |
| genType mod (genType x, genType y) | 返回x – y * floor (x/y) |
| genType min (genType x, genType y) | 返回x和y的较小值 |
| genType max (genType x, genType y) | 返回x和y的较大值 |
| genType clamp (genType x, genType minVal, genType maxVal) | min (max (x, minVal), maxVal)，如果minVal > maxVal，则返回undefined |
| genType mix (genType x, genType y, genType a) | 返回x * (1−a) + y * a |

### 几何函数

| 语法 | 说明 |
| genType length (genType x) | 计算向量的长度， \\(\\sqrt{x1^2+x2^2+...}\\) |
| genType distance (genType p0, genType p1) | p0和p1之间的距离，即length(p0-p1) |
| genType dot (genType x, genType y) | 向量x与向量y的点积 |
| genType cross (vec3 x, vec3 y) | 向量x与向量y的叉积 |
| genType normalize (genType x) | 返回向量x对应的单位向量，即方向相同，长度为1 |
| genType faceforward(genType N, genType I, genType Nref) | 如果dot(Nref, I) < 0，则返回N，否则返回-N |
| genType reflect (genType I, genType N) | I是入射光的方向，N是反射平面的法线，返回值是反射光的方向。I – 2 * dot(N, I) * N。N必须是单位向量。 |
| genType refract(genType I, genType N, float eta) | I是入射光的方向，N是反射平面的法线，折射率是eta，返回值是折射后的光线的向量。I和N必须是单位向量。 |

矩阵函数

| 语法 | 说明 |
| mat matrixCompMult (mat x, mat y) | 返回矩阵x乘以矩阵y的结果。例如result\[i\]\[j\] 等于 x\[i\]\[j\] * y\[i\]\[j\]。注意：如果想进行线性代数里的乘法，请使用符号“*”。 |

向量关系函数

| 语法 | 说明 |
| bvec lessThan(vec x, vec y) bvec lessThan(ivec x, ivec y) | 判断x<y |
| bvec lessThanEqual(vec x, vec y) bvec lessThanEqual(ivec x, ivec y) | 判断x<=y |
| bvec greaterThan(vec x, vec y) bvec greaterThan(ivec x, ivec y) | 判断x>y |
| bvec greaterThanEqual(vec x, vec y) bvec greaterThanEqual(ivec x, ivec y) | 判断x>=y |
| bvec equal(vec x, vec y) bvec equal(ivec x, ivec y) bvec equal(bvec x, bvec y) | 判断x==y |
| bvec notEqual(vec x, vec y) bvec notEqual(ivec x, ivec y) bvec notEqual(bvec x, bvec y) | 判断x!=y |
| bool any(bvec x) | 判断x的元素是否有true |
| bool all(bvec x) | 判断x的元素是否全部为true |
| bool not(bvec x) | <!--TOTO--> |

Texture查找函数

| vec4 texture2D (sampler2D sampler, vec2 coord) vec4 texture2D (sampler2D sampler, vec2 coord, float bias) vec4 texture2DProj (sampler2D sampler, vec3 coord) vec4 texture2DProj (sampler2D sampler, vec3 coord, float bias) vec4 texture2DProj (sampler2D sampler, vec4 coord) vec4 texture2DProj (sampler2D sampler, vec4 coord, float bias) vec4 texture2DLod (sampler2D sampler, vec2 coord, float lod) vec4 texture2DProjLod (sampler2D sampler, vec3 coord, float lod) vec4 texture2DProjLod (sampler2D sampler, vec4 coord, float lod) | 使用texture坐标coord来查找当前绑定到采样器的texture。对于函数名中含有Proj的函数来说，texture的坐标（coord.s,coord.t）会先除以coord的最后一个坐标。对于vec4这种变种来说，坐标的第三个元素直接被忽略。 |
| vec4 textureCube (samplerCube sampler, vec3 coord)vec4 textureCube (samplerCube sampler, vec3 coord, float bias)vec4 textureCubeLod (samplerCube sampler, vec3 coord, float lod) | 使用coord这个坐标去查找当前绑定到采样器的cube map。coord的方向用来表示去查找cube map的哪一个二维平面。OpenGL说明书的2.0版本的3.8.6小节详细描述了这一点。 |

以上就是我们今天的教程啦，感兴趣的伙伴们可以关注我们公众号了解更多教程。