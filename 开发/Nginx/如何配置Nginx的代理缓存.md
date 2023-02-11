# 如何配置Nginx的代理缓存
我们都知道应用程序和网站的性能是他们成功的关键因素。但是，使您的应用程序或网站表现更好的过程并不总是很清楚。

代码质量和基础架构当然至关重要，但在许多情况下，您可以通过专注于一些非常基本的应用程序的交付技术，对应用程序的最终用户体验进行大量改进。

其中一个例子是在应用程序栈中实现和优化缓存。在教程中介绍的技术可以帮助新手和高级用户使用NGINX中包含的内容缓存功能，从而获得更好的性能。

内容缓存位于客户端和源服务器(upstream)之间，并保存它看到的所有内容的副本。如果客户端请求缓存已存储的内容，则它会直接返回内容而不连接源服务器。

这提高了性能，因为内容缓存更靠近客户端，并且更有效地使用应用程序服务器，因为它们不必每次都从头开始生成页面。

Web浏览器和应用程序服务器之间可能存在多个缓存。包括客户端的浏览器缓存，中间缓存，内容交付网络CDN。

以及位于应用程序服务器前面的负载均衡器或反向代理。即使在反向代理/负载均衡器级别，缓存也可以极大地提高性能。

NGINX通常作为应用程序堆栈中的反向代理或负载均衡器部署，并具有一整套缓存功能。本教程中我们将讨论如何使用NGINX配置基本缓存。

配置缓存
----

Nginx只需要两个指令即可启用代理缓存，分别是`[proxy_cache_path](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path)`和`[proxy_cache](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache)`。`proxy_cache_path`指令设置缓存的路径，`proxy_cache`指令用来激活它。

```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

server {
    # ...
    location / {
        proxy_cache myfreax.com_cache;
        proxy_pass http://myfreax.com_upstream;
    }
}
```

`proxy_cache_path`指令的参数定义缓存的本地磁盘目录/path/to/cache/。

`levels`在缓存目录下设置一个两级目录层次的结构。在单个目录中包含大量文件会降低文件访问速度，因此我们建议对大多数部署使用两级目录层次结构。

如果`levels`未包含该参数，Nginx会将所有文件放在同一目录中。

`keys_zone`设置共享内存区域，用于存储缓存键和元数据，例如使用计时器。拥有内存中的密钥副本。

Nginx可以快速确定请求是否是命中缓存`HIT`或`MISS`，而不必读取磁盘，从而加快了检查速度。

1MB区域可以存储大约8,000个密钥的数据，因此示例中配置的10MB区域可以存储大约80,000个密钥的数据。

`max_size`设置缓存大小的上限，在本例中为10G。它是可选的，不指定值允许缓存增长以使用所有可用磁盘空间。

当缓存大小达到限制时，一个称为缓存管理器的进程将删除最近最少使用的缓存，将大小恢复到限制之下。

`inactive`指定项目在未被访问的情况下可以保留在缓存中的时间长度。在此示例中，缓存管理器进程会自动从缓存中删除60分钟未请求过的文件。

无论其是否已过期。默认值为10分钟`10m`。非活动内容与过期内容不同。Nginx不会自动删除缓存中header定义为已过期内容，例如`Cache-Control:max-age=120`。

过期内容仅在指定时间内未被访问时被删除。访问过期内容时，Nginx会从原始服务器刷新它并重置`inactive`计时器。

Nginx首先将发往高速缓​​存的文件写入临时存储区域，`use_temp_path=off`指令指示Nginx将它们写入将被高速缓存的相同目录。

我们建议您将此参数设置`off`为避免在文件系统之间进行不必要的数据复制。`use_temp_path`在Nginx 1.7.10中引入。

最后，`proxy_cache`指令激活缓存，将缓存与父`location`块的URL匹配的所有内容。

您也可以在`server`块中包含`proxy_cache`指令，它适用于没有自己的`location`指令的server块。

提供缓存内容
------

Nginx内容缓存是一个强大的功能，Nginx可以配置在无法从原始服务器获取新内容时从缓存中提供已缓存的内容。

如果缓存资源的所有源服务器都已关闭或暂时占用，则会发生这种情况。Nginx不是将错误传递给客户端，而是从缓存中提供文件的旧版本。

这为Nginx代理的服务器提供了额外的容错能力，并确保在服务器故障或流量高峰时的正常运行时间。要启用此功能，可以使用`proxy_cache_use_stale`指令。

使用此示例配置，如果Nginx从原始服务器收到一个错误`error`，超时`timeout`或任何指定的`5xx`错误。

Nginx在其缓存中读取文件，发送缓存的副本到客户端，而不是将错误转发到客户端。

```
location / {
    # ...
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```

提高缓存性能
------

