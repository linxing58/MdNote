# scss快速入门 - 掘金
## 在使用 scss 之前，我们要知道 Sass、Scss 有什么不同？

SCSS 是 Sass 3 引入新的语法，其语法完全兼容 CSS3，并且继承了 Sass 的强大功能。也就是说，任何标准的 CSS3 样式表都是具有相同语义的有效的 SCSS 文件，[官方解释](https://link.juejin.cn/?target=http%3A%2F%2Fsass.bootcss.com%2Fdocs%2Fscss-for-sass-users%2F "http&#x3A;//sass.bootcss.com/docs/scss-for-sass-users/")。

传统的 css 文件缺失变量等概念，导致需要书写的重复的代码很多。我写 JS 的习惯是遇见重复的代码就要思考是否可以抽出来写成一个可复用的方法，但 css 中不存在变量函数等概念，这时我发现的一个 css 的预编译利器——scss。

scss 具有简单易上手等特点，下面开始编写第一个 scss 文件。

## 准备工作

写在最前面：有小伙伴可能不太会部署前端环境，这里我将代码上传到[github](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flizijie123%2F2019Study%2Ftree%2Fmaster%2Fnote%2Fscss "https&#x3A;//github.com/lizijie123/2019Study/tree/master/note/scss")中，有需要的小伙伴可以把代码下载下来对照着看 SCSS 代码与编译后的 CSS 代码。

scss 需要经过编译为 css 才能被浏览器识别，我这里只做一个小 demo 所以不上 vue-cli，直接使用 webpack 进行编译。

首先安装 css-loader、style-loader、node-sass、sass-loader。

    npm install css-loader style-loader --save-dev
    npm install node-sass sass-loader --save-dev
    复制代码

然后在 webpack.config.js 配置文件中添加对应的 loader, 完整的配置图如下。

    const path = require("path");
    const {VueLoaderPlugin} = require('vue-loader');
    module.exports = {
        entry: './webapp/App.js',
        output: {
            filename: 'App.js',
            path: path.resolve(__dirname, './dist')
        },
    	module: {
    		rules: [
                {
                    test: /\.scss/,
                    use: ['style-loader', 'css-loader','sass-loader']
                },
    			{
    				test: /\.vue$/,
    				use: 'vue-loader'
    			}
    		]
    	},
    	plugins: [
    		new VueLoaderPlugin()
    	],
    	mode: "production"
    }

    复制代码

创建一个 App.scss 文件，接着在入口文件中引入。

    import './App.scss';
    复制代码

后面我们将在 App.scss 中编写 scss 代码。

## 正式开始

### 1. 使用变量

SCSS 中的变量以 $ 开头。

    $border-color:#aaa; //声明变量
    .container {
    $border-width:1px;
        border:$border-width solid $border-color; //使用变量
    }
    复制代码

上述例子中定义了两个变量，其中 $border-color 在大括号之外称为全局变量，顾名思义任何地方都可以使用，$border-width 是在. container 之内声明的，是一个局部变量，只有. container 内部才能使用。

编译后的 CSS

    .container {
        border:1px solid #aaa; //使用变量
    }
    复制代码

我们可以把 SCSS 看做一个模板引擎，编译的过程中用变量的值去替代变量所占据的位置。

tips:SCSS 中变量名使用中划线或下划线都是指向同一变量的，上文中定义了一个变量 $border-color，这时再定义一个变量 $border_color:#ccc, 他们指向同一个变量，.container 的值会被第二次定义的变量覆盖。

    $border-color:#aaa; //声明变量
    $border_color:#ccc;
    .container {
        $border-width:1px;
        border:$border-width solid $border-color; //使用变量
    }
    编译后的CSS
    .container {
        border:1px solid #ccc; //使用变量
    }
    复制代码

这个例子中我们要知道

（1）变量名使用中划线或下划线都是指向同一变量的。

（2）后定义的变量声明会被忽略，但赋值会被执行，这一点和 ES5 中 var 声明变量是一样的。

### 2. 嵌套规则

我们先来看一个例子。

    /*css*/
    .container ul {
        border:1px solid #aaa;
        list-style:none;
    }

    .container ul:after {
        display:block;
        content:"";
        clear:both;
    }

    .container ul li {
        float:left;
    }

    .container ul li>a {
        display:inline-block;
        padding:6px 12px;
    }
    复制代码

这是一个让列表元素横向排列的例子，我们在这个例子中写了很多重复的代码，.container 写了很多遍，下面我将用 SCSS 简写上面的例子。

#### 2.1 嵌套选择器

    /*scss*/
    .container ul {
        border:1px solid #aaa;
        list-style:none;
        
        li {
            float:left;
        }
        
        li>a {
            display:inline-block;
            padding:6px 12px;
        }
    }

    .container ul:after {
        display:block;
        content:"";
        clear:both;
    }
    复制代码

这里我们可以将公共的父元素提取出来。

#### 2.2 嵌套中的父级选择器

SCSS 提供了一个选择器可以选中当前元素的父元素，使用 & 表示，下面用父级选择器继续简化代码。

    /*scss*/
    .container ul {
        border:1px solid #aaa;
        list-style:none;
        
        li {
            float:left;
        }
        
        li>a {
            display:inline-block;
            padding:6px 12px;
        }
        
        &:after {
            display:block;
            content:"";
            clear:both;
        }
    }
    复制代码

父级选择器中需要注意，只能在嵌套内部使用父级选择器，否则 SCSS 找不到父级元素会直接报错。 在各种伪类选择器中，父级选择器是十分常用的。

#### 2.3 嵌套组合选择器

在嵌套规则中可以写任何 css 代码，包括群组选择器（,），子代选择器（>），同层相邻组合选择器（+）、同层全体组合选择器（~）等等，下面继续将自带选择器简化掉。

    /*scss*/
    .container ul {
        border:1px solid #aaa;
        list-style:none;
        
        li {
            float:left;
            
            >a {
                display:inline-block;
                padding:6px 12px;
            }
        }
        
        &:after {
            display:block;
            content:"";
            clear:both;
        }
    }
    复制代码

子代选择器可以写在外层选择器右边（如下述例子）也可以写在内层选择器左边（如上述例子）。

    li >{ 
        a {
            display:inline-block;
            padding:6px 12px;
        }
    }
    复制代码

写在外层选择器右边时要特别注意，他会作用于所有嵌套的选择器上，尽量不要采用这类写法，我认为扩展性不强，也容易出错。

#### 2.4 嵌套属性

先看一个例子

    /*css*/
    li {
        border:1px solid #aaa;
        border-left:0;
        border-right:0;
    }
    复制代码

这个例子中我们只需要两条边框，使用 SCSS 重写一遍。

    /*scss*/
    li {
        border:1px solid #aaa {
            left:0;
            right:0;
        }
    }
    复制代码

