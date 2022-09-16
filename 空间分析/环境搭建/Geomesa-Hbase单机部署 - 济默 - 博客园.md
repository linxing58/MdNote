# Geomesa-Hbase单机部署 - 济默 - 博客园
本文记录一下 Geomesa-Hbase 单机部署，步骤如下：

1\. 在 VMware 下创建虚拟机

2\. 安装 Linux 系统 (我选的是 centos6.8)

[https://www.cnblogs.com/help-silence/p/12515686.html](https://www.cnblogs.com/help-silence/p/12515686.html)

3\. 网络配置

[https://www.cnblogs.com/help-silence/p/12516589.html](https://www.cnblogs.com/help-silence/p/12516589.html)

4\. 关闭防火墙

[https://www.cnblogs.com/help-silence/p/12516931.html](https://www.cnblogs.com/help-silence/p/12516931.html)

5\. 安装 JDK

[https://www.cnblogs.com/help-silence/p/12517693.html](https://www.cnblogs.com/help-silence/p/12517693.html)

6.Hadoop 单机版安装

7.Hbase 单机版部署

1) 在 / ect/profile 中添加环境变量

\#hbase_home
export HBASE_HOME\\=/opt/module/hbase-1.3.1 export PATH\\=$PATH:$HBASE_HOME/bin

2) 修改配置文件

在 hbase 目录下创建 tmp，pids 两个目录  
修改 hbase-env.sh

export JAVA_HOME=/opt/module/jdk1.8.0_211
export HBASE_MANAGES_ZK\\=true #使用 hbase 自带的 zookeeper(就是存储 hadoop 生态下框架状态的文件系统)

修改 hbase-site.xml

![](https://common.cnblogs.com/images/copycode.gif)

<configuration>
    <property>  <name>hbase.rootdir</name>  <value>file:///opt/module/hbase-1.3.1/disk</value>
    </property>
    <property>
    　　<name>hbase.tmp.dir</name>
    　　<value>/opt/module/hbase-1.3.1/tmp</value>
    </property>
    <property>  <name>hbase.cluster.distributed</name>  
　　　　　<value>false</value>
    </property>
</configuration>

![](https://common.cnblogs.com/images/copycode.gif)

3) 启动 hbase

4) shell 操作

8\. 安装 Geomesa-Hbase

1) 解压

2) 修改 conf 目录下的. env.sh 配置文件

export HBASE_HOME=/opt/module/hbase-1.3.1 export PATH\\=$PATH:$HBASE_HOME/bin
export HADOOP_HOME\\=/opt/module/hadoop-2.7.2 export PATH\\=$PATH:$HADOOP_HOME/bin
export GEOMESA_HBASE_HOME\\=/opt/module/geomesa-hbase_2.11-2.1.0 export PATH\\=$PATH:$GEOMESA_HBASE_HOME/bin

3) 安装图形依赖包

$ bin/install-jai.sh
$ bin/install-jline.sh
注：要是抓取不到 jar 包，自己去下载放在 lib 目录下即可

4) GeoMesa 使用 HBase 的自定义过滤器来执行 CQL 查询，为了允许 GeoMesa 使用过滤器，

需要将 ${GEOMESA_HBASE_HOME}/dist/hbase/geomesa-hbase-distributed-runtime_2.11-2.0.0.jar 拷贝到 ${HBase_HOME}/lib 目录下

5) 注册 Coprocessors

Geomesa 使用 HBase 提供的 coprocessor 工具将处理过程移动到服务器端运行来提高查询效率，
最简单的注册方式就是直接修改 hbase-site.xml，增加以下内容： <property>
   <name>hbase.coprocessor.user.region.classes</name>
   <value>org.locationtech.geomesa.hbase.coprocessor.GeoMesaCoprocessor</value>
 </property>

6）查看版本信息

进入 geomesa-hbase 安装目录
执行 bin/geomesa-hbase version
出现版本信息版本信息即为安装成功

7) 测试环境

向 Hbase 中插入 shp 文件

bin/geomesa-hbase ingest --catalog testGeomesa --feature-name gps --input-format shp "/opt/data/gps.shp"

将 gps 点数据展示出来

bin/geomesa-hbase export --output-format leaflet --feature-name gps --zookeepers localhost --catalog testGeomesa

![](https://img2020.cnblogs.com/blog/1735699/202005/1735699-20200502112836513-163402636.png) 
 [https://www.cnblogs.com/help-silence/p/12817447.html](https://www.cnblogs.com/help-silence/p/12817447.html)