NGINX具有丰富的可选设置，可用于微调缓存的性能。这是活其中一些的例子。

```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g 
                 inactive=60m use_temp_path=off;

server {
    # ...
    location / {
        proxy_cache my_cache;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 3;
        proxy_cache_use_stale error timeout updating http_500 http_502
                              http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;

        proxy_pass http://my_upstream;
    }
}
```

这些指令配置的行为包括。

`proxy_cache_revalidate`指示Nginx重新验证缓存。`If-Modified-Since`字段包含在`GET`请求时。

Nginx从源服务器刷新内容。如果客户端请求的缓存与`If-Modified-Since`对比是过期的内容时，NGINX则将它发送到源服务器。

`proxy_cache_min_uses`设置Nginx在缓存该请求之前必须请求多少次才被缓存。如果缓存不够使用的情况下，这将非常有用。

因为它可确保只将最常访问的请求添加到缓存中。默认的`proxy_cache_min_uses`设置为1。

`updating`指令参数[`proxy_cache_use_stale`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_use_stale)和[`proxy_cache_background_update`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_background_update)指令相结合，指示当客户端请求已过期或正在从原始服务器更新的缓存。

Nginx会发送缓存内容。所有更新都将在后台完成。在完全更新之前，将为所有请求返回缓存文件。

如果`proxy_cache_lock`启用，多个客户端请求的文件不在缓存中，只有第一个请求是通过原始服务器的。其余请求将等待第一个请求的缓存，然后从缓存中提取文件。

如果`proxy_cache_lock`未启用，将会使未命中缓存的所有请求都将直接发送到源服务器。

检查是否使用Nginx缓存
-------------

如果需要知道Nginx是否使用了缓存，你可以在响应头中加入$upstream\_cache\_status变量以进行检测。

`add_header X-Cache-Status $upstream_cache_status;`这是在Nginx添加缓存命中状态的add_header指令。

此示例`X-Cache-Status`在响应客户端时添加HTTP标头。以下是`$upstream_cache_status`可能的值。

`MISS`在缓存中找不到响应，因此从原始服务器获取响应。然后缓存响应。`HIT`响应直接来自有效的缓存。

`BYPASS`响应是从原始服务器获取的，而不是从缓存中提供的，因为请求与`proxy_cache_bypass`指令匹配。

`EXPIRED`缓存中的记录已过期。响应包含来自原始服务器的新内容。

`STALE`内容过时，因为源服务器未正确响应但`proxy_cache_use_stale`已配置。

`UPDATING`缓存内容已过时，因为当前正在更新以响应先前的请求，并且`proxy_cache_use_stale updating`已配置。

`REVALIDATED`-`[proxy_cache_revalidate](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_revalidate)`指令已启用，Nginx验证当前缓存的内容是否仍然有效通过`If-Modified-Since`或`If-None-Match`。

缓存控制
----

默认情况下，Nginx遵循`Cache-Control`。它不缓存源服务器响应`Cache-Control`设置为`Private`，`No-Cache`，`No-Store`在响应头。

Nginx只缓存`GET`和`HEAD`的客户端请求。当然如果你需要Nginx可以缓存任何请求。你可使用`proxy_cache_methods`指令设置需要缓存客户端那些请求。

例如Nginx指令`proxy_cache_methods GET HEAD POST;`启用`POST`请求的缓存。

如果`proxy_buffering`设置为`off`，Nginx不会缓存来自源服务器的响应。默认是`on`。使用`proxy_ignore_headers`指令可以忽略包含Cache-Control的header。

`proxy_cache_bypass`指令定义Nginx立即从源服务器请求内容的请求类型，而不是首先尝试在缓存中找到它。这有时被称为缓存打孔。

下面的示例Nginx忽略源服务器的Cache-Control。

```
location /images/ {
    proxy_cache my_cache;
    proxy_ignore_headers Cache-Control;
    # ...
}
```

处理**Byte-Range**请求
------------------

如果文件在高速缓存中是最新的，则NGINX遵循字节范围请求并仅向项目客户端提供项目的指定字节。

如果文件未缓存，或者文件过时，NGINX会从原始服务器下载整个文件。如果请求是针对单个字节范围的，则NGINX会在下载流中遇到该范围后立即将该范围发送到客户端。

如果请求在同一文件中指定了多个字节范围，则NGINX会在下载完成时将整个文件传送到客户端。

下载完成后，NGINX会将整个资源移动到缓存中，以便从缓存中立即满足所有未来的字节范围请求，无论是单个范围还是多个范围。

请注意，`upstream`服务器必须支持Nginx的字节范围请求，以支持字节范围请求。

结论
--

至此，您应该很好地理解`nginx`代理缓存的工作原理以及如何正确配置Nginx代理缓存。如果您有任何问题或反馈，请随时发表评论