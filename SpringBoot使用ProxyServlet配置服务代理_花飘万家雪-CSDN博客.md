# SpringBoot使用ProxyServlet配置服务代理_花飘万家雪-CSDN博客
![](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[花飘万家雪](https://blog.csdn.net/zdx1515888659) 2020-10-22 10:54:05 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 2333 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 收藏  2 

## 引入相关依赖

````null
<groupId>org.mitre.dsmiley.httpproxy</groupId><artifactId>smiley-http-proxy-servlet</artifactId><groupId>com.google.guava</groupId><artifactId>guava</artifactId>```

相关代理配置
------

```null
proxy.servlet_url: /api/*proxy.target_url: http://www.baidu.com```

代理配置类
-----

```null
package com.example.wellnessproxy.config;import com.google.common.collect.ImmutableMap;import org.mitre.dsmiley.httpproxy.ProxyServlet;import org.springframework.beans.factory.annotation.Value;import org.springframework.boot.web.servlet.ServletRegistrationBean;import org.springframework.context.annotation.Bean;import org.springframework.context.annotation.Configuration;import javax.servlet.Servlet;public class ProxyServletConfiguration {@Value("${proxy.servlet_url}")private String servletUrl;@Value("${proxy.target_url}")private String targetUrl;public Servlet createProxyServlet() {return new ProxyServlet();public ServletRegistrationBean proxyServletRegistration() {ServletRegistrationBean registrationBean = new ServletRegistrationBean(createProxyServlet(), servletUrl);Map<String, String> params = ImmutableMap.of(        registrationBean.setInitParameters(params);```

测试效果
----

*   访问本地请求，就可以访问到代理地址了 
 [https://blog.csdn.net/zdx1515888659/article/details/109217480](https://blog.csdn.net/zdx1515888659/article/details/109217480)
````
