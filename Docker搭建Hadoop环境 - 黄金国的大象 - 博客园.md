# Docker搭建Hadoop环境 - 黄金国的大象 - 博客园
> Hadoop 集群环境配置起来相当繁琐，并且在学习 Hadoop 过程中没有一般不会去使用多台设备进行分布式集群配置。因此在一台机器上配置 Hadoop 分布式集群时通常采用虚拟机来模拟多台设备，但虚拟机较为占用系统资源，开多个虚拟机 (模拟 Hadoop 集群通常使用 3 个，一个 master，两个 slave) 对内存要求比较高，因此笔者就想是否能通过 Docker 来配置 Hadoop，并且通过 Jetbrains IDEA 来连接 Docker 容器调试 MapReduce 程序。经过一番折腾，成功地搭建了 Docker+IDEA 的 Hadoop 环境。在此将结合网上其他一些教程和自己的经验讲配置过程记录下来。  
> 注意：文中多次提到容器终端是 hadoop-master 这个容器的终端

## Docker 的安装与使用

> 建议在 Ubuntu 上配置以下内容，因为我自己在 Windows 上多次尝试配置，都出现 Datanode 启动不了的情况，同样的步骤在 Ubuntu 上就没有问题。

看了一些 Docker 教程，笔者认为菜鸟教程里的 Docker 教程不错，包含安装过程和一些基础命令，在这里就不赘述了，读者自行查看 [菜鸟教程 - Docker](http://www.runoob.com/docker/ubuntu-docker-install.html)，另外建议将 Docker 的镜像源[修改为国内镜像源](https://blog.csdn.net/Michel4Liu/article/details/80747676)，下载速度会更快。

## 拉取镜像

镜像，是 Docker 的核心，可以通过从远程拉取镜像即可配置好我们所需要的环境，我们这次需要的是 Hadoop 集群的镜像。我们使用 kiwenlau/hadoop:1.0 这个镜像。

sudo docker pull kiwenlau/hadoop:1.0

显示以下信息说明拉去镜像成功  
![](https://img-blog.csdnimg.cn/20190420171127146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## 克隆配置脚本

这一步是从 github 上克隆配置脚本，脚本的内容是使用 kiwenlau/hadoop:1.0 配置 mater、slave1、slave2 三个容器，其中 slave 数量可以修改：

git clone [https://github.com/kiwenlau/hadoop-cluster-docker](https://github.com/kiwenlau/hadoop-cluster-docker)

切换到合适的路径在进行克隆，显示以下信息说明克隆完成  
![](https://img-blog.csdnimg.cn/20190420172143748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## 创建网桥

由于 Hadoop 的 master 节点需要与 slave 节点通信，需要在各个主机节点配置节点 IP，为了不用每次启动都因为 IP 改变了而重新配置，在此配置一个 Hadoop 专用的网桥，配置之后各个容器的 IP 地址就能固定下来。

sudo docker network create --driver=bridge hadoop

出现以下信息说明网桥创建完成  
![](https://img-blog.csdnimg.cn/20190420172833300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## 执行脚本

此时可以通过前面步骤克隆下来的脚本进行容器创建了，首先看一下脚本内容  
![](https://img-blog.csdnimg.cn/20190420173248771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

为了后续通过 IDEA 连接 IDEA，需要修改脚本，添加一个端口映射，将容器的 9000 端口映射到本地的 9000 端口，在 - p 8088:8088 \\下添加一行如下图所示  
![](https://img-blog.csdnimg.cn/20190421115758940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

执行脚本，脚本在创建完容器之后进入了容器的终端  
![](https://img-blog.csdnimg.cn/20190420173349204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## Docker 命令补充

1、切换到本地终端而不关闭容器：快捷键 Ctrl+P+Q，前一个步骤进入了容器的终端，如果我们想在这个窗口执行**本地命令**，而不想关闭当前容器，可以使用这个快捷前切换回本地终端，如下图所示：  
![](https://img-blog.csdnimg.cn/20190421091020989.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

也可以通过新建一个终端窗口通过以下 attach 命令来进入容器终端，这种方式更方便一点，不需要使用命令来回切换终端  
2、进入容器终端：在一个容器启动的情况下，我们想从**本地终端**切换到**容器终端**，可以使用 docker 的 attach 命令来切换：

\# sudo docker attach \[container-id]
sudo docker attach hadoop-master

![](https://img-blog.csdnimg.cn/20190421091605808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

3、以上两条是我们在后续步骤要多次用到的，在此提出，其他 Docker 命令请查看 [菜鸟教程 - Docker](http://www.runoob.com/docker/ubuntu-docker-install.html)。

## 更换镜像源

-   在下一步中需要安装 vim，在安装 vim 前需要执行 sudo apt-get update 更新软件源，这一步如果使用默认镜像源 (国外源)，更新速度会很慢，因此最好在次之前配置国内镜像源。

-   由于更换镜像源需要复制镜像源到文件中，而容器中只有 vi(非 vim) 编辑器，vi 的粘贴很难用 (我不会用)，所有采用在本机创建一个镜像源文件，然后通过 Docker 命令移动到 Docker 容器中的方法来修改镜像源文件。  
    首先新开一个**本地终端**，创建一个 sources.list 文件

-   将以下内容复制进去

![](https://common.cnblogs.com/images/copycode.gif)

deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic main restricted universe multiverse
deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-security main restricted universe multiverse
deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-updates main restricted universe multiverse
deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-proposed main restricted universe multiverse
deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-backports main restricted universe multiverse
deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic main restricted universe multiverse
deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-security main restricted universe multiverse
deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-updates main restricted universe multiverse
deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-proposed main restricted universe multiverse
deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) bionic-backports main restricted universe multiverse

![](https://common.cnblogs.com/images/copycode.gif)

-   在**容器终端**下备份一下 sources.list 文件

sudo cp /etc/apt/sources.list /etc/apt/sources.list.copy

-   在**本地终端**使用 Docker 命令将本地文件复制到 Docker 容器里

sudo docker cp sources.list  hadoop-master:/etc/apt

-   最后在**容器终端**更新一下软件源

## 安装 vim

-   由于 kiwenlau/hadoop:1.0 这个镜像是没有安装 vim 编辑器，因此我们需要先把 vim 装上，方便后续查看文件，执行以下安装命令即可

\# 安装 vim
sudo apt-get install vim

## 启动 Hadoop

在前面一个步骤的最后进入了 Hadoop 容器的终端 (master 节点)，因为是已经配置好 Hadoop 的容器，所以可以直接使用 Hadoop，在容器根目录下有一个启动 Hadoop 的脚本，脚本代码如下，启动了 dfs 和 yarn：  
![](https://img-blog.csdnimg.cn/20190420225935381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

-   通过以下命令执行：

出现以下信息说明 Hadoop 启动成功  
![](https://img-blog.csdnimg.cn/20190420212325636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## 测试 Word Count

根目录下还有一个测试 WordCount 程序的脚本，WordCount 是 Hadoop 里 “Hello World” 程序，是一个用于文本字符统计的 MapReduce 程序，先来看一下 run-wordcount.sh 这个脚本的内容，脚本往 hdfs 里添加了数据文件，然后执行了 Hadoop 里的例子 hadoop-mapreduce-examples-2.7.2-sources.jar：  
![](https://img-blog.csdnimg.cn/20190420230101993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

-   执行脚本

输出结果没有报异常，且结果如下则 WordCount 程序执行成功  
![](https://img-blog.csdnimg.cn/20190420230613568.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

## 查看 Web 管理页面

-   Name Node：\[Your IP Address]:50070/
    -   进入页面可以看到各节点的情况，注意如果 Summary 下的节点信息异常 (容量为 0、Live Nodes 为 0 等) 可能是配置过程出现了问题。
    -   在导航栏的 Utilities 中有两个选项**Browse the file system**和**Logs**，前者是查看 hdfs 文件系统的，后者是查看 Hadoop 运行日志的，出现任何异常时可以在日志中查看，看能否找出异常原因。

![](https://img-blog.csdnimg.cn/20190421093531891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

-   Resource Manager: \[Your IP Address]:8088/

![](https://img-blog.csdnimg.cn/2019042109335495.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MzQyNzM5,size_16,color_FFFFFF,t_70)

-   容器的 IP 可以在启动容器时的输出信息里找到，或者在开启的本地端口映射的情况下使用本地浏览器，IP 使用 localhost

Docker 配置 Hadoop 环境的过程到此就结束了，下一篇[IDEA 调试 Docker 上的 Hadoop](https://blog.csdn.net/qq_24342739/article/details/89429786)会介绍如何在 IDEA 连接 Docker 配置的 Hadoop 容器，并且调试的文章，以便于 Hadoop 的深入学习。

> 参考” 从 0 开始使用 Docker 快速搭建 Hadoop 集群环境 “：[https://www.jianshu.com/p/b75f8bc9346d](https://www.jianshu.com/p/b75f8bc9346d)
>
> 转载[https://blog.csdn.net/qq_24342739/article/details/89420496](https://blog.csdn.net/qq_24342739/article/details/89420496) 
>  [https://www.cnblogs.com/gaodi2345/p/11829312.html#DockerHadoop_1](https://www.cnblogs.com/gaodi2345/p/11829312.html#DockerHadoop_1)
