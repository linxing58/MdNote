# GeoMesa-kafka（CDH6.2.1集群中安装geomesa-kafka 3.1.0）_tiger-hcx的博客-CSDN博客_geomesa kafka
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

* * *

前提说明：cdh6.2.1 集群已经安装 zookeeper 和 kafka。  
本文目标：安装 geomesa-kafka 3.1.0，使用测试数据进行入库。先在一台上安装然后 scp 到其它台。

下载 geomesa-kafka 3.1.0。  
地址：[https://github.com/locationtech/geomesa/releases/download/geomesa\\\_2.11-3.1.0/geomesa-kafka\\\_2.11-3.1.0-bin.tar.gz，速度有点慢就是了，可以使用 github 代下网站下载或其它方式。](https://github.com/locationtech/geomesa/releases/download/geomesa\_2.11-3.1.0/geomesa-kafka\_2.11-3.1.0-bin.tar.gz，速度有点慢就是了，可以使用github代下网站下载或其它方式。)

上传到服务器，并解压，解压到 / usr/local 目录，解压后的目录：  
![](https://img-blog.csdnimg.cn/20201222183611716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

进入 geomesa-kafka_2.11-3.1.0/conf 目录进行配置

```bash
[root@node3 local]

```

配置文件  
![](https://img-blog.csdnimg.cn/20201222183808495.png)

dependencies.sh 依赖的配置，主要是修改 kafka 的版本和 zookeeper 的版本，我这里不改了。因为 cdh6.2.1 安装的 zookeeper 是 3.4.5 和 kafka 是 2.1.0。所以差别不大就不改了，可以用。  
![](https://img-blog.csdnimg.cn/20201222183910736.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

修改 kafka-env.sh 配置文件  
配置 kafka 和 zookeeper 的 home 目录。  
/opt/cloudera/parcels/CDH/lib/kafka、/opt/cloudera/parcels/CDH/lib/zookeeper  
![](https://img-blog.csdnimg.cn/20201222184116680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

进入 bin 目录

```bash
[root@node3 bin]

```

![](https://img-blog.csdnimg.cn/20201222184511485.png)

执行 install-dependencies.sh 和 install-shapefile-support.sh 脚本  
![](https://img-blog.csdnimg.cn/20201222184631341.png)

直接将 geomesa-kafka_2.11-3.1.0 整个文件夹复制到其它台，即可。

使用 bin/geomesa-kafka 命令将 example.csv 数据入库。example.csv 是安装包中自带的

```bash
[root@node3 bin]

```

\-b 是设置 kafka 的 brokers，-z 是设置 zookeeper  
![](https://img-blog.csdnimg.cn/20201222185753727.png)

成功插入三个要素

到 kafka 中查看，进入 / var/local/kafka/data 目录

![](https://img-blog.csdnimg.cn/20201222185915366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MTg4MTE5,size_16,color_FFFFFF,t_70)

成功

1\. 注意版本的配置  
2. 注意插入语句的参数，不清楚的可以使用 help 查看。  
3. 欢迎互相学习，交流讨论，本人的微信：huangchuanxiaa。 
 [https://blog.csdn.net/qq_18188119/article/details/111562769](https://blog.csdn.net/qq_18188119/article/details/111562769)
