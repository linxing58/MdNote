# 死磕nginx系列--nginx 限流配置
## 限流算法

### 令牌桶算法

![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140242681-53841293.jpg)

算法思想是：

-   令牌以固定速率产生，并缓存到令牌桶中；
-   令牌桶放满时，多余的令牌被丢弃；
-   请求要消耗等比例的令牌才能被处理；
-   令牌不够时，请求被缓存。

### 漏桶算法

![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140248838-284301317.png)

算法思想是：

-   水（请求）从上方倒入水桶，从水桶下方流出（被处理）；
-   来不及流出的水存在水桶中（缓冲），以固定速率流出；
-   水桶满后水溢出（丢弃）。
-   这个算法的核心是：缓存请求、匀速处理、多余的请求直接丢弃。  
    相比漏桶算法，令牌桶算法不同之处在于它不但有一只 “桶”，还有个队列，这个桶是用来存放令牌的，队列才是用来存放请求的。

从作用上来说，漏桶和令牌桶算法最明显的区别就是是否允许突发流量 (burst) 的处理，漏桶算法能够强行限制数据的实时传输（处理）速率，对突发流量不做额外处理；而令牌桶算法能够在限制数据的平均传输速率的同时允许某种程度的突发传输。

Nginx 按请求速率限速模块使用的是漏桶算法，即能够强行保证请求的实时处理速度不会超过设置的阈值。

Nginx 官方版本限制 IP 的连接和并发分别有两个模块：

-   `limit_req_zone` 用来限制单位时间内的请求数，即速率限制, 采用的漏桶算法 "leaky bucket"。
-   `limit_req_conn` 用来限制同一时间连接数，即并发限制。

## limit_req_zone 参数配置

```null
Syntax:	limit_req zone=name [burst=number] [nodelay];
Default:	—
Context:	http, server, location
```

`limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;`

-   第一个参数：$binary_remote_addr 表示通过 remote_addr 这个标识来做限制，“binary\_” 的目的是缩写内存占用量，是限制同一客户端 ip 地址。
-   第二个参数：zone=one:10m 表示生成一个大小为 10M，名字为 one 的内存区域，用来存储访问的频次信息。
-   第三个参数：rate=1r/s 表示允许相同标识的客户端的访问频次，这里限制的是每秒 1 次，还可以有比如 30r/m 的。

`limit_req zone=one burst=5 nodelay;`

-   第一个参数：zone=one 设置使用哪个配置区域来做限制，与上面 limit_req_zone 里的 name 对应。
-   第二个参数：burst=5，重点说明一下这个配置，burst 爆发的意思，这个配置的意思是设置一个大小为 5 的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
-   第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回 503，如果没有设置，则所有请求会等待排队。

例子：

```null
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    server {
        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
}        
```

下面配置可以限制特定 UA（比如搜索引擎）的访问：

```null
limit_req_zone  $anti_spider  zone=one:10m   rate=10r/s;
limit_req zone=one burst=100 nodelay;
if ($http_user_agent ~* "googlebot|bingbot|Feedfetcher-Google") {
    set $anti_spider $http_user_agent;
}
```

其他参数

```null
Syntax:	limit_req_log_level info | notice | warn | error;
Default:	
limit_req_log_level error;
Context:	http, server, location
```

当服务器由于 limit 被限速或缓存时，配置写入日志。延迟的记录比拒绝的记录低一个级别。例子：`limit_req_log_level notice`延迟的的基本是 info。

```null
Syntax:	limit_req_status code;
Default:	
limit_req_status 503;
Context:	http, server, location
```

设置拒绝请求的返回值。值只能设置 400 到 599 之间。

## ngx_http_limit_conn_module 参数配置

这个模块用来限制单个 IP 的请求数。并非所有的连接都被计数。只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

```null
Syntax:	limit_conn zone number;
Default:	—
Context:	http, server, location
```

```null
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /download/ {
        limit_conn addr 1;
    }
```

一次只允许每个 IP 地址一个连接。

```null
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;

server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```

可以配置多个 limit_conn 指令。例如，以上配置将限制每个客户端 IP 连接到服务器的数量，同时限制连接到虚拟服务器的总数。

```null
Syntax:	limit_conn_zone key zone=name:size;
Default:	—
Context:	http
```

```null
limit_conn_zone $binary_remote_addr zone=addr:10m;
```

在这里，客户端 IP 地址作为关键。请注意，不是`$ remote_addr`，而是使用`$ binary_remote_addr`变量。 `$ remote_addr`变量的大小可以从 7 到 15 个字节不等。存储的状态在 32 位平台上占用 32 或 64 字节的内存，在 64 位平台上总是占用 64 字节。对于 IPv4 地址，`$ binary_remote_addr`变量的大小始终为 4 个字节，对于 IPv6 地址则为 16 个字节。存储状态在 32 位平台上始终占用 32 或 64 个字节，在 64 位平台上占用 64 个字节。一个兆字节的区域可以保持大约 32000 个 32 字节的状态或大约 16000 个 64 字节的状态。如果区域存储耗尽，服务器会将错误返回给所有其他请求。

```null
Syntax:	limit_conn_log_level info | notice | warn | error;
Default:	
limit_conn_log_level error;
Context:	http, server, location
```

当服务器限制连接数时，设置所需的日志记录级别。

```null
Syntax:	limit_conn_status code;
Default:	
limit_conn_status 503;
Context:	http, server, location
```

设置拒绝请求的返回值。

## 实战

### 实例一 限制访问速率

```null
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
        limit_req zone=mylimit;
    }
}
```

