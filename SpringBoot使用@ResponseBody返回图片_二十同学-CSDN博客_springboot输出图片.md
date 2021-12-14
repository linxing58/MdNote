# SpringBoot使用@ResponseBody返回图片_二十同学-CSDN博客_springboot输出图片
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[二十同学](https://ershi.blog.csdn.net/) 2019-04-15 17:18:45 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 80933 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 收藏  24 

## **微信搜索：“二十同学” 公众号，欢迎关注一条不一样的成长之路**       

以前使用 HttpServletResponse 可以通过输出流的方式来向前台输出图片。现在大部分都是使用 springboot，在使用 springboot 之后，我们应该如何来修改代码呢？

Spring Boot 项目搭建配置略过，可直接从官网简历一个 demo

首先写一个 Controller 类，包括一个方法，如下：

````null
package com.example.demo.common;import org.springframework.http.MediaType;import org.springframework.web.bind.annotation.GetMapping;import org.springframework.web.bind.annotation.RequestMapping;import org.springframework.web.bind.annotation.ResponseBody;import org.springframework.web.bind.annotation.RestController;import java.io.FileInputStream;@RequestMapping(value="/api/v1")@GetMapping(value = "/image",produces = MediaType.IMAGE_JPEG_VALUE)public byte[] test() throws Exception {File file = new File("E:\\ce\\1.jpg");FileInputStream inputStream = new FileInputStream(file);byte[] bytes = new byte[inputStream.available()];        inputStream.read(bytes, 0, inputStream.available());```

  
我们首先在@GetMapping上加入**produces**告诉Spring，我们要返回的MediaType是一个图片（image/jpeg），然后加上@ResponseBody注解，方法返回byte\[\]，然后将图片读进byte\[\]，**不加produces会报错**。

浏览器访问接口测试一下，返回如下：

![](https://img-blog.csdnimg.cn/20190415170501182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4Mjk4NDM5,size_16,color_FFFFFF,t_70) 
 [https://blog.csdn.net/qq_18298439/article/details/89315478?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.no_search_link&spm=1001.2101.3001.4242.1](https://blog.csdn.net/qq_18298439/article/details/89315478?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.no_search_link&spm=1001.2101.3001.4242.1)
````
