# Vue2项目引入mars3d_～前端小攻城狮的博客-CSDN博客
**1.获取 Mars3D 从 npm 获取**  
使用[Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020 "Node")环境下的现代web前端[技术栈](https://so.csdn.net/so/search?q=%E6%8A%80%E6%9C%AF%E6%A0%88&spm=1001.2101.3001.7020)时，可以使用npm或cnpm或yarn等方式来安装mars3d包

```null
npm install mars3d --save   npm install mars3d-space --save
```

**2.在main.js引入Mars3D类库**  
使用[Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020)环境下的现代web前端[技术栈](https://so.csdn.net/so/search?q=%E6%8A%80%E6%9C%AF%E6%A0%88&spm=1001.2101.3001.7020 "技术栈")时，可以使用npm等来安装mars3d包并import导入后来使用。

```null
import App from './App.vue'import 'mars3d/dist/mars3d.css'import * as mars3d from 'mars3d'Vue.prototype.mars3d = mars3d;Vue.config.productionTip = false
```

**3.安装copy-webpack-plugin**

因为mars3d所依赖的[Cesium](https://so.csdn.net/so/search?q=Cesium&spm=1001.2101.3001.7020 "Cesium")是一个多文件资源的js库，需要对其依赖的一些资源文件进行拷贝处理（这也是容易出问题的地方）， 下面已Webpack技术栈为例，vue等其他技术栈类似（可以参考项目模板说明处理。  
这里指定5.0版本，因为高于这个版本会报错

```null
npm install copy-webpack-plugin@5.0.0 -save 

```

**4.修改项目配置 vue.config.js，如建项目时没有这个文件，可在项目根目录新建，配置文件配置方法代码如下：** 

```null
// webpack.config.js  或 vue.config.jsconst path = require('path')const webpack = require('webpack')const CopyWebpackPlugin = require('copy-webpack-plugin')const cesiumSource = 'node_modules/mars3d-cesium/Build/Cesium/'    productionSourceMap: false,            new webpack.DefinePlugin({                CESIUM_BASE_URL: JSON.stringify('static')           /* new CopyWebpackPlugin({                patterns: [{ from: path.join(cesiumSource,'Workers'), to: 'static/Workers' }]                patterns: [{ from: path.join(cesiumSource,'Assets'), to: 'static/Assets' }]                patterns: [{ from: path.join(cesiumSource,'ThirdParty'), to: 'static/ThirdParty' }]                patterns: [{ from: path.join(cesiumSource,'Widgets'), to: 'static/Widgets' }]            new CopyWebpackPlugin([{ from: path.join(cesiumSource,'Workers'), to: 'static/Workers' }]),            new CopyWebpackPlugin([{ from: path.join(cesiumSource,'Assets'), to: 'static/Assets' }]),            new CopyWebpackPlugin([{ from: path.join(cesiumSource,'ThirdParty'), to: 'static/ThirdParty' }]),            new CopyWebpackPlugin([{ from: path.join(cesiumSource,'Widgets'), to: 'static/Widgets' }])        //可以配置多个,(如果要同时请求第三方和后台服务器，可以配置多个，分别代理)'target': 'http://localhost:8080/', //第三方接口地址'secure': true, // false为http访问，true为https访问'changeOrigin': true, // 跨域访问设置，true代表跨域'pathRewrite': { // 路径改写规则'^/proxy': '/' // 以/proxy/为开头的改写为'/'             
```

**5.新建div容器并使用mars3d.Map方法创建地球(这里为了方便演示我直接写在App.vue里)**

```null
<div id="mars3dContainer" class="mars3d-container"></div>basemaps: [{ name: "天地图", type: "tdt", layer: "img_d", show: true }],var map = new mars3d.Map("mars3dContainer", mapOptions);
```

**6.npm run serve 运行项目实例，即可出现一个白色地球，地图的各种样式在上面的init()方法中自行创建**![](https://img-blog.csdnimg.cn/aa61e3d2a58c4eee940d8f55e490beba.png)