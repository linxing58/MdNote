# Spring Cloud(6)：保护微服务（Security） - OAuth2.0 - Storm_L - 博客园
OAuth2是一个**授权（Authorization）**协议。我们要和Spring Security的**认证（Authentication）**区别开来，认证（Authentication）证明的你是不是这个人，而授权（Authorization）则是证明这个人有没有访问这个资源（Resource）的权限。  
下面这张图来源于[OAuth 2.0 authorization framework RFC Document](https://tools.ietf.org/html/rfc6749)，是OAuth2的一个抽象流程。

![](https://common.cnblogs.com/images/copycode.gif)

     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization **Grant** \---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization **Grant** \-->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+

![](https://common.cnblogs.com/images/copycode.gif)

先来解释一下上图的名词：

**Resource Owner**：资源所有者，即用户

**Client**：客户端应用程序（Application）

**Authorization Server**：授权服务器

**Resource Server**：资源服务器

再来解释一下上图的大致流程：

（A） 用户连接客户端应用程序以后，客户端应用程序要求用户给予授权

（B） 用户同意给予客户端应用程序授权

（C） 客户端应用程序使用上一步获得的授权（Grant），向授权服务器申请令牌

（D） 授权服务器对客户端应用程序的授权（Grant）进行验证后，确认无误，发放令牌

（E） 客户端应用程序使用令牌，向资源服务器申请获取资源

（F） 资源服务器确认令牌无误，同意向客户端应用程序开放资源

从上面的流程可以看出，如何获取**授权（Grant）**才是关键。在OAuth2中有4种授权类型：

  **1\. Authorization Code（授权码模式）**：功能最完整、流程最严密的授权模式。通过第三方应用程序服务器与认证服务器进行互动。广泛用于各种第三方认证。

  2\. Implicit（简化模式）：不通过第三方应用程序服务器，直接在浏览器中向认证服务器申请令牌，更加适用于移动端的App及没有服务器端的第三方单页面应用。

  **3\. Resource Owner Password（密码模式）**：用户向客户端服务器提供自己的用户名和密码，用户对客户端高度信任的情况下使用，比如公司、组织的内部系统，SSO。

  4\. Client Credentials（客户端模式）：客户端服务器以自己的名义，而不是以用户的名义，向认证服务器进行认证。

下面主要讲最常用的（1）和（3）。此外，还有一个模式叫Refresh Token，也会在下面介绍。

**Authorization Code（授权码模式）**

![](https://common.cnblogs.com/images/copycode.gif)

     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)

   Note: The lines illustrating steps (A), (B), and (C) are broken into
   two parts as they pass through the user-agent.

![](https://common.cnblogs.com/images/copycode.gif)

它的步骤如下：

（A） 用户（Resource Owner）通过用户代理（User-Agent）访问客户端（Client），客户端索要授权，并将用户导向认证服务器（Authorization Server）。

（B） 用户选择是否给予客户端授权。

（C） 假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。

（D） 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。

（E） 认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。这一步也对用户不可见。

**Resource Owner Password（密码模式）**

![](https://common.cnblogs.com/images/copycode.gif)

     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          v
          |    Resource Owner
         (A) Password Credentials
          |
          v
     +---------+                                  +---------------+
     |         |>--(B)---- Resource Owner ------->|               |
     |         |         Password Credentials     | Authorization |
     | Client  |                                  |     Server    |
     |         |<--(C)---- Access Token ---------<|               |
     |         |    (w/ Optional Refresh Token)   |               |
     +---------+                                  +---------------+

            Figure 5: Resource Owner Password Credentials Flow

![](https://common.cnblogs.com/images/copycode.gif)

它的步骤如下：

（A） 用户（Resource Owner）向客户端（Client）提供用户名和密码。

（B） 客户端将用户名和密码发给认证服务器（Authorization Server），向后者请求令牌。

（C） 认证服务器确认无误后，向客户端提供访问令牌。

**令牌刷新（refresh token）**

![](https://common.cnblogs.com/images/copycode.gif)

  +--------+                                           +---------------+
  |        |--(A)------- Authorization Grant --------->|               |
  |        |                                           |               |
  |        |<-(B)----------- Access Token -------------|               |
  |        |               & Refresh Token             |               |
  |        |                                           |               |
  |        |                            +----------+   |               |
  |        |--(C)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(D)- Protected Resource --| Resource |   | Authorization |
  | Client |                            |  Server  |   |     Server    |
  |        |--(E)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(F)- Invalid Token Error -|          |   |               |
  |        |                            +----------+   |               |
  |        |                                           |               |
  |        |--(G)----------- Refresh Token ----------->|               |
  |        |                                           |               |
  |        |<-(H)----------- Access Token -------------|               |
  +--------+           & Optional Refresh Token        +---------------+

![](https://common.cnblogs.com/images/copycode.gif)

具体流程不再分析，我们已知，当我们申请token后，Authorization Server不仅给了我们Access Token，还有Refresh Token。当Access Token过期后，我们用Refresh Token访问/refresh端点就可以拿到新的Access Token了。

本节只讲概念，在接下来的几节中会搭建一组含有Client Application, Authorization Server, Resource Server的微服务群，并使用Eureka和Config组件管理。