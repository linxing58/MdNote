# nginx缓存配置及开启gzip压缩 - 龙恩0707 - 博客园
## [nginx 缓存配置及开启 gzip 压缩](https://www.cnblogs.com/tugenhua0707/p/10841267.html)

2019-05-09 21:42  [龙恩 0707](https://www.cnblogs.com/tugenhua0707/)  阅读 (43881)  评论 ()  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=10841267)  收藏  举报

**阅读目录**

-   [一：nginx 缓存配置](#_labe0)
-   [二：nginx 开启 gzip](#_labe1)

一：nginx 缓存配置

在前一篇文章，我们理解过 http 缓存相关的知识点, 请看[这篇文章](https://www.cnblogs.com/tugenhua0707/p/10807289.html). 今天我们来学习下使用 nginx 服务来配置缓存的相关的知识。

nginx 配置缓存的优点：可以在一定程度上，减少服务器的处理请求压力。比如对一些图片，css 或 js 做一些缓存，那么在每次刷新浏览器的时候，就不会重新请求了，而是从缓存里面读取。这样就可以减轻服务器的压力。

nginx 可配置的缓存又有 2 种：  
1）客户端的缓存 (一般指浏览器的缓存)。  
2）服务端的缓存 (使用 proxy-cache 实现的)。

客户端的缓存一般有如下两种方式实现：

协商缓存和强缓存。具体理解什么是协商缓存或强缓存，可以看我之前的[这篇文章](https://www.cnblogs.com/tugenhua0707/p/10807289.html).

在配置之前，我们来看下我们的项目基本架构如下：

![](https://common.cnblogs.com/images/copycode.gif)

|---- 项目 demo |  |--- .babelrc       # 解决 es6 语法问题 |  |--- node_modules   # 所有依赖的包 |  |--- static |  | |--- index.html  # html 页面 |  | |--- css         # 存放 css 文件夹 |  | | |--- base.css  # css 文件，是从网上随便复制过来的很多 css 的 |  | |--- js          # 存放 js 的文件夹 |  | | |--- jquery-1.11.3.js  # jquery 文件 |  | |--- images      # 存放 images 文件夹 |  | | |-- 1.jpg      # 图片对应的文件 |  |--- app.js         # 编写 node 相关的入口文件 |  |--- package.json   # 依赖的包文件

![](https://common.cnblogs.com/images/copycode.gif)

package.json 代码如下：

![](https://common.cnblogs.com/images/copycode.gif)

{"name": "xxx", "version": "1.0.0", "description": "","main":"index.js","scripts": {"dev":"nodemon ./index.js"},"license":"MIT","devDependencies": {},"dependencies": {"@babel/core":"^7.2.2","@babel/preset-env":"^7.2.3","@babel/register":"^7.0.0","koa":"^2.7.0","koa-static":"^5.0.0","nodemon":"^1.19.0","path":"^0.12.7" }
}

![](https://common.cnblogs.com/images/copycode.gif)

app.js 代码如下：

![](https://common.cnblogs.com/images/copycode.gif)

import path from 'path';
import Koa from 'koa'; // 静态资源中间件
import resource from 'koa-static';
const app \\= new Koa();
const host \\= 'localhost';
const port \\= 7878;

app.use(resource(path.join(\_\_dirname, './static')));

app.listen(port, () \\=> {
  console.log(\`server is listen in ${host}:${port}\`);
});

![](https://common.cnblogs.com/images/copycode.gif)

index.js 代码如下：

require('@babel/register');
require('./app.js');

index.html 代码如下：

![](https://common.cnblogs.com/images/copycode.gif)

<!DOCTYPE HTML\>

<html lang\="en"\>
  <head\>
    <meta charset\="UTF-8" />
    <meta name\="viewport" content\="width=device-width, initial-scale=1.0" />
    <title\>前端缓存</title\>
    <style\> .web-cache img { display: block; width: 100%;
      }
    </style\>
    <link href\="./css/base.css" rel\="stylesheet" />
    <script type\="text/javascript" src\="./js/jquery-1.11.3.js"\></script\>
  </head\>
  <body\>
    <div class\="web-cache"\> 1111112224546664456999000 <img src\="./images/1.jpg" />
    </div\>
  </body\>
</html\>

![](https://common.cnblogs.com/images/copycode.gif)

如上就是一些基本的代码结构，当我们在 nginx 没有配置任何的时候，我们直接在命令行中运行 npm run dev 的时候，然后我们在浏览器访问 [http://localhost:7878/](http://localhost:7878/) 时候，可以看到不管我刷新多少次，浏览器下图片，css，js 所有的请求都会返回 200，不会有任何缓存。如下所示：  
![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213224686-2092390794.png)

现在我们去我本地安装的 nginx 中去配置下哦，打开 nginx.conf 配置文件。具体本地 mac 系统下安装 nginx，可以去我之前的文章查看下如何安装，[mac 下安装 nginx](https://www.cnblogs.com/tugenhua0707/p/9863885.html).

打开 nginx.conf，使用 cat /usr/local/etc/nginx/nginx.conf (或者使用 sudo open /usr/local/etc/nginx/nginx.conf -a 'sublime text' 使用编辑器 sublime 打开)。

在 nginx.conf 加入如下规则：

![](https://common.cnblogs.com/images/copycode.gif)

server {
  location ~\* \\.(html)$ {
    access_log off;
    add_header  Cache-Control  max-age=no-cache;
  }

  location ~\* \\.(css|js|png|jpg|jpeg|gif|gz|svg|mp4|ogg|ogv|webm|htc|xml|woff)$ {

    # 同上，通配所有以.css/.js/...结尾的请求
    access\_log off;
    add\_header    Cache\-Control  max-age=360000;

  }
}

![](https://common.cnblogs.com/images/copycode.gif)

如上配置解析含义如下：

**~\* 的含义是**：通配任意字符（且大小写不敏感），\\转义字符，因此 ~\* \\.(html)$ 的含义是：匹配所有以. html 结尾的请求  
**access_log off;** 的含义是 关闭日志功能。

**add_header Cache-Control max-age=no-cache;** 的含义：html 文件不设置强制缓存时间，协商缓存，使用 Last-Modified。no-cache 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则不会下载，返回 304。

如下图所示：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213407462-1288458996.png)

**add_header Cache-Control max-age=360000;** 的含义给上面匹配后缀的文件设置强制缓存，且缓存的时间是 360000 秒，第一次访问的时候，从服务器请求，当除了第一次以外，再次刷新浏览器，会从浏览器缓存读取，那么强制缓存一般是从内存里面先读取，如果内存没有，再从硬盘读取。

如下图所示：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213446300-861238314.png)

注意：如上只是不对反向代理的页面进行缓存设置的，但是如果是反向代理后的页面，如上设置是不生效的。比如说我 node 起了一个服务，然后通过访问 nginx 反向代理的方式代理到我 node 服务来，上面的配置是不生效的。因此我们需要如下处理配置。

**解决 nginx 反向代理缓存不起作用的问题**

比如我上面的 node 服务端口是 7878 端口。nginx 需要如下配置：

![](https://common.cnblogs.com/images/copycode.gif)

server {
  listen 8081;
  server_name  xxx.abc.com;
  location / {
    proxy_pass [http://localhost:7878](http://localhost:7878);
    add_header  Cache-Control  max-age=no-cache;
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

1) 如果我们要添加缓存功能的话，需要创建一个用于存放缓存文件的文件夹。比如我们这里使用 /data/nuget-cache。

在 / usr/local/etc/nginx 目录下新建。比如使用命令：mkdir /data/nuget-cache. 创建完成后，我们来查看下：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213547291-1825222788.png)

2）然后我们需要在 nginx.conf 的 http 设置部分添加 proxy_cache_path 的设置，如下：

![](https://common.cnblogs.com/images/copycode.gif)

http { // ..... 其他的配置
  proxy_cache_path  /data/nuget-cache levels=1:2 keys_zone=nuget-cache:20m max_size=50g inactive=168h;
  server {
    listen 8081;
    server_name  xxx.abc.com;
    location / {
      proxy_pass [http://localhost:7878](http://localhost:7878);
      add_header  Cache-Control  max-age=no-cache;
    }
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

**proxy_cache_path** 各个配置值的含义解析如下：

**proxy_cache_path** 指缓存的目录，目录为：/data/nuget-cache。  
**levels=1:2** 表示采用 2 级目录结构；  
**keys_zone**指的是缓存空间名称，叫 nuget-cache。缓存内存的空间为 20M。  
**max_size**指的是缓存文件可以占用的最大空间。为 50G.  
**inactive=168h;** 默认过期时间为 168 个小时。为 7 天，也可以写成：inactive=7d; 这样的。

3）我们还需要在 server 设置部分添加 proxy_cache 与 proxy_cache_valid 的设置：如下代码：

![](https://common.cnblogs.com/images/copycode.gif)

http { // ..... 其他的配置
  proxy_cache_path  /data/nuget-cache levels=1:2 keys_zone=nuget-cache:20m max_size=50g inactive=168h;
  server {
    listen 8081;
    server_name  xxx.abc.com;
    location / {
      proxy_pass [http://localhost:7878](http://localhost:7878);
      add_header  Cache-Control  max-age=no-cache;
      proxy_cache nuget-cache;
      proxy_cache_valid 168h;
    }
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

**proxy_cache** 设置的是 proxy_cache_path 中的 keys_zone 的值。  
**proxy_cache_valid：** 设置的是缓存过期时间，比如设置 168 个小时过期。

如上配置完成后，我们保存 nginx.conf 配置后，重新启动下 nginx 后，发现还是不能缓存文件了。因此我们还需要进行如下配置：

需要在 server 中再加上如下代码：

proxy_ignore_headers Set-Cookie Cache-Control;
proxy_hide_header Cache-Control;
proxy_hide_header Set-Cookie;

**proxy_ignore_headers 的含义是：** 忽略 Cache-Control 的请求头控制，依然进行缓存，比如对请求头设置 cookie 后，默认是不缓存的，需要我们增加忽略配置。

因此所有配置变成如下了：

![](https://common.cnblogs.com/images/copycode.gif)

http { // ..... 其他的配置
  proxy_cache_path  /data/nuget-cache levels=1:2 keys_zone=nuget-cache:20m max_size=50g inactive=168h;
  server {
    listen 8081;
    server_name  xxx.abc.com;
    location / {
      proxy_pass [http://localhost:7878](http://localhost:7878);
      add_header  Cache-Control  max-age=no-cache;
      proxy_cache nuget-cache;
      proxy_cache_valid 168h;
      proxy_ignore_headers Set-Cookie Cache-Control;
      proxy_hide_header Cache-Control;
      proxy_hide_header Set-Cookie;
    }
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

但是如上写法看起来很繁琐，因此我们可以使用 include 命令把文件包含进来，因此我在 /usr/local/etc/nginx 目录下新建一个 nginx_proxy.conf 配置文件，把上面的 proxy 相关的配置放到该文件里面，如下所示：  
![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213826905-2006958141.png)

然后我们的配置就变成如下了：

![](https://common.cnblogs.com/images/copycode.gif)

http { // ..... 其他的配置
  proxy_cache_path  /data/nuget-cache levels=1:2 keys_zone=nuget-cache:20m max_size=50g inactive=168h;
  include nginx_proxy.conf;
  server {
    listen 8081;
    server_name  xxx.abc.com;
    location / {
      proxy_pass [http://localhost:7878](http://localhost:7878);
      add_header  Cache-Control  max-age=no-cache;
    }
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

如上是对页面使用协商缓存的，但是对于图片，css, 或 js 这样的，我想使用强制缓存，因此对于其他的类型文件我们统一如下这样处理：

![](https://common.cnblogs.com/images/copycode.gif)

server {
  listen 8081;
  server_name  xxx.abc.com;
  location / {
    proxy_pass [http://localhost:7878](http://localhost:7878);
    add_header  Cache-Control  max-age=no-cache;
  }
  location ~\* \\.(css|js|png|jpg|jpeg|gif|gz|svg|mp4|ogg|ogv|webm|htc|xml|woff)$ {
    access_log off;
    add_header Cache-Control "public,max-age=30\*24\*3600";
    proxy_pass [http://localhost:7878](http://localhost:7878);
 }
  error_page 500 502 503 504  /50x.html;
  location = /50x.html {root   html;}
}

![](https://common.cnblogs.com/images/copycode.gif)

如上 css 或 js 文件等缓存的时间是 30 天。使用的是 max-age 强制缓存。因此如上，如果是页面第二次访问的话，会返回 304，如下所示：  
![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213920986-1728271796.png)

如果是 css 或 js 这样的访问的话，就是强制缓存了，状态码还是 200，但是先从内存里面读取的。当然如果进程结束了，比如浏览器关闭了，再打开，那么是从硬盘上读取的了。如下所示：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509213935225-1904647729.png)

因此 nginx.conf 所有的配置文件代码如下：

![](https://common.cnblogs.com/images/copycode.gif)

worker_processes  1;
events {
  worker_connections 1024;
}
http {
  include       mime.types;
  default_type  application/octet-stream;
 sendfile        on;
  \#tcp_nopush     on;
  \#keepalive_timeout 0;
  keepalive_timeout 65;
  include nginx_proxy.conf;
  proxy_cache_path /data/nuget-cache levels=1:2 keys_zone=nuget-cache:20m max_size=50g inactive=168h;
  \#gzip  on;
  server {
    listen 8081;
    server_name  xxx.abc.com;
    location / {
      proxy_pass [http://localhost:7878](http://localhost:7878);
      add_header  Cache-Control  max-age=no-cache;
    }
    location ~\* \\.(css|js|png|jpg|jpeg|gif|gz|svg|mp4|ogg|ogv|webm|htc|xml|woff)$ {
      access_log off;
      add_header Cache-Control "public,max-age=30\*24\*3600";
      proxy_pass [http://localhost:7878](http://localhost:7878);
 }
    error_page 500 502 503 504  /50x.html;
    location = /50x.html {root   html;}
  }
}

![](https://common.cnblogs.com/images/copycode.gif)

如上上面的 css，js 这些我时间设置短一点，比如设置 60 秒过期的话，那么过期后，我再刷新浏览器，浏览器会去询问服务器端，检查资源是否被更新了，如果资源没有被更新的话，那么服务器端会范湖 304. 资源依然读取本地的。如下所示：  
![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509214016701-2032923329.png)

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509214023011-273400707.png)

然后再继续刷新的话，它之后又从内存里面读取了。依次这样循环下去。

二：nginx 开启 gzip

 开启 gzip 配置是在 http 层加的。基本配置代码如下：

![](https://common.cnblogs.com/images/copycode.gif)

\# 开启 gzip
gzip on;

# 启用 gzip 压缩的最小文件；小于设置值的文件将不会被压缩

gzip_min_length 1k;

# gzip 压缩级别 1-10 gzip_comp_level 2;

# 进行压缩的文件类型。

gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

# 是否在 http header 中添加 Vary: Accept-Encoding，建议开启

gzip_vary on;

![](https://common.cnblogs.com/images/copycode.gif)

我们如上的配置加上去后，如在 http 下加上上面的 gzip 代码：

![](https://common.cnblogs.com/images/copycode.gif)

http {

# 开启 gzip

  gzip on;

# 启用 gzip 压缩的最小文件；小于设置值的文件将不会被压缩

  gzip_min_length 1k;

# gzip 压缩级别 1-10 gzip_comp_level 2;

# 进行压缩的文件类型。

  gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

# 是否在 http header 中添加 Vary: Accept-Encoding，建议开启

  gzip_vary on;
}

![](https://common.cnblogs.com/images/copycode.gif)

我们可以先来对比下，如果我们没有开启 zip 压缩之前，我们的对应的文件大小，如下所示：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509214135708-143961921.png)

现在我们开启了 gzip 进行压缩后的文件的大小，可以看到如下所示：

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509214148865-184228953.png)

并且我们查看响应头会看到 gzip 这样的压缩，如下所示

![](https://img2018.cnblogs.com/blog/561794/201905/561794-20190509214202019-1407843314.png)

-   分类 [nginx 相关的](https://www.cnblogs.com/tugenhua0707/category/1205593.html) , [缓存相关的](https://www.cnblogs.com/tugenhua0707/category/1456995.html)
-   标签 [nginx 缓存配置及开启 gzip 压缩](https://www.cnblogs.com/tugenhua0707/tag/nginx%E7%BC%93%E5%AD%98%E9%85%8D%E7%BD%AE%E5%8F%8A%E5%BC%80%E5%90%AFgzip%E5%8E%8B%E7%BC%A9/) 
    [https://www.cnblogs.com/tugenhua0707/p/10841267.html](https://www.cnblogs.com/tugenhua0707/p/10841267.html)