上述规则限制了每个 IP 访问的速度为 2r/s，并将该规则作用于根目录。如果单个 IP 在非常短的时间内并发发送多个请求，结果会怎样呢？  
![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140422316-147466052.jpg)

我们使用单个 IP 在 10ms 内发并发送了 6 个请求，只有 1 个成功，剩下的 5 个都被拒绝。我们设置的速度是 2r/s，为什么只有 1 个成功呢，是不是 Nginx 限制错了？当然不是，是因为 Nginx 的限流统计是基于毫秒的，我们设置的速度是 2r/s，转换一下就是 500ms 内单个 IP 只允许通过 1 个请求，从 501ms 开始才允许通过第二个请求。

### 实例二 burst 缓存处理

我们看到，我们短时间内发送了大量请求，Nginx 按照毫秒级精度统计，超出限制的请求直接拒绝。这在实际场景中未免过于苛刻，真实网络环境中请求到来不是匀速的，很可能有请求 “突发” 的情况，也就是 “一股子一股子” 的。Nginx 考虑到了这种情况，可以通过 burst 关键字开启对突发请求的缓存处理，而不是直接拒绝。  
来看我们的配置：

```null
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
        limit_req zone=mylimit burst=4;
    }
}
```

我们加入了 burst=4，意思是每个 key(此处是每个 IP) 最多允许 4 个突发请求的到来。如果单个 IP 在 10ms 内发送 6 个请求，结果会怎样呢？  
![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140430356-1366473245.jpg)

相比实例一成功数增加了 4 个，这个我们设置的 burst 数目是一致的。具体处理流程是：1 个请求被立即处理，4 个请求被放到 burst 队列里，另外一个请求被拒绝。通过 burst 参数，我们使得 Nginx 限流具备了缓存处理突发流量的能力。

但是请注意：burst 的作用是让多余的请求可以先放到队列里，慢慢处理。如果不加 nodelay 参数，队列里的请求不会立即处理，而是按照 rate 设置的速度，以毫秒级精确的速度慢慢处理。

### 实例三 nodelay 降低排队时间

实例二中我们看到，通过设置 burst 参数，我们可以允许 Nginx 缓存处理一定程度的突发，多余的请求可以先放到队列里，慢慢处理，这起到了平滑流量的作用。但是如果队列设置的比较大，请求排队的时间就会比较长，用户角度看来就是 RT 变长了，这对用户很不友好。有什么解决办法呢？nodelay 参数允许请求在排队的时候就立即被处理，也就是说只要请求能够进入 burst 队列，就会立即被后台 worker 处理，请注意，这意味着 burst 设置了 nodelay 时，系统瞬间的 QPS 可能会超过 rate 设置的阈值。nodelay 参数要跟 burst 一起使用才有作用。

延续实例二的配置，我们加入 nodelay 选项：

```null
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
        limit_req zone=mylimit burst=4 nodelay;
    }
}
```

单个 IP 10ms 内并发发送 6 个请求，结果如下：  
![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140438018-343740650.jpg)

跟实例二相比，请求成功率没变化，但是总体耗时变短了。这怎么解释呢？实例二中，有 4 个请求被放到 burst 队列当中，工作进程每隔 500ms(rate=2r/s) 取一个请求进行处理，最后一个请求要排队 2s 才会被处理；实例三中，请求放入队列跟实例二是一样的，但不同的是，队列中的请求同时具有了被处理的资格，所以实例三中的 5 个请求可以说是同时开始被处理的，花费时间自然变短了。

但是请注意，虽然设置 burst 和 nodelay 能够降低突发请求的处理时间，但是长期来看并不会提高吞吐量的上限，长期吞吐量的上限是由 rate 决定的，因为 nodelay 只能保证 burst 的请求被立即处理，但 Nginx 会限制队列元素释放的速度，就像是限制了令牌桶中令牌产生的速度。

看到这里你可能会问，加入了 nodelay 参数之后的限速算法，到底算是哪一个 “桶”，是漏桶算法还是令牌桶算法？当然还算是漏桶算法。考虑一种情况，令牌桶算法的 token 为耗尽时会怎么做呢？由于它有一个请求队列，所以会把接下来的请求缓存下来，缓存多少受限于队列大小。但此时缓存这些请求还有意义吗？如果 server 已经过载，缓存队列越来越长，RT 越来越高，即使过了很久请求被处理了，对用户来说也没什么价值了。所以当 token 不够用时，最明智的做法就是直接拒绝用户的请求，这就成了漏桶算法。

### 示例四 自定义返回值

```null
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
        limit_req zone=mylimit burst=4 nodelay;
        limit_req_status 598;
    }
}
```

默认情况下 没有配置 status 返回值的状态：  
![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140535388-1660896627.jpg)

自定义 status 返回值的状态：  
![](https://images2018.cnblogs.com/blog/802666/201805/802666-20180502140542215-2050682677.jpg)

## 参考文档

[Nginx 限制访问速率和最大并发连接数模块 --limit (防止 DDOS 攻击)](http://www.cnblogs.com/wjoyxt/p/6128183.html)  
[Nginx 限流](http://colobu.com/2015/10/26/nginx-limit-modules/)  
[关于 nginx 的限速模块](https://www.cnblogs.com/chenpingzhao/p/4971308.html)  
[Nginx 源代码笔记 - HTTP 模块 - 流控](http://ialloc.org/posts/2014/07/26/ngx-notes-module-http-limit/)  
[Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)  
[Module ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)  
[Nginx 限速模块初探](https://www.cnblogs.com/CarpenterLee/p/8084533.html) 
 [https://www.cnblogs.com/biglittleant/p/8979915.html](https://www.cnblogs.com/biglittleant/p/8979915.html)
