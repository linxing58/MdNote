# CSS - flex弹性布局+rem适配布局 - 掘金
flex弹性布局
--------

### 1.flex布局体验

```css
    div {
        display: flex;
        width: 80%;
        height: 300px;
        justify-content: space-around;
    }
    
    div span {
        
        height: 100px;
        margin-right: 5px;
        flex: 1;
    }
    

```

### 2.flex布局原理

flex是flexible Box的缩写

flex容器（flex container）中的子盒子就是 flex项目（flex item）

> 父盒子设为flex布局，子元素的float，clear，vertical-align属性失效！

**原理： 通过给父盒子添加flex属性，来控制子盒子的位置和排列方式**

### 3.flex布局父项常见属性

*   flex-direction：设置主轴的方向
    
    ```css
      flex-direction：row; 
      flex-direction：column; 
    
    ```
    
*   justify-content：设置主轴上的子元素排列方式（特别注意是在主轴上）
    
    ```scss
      justify-content：flex-start(默认贴着左边)/end(贴着右边)/center(在主轴居中对齐) 
      /space-around(平分剩余空间)/space-between(先两边贴边，再平分剩余空间)
    
    ```
    
*   flex-wrap：设置子元素是否换行
    
    flex布局中，默认的子元素是不换行的 //wrap换行
    
    ```css
      flex-wrap: wrap;
    
    ```
    
*   align-content：设置侧轴上的子元素排列方式（多行）---在单行下是没效果的
    
    ```scss
      align-content：flex-start(从上到下)/end(从下到上)/center(垂直居中)/stretch(拉伸 默认值)
     /space-between(先两边贴边，再平分剩余空间)/space-around(平分剩余空间)
    
    ```
    
*   align-items：设置侧轴上的子元素排列方式（单行）
    
    ```css
      align-items: flex-start(从上到下)/end(从下到上)/center(垂直居中)/stretch(拉伸 默认值)
    
    ```
    
*   flex-flow：复合属性，相当于同时设置了flex-direction和flex-wrap
    
    ```css
      flex-flow: column wrap；
    
    ```
    

### 4.flex布局子项常见属性

*   flex子项目的份数 (1.剩余空间2.份数)
    
    ```css
       flex: <number>;
    
    ```
    
*   align-self控制子项自己在侧轴的排列方式(了解，但面试会问到)
    
    父级flex是统一调整子元素位置，align-self可控制某一子元素的位置
    
    ```css
      align-self: flex-end;
    
    ```
    
*   order属性定义子项的排列顺序（前后顺序） 默认的是0，数值越小越靠前 与z-index不一样
    
    ```css
      
      order: -1;
    
    ```
    

rem适配布局
-------

### 1.rem基础

rem单位（root em）是一个相对单位，类似em em是父元素字体的，不同的是rem的基准是相对于html元素的字体大小

比如，根元素（html）设置font-size=12px；非根元素设置width：2rem；则换成px表示就是24px

两者比较：

*   em相对于父元素 的字体大小来说的
    
*   rem相对于html 元素的字体大小来说的
    
*   rem的优点 可以通过修改html里面的文字大小 来改变页面中元素的大小 可以整体控制
    

### 2.媒体查询（Media Query）

@media可以针对不同的屏幕尺寸设置不同的样式

语法规范：

```css
@media mediatype and|not|only(media feature){
CSS-Code;
}

```

*   mediatype查询类型-----将不同的终端划分成不同的类型，称为媒体类型
    
    all 用于所有的设备 / print 用于打印机和打印预览 / screen 用于电脑屏幕，平板电脑，智能手机等
    
*   关键字：and 将多个媒体类型连在一起 /not 排除某个媒体类型 /only 指定某个特定的媒体类型，可以省略
    
*   媒体特性：小括号包含（width/min-width/max-width）
    
    媒体查询可以根据不同的屏幕尺寸再改变不同的样式， 从小到大更简洁----层叠性
    

> 注意点：screen 和 and不可省略 数字后面必须跟单位 封号得去掉

### 3.媒体查询+rem实现元素动态大小变化

```css
  * {
        margin: 0;
        padding: 0;
    }
      
    @media screen and (min-width: 320px) {
        html {
            font-size: 50px;
        }
    }
    
    @media screen and (min-width: 640px) {
        html {
            font-size: 100px;
        }
    }
    
    div {
        height: 1rem;
        font-size: .5rem;
        background-color: green;
        color: #fff;
        text-align: center;
        line-height: 1rem;
    }

```

### 4.引入资源（理解）

引入资源就是 针对于不同的屏幕尺寸 调用不同的css文件

```xml
 <style>
    /*  */
    /* 当我们屏幕小于640px 的时候 我们让div一行显示一个 */
    /* 建议： 我们媒体查询最好的方法是从小到大 */
  </style>
 <link rel="stylesheet" href="style320.css" media="screen and (min-width:320px)">
 <link rel="stylesheet" href="style640.css" media="screen and (min-width:640px)">

```

### 5.less基础

1.Less介绍

Less是css的扩展语言，也是css的预处理器。Less是一门CSS预处理语言，它扩展了CSS的动态特性

2.维护css的弊端，css的选择器很是头疼，有些标签要反复写，效率低且易出错，现在用less写，只需要按照html的DOM树方式写一遍，然后在里面加样式就行了

例如：

```css
.nav{
     .container{
                 ul{
                    li{
                        ...
                        a{
                            ...
                            &:hover{
				...
                               }
                        }
                  }
              }
        }
  }

```

3.Less编译

*   正确的变量名 @color: green;
*   错误的变量名 @1color @color~#
*   变量名区分大小写 @color 和 @COLOR 是两个不同的变量

4.less运算 + - * /

注意点：

*   运算符中间左右有个空格隔开1px + 5
*   两个数参与运算，如果只有一个数有单位 ，则结果以这个单位为准
*   两个数参与运算，如果两个数都有单位，而且有不同的单位，最后的结果以第一个单位为准

rem适配方案
-------

1.rem实际开发适配方案

*   ①按照设计稿与设备宽度的比例，动态计算并且设置html跟标签的font-size的大小（媒体查询）
    
*   ②CSS中，设计稿元素的宽，高，相对位置等取值，按照同等比例换算为rem单位的值
    

2.rem实际开发适配方案1（基本以750px为准） 划分15等份/20

元素大小的取值方法：

*   ①最后的公式：页面元素的rem值=页面元素值（px） / (屏幕宽度/划分的份数)
*   ②屏幕宽度/划分的份数就是html中font-size的大小
*   ③或者：页面元素的rem值= 页面元素值（px） / html中font-size的字体大小 3.rem实际开发适配方案2 （flexable.js的引用） -- 推荐

官网下载js文件，默认将屏幕划分为十等份，引入之后仅需设置html的font-size即可，剩下的由flexable.js计算