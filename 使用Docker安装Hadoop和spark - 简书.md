# 使用Docker安装Hadoop和spark - 简书
[![](https://cdn2.jianshu.io/assets/default_avatar/1-04bbeead395d74921af6a4e8214b4f61.jpg)
](https://www.jianshu.com/u/32f6cd6e553b)

0.5682018.11.03 16:30:39 字数 662 阅读 4,295

使用 docker 配置安装 hadoop 和 spark

## 安装 hadoop 镜像

选择的 docker[镜像地址](https://hub.docker.com/r/uhopper/hadoop/)，这个镜像提供的 hadoop 版本比较新，且安装的是 jdk8，可以支持安装最新版本的 spark。

    docker pull uhopper/hadoop:2.8.1 

## 安装 spark 镜像

如果对 spark 版本要求不是很高，可以直接拉取别人的镜像，若要求新版本，则需要对 dockerfile 进行配置。

### 环境准备

1.  下载 sequenceiq/spark 镜像构建源码

        git clone https://github.com/sequenceiq/docker-spark 
2.  从 Spark 官网下载 Spark 2.3.2 安装包

    -   下载地址：[http://spark.apache.org/downloads.html](http://spark.apache.org/downloads.html)
3.  将下载的文件需要放到 docker-spark 目录下
4.  查看本地 image，确保已经安装了 hadoop
5.  进入 docker-spark 目录，确认所有用于镜像构建的文件已经准备好

    -   ![](https://upload-images.jianshu.io/upload_images/10582164-89fc5e3ec50133de.jpg)

        image-20181030125353071

### 修改配置文件

-   修改 Dockerfile 为以下内容

    -       FROM sequenceiq/hadoop-docker:2.7.0
            MAINTAINER scottdyt

            #support for Hadoop 2.7.0
            #RUN curl -s http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz | tar -xz -C /usr/local/
            ADD spark-2.3.2-bin-hadoop2.7.tgz /usr/local/
            RUN cd /usr/local && ln -s spark-2.3.2-bin-hadoop2.7 spark
            ENV SPARK_HOME /usr/local/spark
            RUN mkdir $SPARK_HOME/yarn-remote-client
            ADD yarn-remote-client $SPARK_HOME/yarn-remote-client

            RUN $BOOTSTRAP && $HADOOP_PREFIX/bin/hadoop dfsadmin -safemode leave && $HADOOP_PREFIX/bin/hdfs dfs -put $SPARK_HOME-2.3.2-bin-hadoop2.7/jars /spark && $HADOOP_PREFIX/bin/hdfs dfs -put $SPARK_HOME-2.3.2-bin-hadoop2.7/examples/jars /spark 


            ENV YARN_CONF_DIR $HADOOP_PREFIX/etc/hadoop
            ENV PATH $PATH:$SPARK_HOME/bin:$HADOOP_PREFIX/bin
            # update boot script
            COPY bootstrap.sh /etc/bootstrap.sh
            RUN chown root.root /etc/bootstrap.sh
            RUN chmod 700 /etc/bootstrap.sh

            #install R 
            RUN rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
            RUN yum -y install R

            ENTRYPOINT ["/etc/bootstrap.sh"] 
-   修改 bootstrap.sh 为以下内容

        #!/bin/bash

        : ${HADOOP_PREFIX:=/usr/local/hadoop}

        $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh

        rm /tmp/*.pid

        # installing libraries if any - (resource urls added comma separated to the ACP system variable)
        cd $HADOOP_PREFIX/share/hadoop/common ; for cp in ${ACP//,/ }; do  echo == $cp; curl -LO $cp ; done; cd -

        # altering the core-site configuration
        sed s/HOSTNAME/$HOSTNAME/ /usr/local/hadoop/etc/hadoop/core-site.xml.template > /usr/local/hadoop/etc/hadoop/core-site.xml

        # setting spark defaults
        echo spark.yarn.jar hdfs:///spark/* > $SPARK_HOME/conf/spark-defaults.conf
        cp $SPARK_HOME/conf/metrics.properties.template $SPARK_HOME/conf/metrics.properties

        service sshd start
        $HADOOP_PREFIX/sbin/start-dfs.sh
        $HADOOP_PREFIX/sbin/start-yarn.sh



        CMD=${1:-"exit 0"}
        if [[ "$CMD" == "-d" ]];
        then
          service sshd stop
          /usr/sbin/sshd -D -d
        else
          /bin/bash -c "$*"
        fi 

    ### 构建镜像

        docker build --rm -t scottdyt/spark:2.3.2 . 

    ![](https://upload-images.jianshu.io/upload_images/10582164-2ccb5651fadc1105.jpg)

    Screen Shot 2018-10-30 at 10.58.21 AM

    ### 查看镜像

    ![](https://upload-images.jianshu.io/upload_images/10582164-228007be3a863a35.jpg)

    Screen Shot 2018-10-30 at 12.06.19 PM

启动一个 spark2.3.1 容器

    docker run -it -p 8088:8088 -p 8042:8042 -p 4040:4040 -h sandbox scottdyt/spark:2.3.2 bash 

启动成功：

![](https://upload-images.jianshu.io/upload_images/10582164-cb3466be40a226e3.jpg)

Screen Shot 2018-10-30 at 12.10.59 PM

如果想偷懒一点，直接安装装好 spark 和 hadoop 的镜像，镜像地址在[这里](https://hub.docker.com/r/uhopper/hadoop-spark/)。

或者直接在终端输入：

    docker pull uhopper/hadoop-spark:2.1.2_2.8.1 

安装完成：

![](https://upload-images.jianshu.io/upload_images/10582164-68f43bde54f0eb0e.jpg)

image-20181030124727908

## 参考

1.  [https://blog.csdn.net/farawayzheng_necas/article/details/54341036](https://blog.csdn.net/farawayzheng_necas/article/details/54341036)

更多精彩内容，就在简书 APP

"小礼物走一走，来简书关注我"

还没有人赞赏，支持一下

[![](https://cdn2.jianshu.io/assets/default_avatar/1-04bbeead395d74921af6a4e8214b4f61.jpg)
](https://www.jianshu.com/u/32f6cd6e553b)

总资产 22 共写了 6.3W 字获得 50 个赞共 28 个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   最近在学习大数据技术，朋友叫我直接学习 Spark，英雄不问出处，菜鸟不问对错，于是我就开始了 Spark 学习。 为什...

    [![](https://upload-images.jianshu.io/upload_images/1268058-94ec7a6e5645f794.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/ee210190224f)
-   感激 101-017 感激上午儿子平和的和我讲了他的一些想法，连接到儿子内心的感觉让我心安和平静。感激我又一次意识到...


-   陈玲，焦点网络中级四期坚持分享，第 875 天。 今天做了我的人生 900 格之后，突然很伤感，看那些黑色和灰色的部分想落...

    [![](https://upload.jianshu.io/users/upload_avatars/7745032/194a5163-9e5c-4164-9e2d-674d9b90b89e.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    煙花](https://www.jianshu.com/u/686cec197d9e)阅读 84 评论 0 赞 0
-   01 世人都说: 鞋子合不合脚只有脚知道。 可是，可是，为什么一双鞋子别人都那么合脚，偏偏你的脚百般不自在？ 你的脚...

    [![](https://upload.jianshu.io/users/upload_avatars/7517156/77bc5ef9-9429-4751-9f90-f58a0f96144e.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    茹小未](https://www.jianshu.com/u/59ec824d27c5)阅读 192 评论 0 赞 4

    [![](https://upload-images.jianshu.io/upload_images/7517156-3cc4154bd3886674.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/839f47588594)
-   A 标签 a 标签是我们在写页面的时候最常用的标签了，相信大家都已经熟悉它的用法了，在这里我只是写一下在开发中注意的... 
    [https://www.jianshu.com/p/6ea173bcc690](https://www.jianshu.com/p/6ea173bcc690)
