# Cesium加载大量Label实体时卡顿的一种解决方法 - 掘金
## 前言

笔者接触 CesiumJS 是由于公司项目需要，直接边学边开发，两三个月来都挺顺风顺水的。直到数据量越来越大，地图上的实体越来越多，首屏加载的时候经常会卡顿，客户那边的机器性能太差，有时候卡顿的同时还出现浏览器无响应问题。测试把这个问题归为 BUG，要求必须解决，搞得我头大。

## 加载成百上千实体时出现的问题

让我们先来看下加载 Label 实体出现了什么问题？

    function addBillboard(bsm, name, buildingType, position) {
        this.viewer.entities.add({
            id: bsm,
            name,
            position: Cartesian3.fromDegrees(position[0], position[1], position[2]),
            label: {
              text: name,
              font: style == 1 ? '500 40px Microsoft YaHei' : '500 30px Helvetica', // 15pt monospace
              outlineColor: Color.WHITE, // 边框颜色
              outlineWidth: 1, // 边框宽度
              scale: 0.5,
              style: style == 1 ? LabelStyle.FILL_AND_OUTLINE : LabelStyle.FILL,
              fillColor: style == 1 ? Color.BLACK : Color.WHITE,
              pixelOffset: new Cartesian2(110, -70), // 偏移量
              showBackground: true,
              backgroundColor: backColor,
              distanceDisplayCondition: new DistanceDisplayCondition(10.0, 8000.0),
              disableDepthTestDistance: 100.0,
              scaleByDistance: new NearFarScalar(500, 1, 1400, 0.0),
              translucencyByDistance: new NearFarScalar(500, 1, 1400, 0.0)
            },
            billboard: {
              image: buildingTypeImage,
              width: 300,
              height: 140,
              pixelOffset: new Cartesian2(90, -25), // 偏移量
              distanceDisplayCondition: new DistanceDisplayCondition(10.0, 2000.0),
              disableDepthTestDistance: 100.0,
              scaleByDistance: new NearFarScalar(500, 1, 1400, 0.0),
              translucencyByDistance: new NearFarScalar(500, 1, 1400, 0.0)
            },
            properties: {
              poiType: 'buildingPOI',
              name,
              buildingType,
              bsm,
              position
            }
          });
    }
    复制代码

当我需要创建 400 多个上面这样的实体时，页面总会卡顿一段时间。于是我上网查找相关资料，可能我表述有问题，一直没找到我想要的答案。所以我进行了浏览器 DevTools。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df17ee2471bf4f60bab722ab2d65c7c2~tplv-k3u1fbpfcp-watermark.awebp)
 在加载地图的时候，js 执行了 4.75 秒。。。。，公司电脑配置如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c2f0aa00bda4afa82a3ffd0d56ec98d~tplv-k3u1fbpfcp-watermark.awebp)
 性能不算差吧  
继续看 Performance 分析吧

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d8f8879b14f4003b68bd8a05b1361f7~tplv-k3u1fbpfcp-watermark.awebp)
 这个 getImageData 怎么执行了 2 秒多，我当时想难道我传进去的图片 Cesium 要做其他的处理？ 于是我把实体的`billboard`属性剔除掉了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/958b693721594965938891549e942206~tplv-k3u1fbpfcp-watermark.awebp)
 少了 20% 的时间，噢，不对我已经没传图片进去了，`Cesium`依然调了`getImageData`，这就奇怪了，难道这是`Cesium`在生成图片，`entitie`属性只剩下 label/position/properties/id/name 了，除了 label，其他都是纯数据的。难道 label 不是文本？  

带着这个问题，我把实体的`label`属性也剔除掉了。进行了一次`Performance`。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b49295d6b596470abed634558ad531f4~tplv-k3u1fbpfcp-watermark.awebp)
 `getImageData`方法没有调用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f58e6b4e535d45038d16cbd9d74eda75~tplv-k3u1fbpfcp-watermark.awebp)
 至此，我断定`Cesium`在创建含有 label 的`entitie`时，会动态生成对应 label.name 值的图片。

