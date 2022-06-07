# DEM数据下载
**DEM 数据的下载**

**一、SRTM（Shuttle Radar Topography Mission）**

SRTM 地形数据按精度可以分为 SRTM1 和 SRTM3，分别对应的分辨率精度为 30 米和 90 米数据，90 米数据可以去地理空间数据云下载，30 米的可以去美国 USGS 下载（USGS 网站目前有时需要 “翻墙”、国内推荐使用地理空间数据云）。

下载地址：

**30mDEM：** 

[http://srtm.csi.cgiar.org/srtmdata/](https://link.zhihu.com/?target=http%3A//srtm.csi.cgiar.org/srtmdata/)

USGS: [https://glovis.usgs.gov/](https://link.zhihu.com/?target=https%3A//glovis.usgs.gov/)

**（90mDEM）:**

USGS：[http://dds.cr.usgs.gov/srtm/version2_1/SRTM3/Eurasia/](https://link.zhihu.com/?target=http%3A//dds.cr.usgs.gov/srtm/version2_1/SRTM3/Eurasia/)

地理空间数据云[http://www.gscloud.cn/](https://link.zhihu.com/?target=http%3A//www.gscloud.cn/)

**二、ASTER GDEM**  
ASTER GDEM V2 全球数字高程数据于 2015 年 1 月 6 日正式发布，用户可以通过地理空间数据云下载使用。  
USGS 下载地址：[http://earthexplorer.usgs.gov/](https://link.zhihu.com/?target=http%3A//earthexplorer.usgs.gov/)

**三、GTOPO30(Global 30 Arc-Second Elevation)**

USGS 下载地址：[https://www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-global-30-arc-second-elevation-gtopo30?qt-science_center_objects=0#qt-science_center_objects](https://link.zhihu.com/?target=https%3A//www.usgs.gov/centers/eros/science/usgs-eros-archive-digital-elevation-global-30-arc-second-elevation-gtopo30%3Fqt-science_center_objects%3D0%23qt-science_center_objects)

**四、GLS 2005 DEM**  
下载方法：打开 [http://glovis.usgs.gov/](https://link.zhihu.com/?target=http%3A//glovis.usgs.gov/) 选择数据集 Global Land Survey，GLS2005，点击 View Image. 在弹出的 USGS Global Visualization Viewer 窗口中选择你感兴趣的区域，选择你要下载的场景 (只有 LT5 的包中有 DEM，LT7 的包里面没有) 点击 Add, 点击 Download。  
**五、TANDEM**  
有关 TanDEM-X 90m DEM 参阅以下网址：https：//[http://tandemx-90m.dlr.de/](https://link.zhihu.com/?target=http%3A//tandemx-90m.dlr.de/)

有关 TanDEM-X 任务的更多信息，请参阅以下网址：[https://\[www.dlr.de/dlr/en/desktopdefault.aspx/tabid-10378/\](https://link.zhihu.com/?target=http%3A//www.dlr.de/dlr/en/desktopdefault.aspx/tabid-10378/](<https://[www.dlr.de/dlr/en/desktopdefault.aspx/tabid-10378/](https://link.zhihu.com/?target=http%3A//www.dlr.de/dlr/en/desktopdefault.aspx/tabid-10378/>))

公开的 TanDEM-X - Digital Elevation Model (DEM) - Global, 90mDEM 下载链接为： [https://download.geoservice.dlr.de/TDM90/](https://link.zhihu.com/?target=https%3A//download.geoservice.dlr.de/TDM90/)

**六、GMTED2010DEM(The Global Multi-resolution Terrain Elevation Data 2010 ）**

USGS 下载地址：[http://earthexplorer.usgs.gov](https://link.zhihu.com/?target=http%3A//earthexplorer.usgs.gov/)

* * *

## **示例：12.5mDEM 数据下载**

**官网**

NASA（美国国家航空航天局）[https://search.asf.alaska.edu/#/](https://link.zhihu.com/?target=https%3A//search.asf.alaska.edu/%23/)

## **第一步 注册**

首先我们打开网站，显示网站的界面

![](https://pic3.zhimg.com/v2-0b716f0f66e6b2cb338188a097eb14ce_b.jpg)

注册账号。点击右上角的**【Sign in】** 按钮，进入注册面板。

在注册账号面板填写**用户名称**与**密码**后点击**【LOG IN】** 就完成了注册

![](https://pic2.zhimg.com/v2-f8865443911af4ca89cea48013c9d405_b.jpg)

如下显示我们注册的账号已经登录上去

![](https://pic1.zhimg.com/v2-30878b0a4fc8c44920871bc0fe4663f0_b.jpg)

## **第二步 寻找数据**

![](https://pic1.zhimg.com/v2-2cc1dd5192cbecceb42df1b06e69f26c_b.jpg)

在**Dataset（数据集）选项**中选择**ALOS PALSAR**

![](https://pic3.zhimg.com/v2-71dd5c767a2a537245d7c160ee5f5da2_b.jpg)

使用**框选**工具（可选），选择你要下载的区域（因精度较高，数据量较大建议框选小区域的县域）

![](https://pic4.zhimg.com/v2-63e0ee05fd0e52d9bf0ab02a3632df67_b.jpg)

点击**Filters**面板（过滤）

![](https://pic2.zhimg.com/v2-a3459a73cae46dcfe2724cebbf721c25_b.jpg)

在 Filters 面板填写信息。**Beam Mode**中选择 **FBS 与 FBD**

（或 File Type 中直接选择 Hi-Res Terrian Corrected 这里不做演示）

![](https://pic4.zhimg.com/v2-b10ca5e497d2d1b1b850c31d912eeac7_b.jpg)

一切就绪后点击 **SEARCH（**搜索**）**

**但是发现我们的数据并不全**

![](https://pic1.zhimg.com/v2-d6e4d4ca4e8e9c5ebec4031fa2123084_b.jpg)

**再次进入 Filters 面板，在里面清楚时间选择，File Type 填写**Hi-Res Terrian Corrected，**Beam Mode**中选择 全部数据

![](https://pic1.zhimg.com/v2-46b1eb1ef9565ea964966f1235690098_b.jpg)

**如下显示我们想要的范围都有数据**

![](https://pic2.zhimg.com/v2-8834bd1fc56630cc846f3f9df578e835_b.jpg)

## **第三步 进行下载**

**点击图层的图块，选中会显示红框，点击下载选项面板的下载即可**

![](https://pic3.zhimg.com/v2-d5c11c113c4871397c17a2c2c598170e_b.jpg)

**数据开始下载，只不过很满很慢（关键在于你的网速）**

![](https://pic3.zhimg.com/v2-81f60527fa5a9bcbf3ad1718149d0e16_b.jpg)

**另外，我们可以把数据一起先放进购物车，之后可以进行统一下载**

![](https://pic2.zhimg.com/v2-072fc040dc136c3fd2b93f98d3015395_b.jpg) 
 [https://zhuanlan.zhihu.com/p/375513290](https://zhuanlan.zhihu.com/p/375513290)
