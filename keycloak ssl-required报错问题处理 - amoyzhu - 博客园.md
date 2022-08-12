# keycloak ssl-required报错问题处理 - amoyzhu - 博客园
 两台主机，网段不同，第一台129.30.108.179/24    第二台172.16.160.92/24 

 都安装keycloak :    docker run -d --name keycloak -e KEYCLOAK\_USER=admin -e KEYCLOAK\_PASSWORD=admin -p 8080:8080 jboss/keycloak    都可以正常启动，第二台可以正常登录admin后台，第一台登录admin后台会报HTTPS required错识，这可能是哪一方面的原因？  
第一台docker日志是： 

![](https://images2018.cnblogs.com/blog/862626/201804/862626-20180421175601201-379529681.png)

![](https://images2018.cnblogs.com/blog/862626/201804/862626-20180421175701253-185589131.png)

![](https://images2018.cnblogs.com/blog/862626/201804/862626-20180421175713124-1184754963.png)

原因是：一个公网IP地址，一个私有IP地址

keycloak官网：

https://www.keycloak.org/docs/latest/server\_installation/index.html#setting-up-https-ssl

![](https://images2018.cnblogs.com/blog/862626/201804/862626-20180421175906740-2897235.png)

Keycloak can run out of the box without SSL so long as you stick to private IP addresses like `localhost`, `127.0.0.1`, `10.0.x.x`, `192.168.x.x`, and `172.16.x.x`. If you don???t have SSL/HTTPS configured on the server or you try to access Keycloak over HTTP from a non-private IP adress you will get an error.

keycloak如果用私有地址才能不使用ssl登录方式，如果用公网就需要用ssl登录方式

解决办法是：

https://stackoverflow.com/questions/30622599/https-required-while-logging-in-to-keycloak-as-admin

登录mysql数据库修改一个字段的内容：

```

```

这个问题困扰了我两天，标记一下

```

```