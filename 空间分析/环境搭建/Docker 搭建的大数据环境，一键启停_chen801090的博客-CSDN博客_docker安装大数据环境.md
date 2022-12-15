# Docker 搭建的大数据环境，一键启停_chen801090的博客-CSDN博客_docker安装大数据环境
**代码未动，环境先行**

我是一个[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)爱好者。我在学习大数据相关技术的时候，想到了一个点子：

-   用 docker 搭建一个大数据开发环境！

**这么做有什么好处呢 ？**

我只要有了这个 docker-compose.yml 容器编排描述文件，我就可以在任何一个安装 docker 软件的机器里，启动我的大数据环境。  
一劳永逸的事情，不正是我们程序员每天都在做并且是努力的目标吗？

**如何做？**

找遍了国内的博客和帖子，都没有合适的答案。  
我只能自己来。

### docker hub

首先我去到 docker hub 。 这个就是 github 的 docker 版本。  
我在里面搜索了 很多 Hadoop ， spark 等等关键词，找到了一家公司；

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzIwMDU3ZTg0MGVlNzQ2OTdhY2I3Y2U4ZTA5Y2I3ZmRl?x-oss-process=image/format,png)

这家公司 几乎把所有的 大数据组件都做成了 docker image 。 而且是细粒度，分角色 去划分的。真的太棒了。  
比如 你现在看到的这个图片，就是 他针对于 Hadoop 中 namenode 这一角色做的 docker image。如果你在其之上做一些封装和个性化定制将会变得特别容易。

于是我就从他的 Registry 中找我想要的大数据组件

-   Hadoop
-   Hive
-   Spark

easy , 全都找到了。

**虚拟机**

接线来我们就需要 在虚拟机中安装 docker 了。  
什么 还需要虚拟机 ？  
这里我说一下，安装一个虚拟机吧，windows 各种不方便。（mac 的朋友可以飘过）。

虚拟机我使用 virtual box ， 安装的 ubuntu 。  
然后我就开始安装 docker 了。  
安装了 docker 还需要安装他的孪生兄弟，docker-compose

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzFkY2ZjZTYyYmFhNTRhNzVhNTQ5NzAwMjA3OTIzZjNl?x-oss-process=image/format,png)

**docker-compose.yml**

docker-compose 让 docker 容器的编排变得简单。  
docker-compose.yml 记录了如何编排的过程。他是一个描述文件！  
如下 是我大数据环境的 docker-compose.yml 文件！

````null
version: '2' services:  namenode:    image: bde2020/hadoop-namenode:1.1.0-hadoop2.8-java8    container_name: namenode    volumes:      - ./data/namenode:/hadoop/dfs/name    environment:      - CLUSTER_NAME=test    env_file:      - ./hadoop-hive.env    ports:      - 50070:50070      - 8020:8020    datanode:    image: bde2020/hadoop-datanode:1.1.0-hadoop2.8-java8    depends_on:       - namenode    volumes:      - ./data/datanode:/hadoop/dfs/data    env_file:      - ./hadoop-hive.env    ports:      - 50075:50075  hive-server:    image: bde2020/hive:2.1.0-postgresql-metastore    container_name: hive-server    env_file:      - ./hadoop-hive.env    environment:      - "HIVE_CORE_CONF_javax_jdo_option_ConnectionURL=jdbc:postgresql://hive-metastore/metastore"    ports:      - "10000:10000"  hive-metastore:    image: bde2020/hive:2.1.0-postgresql-metastore    container_name: hive-metastore    env_file:      - ./hadoop-hive.env    command: /opt/hive/bin/hive --service metastore    ports:      - 9083:9083  hive-metastore-postgresql:    image: bde2020/hive-metastore-postgresql:2.1.0    ports:      - 5432:5432    volumes:      - ./data/postgresql/:/var/lib/postgresql/data  spark-master:    image: bde2020/spark-master:2.1.0-hadoop2.8-hive-java8    container_name: spark-master    ports:      - 8080:8080      - 7077:7077    env_file:      - ./hadoop-hive.env  spark-worker:    image: bde2020/spark-worker:2.1.0-hadoop2.8-hive-java8    depends_on:      - spark-master    environment:      - SPARK_MASTER=spark://spark-master:7077    ports:      - "8081:8081"    env_file:      - ./hadoop-hive.env  mysql-server:    image: mysql:5.7    container_name: mysql-server    ports:      - "3306:3306"    environment:      - MYSQL_ROOT_PASSWORD=zhangyang517    volumes:      - ./data/mysql:/var/lib/mysql  elasticsearch:    image: elasticsearch:6.5.3    environment:      - discovery.type=single-node    ports:      - "9200:9200"      - "9300:9300"    networks:       - es_network  kibana:    image: kibana:6.5.3    ports:      - "5601:5601"    networks:       - es_networknetworks:  es_network:    external: true```

后来我需要用到 elasticsearch 和 kibana , 我就直接加上去了。真的非常方便。  
最重要的是 他可以 轻松的 share 给你的小伙伴，好基友。

接下来我们需要写一个启动脚本，一个停止脚本。这样就能实现一键启停了。  
run.sh

```null
#!/bin/bashdocker-compose -f docker-compose.yml up -d namenode hive-metastore-postgresqldocker-compose -f docker-compose.yml up -d datanode hive-metastoresleep 5docker-compose -f docker-compose.yml up -d hive-serverdocker-compose -f docker-compose.yml up -d spark-master spark-workerdocker-compose -f docker-compose.yml up -d mysql-server#docker-compose -f docker-compose.yml up -d elasticsearch#docker-compose -f docker-compose.yml up -d kibanamy_ip=`ip route get 1|awk '{print $NF;exit}'`echo "Namenode: http://${my_ip}:50070"echo "Datanode: http://${my_ip}:50075"echo "Spark-master: http://${my_ip}:8080"```

stop.sh

来看下效果：

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzNhYzFiMjU2MDdmZDRjYmVhZTgwMDc0Mzc3YjY1YTAw?x-oss-process=image/format,png)

启动成功了。验证一下

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzYxMWQwYmE0OTIwMDRkN2RhNWU4ZWIzNWJlZWU3OTFh?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AxLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzAxMmU0MGJhY2Y2ZTQ0NWJhN2MyOWIyYmFmNWU3YjNk?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cDovL3AzLnBzdGF0cC5jb20vbGFyZ2UvcGdjLWltYWdlLzg4MDE1NDQ2ZDc0YzQ5NzhhOGYyY2NjOTRjYzMyOGMy?x-oss-process=image/format,png) 
 [https://blog.csdn.net/chen801090/article/details/103977452](https://blog.csdn.net/chen801090/article/details/103977452)
````