scss 识别一个属性以分号结尾时则判断为一个属性，以大括号结尾时则判断为一个嵌套属性，规则是将外部的属性以及内部的属性通过中划线连接起来形成一个新的属性。

### 3. 导入 SCSS 文件

大型项目中 css 文件往往不止一个，css 提供了 @import 命令在 css 内部引入另一个 css 文件，浏览器只有在执行到 @import 语句后才会去加载对应的 css 文件，导致页面性能变差，故基本不使用。SCSS 中的 @import 命令跟原生的不太一样，后续会讲解到。

#### 3.1 导入变量的优先级问题 - 变量默认值

    /*App1.scss*/
    $border-color:#aaa; //声明变量
    @import App2.scss;  //引入另一个SCSS文件
    .container {
        border:1px solid $border-color; //使用变量
    }
    /*App2.scss*/
    $border-color:#ccc; //声明变量

    /*生成的css文件*/
    .container {
        border:1px solid #ccc; //使用变量
    }
    复制代码

这可能并不是我们想要的，有时候我们希望引入的某些样式不更改原有的样式，这时我们可以使用变量默认值。

    /*App1.scss*/
    $border-color:#aaa; //声明变量
    @import App2.scss;  //引入另一个SCSS文件
    .container {
        border:1px solid $border-color; //使用变量
    }
    /*App2.scss*/
    $border-color:#ccc !default; //声明变量

    /*生成的css文件*/
    .container {
        border:1px solid #aaa; //使用变量
    }
    复制代码

导入的文件 App2.scss 只在文件中不存在 $border-color 时起作用，若 App1.scss 中已经存在了 $border-color 变量，则 App2.scss 中的 $border-color 不生效。

!default 只能使用与变量中。

#### 3.2 嵌套导入

