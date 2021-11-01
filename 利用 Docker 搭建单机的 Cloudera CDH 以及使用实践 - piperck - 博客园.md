# 利用 Docker 搭建单机的 Cloudera CDH 以及使用实践 - piperck - 博客园
想用 CDH 大礼包，于是先在 Mac 上和 Centos7.4 上分别搞个了单机的测试用。其实操作的流和使用到的命令差不多就一并说了:

首先前往官方下载包：

[https://www.cloudera.com/downloads/quickstart\\\_vms/5-13.html](https://www.cloudera.com/downloads/quickstart\_vms/5-13.html)

如果使用 mac 并且安装 docker。 我们可以很轻松的使用 kitematic 来获取最新版本的 cloudera docker 镜像。只需要搜 cloudera/quickstar 即可这是地址：

[https://hub.docker.com/r/cloudera/quickstart/](https://hub.docker.com/r/cloudera/quickstart/)

当我们下载好镜像之后就可以愉快的将进行加载起来。macos 基本是全程无脑，linux 稍微麻烦一点使用

docker import cloudera-quickstart-vm-5.13.0-0-beta-docker.tar

将镜像 import 进来。

然后使用命令启动就可以了。

Cloudera 的 docker 版本分成两部分启动。一方面是大礼包的启动 /usr/bin/docker-quickstart，一方面是 Cloudera manager 本身的启动 /home/cloudera/cloudera-manager

这里我们使用命令

docker run --name cdh --hostname=quickstart.cloudera --privileged=true -t -i -p 8020:8020 -p 8022:8022 -p 7180:7180 -p 21050:21050 -p 50070:50070 -p 50075:50075 -p 50010:50010 -p 50020:50020 -p 8890:8890 -p 60010:60010 -p 10002:10002 -p 25010:25010 -p 25020:25020 -p 18088:18088 -p 8088:8088 -p 19888:19888 -p 7187:7187 -p 11000:11000 cloudera/quickstart /bin/bash -c '/usr/bin/docker-quickstart && /home/cloudera/cloudera-manager --express && service ntpd start'

直接启动两个程序。这里注意参数都可以从下面 refrence 查询到大概是什么意思，合理之所以要写这么多端口映射也是为了方便我们外面的机器可以方面的访问 docker 内部的这些端口，访问这些服务。 Cloudera 本身的 manager 是 7180 端口。当这些启动起来之后就可以访问目标机器 ip 的 7180 端口访问 Cloudera manager 了。

![](https://img2018.cnblogs.com/blog/742678/201811/742678-20181106183319712-459447245.png)

上图就是一个 dashbord 的样子。另外在 linux 机器有一个地方需要注意的是，可能你的 docker 用上面命令起起来之后，docker 内的实例没有办法访问外网，这里我们配置一下 docker 创建容器时候的参数增加 -net host 即可。也可以在宿主机器上在 /etc/default/docker 文件。并且配置上 DOCKER_OPTS="--dns host_ip" 即可。

从上图我们还可以注意到另外一个问题，除了主机和 manager 都没有启动。在 Cloudera 大礼包中，只有 hue 和 manager 本身是什么服务都不依赖的可以在任何时候选择启动和关闭。其他的应用多多少少存在着一些启动顺序上的依赖这个要注意。 

现在我们来启动几个我们关心的服务，我们先来启动 HDFS。

![](https://img2018.cnblogs.com/blog/742678/201811/742678-20181106204224839-707988631.png)

这里我已经把它启动起来了当没有启动的时候点击 start cm(Cloudera manager) 就会把这个给启动起来。

点一下已经启动起来的 HDFS 就会到这个应用的 dashborad cm 给我们提供了非常多的图表以及面板可以关注目前机器和集群的情况如下图：

![](https://img2018.cnblogs.com/blog/742678/201811/742678-20181106204700838-304212308.png)

目前看到的都是单节点的情况。让我意外的是启动的时候竟然还会有 Canary 模式。在这个界面点击右上角的 NameNode Web UI 就可以看到老板我们熟悉的

社区版的 HDFS 界面了。比较方便的是当我们点击 Configuration 就可以进到 HDFS 的一些配置包括块大小之类的配置这里都可以方便设置。

可以看到这一套东西真的是把能包好的东西都已经给我们列出来了。

我暂时在单机上面启了两个 app 一个 HDFS 一个 Spark ，内存基本被打到了 5 个 G. 可以看出来其实 CDH 大礼包其实还是非常吃内存的。当我们在进行线上环境配置的

时候占用的资源肯定是只增不减。这里抛砖引玉了一个 app 接下来大家可以按照这个方法继续探索。

既然 HDFS 已经启动让我们来尝试使用 python 来操作一下 HDFS

pip install hdfs 安装 hdfscli 包

from hdfs.client import Client
client \\= Client("[http://127.0.0.1:50070"](http://127.0.0.1:50070"), root="/", timeout=100)

print(client.list("/"))
client.upload("/", "/Users/piperck/Desktop/About_me/dragen.wma")

可以看到我们可以直接创建连接，client.list 是列出 HDFS 目前根目录的情况。 下面我们调用 client.upload 上传文件。

上传文件的时候可能遇到很多问题，因为我们这里使用的是 docker 搭建的 CDH , 所以一般会报这个错误：

('&lt;requests.packages.urllib3.connection.HTTPConnection object at 0x00000000043A3FD0>: Failed to establish a new connection:
 \[Errno 11004] getaddrinfo failed',))

这个时候我们需要去 docker 里面 hostname 一下会得到 quickstart.cloudera。我用的 macos 所以把这个直接配置进我电脑的 /etc/hosts 里。

127.0.0.1       quickstart.cloudera

否则，永远报错。。这里搞了非常久需要注意一下。

之后继续尝试连接，应该还会报另外一个错误：

Permission denied: user=root, access=WRITE, inode="/":hdfs:supergroup:drwxr-xr-x 

很明显权限问题，因为我们并没有登陆而且在本地使用的权限也不明。755 的权限导致我们无法上传文件，这里的 root 权限是 hdfs 用户，所以会失败

这里有两个办法可以解决这个问题：

1. 调整 hdfs 的权限检查将

<property>
  <name>dfs.permissions</name>
  <value>false</value>
</property>

设置为 False 关闭权限检查。

2. 增加一个由这个用户创建的文件夹在根目录，然后将文件往那里面传就可以了。 

现在我们再将传上去的文件下载回来：

from hdfs.client import Client
client \\= Client("[http://127.0.0.1:50070"](http://127.0.0.1:50070"), root="/", timeout=100)

print(client.list("/"))
client.download("/dragen.wma", "/Users/piperck/Desktop")

很轻松成功了，没有再出什么幺蛾子。

Reference:

[https://www.cloudera.com/documentation/enterprise/5-15-x/topics/quickstart\\\_docker\\\_container.html](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/quickstart\_docker\_container.html)  ---docker 安装启动文档

[https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm\\\_mc\\\_start\\\_stop\\\_service.html#cmug\\\_topic\\\_5\\\_6](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm\_mc\_start\_stop\_service.html#cmug\_topic\_5\_6)  --- 启动 hdfs 服务教程

[https://blog.csdn.net/g11d111/article/details/72902112](https://blog.csdn.net/g11d111/article/details/72902112)

[https://dxysun.com/2018/07/19/hadoopForPythonHdfs/](https://dxysun.com/2018/07/19/hadoopForPythonHdfs/)  PYTHONHDFS 使用教程

[https://blog.csdn.net/Gamer\\\_gyt/article/details/52446757](https://blog.csdn.net/Gamer\_gyt/article/details/52446757)  使用 python 的 hdfs 包操作分布式文件系统（HDFS）

[https://segmentfault.com/a/1190000002672666](https://segmentfault.com/a/1190000002672666)  hadoop 常用文件的操作命令

 
 [https://www.cnblogs.com/piperck/p/9917118.html](https://www.cnblogs.com/piperck/p/9917118.html)
