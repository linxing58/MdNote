# 文章编写实用工具——Typora+PicGo+Gitee - 简书
[![](https://upload.jianshu.io/users/upload_avatars/5330898/e2150f12-89cd-45b5-a071-7c7d21b05e61?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/5ea447f0ded8)

22021.08.27 22:35:28 字数 852 阅读 1,483

## 背景

伴随着自己日常记录越来越多，就想着使用 Markdown 在自己电脑上直接写 md 文件，于是乎找了一下网上比较常见的几个工具，最终比较发现 Typora+Pigo+Gitee 很适合我，于是我把它整理出来，分享给大家。

## 环境

-   系统：windows 10
-   工具：
    -   [Typora](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.typora.io%2F) 一款 Markdown 编辑工具
    -   [PicGo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo%2Freleases)：一款图床工具
-   图床存储：Gitee
-   上述工具[快速下载包](https://links.jianshu.com/go?to=https%3A%2F%2Fdownload.csdn.net%2Fdownload%2Fcsde12%2F21779371)

## 正文

### 1、首先我们先进行安装图床工具，方法很简单，直接从 GitHub 上面下载[PicGo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo%2Freleases)即可

![](https://upload-images.jianshu.io/upload_images/5330898-6a1307364b863e06.png)

image.png

这里我们根据自己的系统进行下载。我使用的时 windows 10 x64，因此这里直接下载的[PicGo-Setup-2.3.0-beta.8-x64.exe](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo%2Freleases%2Fdownload%2Fv2.3.0-beta.8%2FPicGo-Setup-2.3.0-beta.8-x64.exe)

### 2、运行刚才下载的[PicGo-Setup-2.3.0-beta.8-x64.exe](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo%2Freleases%2Fdownload%2Fv2.3.0-beta.8%2FPicGo-Setup-2.3.0-beta.8-x64.exe)进行安装。

![](https://upload-images.jianshu.io/upload_images/5330898-55a7f81b6c935688.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-feb1cd083bedcee2.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-c47ce5fe12e86de3.png)

image.png

### 3、打开 PicGo，进行设置自己的图床

刚安装完找不到的同学可以到任务栏收起的图标里面找找，打开 PicGo 设置菜单，按照下图进行设置

![](https://upload-images.jianshu.io/upload_images/5330898-fb7e39ce9b708e0e.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-dc2bf68c9b967db6.png)

image.png

这里面我们可以看到它支持很多种图床，而我们要使用的是 Gitee 里面没有，下面我们就需要安装插件

### 4、点击插件设置，并搜索 gitee，安装插件

![](https://upload-images.jianshu.io/upload_images/5330898-5cfa197103f69aab.png)

image.png

注意，这里如果你电脑上面没有[nodejs](https://links.jianshu.com/go?to=http%3A%2F%2Fnodejs.cn%2Fdownload%2Fcurrent%2F)，则需要先安装 nodejs 再进行插件安装。  
再次点击 PigGo 设置，就发现已经有了 gitee 这个选项了。  

![](https://upload-images.jianshu.io/upload_images/5330898-41df086e00a588b7.png)

image.png

### 5、下面我们打开 gitee 并创建一个仓库

![](https://upload-images.jianshu.io/upload_images/5330898-2cb0a6139ce6809b.png)

image.png

记住这个仓库地址  
名字可以自己随便起

### 6、进入 Gitee 的设置页面，进行设置一个私有秘钥，并记录好这个秘钥，下面我们需要使用。

![](https://upload-images.jianshu.io/upload_images/5330898-ebf61d1f83839325.png)

image.png

### 7、打开 PicGo，并点击图床设置——gitee，按照下图进行填入刚才记录的仓库地址和秘钥

![](https://upload-images.jianshu.io/upload_images/5330898-afbbbf74bf578764.png)

image.png

注意，相对地址通常为 \[gitee 账号名称 / 仓库名称] 组合而成  
到这里，我们已经完成了 PicGo+Gitee 图床的设置，有时我们的一些图片可以通过这个工具直接上传的到我们的仓库中，进行存储。

**下面我们开始安装 Typora**

### 1、进行下载[Typora](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.typora.io%2F) ，并进行安装

![](https://upload-images.jianshu.io/upload_images/5330898-69b37cf1dbab77ce.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-c67b0dc37670345f.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-cb7f6b9ebe3eb173.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-e1d9813bc7fbe946.png)

image.png

### 2、打开文件——偏好设置，按照下图进行基础设置和与 PicGo 关联设置。

![](https://upload-images.jianshu.io/upload_images/5330898-1527529f3edc5cff.png)

image.png

![](https://upload-images.jianshu.io/upload_images/5330898-40c6902dd05dc3fd.png)

image.png

注意，PicGo 路径需要选择我们刚才安装完毕的那个，并按照图片中进行勾选，这样可以使其我们本地编写直接粘贴进去的网络地址图片、本地截图、复制过来的文章中的图片都自动转换为使用 PicGo 上传完成后生成的地址。

### 3、点击验证图片上传选项

![](https://upload-images.jianshu.io/upload_images/5330898-f00dcdd9ccdbdbbd.png)

image.png

如图显示成功，则表示已经成功上传，后续我们就可以在 Typora 中进行编写文章了。

###### 本文声明：

![](https://upload-images.jianshu.io/upload_images/5330898-feced55d7a85b663.png)

88x31.png

[知识共享许可协议](https://links.jianshu.com/go?to=http%3A%2F%2Fcreativecommons.org%2Flicenses%2Fby-nc%2F4.0%2F)  
本作品由 [cn 華少](https://links.jianshu.com/go?to=www.cnhuashao.com) 采用 [知识共享署名 - 非商业性使用 4.0 国际许可协议](https://links.jianshu.com/go?to=http%3A%2F%2Fcreativecommons.org%2Flicenses%2Fby-nc%2F4.0%2F) 进行许可。

更多精彩内容，就在简书 APP

"如果喜欢是否可以请我喝杯茶~ 感谢"

还没有人赞赏，支持一下

[![](https://upload.jianshu.io/users/upload_avatars/5330898/e2150f12-89cd-45b5-a071-7c7d21b05e61?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/5ea447f0ded8)

总资产 18 共写了 8.0W 字获得 178 个赞共 70 个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   为什么要用图床 写 Markdown 插入图片时，可以插入本地图片或网络图片。 本地图片自然只能本地看。不少博客平台可...

-   我们写博客的时候经常会需要配图，特别是 markdown 写的时候只能通过网络链接来展示图片。 首先来说存储仓库。测试...

    [![](https://upload-images.jianshu.io/upload_images/5558535-4c1fd7a80cc83fbc.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/eac284babf70)

-   markdown\[[https://baike.baidu.com/item/markdown/3245829?fr..](https://baike.baidu.com/item/markdown/3245829?fr..).

    [![](https://upload-images.jianshu.io/upload_images/20524049-e6194df4e0b60535?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/213a8586ade2)

-   首先下载安装，链接如下 Typora：[https://www.typora.io/PicGo：https://mo..](https://www.typora.io/PicGo：https://mo..).

    [![](https://upload-images.jianshu.io/upload_images/15628144-f009a699eed3b469?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/cd7ab805571b)

-   图床，同字面意思，即存放图片的地方。 0. 在线图床：ImgURL\[[https://imgurl.org/\\](https://imgurl.org/\)]： Im...
       [![](https://upload.jianshu.io/users/upload_avatars/21010499/d8ff0964-4340-4db9-a53d-35e1614b288b.png?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
       阿 ll](https://www.jianshu.com/u/8b77e3734bdd)阅读 301 评论 0 赞 1
       [![](https://upload-images.jianshu.io/upload_images/21010499-ff18ae6b0679dfbb.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
       ](https://www.jianshu.com/p/4111f11cadad) 
    [https://www.jianshu.com/p/1152941beaa7](https://www.jianshu.com/p/1152941beaa7)
