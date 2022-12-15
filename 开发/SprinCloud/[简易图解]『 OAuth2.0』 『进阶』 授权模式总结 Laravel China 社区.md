# [简易图解]『 OAuth2.0』 『进阶』 授权模式总结 | Laravel China 社区
 [![](https://cdn.learnku.com/uploads/avatars/29862_1634310650.png!/both/100x100)
 chihokyo 的个人博客](https://learnku.com/blog/chihokyo) /  7403 /  26 /  创建于 3年前  / 更新于 3年前 

一，到底 OAuth 是什么？
---------------

### 写在前面的[#](#a76cfc)

上一次简单傻瓜式的用图来说明了到底什么是 OAuth，没看过的童靴可以点击下面这条链接看一下。  
看完这个在来看这篇可能理解起来效果拔群。  
[【简易图解】『 OAuth2.0』 猴子都能懂的图解](https://learnku.com/articles/20031)  
简而言之就是一个程序想借用下_你的数据_ ( **想借你微信朋友圈内容看一下啦，想借你微博图片下载一下啦**) 所产生的一系列问题 (**比如安全，如何授权等等**)，OAuth 就是关于这些的规范。

> *   接下来我想就授权模式做一下总结，还是用图。
> *   这次的图可能没有上次这么傻瓜，只要仔细阅读一下，就连我非计算机专业出身的人都能懂的话，一般的猴子们估计也没问题。觉得太长的可以只从图解看起，或者只看授权码模式就好。
> *   授权模式主要参考来自 [RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.1)

### 认证 Authentication VS 授权 Authorization

看授权模式图之前，先区别下这两个概念。

*   **认证**就是我要输入帐号和密码来证明我是我
*   **授权**就是并非通过帐号和密码来把我的东西借给其他人
*   这其中的关键就是，**_是否需要输入帐号密码_**。记住，OAuth 不需要输入帐号和密码，你要做的只是授权。  
    下面这张图清晰的说明了，认证和授权。  
    ![](https://cdn.learnku.com/uploads/images/201811/22/29862/mZpBTyIkU1.png!large)
    
    二，授权模式[#](#c4f7b7)
    ------------------
    
    如果有觉得画质太差的，没办法，上传之后就这样了。。  
    这里是 PPT 链接。
    
*   [授权码模式](https://docs.google.com/presentation/d/1UV0GTPa6oPefhz4w6SjSZ6FDFfoqs-aOJvDluNTM13Y/edit?usp=sharing)
*   [简化模式](https://docs.google.com/presentation/d/1mv7U3J35ZLyeHZdq4t1xf_kTJs0Mq3Oa-VYrwdFofgM/edit?usp=sharing)
*   [密码模式](https://docs.google.com/presentation/d/1ucSpJrXFglPx9CzTsOxa_XHFjz2fAkPWBjb1Xy1bmIQ/edit?usp=sharing)
*   [客户端模式](https://docs.google.com/presentation/d/1-YXe8BCMUePeQm_KC2CO0rKVJi6FNqgFnEQG6zjTzjI/edit?usp=sharing)
*   [刷新令牌](https://docs.google.com/presentation/d/1yL9Czy2IzVp4oXV5Wc_sD0DBJQ1dVyo8LtBG0gX_6PI/edit?usp=sharing)

### 1\. 授权码模式 Authorization Code

#### 1.3 授权响应 Authorization Response[#](#0a7786)

```
HTTP/1.1 302 Found
Location: {重定向URI}
?code={授权码}          // 必填
&state={任意文字}       // 如果授权请求中包含 state的话那就是必填
```

#### 1.4 令牌请求 Access Token Request[#](#b98ca6)

```
POST {令牌终点} HTTP/1.1
Host: {认证服务器}
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code      // 必填
&code={授权码}                     // 必填　必须是认证服务器响应给的授权码
&redirect_uri={重定向URI}          // 如果授权请求中包含 redirect_uri 那就是必填
&code_verifier={验证码}            // 如果授权请求中包含 code_challenge 那就是必填
```

> 根据具体情况有可能是向客户端服务器进行请求，这时候请加上 Basic 认证（Authorization 头部）或者是 参数 client\_id & client\_secret

#### 1.5 令牌响应 Access Token Response[#](#78afce)

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"{访问令牌}",      // 必填
  "token_type":"{令牌类型}",      // 必填
  "expires_in":{过期时间},        // 任意
  "refresh_token":"{刷新令牌}",   // 任意
  "scope":"{授权范围}"            // 如果请求和响应的授权范围不一致就必填
}
```

### 2\. 简化模式 Implicit[#](#b2d7f4)

#### 2.2 授权请求 Authorization Request[#](#db5cd4)

```
GET {授权终点}
  ?response_type=token             // 必填
  &client_id={客户端ID}      // 必填
  &redirect_uri={重定向URI}  // 可选。授权成功后的重定向地址
  &scope={授权范围}              // 任意
  &state={任意文字}              // 推荐
  HTTP/1.1
HOST: {认证服务器}
```

#### 2.3 授权响应 Authorization Response[#](#4fa144)

```
HTTP/1.1 302 Found
Location: {重定向URI}
  #access_token={令牌码} // 必填
  &token_type={令牌类型}     // 必填
  &expires_in={过期时间}       // 任意
  &state={任意文字}            // 如果授权请求中包含 state 那就是必填
  &scope={授权范围}            // 如果请求和响应的授权范围不一致就必填
```

### 3\. 密码模式 Resource Owner Password Credentials[#](#34b861)

#### 3.2 令牌请求 Access Token Request[#](#af1086)

```
POST {令牌终点} HTTP/1.1
Host: {认证服务器}
Content-Type: application/x-www-form-urlencoded

grant_type=password       // 必填
&username={用户ID}    // 必填
&password={密码}    // 必填
&scope={授权范围}       // 任意
```

> 根据具体情况有可能是向客户端服务器进行请求，这时候请加上 Basic 认证（Authorization 头部）或者是 参数 client\_id & client\_secret

#### 3.3 令牌响应 Access Token Response[#](#56c0cc)

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"{访问令牌}",   // 必填
  "token_type":"{令牌类型}",      // 必填
  "expires_in":{过期时间},        // 任意
  "refresh_token":"{刷新令牌}", // 任意
  "scope":"{授权范围}"              // 如果请求和响应的授权范围不一致就必填
}
```

### 4\. 客户端模式 Client Credentials[#](#376265)

#### 4.2 令牌请求 Access Token Request[#](#74dd28)

```
POST {令牌终点} HTTP/1.1
Host: {认证服务器}
Authorization: Basic {客户端模式}
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials     // 必填
&scope={授权范围}               // 任意
```

#### 4.3 令牌响应 Access Token Response[#](#cb9a6d)

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"{访问令牌}",  // 必填
  "token_type":"{令牌类型}",      // 必填
  "expires_in":{过期时间},              // 任意
  "scope":"{授权范围}"                // 如果请求和响应的授权范围不一致就必填
}
```

### 5\. 刷新令牌 Refresh Token[#](#887aff)

#### 5.1 图解[#](#53bccf)

> 第三方已存在令牌码为前提进行更新令牌  
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/6gA3JVRIoU.png!large)
>   
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/tW5dr3TqM1.png!large)
>   
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/ytqK11i2I4.png!large)
>   
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/h8oMy0ivAH.png!large)
>   
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/1jjM4hEEQr.png!large)
>   
> ![](https://cdn.learnku.com/uploads/images/201812/20/29862/FWXv0Lp2yO.png!large)

#### 5.2 令牌请求 Access Token Request[#](#98391a)

```
POST {令牌终点} HTTP/1.1
Host: {认证服务器}
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token           // 必填
&refresh_token={刷新令牌}        // 必填
&scope={授权范围}                    // 任意
```

> 根据具体情况有可能是向客户端服务器进行请求，这时候请加上 Basic 认证（Authorization 头）或者是 参数 client\_id & client\_secret

#### 5.3 令牌响应 Access Token Response[#](#56cc23)

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"{访问令牌}",      // 必填
  "token_type":"{令牌类型}",          // 必填
  "expires_in":{过期时间},                  // 任意
  "refresh_token":"{刷新令牌}", // 任意
  "scope":"{授权范围}"                    // 如果请求和响应的授权范围不一致就必填
}
```

三，总结[#](#06a850)
----------------

| 授权模式 | 授权终点 | 令牌终点 |
| --- | --- | --- |
| 授权码模式 | 使用 | 使用 |
| 简化模式 | 使用 | 不使用 |
| 密码模式 | 不使用 | 使用 |
| 客户端模式 | 不使用 | 使用 |
| 刷新令牌 | 不使用 | 使用 |

> 其实授权终点就是授权请求和响应  
> 令牌终点就是令牌的请求和响应

感谢您的阅读！

> 本作品采用[《CC 协议》](https://learnku.com/docs/guide/cc4.0/6589)，转载必须注明作者和本文链接

本帖由系统于 3年前 自动加精