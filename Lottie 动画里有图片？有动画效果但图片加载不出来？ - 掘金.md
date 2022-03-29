# Lottie 动画里有图片？有动画效果但图片加载不出来？ - 掘金
> `Lottie` 是 Airbnb 开源的一套跨平台的完整的动画效果解决方案，设计师只需要使用 `After Effectes` （之后简称 AE）设计出动画之后，使用 Lottic 提供的 Bodymovin 插件将设计好的动画导出成 `JSON` 格式，就可以直接运用在 iOS、Android 和 React Native 之上，无需关心中间的实现细节。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a98318a7fc314db39511dbd7b8ad7051~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

公司使用 lottie-web 动画框架，设计师给了一个带有图片资源的 json 文件，渲染出来之后会发现具有动画效果，但是图片未加载出来，如下图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8210685c3a854d0db51040cee6b3e58b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

设计师给的文件如下：

> \*\*.json  
> images/\*\*.png

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52099b235d8d48da9117585db1ed0a89~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

前端代码如下：

```js
import ExchangeSuccessJson from 'static/json/exchange_v_success1.json';

lottie.loadAnimation({
  container: ele,
  renderer: 'svg',
  loop: true,
  autoplay: true,
  animationData: ExchangeSuccessJson,
});
```

其实 Lottie 动画所需要的图片资源也是可以集成在动画的 JSON 文件中，因此解决方法就是**将图片资源整合到动画的 JSON 文件**。

我在这篇文章[Lottie 动画里有图片怎么办？设计师小姐姐也能帮你减少开发量！](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzIxNjc0ODExMA%3D%3D%26mid%3D2247485270%26idx%3D1%26sn%3Dea6c76cdab342cde8a7af247960e498f%26chksm%3D97851e77a0f297619f28aadc1ef51c3ea019d318d77d8377b3841eb547acbe4ece423fd8f0fa%26scene%3D178%26cur_album_id%3D1439395363538173953%23rd "https&#x3A;//mp.weixin.qq.com/s?\_\_biz=MzIxNjc0ODExMA==&mid=2247485270&idx=1&sn=ea6c76cdab342cde8a7af247960e498f&chksm=97851e77a0f297619f28aadc1ef51c3ea019d318d77d8377b3841eb547acbe4ece423fd8f0fa&scene=178&cur_album_id=1439395363538173953#rd")里看到了一些解决方案，但是我发现需要跟设计师沟通，让设计师去将图片整合进动画资源，然后再重新生成发给前端。然而作为一名有追求的前端工程师，是否可以自己去快速解决这个问题呢？有的！

## 前端快速整合图片进动画文件

把文件打包成 gzip，上传到[design.alipay.com/lolita](https://link.juejin.cn/?target=https%3A%2F%2Fdesign.alipay.com%2Flolita "https&#x3A;//design.alipay.com/lolita")，压缩以后导出成单独一个 json 文件。项目中使用这个新的 json 文件即可。

自己动手，丰衣足食！到此就可以轻松加载图片，并还原动画效果了。

上述使用的动画，图片问题已经解决了，但是设计师说他想要的动画效果是前几帧播放一次，后面所有的帧数重复播放。而设计师做出来的 lottie 动画肯定是播放一次的，前端需要就后几帧进行重复播放，如何做？

```js
import ExchangeSuccessJson from 'static/json/exchange_v_success1.json';

lottieAnimate.current = lottie.loadAnimation({
  container: ele,
  renderer: 'svg',
  loop: true,
  autoplay: true,
  animationData: ExchangeSuccessJson,
});

lottieAnimate.current.addEventListener('complete', () => {
	
	lottieAnimate.current && lottieAnimate.current.goToAndPlay(43, true);
});
```

 [https://juejin.cn/post/7079975710093213709](https://juejin.cn/post/7079975710093213709)
