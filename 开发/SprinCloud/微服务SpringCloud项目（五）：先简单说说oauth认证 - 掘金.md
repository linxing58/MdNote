# 微服务SpringCloud项目（五）：先简单说说oauth认证 - 掘金
**小知识，大挑战！本文正在参与「[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https&#x3A;//juejin.cn/post/7008476801634680869")」创作活动**

**本文已参与 [「掘力星计划」](https://juejin.cn/post/7012210233804079141 "https&#x3A;//juejin.cn/post/7012210233804079141") ，赢取创作大礼包，挑战创作激励金。** 

## 📖前言

    心态好了，就没那么累了。心情好了，所见皆是明媚风景。
    复制代码

> **`“一时解决不了的问题，那就利用这个契机，看清自己的局限性，对自己进行一场拨乱反正。” 正如老话所说，一念放下，万般自在。如果你正被烦心事扰乱心神，不妨学会断舍离。断掉胡思乱想，社区垃圾情绪，离开负面能量。心态好了，就没那么累了。心情好了，所见皆是明媚风景。`**

### 🚓`SpringCloudOauth2`

### `Oauth2` 简介

`OAuth 2.0` 是用于授权的行业标准协议。`OAuth 2.0` 致力于简化客户端开发人员，同时为 `Web` 应用程序，桌面应用程序，移动电话和客厅设备提供特定的授权流程。该规范及其扩展正在 `IETF OAuth` 工作组内开发。

`OAuth 2.0` 的标准是 `RFC 6749` 文件。`OAuth` 的核心就是向第三方应用颁发令牌。它定义了获得令牌的四种授权方式（`authorization grant` ）。

授权流程图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09c21ee17c454489a78096ae4c32c599~tplv-k3u1fbpfcp-watermark.awebp?)

简单一点的诉述就是发起一个认证请求，根据授权类型去认证服务器认证，如果成功就返回 `token`，再使用 `token` 去访问资源服务器，`token` 验证通过就返回被保护的资源。

### 包含的角色

`OAuth` 定义了四个角色：

-   **资源所有者**：能够授予对受保护资源的访问权限的实体。当资源所有者是一个人时，它称为最终用户。
-   **资源服务器**：托管受保护资源的服务器，能够接受并使用访问令牌响应受保护的资源请求。
-   **客户端**：对我们的产品来说，QQ、微信登录是第三方登录系统。我们又需要第三方登录系统的资源（头像、昵称等）
-   **授权服务器**：请求授权成功后，服务器向客户端发布访问令牌认证资源所有者并获得授权

#### 授权模式（authorization-code）

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌漏。

第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。

```java
https:
  response_type=code&  #  固定写法
  client_id=CLIENT_ID& #  client_id 自己定义的
  redirect_uri=CALLBACK_URL& #回调地址，授权成功跳转到地址并携带code=XXXX，之后再用XXXX去申请令牌
  scope=read  #非必须
```

第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 `A` 网站授权。用户表示同意，这时 `B` 网站就会跳回 `redirect_uri` 参数指定的网址。跳转时，会传回一个 `授权码`，就像下面这样。

`https://CALLBACK_URL?code=AUTHORIZATION_CODE`

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

```java
https:
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

上面 `URL` 中，`client_id` 参数和 `client_secret 参数` 用来让 B 确认 A 的身份（`client_secret 参数是保密的，因此只能在后端发请求`），`grant_type` 参数的值是 `authorization_code`，表示采用的授权方式是授权码，`code 参数` 是上一步拿到的授权码，`redirect_uri` 参数是令牌颁发后的回调网址。

第四步，B 网站收到请求以后，就会颁发令牌。

```json
{
    "access_token": "xxxxxxxxxxxxxxxxxxxxxxxxx",
    "token_type": "bearer",
    "refresh_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "expires_in": 86399,
    "scope": "all",
    "username1": "demo",
    "license": "sunnychen"
}
```

#### 隐藏式（implicit）

有些 `Web` 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。也就有了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为 "隐藏式"（implicit）。

第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用

```java
https:
  response_type=token& #response_type参数为token，表示要求直接返回令牌
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回 redirect_uri 参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

`https:// 服务器 IP: 服务器 PORT/callback#token=ACCESS_TOKEN #token=ACCESS_TOKEN` 这里就是 token

> **PS：注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在 "中间人攻击" 的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。** 

**`这种方式安全性很低，所以一般用于对安全性要求不高的场景，并且 token 有效期很短，一般也就是当前 session，会话结束就失效。`**

#### 密码式（password）：

如果你高度信任某个应用，可以直接把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为 "密码式"（password）。

第一步，A 要求用户提供 B 的用户名和密码。拿到以后，A 就直接向 B 请求令牌。

```java
https:
  grant_type=password&  #password指明使用密码模式
  username=USERNAME&    #用户名
  password=PASSWORD&    #密码
  client_id=CLIENT_ID   #定义的client
```

第二步，B 验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 `JSON` 数据里面，作为 `HTTP` 回应，A 因此拿到令牌。

```json
{
    "access_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "token_type": "bearer",
    "refresh_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "expires_in": xxxx,
    "scope": "all",
    "username1": "demo",
    "license": "sunnychen"
}
```

这种方式毕竟是直接给出了用户的账号密码，最好是在自己的系统内部使用的场景。如果不是都必须是一种相互高度信任的场景

#### 客户端凭证（client credentials）

用于没有前端的命令行应用，即在命令行下请求令牌。

第一步，A 应用在命令行向 B 发出请求。

```java
https:
  grant_type=client_credentials&  #指定使用客户端凭证模式
  client_id=CLIENT_ID&            #定义的client
  client_secret=CLIENT_SECRET     #定义的secret
```

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

刷新令牌 OAuth 2.0 允许用户自动更新令牌，当令牌的有效期到了以后，如果让用户重新走一次上面的流程，对于用户的体验来说相当的不友好。

```json
{
    "access_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "token_type": "bearer",
    "refresh_token": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "expires_in": 86399,
    "scope": "all",
    "username1": "demo",
    "license": "sunnychen"
}
```

在我们上面请求到的 token 中，都是同时颁发了 access_token 以及 refresh_token。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

```java
https:
  grant_type=refresh_token&  #指定是刷新令牌
  client_id=CLIENT_ID&       #定义的client
  client_secret=CLIENT_SECRET& #定义的secret
  refresh_token=REFRESH_TOKEN  #使用获取令牌时同时返回的refresh_token
```

B 网站验证通过以后，就会颁发新的令牌。

### ✨Spring Security

#### 特点

对身份验证和授权的全面且可扩展的支持

防止攻击，例如会话固定，点击劫持，跨站点请求伪造等

#### `Servlet API` 集成

`Spring Web MVC` 的可选集成

### 🎊Spring Cloud Security

#### 简介

`Spring Cloud Security` 提供了一组原语，用于以最少的麻烦构建安全的应用程序和服务。可以在外部（或中央）进行大量配置的声明式模型通常可以通过中央身份管理服务来实现大型的，相互协作的远程组件系统。在 `Cloud Foundry` 等服务平台中使用它也非常容易。在 `Spring Boot` 和 `Spring Security OAuth2` 的基础上，我们可以快速创建实现常见模式（如单点登录，令牌中继和令牌交换）的系统。

**`根据最新的相关资料显示，Spring 官方已经不支持 Spring Security Oauth2 了，他们将对这技术进行迁移，换句话说就是打算自己搞一个。`**

* * *

**下节将 `oauth` 认证的搭建和初步使用，欢迎持续关注~ 最后感谢大家耐心观看完毕，留个点赞收藏是您对我最大的鼓励！**

* * *

### 🎉总结：

-   **更多参考精彩博文请看这里：[《陈永佳的博客》](https://juejin.cn/user/862483929905639/posts "https&#x3A;//juejin.cn/user/862483929905639/posts")**
-   **喜欢博主的小伙伴可以加个关注、点个赞哦，持续更新嘿嘿！** 
    [https://juejin.cn/post/7016911111463125000](https://juejin.cn/post/7016911111463125000)
