# cesium-Web网页优化进阶_孙霸天的博客-CSDN博客_cesium 水
之前我们基于常见的cesium问题已经进行总结了，常见的问题可以基于这篇进行优化

[cesium-Web页面优化总结](https://jackie-sun.blog.csdn.net/article/details/124412982)

但是这篇总结的方法都是基于cesium的方法参数进行调优，这在有一些情况下并不能解决问题。

介绍
--

我们公司有个三维项目，前台使用cesium进行三维[可视化](https://so.csdn.net/so/search?q=%E5%8F%AF%E8%A7%86%E5%8C%96&spm=1001.2101.3001.7020)，页面上我们加载了多个图层，主要的图层是使用geoserver进行发布的，包括影像图和注解图层。

叠加多个图层后我们在移动地图时，会发现底图加载会有明显的卡顿，如下所示可以明显看到影像数据加载延迟

![](https://img-blog.csdnimg.cn/img_convert/ce0407c292a17e6054305d5a00ec3405.gif)

思考
--

这里我们基于数据加载方面进行分析（该系统还有其他方面进行优化），这个页面主要加载了以下几个图层

*   大范围的地形切片数据
*   大范围的影像接片数据
*   大范围的注解切片图层（有两个）

这里GeoServer使用Tomcat进行部署，同时地形切片数据也是使用Tomcat进行发布的，使用的是同一个tomcat

这里我们可以想象到，我们拖拽一次将会有大量的切片数据请求，前台需要处理这些请求并将数据加载到cesium中去，如下所示

![](https://img-blog.csdnimg.cn/img_convert/2a4ad5fe4fdf29bafc25e96fbaa91f19.png)

正常情况下前台请求是异步的，并不会因为并发大就过慢，但这里明显感觉到延迟和卡顿就像是同步请求一样。

这时我的同事给我发来了一篇文章让我看看

[cesium 页面多 viewer 地图加载过缓解决方案](https://blog.csdn.net/lovefengruoqing/article/details/111871408)，这位博主介绍了他遇到的问题以及解决方法，但没有深入去分析底层原因。

不过得谢谢这位博主，我受到了他的启发。就去仔细查看了cesium的源码，终于让我发现了端倪。

### 分析

查看源码和官方文档分析可知，cesium进行数据请求主要通过三个类完成：Resource、RequestScheduler、Request

1.  Resource：负责资源请求
2.  RequestScheduler：负责请求调度管理
3.  Request：设置请求参数

单帧渲染逻辑：

1.  scene.render中准备了需要请求的数据列表selectTiles；
    
2.  数据provider类更新中发起selectTiles需要的请求；构造请求管理参数Request,并发起Resource.fecthArrayBuffer请求，进入RequestScheduler调度管理；
    
3.  所有进入RequestScheduler的请求，按照优先级排序过滤，每帧仅有50条级别较高的请求进入等待队列requestHeap；
    
4.  requestHeap中的请求列表按照优先级排序，每次最多向发起6条数据请求，其他请求在当前帧被取消；标记每条tile中的请求状态；
    
5.  如果窗口改变，数据不再可见，requestHeap中的请求或者已经发送的请求将被取消；
    
6.  下一帧显示调度根据tile的请求状态和窗口变化更新selectTiles，再次发起数据请求，请求管理按优先级发出数据请求，如此往复直到selectTiles为空；
    

这里可以知道，并发的限制是由RequestScheduler这个类来控制的，我们在cesium的API文档中进行查看，如下

![](https://img-blog.csdnimg.cn/img_convert/0298e4ce9e9583794dd088739863d717.png)

翻译一下

![](https://img-blog.csdnimg.cn/img_convert/4d1bc398b680fbac2cc60b05dbb2e0a7.png)

这里可以看到并发请求由maximumRequests和maximumRequestsPerServer两个变量决定

*   maximumRequests：同时活动请求的最大数量，不受限制的请求不遵守此限制。默认为50
    
*   maximumRequestsPerServer：每台服务器的最大同时活动请求数，不受限制的请求不遵守此限制。默认为6
    

这里可以知道cesium向同一台服务器进行请求时，最多一次并发请求6次，到这里我们基本上知道问题的原因了。

但我还想深挖一下，我又去看了Request类，这里有个参数\*\*throttleByServer \*\*（也是之前那个博主修改的部分）

![](https://img-blog.csdnimg.cn/img_convert/5671c1cc2a4edefe5da4090a2580ebde.png)

翻译一下：

是否通过服务器限制请求。浏览器通常支持大约 6-8 个 HTTP/1 服务器的并行连接，以及无限数量的 HTTP/2 服务器连接。将此值设置`true`为更适合通过 HTTP/1 服务器的请求。

cesium中request默认是设置为false的，也就是上面RequestScheduler中说的不受限制的请求，那就是不受上面的限制的。

但是查看源码可以发现ImageryLayer中在doRequest方法中将\*\*throttleByServer \*\*参数设置为true了，就会受到RequestScheduler的限制（其他数据源中也开启了比如3dtiles类）

![](https://img-blog.csdnimg.cn/img_convert/dee3f65394f196dbba9c946da1c7c7b0.png)

解决
--

结合上面的分析可以知道，cesium在请求时如果设置**throttleByServer：true **，就会受到**RequestScheduler**类中设置的限制，每一帧最多处理50个请求，每一帧请求时最多向同一个服务器请求6次，而我们的图层数据都是放在同一个tomcat服务中的，这就造成了数据并发时明显的卡顿。

这里的解决方式就有几种了：

*   直接修改RequestScheduler源码，将maximumRequests(默认50)和maximumRequestsPerServer(默认6)两个变量的数值调大
    
*   修改对应的数据类请求代码，这里我们系统加载了地形图层和影像切片图层，那我就修改这两个类的请求，设置**throttleByServer：false**
    

![](https://img-blog.csdnimg.cn/img_convert/e5092aeac411430e43d596b5767f6767.png)

使用第一种方式会修改cesium中所有类型数据的请求，这不合理，可能会造成客户机的CPU压力过大（比如不受限制加载倾斜摄影切片），会造成浏览器主线程卡顿；并发过大也会对服务器造成压力，这就类似[拒绝服务攻击](https://baike.baidu.com/item/%E6%8B%92%E7%BB%9D%E6%9C%8D%E5%8A%A1%E6%94%BB%E5%87%BB/421896?fr=aladdin)。我们还是使用第二种方式，对需要的数据类型进行修改。（当然也可以两种方式一起修改，这个时候就要调试了，选择一种适合你的方式）

修改完源码，我们使用命令进行打包

```bash
npm run minifyRelease

```

![](https://img-blog.csdnimg.cn/img_convert/bc276d199d26ea98b03f416e3d45dbf3.png)

![](https://img-blog.csdnimg.cn/img_convert/14dfa287a27e0b29958329a4ccc2a230.png)

将项目中的cesium依赖替换为打包好的版本，查看效果如下

![](https://img-blog.csdnimg.cn/img_convert/36a2e8b4b153d0322b044e3debe82381.gif)

总结
--

可以理解cesium为什么会有这样的限制，考虑到客户机和服务器的压力，要保证兼容更多不同设备。不过现在的电脑越来好，CPU和[GPU](https://so.csdn.net/so/search?q=GPU&spm=1001.2101.3001.7020)的性能都在变强，内存也越来越大，cesium官方是不是可以考虑将这几个参数暴露出来由我们自己决定。  
后面再优化，我会考虑优化数据服务，比如把一个tomcat服务拆成几个，或者使用nginx替代tomcat发布部分数据

扩展
--

参考这位博主的博文[HTTP1.0和HTTP1.1和HTTP2.0的区别](https://blog.csdn.net/ailunlee/article/details/97831912)