## 解决方案——自行动态生成图片

找到问题，接下来就好办了，自己尝试构造这个 label，不要用到 Cesium 的`Label`。原本项目中实体的`billboard`是一张`png`格式的底图。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/202fb47c3a8448e09e781007274a5a38~tplv-k3u1fbpfcp-watermark.awebp)

**思路: 使用 Canvas 绘制一张带有文本的图片，再将 base64 码给`entitie`的`billboard`**  

    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    const image = new Image();
    image.style.width = '400px';
    const name = '韭年';
    image.onload = () => {
        canvas.width = image.width;
        canvas.height = image.height;
        ctx.drawImage(image, 0, 0, image.width, image.height);
        ctx.font = 'bold 20px Arial';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'bottom';
        ctx.fillStyle = '#FFF';
        ctx.fillText(name, 220, 70);
        this.imageUrl = canvas.toDataURL('image/png');
      };
    image.src = Iurl;
    复制代码

图片生成完成，把生成的图片加到`billboard`中，再进行一次`Performance`看看：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c37acac30bcb4375903cf1411d63ad0c~tplv-k3u1fbpfcp-watermark.awebp)

页面秒开了，没有卡顿！！

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea1df7b43b264edd964840e7fb103dd7~tplv-k3u1fbpfcp-watermark.awebp)
 标注也正常显示。

## 新的问题

考虑到客户的电脑性能不好，每次开启页面都要动态生成几百张图片，也挺浪费性能的。那就做存储吧。  
存后端 Or 前端？ 由于这些点位信息是动态配置的，有专门的页面进行管理，所以这些文本和底图会有改变的可能。这种情况把图片存在后端只能手动去控制生成图片。  

### IndexedDB 存储

大量图片，体积是很大的，`Storage`存不了这么多，只能放到`WebSQL`或`IndexedDB`,`WebSQL`已经不维护了，那就放在`IndexedDB`吧。

    // 项目入口文件中创建并连接IndexedDB
    //FILE -> main.js

    const indexedDB = window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB;
    DBOpenRequest.onupgradeneeded = (e) => {
      console.info('DB upgradeneeded')
      const db = e.target.result;
      if (!db.objectStoreNames.contains('buildingImage')) {
        db.createObjectStore('buildingImage', { keyPath: 'bsm' });
      }
    }
    DBOpenRequest.onsuccess = (e) => {
      console.info('DB链接成功')
      const db = e.target.result;
      Vue.prototype.$DB = db;

      new Vue({
        router,
        store,
        beforeDestroy() {
          DBOpenRequest.result.close();
        },
        render: h => h(App)
      }).$mount('#app');
    }
    DBOpenRequest.onerror = (e) => {
      console.ERROR('Can not open indexedDB', e);

      new Vue({
        router,
        store,
        beforeDestroy() {
          DBOpenRequest.result.close();
        },
        render: h => h(AppWith404)
      }).$mount('#app');
    };
    复制代码

最后： 在添加`entitie`的时候先查询库里对应的 bsm 存不存在，若存在，判断各类信息是否一致，一致的话使用存储的 base64 图片作为`billboard`的`image`；若不存在或者各类信息不一致，重新生成图片并存入库，再作为`billboard`的`image`。

## 结语

至此，Cesium 加载大量 entitie 卡顿的问题基本解决，进入页面也能达到秒开的速度。  
国内对`CesiumJS`的知识的分享过少，有时候遇到问题，网上找不到答案；并且`CesiumJS`的 API 超级多，很难在短时间内找到想要的功能 API。若有想一起学习探讨的大佬，请私信！！！ 本人对`CesiumJS`的认知就像学前端的时候刚学`JS`一样，一脸懵逼，文章中若有错误或讲得不对的，请批评指出。谢谢 (●'◡'●)。 
 [https://juejin.cn/post/6948613248941817893](https://juejin.cn/post/6948613248941817893)