上一个例子中我们是在全局中导入的 App2.scss，现在我们在为 App2.scss 添加一些内容，并在局部中导入。

    /*App1.scss*/
    $border-color:#aaa; //声明变量
    .container {
        @import App2.scss;  //引入另一个SCSS文件
        border:1px solid $border-color; //使用变量
    }
    /*App2.scss*/
    $border-color:#ccc !default; //声明变量
    p {
        margin:0;
    }

    /*生成的css文件*/
    .container {
        border:1px solid #aaa; //使用变量
    }
    .container p {
        margin:0;
    }
    复制代码

可以看得出来，就是将 App2.scss 中的所有内容直接写入到 App1.scss 的. container 选择器中。

#### 3.3 使用原生 @import

前面我们说到基本不使用原生 @import，但某些情况下我们不得不使用原生 @import 时了，SCSS 也为我们处理了这种情况，直接导入 css 文件即可。

    @import 'App.css';
    复制代码

### 4. 注释

SCSS 中的注释有两种

（1）/\*注释\*/: 这种注释会被保留到编译后的 css 文件中。

（2）// 注释: 这种注释不会被保留到编译后生成的 css 文件中。

### 5. 混合器（函数）

#### 5.1 声明一个函数

使用 @mixin 指令声明一个函数，看一下自己的 css 文件，有重复的代码片段都可以考虑使用混合器将他们提取出来复用。

    @mixin border-radius{
        -moz-border-radius: 5px;
        -webkit-border-radius: 5px;
        border-radius: 5px;
        color:red;
    }
    复制代码

混合器作用域内的属性都是 return 的值，除此之外，还可以为函数传参数。

    @mixin get-border-radius($border-radius,$color){
        -moz-border-radius: $border-radius;
        -webkit-border-radius: $border-radius;
        border-radius: $border-radius;
        color:$color;
    }
    复制代码

也可以设置混合器的默认值。

    @mixin get-border-radius($border-radius:5px,$color:red){
        -moz-border-radius: $border-radius;
        -webkit-border-radius: $border-radius;
        border-radius: $border-radius;
        color:$color;
    }
    复制代码

#### 5.2 使用函数

使用函数的关键字为 @include

    .container {
        border:1px solid #aaa;
        @include get-border-radius;         //不传参则为默认值5px
        @include get-border-radius(10px,blue);   //传参
    }
    /*多个参数时，传参指定参数的名字，可以不用考虑传入的顺序*/
    .container {
        border:1px solid #aaa;
        @include get-border-radius;         //不传参则为默认值5px
        @include get-border-radius($color:blue,$border-radius:10px);   //传参
    }
    复制代码

我们可能会想到，直接将混合器写成一个 class 不就行了，但是写成一个 class 的时候是需要在 html 文件中使用的，而使用混合器并不需要在 html 文件中使用 class 既可达到复用的效果。

tips: 混合器中可以写一切 scss 代码。

### 6. 继承

继承是面向对象语言的一大特点，可以大大降低代码量。

#### 6.1 定义被继承的样式

    %border-style {
      border:1px solid #aaa;
      -moz-border-radius: 5px;
      -webkit-border-radius: 5px;
      border-radius: 5px;
    }
    复制代码

使用 % 定义一个被继承的样式，类似静态语言中的抽象类，他本身不起作用，只用于被其他人继承。

#### 6.2 继承样式

通过关键字 @extend 即可完成继承。

    .container {
    	@extend %border-style;
    }
    复制代码

上述例子中看不出混合器与继承之间的区别，那么下一个例子可以看出继承与混合器之间的区别。

    .container {
    	@extend %border-style;
    	color:red;
    }
    .container1 {   //继承另一个选择器
    	@extend .container;
    }
    复制代码

### 7. 小结

通过本文的介绍，相信大家已经对 SCSS 有了一定的认知，但是光看是学不会的，我建议各位小伙伴要有强上的精神，先强行使用到目前正在做的项目中，熟悉一段时间，自然可以写出优质的 SCSS 代码。

### 8. 其他

#### 8.1 操作符

SCSS 提供了标准的算术运算符，例如 +、-、\*、/、%。

    /*SCSS*/
    width: 600px / 960px * 100%;
    /*编译后的CSS*/
    width: 62.5%;
    复制代码

除此之外 SCSS 还有很多功能，各位小伙伴可以先将目前的知识运用熟练再去学习也不迟。

### 9. 交流

如果这篇文章帮到你了，觉得不错的话来点个 Star 吧。 [github.com/lizijie123](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Flizijie123%2F2019Study "https&#x3A;//github.com/lizijie123/2019Study") 
 [https://juejin.cn/post/6844903859010158600](https://juejin.cn/post/6844903859010158600)
