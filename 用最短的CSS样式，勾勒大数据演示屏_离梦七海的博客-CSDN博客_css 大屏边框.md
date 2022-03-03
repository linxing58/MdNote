# 用最短的CSS样式，勾勒大数据演示屏_离梦七海的博客-CSDN博客_css 大屏边框
## 效果预览

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2dc501d3b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2dc718e10?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2dc623c14?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2dc85cb11?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2df5c0688?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## ↓↓↓ 动画效果总览：GIF 加载需要稍等一下 ↓↓↓

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda2df6929c6?imageslim)

## 常用效果的实现（最精简的代码）

再漂亮的设计，也可以拆解成基础效果的组合，在这里，我把一些常用的效果进行了罗列，并附加代码

## 优雅的呈现

### 蒙版文字

覆盖一层蒙版，并在蒙版上书写文字。需要 2 层，从低到上为图片层、蒙版层 (上面可写字)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda38f7596ad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

````null
<div class='modal'></div> height: 200px; width: 300pxbackground-image: url(http://www.imaoda.com/s/img/github/21.jpg);height: 100%; width: 100%;height: 20%; width: 100%;background: rgba(0,0,0,.5)```

### 毛玻璃蒙版

毛玻璃蒙版，本质上讲是两层图，底层图清晰完整，顶层图模糊残缺，并且两者位移完全一致，因此叠加到一起后像是一张图。为了确保位移一致，我们用了 `background : fixed` ，他能使背景图片相对于浏览器进行固定。

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda45ea7a1d5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
<div class='blur-modal'></div><div class='black-modal'> 描述文字 </div>position: relative; height:200px; width:300px;background: fixed url(http://www.imaoda.com/s/img/github/21.jpg);position: absolute; height:50px; width:300px;background: fixed url(http://www.imaoda.com/s/img/github/21.jpg);position: absolute; height:50px; width:300px;background: rgba(0,0,0,.2);```

### 高斯模糊背景图

电影介绍的时候，其背景色跟海报差不多，其实就是放大了海报的一个角落 + 高斯模糊 + 黑色半透明蒙版的效果

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4482d6244?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda465889dcd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
<div class='figure'></div>background: url(http://www.imaoda.com/s/img/github/21.jpg);height: 200px; width: 400px;height: 200px; width: 400px; top: 0px; left: 0px; background: rgba(0,0,0,.5);color: white; text-align: center```

filter 的还有一个特效是能让图像在边缘地带跟背景很自然的过渡到一块去

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4910840fe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 图片边框

过去做背景边框，div嵌套，外层的背景用图片，内层略小居中，但这种方法需要精准度量。更方便的方法是使用 border-image

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4770f34c8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
<div class='back'><br> 运行状态监控 </div>height: 180px;width: 400px;background: url(http://www.imaoda.com/s/img/tpl/content-frame1.png) no-repeat;background-size: contain; background-position: center;```

### 科技感内发光边框

边框内发光可以带来一种 `未来科技感`（配上动画效果更棒）

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4a6dba8d6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
background: black;padding: 10pxheight: 20rem; width: 40rem; color:white; padding: 1rem;box-shadow: 0 0 3rem rgba(100,200,255,.5) inset;background: rgba(0,0,0,.3); ```

### 科技感图片边框

选用图片边框，可以实现更多效果，当然代价就是需要先在PS里设计一下

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4ab33ec10?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> border-image 的 5% 决定了边框的厚度

```null
<div class='panel'>正常</div><div class='panel'>外发光</div>height: 10rem; width: 20rem; color:rgba(255,255,255,.9);border: 20px solid transparent;border-image: url(http://www.imaoda.com/s/img/tpl/border.png) 5%;background:rgba(0,0,0,.3);box-shadow:  0 0 5rem rgb(0,110,150)```

### 科技感图片边框2

这个有点像是游戏中的边框，效果偏重一些

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda4d4c446e2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
background: rgb(22,22,22); padding:10remheight: 10rem; width: 30rem;border: 1.5rem solid transparent;border-image: url(http://www.imaoda.com/s/img/tpl/border1.png) 15% 5%;```

### 半透明面板

科幻片里的未来科技，都是半透明的

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda520a67660?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 主要利用透明度的渐变，少量的内发光可增加立体感

```null
<div class='panel'>实时监控数据</div>background: url(http://www.imaoda.com/s/img/tpl/sky1.jpg); padding: 10remheight: 12rem; width: 24rem; color:rgba(60,240,250,1); font-size: 1.5rem; padding:1rem;border: 1px solid rgba(0,180,220,0.5);background: linear-gradient(180deg,rgba(0,180,220,0.3),rgba(0,180,220,0.1),rgba(0,180,220,0.1),rgba(0,180,220,0.3));box-shadow: 0 0 2rem rgba(0,180,220,.1) inset```

### 立体感面板-带标题栏

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda54ffe6676?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   面板头采用上深下浅过渡，面板采用半透明黑
*   侧滑弹出动画效果更佳

```null
<div class='panel-head'>用户行为分析</div><div class='panel-body'></div>display: flex; flex-flow: column nowrap;height: 23rem; width: 40rem;box-shadow: 0 0 1rem rgba(0,0,0,.5);flex: 0 0 3rem; font-size: 1.5rem; color: rgba(255,255,255,.7); line-height: 3rem; text-align: center;background: linear-gradient(rgb(0,20,30), rgb(0,40,70));border: 2px solid rgba(0,90,120,.3);background: rgba(0,0,0,.3);border: 2px solid rgba(0,90,120,.4);```

### 角标面板

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda56663c22c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   角标与边框同色
*   缺角矩形利用了 linear-gradient 透明效果

```null
<div class='panel' data-corner='New'></div>padding: 10rem;background: rgba(0,0,0,.9)height: 12rem; width: 24rem;border: 1px solid rgba(22,78,137,1);background: rgba(13,35,61,1);content: attr(data-corner);display: block; position: absolute; top: 0; right: 0; width: 5rem; padding: 0 1rem; overflow: hidden; text-align: right; color: rgba(255,255,255,.9);background: linear-gradient(45deg,transparent 0% , transparent 30%,  rgba(22,78,137,1) 30%,  rgba(22,78,137,1) 100%)```

### 弧面屏卡牌

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda571a0157d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

用到内阴影凸显卡牌的立体感和边框感

```null
height: 406px; width: 206px; background: linear-gradient(180deg,#116448,#116448);border: 1px solid rgba(0,0,0,.2);box-shadow: 0 0 1px blackheight: 400px; width: 200px; margin: 2px; border: 1px solid rgba(0,0,0,.2);box-shadow: 0 0 15px black inset;```

### 动态旋转边框

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda59490e7e5?imageslim)

*   图中两个外围的圆环采用 absolute 定位，background-position:center 居中，background-size 调整大小，并添加 animation 使其动起来
*   任务图像靠外层的 flex 布局实现居中

```null
<div class='ring-outer'></div><div class='ring-inner'></div><div class='avatar'></div>position: relative; width: 220px; height: 220px;display: flex; justify-content: center; align-items: centerheight: 220px; width: 220px; position: absolute;background: url(http://www.imaoda.com/s/img/tpl/ring-inner.png) no-repeat;background-size: 90%; background-position: center;animation: clockwise 3s linear infinite;height: 220px; width: 220px; position: absolute;background: url(http://www.imaoda.com/s/img/tpl/ring-outer.png) no-repeat;background-position: center;animation: anti-clockwise 3s linear infinite;height: 160px; width: 160px;background: url(http://www.imaoda.com/s/img/lessons/92.jpg);transform: rotate(360deg)@keyframes anti-clockwise{transform: rotate(360deg)```

### 元素背景（框）

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda5c0f0a7b9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
 ![](https://user-gold-cdn.xitu.io/2018/6/11/163edda5c3f496c9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   承载背景的外层，其长宽比和图近似
*   背景图片采用 `background-position: center; background-size: contain;` 实现自适应大小居中
*   内层用来承载内容，略小于外框，通过外框的flex实现居中
*   要让文字清晰于各种背景：白色文字带黑色发光，黑色文字带白色发光

```null
<li> <div class='content'> 第一名 </div> </li><li> <div class='content'> 第二名 </div> </li><li> <div class='content'> 第三名 </div> </li>background: linear-gradient(0deg,#006666,#003333)list-style: none; margin: 10px; height: 150px; width: 200px;background: url(http://www.imaoda.com/s/img/github/31.png) no-repeat;background-position: center; background-size: 100%;color: white; text-shadow: 0 0 2px #222222;display: flex; justify-content: center; align-items: centerheight: 80px; width: 120px;```

### 渐变色边框

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda5ce644ec6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 虽然用到了border-image,但是实际传递的只是linear-gradient

```null
	height: 400px; width: 400px;	border: 30px solid transparent;	border-image: linear-gradient(45deg,red,blue) 10%```

上述方法无法实现 border-radius，可采用双层 div 的方式，底层渐变层，上层内容层。缺点是大小需要精确度量。

```null
<div class='content'></div>width: 300px; height: 600px; border-radius: 20px;background: linear-gradient(45deg,red,blue);width: 260px; height: 560px;```

### 水晶导航侧栏

这里主要做了一套左侧的nav，涉及以下要点：

*   白色文字带黑阴影，可避免背景过白显示不清
*   ul 统一带右侧 border （比按钮最暗色rgb+10） 和右侧 1px #222222 深灰色阴影
*   li采用下暗上亮的方式；hover后采用更亮的左暗右亮
*   字体采用darkgray，hover后采用白色，带上阴影
*   hover 后也可采用 brightness 方案
*   li 的过渡是为了显示出3d效果，因此过渡为\[暗，暗，亮\]
*   li:hover为了凸显，过渡效果为\[暗，亮，亮\]

```null
background: rgb(14,28,44)text-align: center; width: 6rem; color: darkgray;border-right:1px solid rgb(41,51,3); box-shadow: 1px 0 0px #222222 ; list-style: none; font-size: 2rem; border-top:1px solid rgb(91,101,123);background: linear-gradient(180deg,rgb(51,61,83),rgb(31,41,63),rgb(31,41,63));background: linear-gradient(90deg,rgb(34,54,86),rgb(36,171,232) ,rgb(36,171,232));text-shadow: 1px 1px 1px black```

### hover浮出导航侧栏

内发光能给人凸起的感觉，菜单高亮时也可采用该效果，不过，只要上下两侧发光（用渐变背景实现）

渐变采用180，亮暗暗亮，其中亮的部分三通道亮度加20即可

```null
height: 3rem; width:10rem; line-height: 3rem; color: rgba(255,255,255,.8); cursor: pointer; list-style: none; background: rgb(8,21,37);border-bottom: 3px solid rgba(0,0,0,.5);transition: margin-left .5s;background: linear-gradient(180deg,rgb(28,41,57),rgb(8,21,37),rgb(8,21,37),rgb(28,41,57));```

### 立体按钮按压效果

按压前，按钮浮起，有阴影；按压后，按钮下去，阴影消失，体积缩小（离视野更远）

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda61ca62224?imageslim)

```css
height: 100px; width: 300px; background: green; margin: 2rem;box-shadow: 0 0 1rem gray;box-shadow: 0 0 0px gray; transform: scale(0.99)```

### 金属质感过渡

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6550705c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

45度角，两侧深，中间浅

```null
	height: 3rem; width: 5rem;	background: linear-gradient(45deg, rgb(31,89,146),rgb(35,175,230),rgb(29,136,203) )```

所以需要仔细斟酌按钮过渡，按钮按下时，可将外阴影改为内阴影，同时scale(0.95)。加上.1s的过渡，效果更佳

```null
height: 3rem; width: 3rem; line-height: 2.8rem; text-align: center; text-shadow: 1px 1px 5px black;background: linear-gradient(225deg, rgb(31,89,146),rgb(31,89,146) 10% ,rgb(35,175,230) 60%,rgb(35,175,230) 70%,rgb(29,136,203) );box-shadow: 0 0 5px blackbox-shadow: 0 0 5px gray inset;```

### 质感进度条

进度条要中间亮，两边暗，显得圆鼓鼓的，进度条容器要有内阴影，显示出凹进去的感觉

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6746b1bde?imageslim)

```null
<div class='progress'></div>height: 2rem; width: 30rem; box-shadow: 0 0 5px white inset;background: linear-gradient(180deg, #00CCCC,#00CCCC,#009999);box-shadow: 0 0px 5px white```

### 黄金色惊喜字体

```null
	color: rgb(253,219,94); font-size: 3rem; 	text-shadow: 1px 2px 2px black```

### 分界线

有时候我们的边框，只需要一根分界线，而且颜色不能太显眼。

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6a13df5ea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
 ![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6bbe2064c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*   可以直接利用一侧的 border
*   边框颜色为黑色，但是透明度只有 0.2 （以便溶于底色中）

```null
background: linear-gradient(0deg,#006666,#003333)list-style: none; width: 100px; line-height: 40px; color: white; text-align: center;border-right: 2px solid rgba(255,255,255,.2);```

### 列表前置短竖线

*   用伪类 before 为 title 添加一个前置子元素
*   这个子元素采用 inline-block 布局，尺寸用 em 度量

```null
background: linear-gradient(0deg,#006666,#003333)font-size: 2rem; line-height: 5rem; color: whitewidth: .2em; background: yellow; height: .8em; margin-right: .5em;```

### 文字颜色

文字颜色用纯白、纯黑，太醒目突兀。通常我们会选择“不那么白”或者“不那么黑”的颜色，让文字更柔和。如果嫌选颜色麻烦，可以用纯黑或者纯白，加上透明度，融入背景中，显得柔和很多。

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6d2e08616?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```css
rgba(255,255,255,.9)
````

## 提示效果

### 区域凸显

让区域高亮一般用于凸显用户选择，常用方法如下：

-   修改 background 的 RGBA 颜色，适量调高每个分量（或 A 分量调整背景透明度）|| 修改 filter: brightness 的值
-   可考虑用 transfrom: scale(1.1) 放大凸显 && 增加阴影
-   加上极少量的 transition 增加过渡效果

### 区域暗淡

暗淡部分区域，降低用户注意力，比如已完成，或者未解锁的：

-   修改 background 的 RGBA 颜色，适量降低每个分量 || 修改 filter: brightness
-   绝对定义一张黑色半透明蒙版 (屏蔽点击) 置于其上，可用半透明 + 高斯模糊加强效果，通过 after/before 伪类实现（attr）批量设置

### 旋转光芒效果

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6e23e12fa?imageslim)
 ![](https://user-gold-cdn.xitu.io/2018/6/11/163edda6fb197b8c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如果有现成的 PS 制作的光芒更好，低配版的光芒制作方法：

1.  在 ppt 中创建一个三角，选择线性过渡，顶部黄色，底部黄色透明
2.  三角复制成两份，做成 8 子形状，然后复制 n 个 8，旋转成光芒

旋转光芒利用了 2 个要素，`animation:my 5s linear 0s infinite` ，`transform:rotate(360deg)`。动态修改动画持续时间，如改成 2s，可以实现旋转加速

````null
<img class='light' src="http://www.imaoda.com/s/img/tpl/light.png" alt=""><div class='btn'>开始</div>background: rgb(14,28,44)position: relative; height: 400px; width: 400px; display: flex; justify-content: center; align-items: center;width: 200px; height: 200px;	filter: drop-shadow(0 0 5px yellow);animation: rotate linear 3s infinite;top: 50%; left: 50%; transform: translate(-50%,-50%); padding:3rem;background:green; color: white; border-radius: 50%;transform: rotate(360deg)```

*   光芒元素用 static 定位，flex 居中，如果用 top/left+tranlate 居中，与旋转 rotate 冲突
*   按钮元素用 absolute 定位，top/left+tranlate 居中 （因为 absolute 元素不被 flex 控制）

### 聚光效果

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda70caac857?imageslim)
 ![](https://user-gold-cdn.xitu.io/2018/6/11/163edda74a6815ee?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

一个外放光的圆形 border 逐渐缩小的动画：

*   外层固定大小，采用flex布局确定内层绝对居中
*   对长宽进行动画
*   border-radius: 50% 百分比固定，保持圆形

```null
<div class='circle'></div><div class='btn'>开始</div>height: 300px; width: 300px;display: flex; justify-content: center; align-items: center;height: 200px; width: 200px;border: 10px solid yellow;filter: drop-shadow(0 0 10px white);animation: shrink 1s infinite;top: 50%; left: 50%; transform: translate(-50%,-50%); padding:3rem;background:green; color: white; border-radius: 50%;height: 10px; width: 10px;```

### 亮框提示

用户点击或者 hover 的时候，边框高亮、发光，以起到提示作用

![](https://user-gold-cdn.xitu.io/2018/6/11/163edda7754199b4?imageslim)
 ![](https://user-gold-cdn.xitu.io/2018/6/11/163edda82c425672?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```null
display: inline-block; height: 200px; width: 150px;background: url(http://www.imaoda.com/s/img/github/sgs.jpg);box-shadow: 0 2px 2px #222222;border: 2px solid transparent;box-shadow: 0 0 30px yellow;border: 2px solid yellow;transform: translate(0,-10%);```

最后再巩固一下其中的基础知识
--------------

### RGBA 颜色

可以在常规 RGB 颜色后面，增加透明度 0 为全完透明，1 为完全不透明。可用在：

*   color
*   background
*   gradient
*   box-shadow
*   text-shadow

```null
rgba(123,222,22,.5)
````

### 渐变

线性渐变：角度，颜色 1，颜色 2，...，颜色 n。可后面追加 %，如果省略，则均分

```null
linear-gradient(45deg,rgba(0,0,255,.1) 0%,rgba(0,255,0,.1) 90%,rgba(255,0,0,.6) 100%)
```

辐射渐变：颜色 1，颜色 2，...，颜色 n。可后面追加 %，如果省略，则均分

```null
linear-gradient(rgba(0,0,255,.1) 0%,rgba(0,255,0,.1) 90%,rgba(255,0,0,.6) 100%)
```

### filter 滤镜

filter 会对当前元素的全部内容（含子元素，含图文边框等）进行滤镜。如果想只对图片滤镜，那么需要使用 position 将文字元素定位到其上方。

````null
filter: blur(.1rem) grayscale(.5);```

常用滤镜包括：

*   blur：失焦感的高斯模糊，默认0rem
*   opacity：透明度，默认1，全透明0
*   brightness：亮度，默认1，全黑0，曝光过度2+
*   saturate：饱和度，默认1，灰色0，艳丽2+
*   contrast：对比度，默认1，全灰0，黑白分明2+
*   grayscale：叠加灰度，默认0，灰色1
*   sepia：叠加褐色，默认0，褐色1
*   invert：反色，默认0，全灰0.5，反色1
*   drop-shadow：透明区域无阴影的 box-shadow

### border-image 图片边框

以前要实现 border-image 的效果，需要用一个稍微大一圈的图片打底

```null
border: 30px solid transparent;border-image: url(b.png) 10%; ```

### background-attachment 固定背景

如果是fixed，背景不跟随scroll滚动

### background-blend-mode

multiply 正片叠底

### 部分圆角矩形

border-radius 可针对左上、右上、右下、左下进行分别设置。因此可以设定出部分圆角矩形

```css
height: 400px; width: 400px; background: green;border-radius: 0 2rem 0 2rem ```

### 物质阴影/发光

box-shadow 可设置 右侧阴影、下方阴影、高斯模糊、颜色。其中阴影是实色，层次感稍强，一般添加模糊即可。模糊会向四个方向进行扩散，与

```css
height: 400px; width: 400px; background: black; margin: 2rem; padding: 2rem; color: lightgreen; font-size: 28px;box-shadow: 0 0 1rem gray;```

发光和阴影本质一样，只不过颜色用浅色系(white/yellow等)，所处环境为深色，加上 border-radius 效果更自然

`内阴影`，能够凸显出凹陷的感觉，只需在 box-shadow 的最后加上 `inset` 关键词

### 字体发光/黑框

text-shadow，只需模糊即可，白光可让字体发光，黑光可让白色字体能适应图片背景

```css
height: 400px; width: 400px; background: black; margin: 2rem; padding: 2rem; color: lightgreen; font-size: 28px;text-shadow: 0 0 .5rem yellow```

作者：maodayeyeye  
链接：https://juejin.im/post/5b1e2b50f265da6e5546c15d  
来源：掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。 
 [https://blog.csdn.net/qq_42316954/article/details/80886992](https://blog.csdn.net/qq_42316954/article/details/80886992)
````
