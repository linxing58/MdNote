# Hexo博客写文章及基本操作 - 知乎
上一篇文章中我们简单地 DIY 了下博客，虽然很有意思，但还是不要忘记搭建博客的初心。接下来进入正题，使用`Hexo`写博客~

* * *

## 编写文章的工具

### 1.Markdown

写博客文章我们会使用`Markdown`来排版，它通过一些简单的标记语法让文本具有一定的格式。写作体验和`Word`这种机械式的排版工具比起来完全不是一个`level`的。

### 2.Typora

接下来要介绍的主角是一个`Markdown`编辑器，它能让我们的写作体验达到顶峰。我个人是离不开这个编辑器了，它就是[Typora](https://link.zhihu.com/?target=https%3A//www.typora.io/)

### 好处

下面是我觉得比较好的两个点：

①实时预览：在`Typora`中我们输入标记语法就能实时看到排版效果，解决了传统`Markdown`编辑器左右分屏式看起来麻烦的烦恼。

②快捷键输入：很多标记语法我们都可以使用快捷键输入，免去了手动敲语法的烦恼。

> **当然还有很多强大的功能需要我们细品，这里就不一一赘述了，大家自行探索~**

### 常用快捷键和语法

标题：Ctrl+1、2、3... 对应一、二、三... 级标题（光标定位到需要设置为标题的行，按快捷键）

**加粗**：Ctrl+B（选中要加粗的文本，按快捷键）

_斜体_：Ctrl+I（选中要设置斜体的文本，按快捷键）

下划线

：Ctrl+U（选中要加下划线的文本，按快捷键）

删除线：Alt+Shift+5（选中要加删除线的文本，按快捷键）

`代码片段`：Ctrl+Shift+\`（选中要设置为代码片段的文本，按快捷键）

代码块：Ctrl+Shift+K（任意位置按快捷键，选择编程语言然后在代码块中输入代码）

切换到下一行：Ctrl+Enter（任意位置按快捷键，在代码块中可以跳出代码块另起一行）

[链接](https://link.zhihu.com/?target=https%3A//www.baidu.com/)：Ctrl+K（先复制链接，然后选中要加链接的文本，按快捷键。Ctrl + 左键点击文本可跳转到对应链接）

取消格式：再次按相同的快捷键即可

有序列表：数字 + 点 + 空格

任务列表：加号或减号 + 空格

切换到列表下一行：Space+Enter

嵌套列表：按 Tab 键

退出列表：按 Shift+Tab

插入表格：Ctrl+T

引用：输入 > 后面加空格，或者 Ctrl+Shift+Q

## Hexo 文章管理

### 1. 创建一个 md 文件

md 文件也就是`Markdown`文件，通过以下命令来创建：

```text
$ hexo new <title>
$ hexo new "我的第一篇文章"
```

### 2. 布局（layout）

-   创建 md 文件时，我们可以指定布局

`$ hexo new [layout] <title>  
$ hexo new page "我的页面"`

-   布局有三种：`post`（文章）、`draft`（草稿）、`page`（页面）

在新建文件时，Hexo 会根据 `scaffolds` 文件夹内相对应的文件（可以理解为模板）来建立 md 文件：

![](https://pic2.zhimg.com/v2-2683cfb7e862381166e609ef37210a99_b.png)

-   如果没有指定布局类型，则为默认布局`post`，可以在站点配置文件修改 `default_layout` 参数来修改默认布局。  
-   当我们创建不同布局的 md 文件时，它们会存储在不同路径：

![](https://pic3.zhimg.com/v2-81c1aa7b55f1ae6767b3563127a22156_b.jpg)

> 对于独立页面来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 `index.md` 文件，页面布局顾名思义就是用来 DIY 我们博客页面的。

### 3. 草稿（draft）

`draft`这种布局在建立时会被保存到 `source/_drafts` 文件夹中，但不会显示在页面上，如果我们不想某一篇文章显示在页面上，那么就可以把它移动到`_drafts`文件夹中。

-   我们可在启动服务器时加上 `--draft` 参数来查看草稿。

`$ hexo server --draft`

-   还可以在站点配置文件中把 `render_drafts` 参数设为 `true` 来预览草稿。  
-   我们可以通过 `publish` 命令将草稿发布文章或者页面，它将会被移动到指定的文件夹。

`$ hexo publish [layout] <title>`

### 4.Front-matter

当我们创建一个 md 文件后，打开后会看到一些内容，这些称为`Front-matter`，它是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```yaml
---
title: Hello World # 标题就是我们上面创建的时候指定的名字
date: 2013/7/13 20:46:25 # 文件创建的时间
---
```

> 在`Typora`中我们在 md 文件的首行（必须是第一行）输入`---` ，然后按回车就可以插入`Front-matter`了。

Front-matter 预定义参数

```text
  layout  布局  默认为true，如果你不想你的文章被处理，可以设置为false
  title  标题  标题会显示在最上方居中位置     
  date  建立日期    如果不指定则为默认值-文件创建日期，可以自定义。
  update  更新日期  如果不指定则为默认值-文件修改后重新生成静态文件的日期。
  comments  是否开启文章的评论功能 默认值为true
  tags  标签（不适用于页面page布局）
  categoreies  分类（不适用于页面page布局）
  permalink  覆盖文章网址
  keywords  仅用于 meta 标签和 Open Graph 的关键词（不推荐使用）
```

### 为文章添加分类与标签

只有文章（post 布局）支持分类和标签，需要在`Front-matter`中设置。分类有层级关系，标签没有。

举个例子：  
1）下面文章它的标签是：Hexo、博客  
2）分类是： 个人博客 > Hexo 博客  
3）“Hexo 博客” 是 “个人博客” 的子分类

```yaml
categories:
- 个人博客（第一层级）
- Hexo博客（第二层级）
tags:
- Hexo
- 博客
```

### 为文章添加多个分类

1）下面文章属于三个分类：日常 > 生活，日常 > 随想，日记  
2）其中生活、随想为日常的子分类，日常和日记为同级分类

```yaml
categories:
- [日常, 生活]
- [日常, 随想]
- [日记]
```

## 基本操作

### 常用命令

-   **清除缓存：** `hexo clean`  
-   **生成静态文件：** `hexo generate`可简写为 `hexo g`  
-   **启动服务器：** `hexo server`或者 `hexo s` 常用参数：`-p（--port）`重设端口  
-   **部署：** `hexo deploy`可简写为`hexo d`，用于将网站部署到服务器上。（暂时用不到，目前都是在本地，后面我们将博客托管到`GitHub Pages`或`Gitee Pages`时才会用到此命令）  
    常用参数：`-g（--generate）`，`hexo d -g`部署前预先生成静态文件，等同于 `hexo g -d`

**一般发布文章或者修改博客后需要这些操作：** 清除缓存 > 生成静态文件 > 启动服务器，测试没问题后再部署。

```csharp
// 我们可以写成一条命令
$ hexo clean && hexo g && hexo s
$ hexo d

```

> 更多细节请查看：[Hexo 官方文档](https://link.zhihu.com/?target=https%3A//hexo.bootcss.com/docs/)

* * *

**The last but not the end.** 
 [https://zhuanlan.zhihu.com/p/156915260](https://zhuanlan.zhihu.com/p/156915260)
