# Flink CDC 系列 - 构建 MySQL 和 Postgres 上的 Streaming ETL-阿里云开发者社区
> 本篇教程将展示如何基于 Flink CDC 快速构建 MySQL 和 Postgres 的流式 ETL。
> 
> Flink-CDC 项目地址：  
> [https://github.com/ververica/flink-cdc-connectors](https://github.com/ververica/flink-cdc-connectors)

本教程的演示基于 Docker 环境，都将在 Flink SQL CLI 中进行，只涉及 SQL，无需一行 Java/Scala 代码，也无需安装 IDE。

假设我们正在经营电子商务业务，商品和订单的数据存储在 MySQL 中，订单对应的物流信息存储在 Postgres 中。

对于订单表，为了方便进行分析，我们希望让它关联上其对应的商品和物流信息，构成一张宽表，并且实时把它写到 ElasticSearch 中。

接下来的内容将介绍如何使用 Flink Mysql/Postgres CDC 来实现这个需求，系统的整体架构如下图所示：

![](https://img.alicdn.com/imgextra/i2/O1CN01U3K2gs25Gaow9wrA8_!!6000000007499-0-tps-1589-552.jpg)

一、准备阶段
------

准备一台已经安装了 Docker 的 Linux 或者 MacOS 电脑。

### 1.1 准备教程所需要的组件

接下来的教程将以 `docker-compose` 的方式准备所需要的组件。

使用下面的内容创建一个 `docker-compose.yml` 文件：

```null
version: '2.1'
services:
  postgres:
    image: debezium/example-postgres:1.1
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=1234
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  mysql:
    image: debezium/example-mysql:1.1
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_USER=mysqluser
      - MYSQL_PASSWORD=mysqlpw
  elasticsearch:
    image: elastic/elasticsearch:7.6.0
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    ports:
      - "9200:9200"
      - "9300:9300"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
  kibana:
    image: elastic/kibana:7.6.0
    ports:
      - "5601:5601"
```

该 Docker Compose 中包含的容器有：

*   MySQL：商品表 `products` 和 订单表 `orders` 将存储在该数据库中， 这两张表将和 Postgres 数据库中的物流表 `shipments` 进行关联，得到一张包含更多信息的订单表 `enriched_orders`；
*   Postgres：物流表 `shipments` 将存储在该数据库中；
*   Elasticsearch：最终的订单表 `enriched_orders` 将写到 Elasticsearch；
*   Kibana：用来可视化 ElasticSearch 的数据。

在 `docker-compose.yml` 所在目录下执行下面的命令来启动本教程需要的组件：

```null
docker-compose up -d
```

该命令将以 detached 模式自动启动 Docker Compose 配置中定义的所有容器。你可以通过 docker ps 来观察上述的容器是否正常启动了，也可以通过访问 [http://localhost:5601/](http://localhost:5601/) 来查看 Kibana 是否运行正常。

注：本教程接下来用到的容器相关的命令也都需要在 `docker-compose.yml` 所在目录下执行。

### 1.2 下载 Flink 和所需要的依赖包

1.  下载 [Flink 1.13.2](https://downloads.apache.org/flink/flink-1.13.2/flink-1.13.2-bin-scala_2.11.tgz) \[1\] 并将其解压至目录 `flink-1.13.2`
2.  下载下面列出的依赖包，并将它们放到目录 `flink-1.13.2/lib/` 下
    
    *   [flink-sql-connector-elasticsearch7\_2.11-1.13.2.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-elasticsearch7_2.11/1.13.2/flink-sql-connector-elasticsearch7_2.11-1.13.2.jar)
    *   [flink-sql-connector-mysql-cdc-2.1.0.jar](https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.0/flink-sql-connector-mysql-cdc-2.1.0.jar)
    *   [flink-sql-connector-postgres-cdc-2.1.0.jar](https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-postgres-cdc/2.1.0/flink-sql-connector-postgres-cdc-2.1.0.jar)

> \[1\] [https://downloads.apache.org/flink/flink-1.13.2/flink-1.13.2-bin-scala\_2.11.tgz](https://downloads.apache.org/flink/flink-1.13.2/flink-1.13.2-bin-scala_2.11.tgz)

### 1.3 准备数据

#### 1.3.1 在 MySQL 数据库中准备数据

1.  进入 MySQL 容器：
    
    ```null
    docker-compose exec mysql mysql -uroot -p123456
    ```
    
2.  创建数据库和表 `products`，`orders`，并插入数据：
    
    ```null
    
    CREATE DATABASE mydb;
    USE mydb;
    CREATE TABLE products (
      id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(255) NOT NULL,
      description VARCHAR(512)
    );
    ALTER TABLE products AUTO_INCREMENT = 101;
    
    INSERT INTO products
    VALUES (default,"scooter","Small 2-wheel scooter"),
           (default,"car battery","12V car battery"),
           (default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
           (default,"hammer","12oz carpenter's hammer"),
           (default,"hammer","14oz carpenter's hammer"),
           (default,"hammer","16oz carpenter's hammer"),
           (default,"rocks","box of assorted rocks"),
           (default,"jacket","water resistent black wind breaker"),
           (default,"spare tire","24 inch spare tire");
    
    CREATE TABLE orders (
      order_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
      order_date DATETIME NOT NULL,
      customer_name VARCHAR(255) NOT NULL,
      price DECIMAL(10, 5) NOT NULL,
      product_id INTEGER NOT NULL,
      order_status BOOLEAN NOT NULL 
    ) AUTO_INCREMENT = 10001;
    
    INSERT INTO orders
    VALUES (default, '2020-07-30 10:08:22', 'Jark', 50.50, 102, false),
           (default, '2020-07-30 10:11:09', 'Sally', 15.00, 105, false),
           (default, '2020-07-30 12:00:30', 'Edward', 25.25, 106, false);
    ```
    

#### 1.3.2 在 Postgres 数据库中准备数据

1.  进入 Postgres 容器：
    
    ```null
    docker-compose exec postgres psql -h localhost -U postgres
    ```
    
2.  创建表 `shipments`，并插入数据：
    
    ```null
    
    CREATE TABLE shipments (
       shipment_id SERIAL NOT NULL PRIMARY KEY,
       order_id SERIAL NOT NULL,
       origin VARCHAR(255) NOT NULL,
       destination VARCHAR(255) NOT NULL,
       is_arrived BOOLEAN NOT NULL
     );
     ALTER SEQUENCE public.shipments_shipment_id_seq RESTART WITH 1001;
     ALTER TABLE public.shipments REPLICA IDENTITY FULL;
     INSERT INTO shipments
     VALUES (default,10001,'Beijing','Shanghai',false),
            (default,10002,'Hangzhou','Shanghai',false),
            (default,10003,'Shanghai','Hangzhou',false);
    ```
    

二、启动 Flink 集群和 Flink SQL CLI
----------------------------

1.  使用下面的命令跳转至 Flink 目录下：
    
    ```null
    cd flink-1.13.2
    ```
    
2.  使用下面的命令启动 Flink 集群：
    
    ```null
    ./bin/start-cluster.sh
    ```
    
    启动成功的话，可以在 [http://localhost:8081/](http://localhost:8081/) 访问到 Flink Web UI，如下所示：
    
    ![](https://img.alicdn.com/imgextra/i4/O1CN01txQ7XN1QkSOqeN1RD_!!6000000002014-0-tps-3436-878.jpg)
    
3.  使用下面的命令启动 Flink SQL CLI
    
    ```null
    ./bin/sql-client.sh
    ```
    
    启动成功后，可以看到如下的页面：
    

![](https://img.alicdn.com/imgextra/i3/O1CN01hW77jo1mH4XEGLP0b_!!6000000004928-0-tps-946-946.jpg)

三、在 Flink SQL CLI 中使用 Flink DDL 创建表
-----------------------------------

首先，开启 checkpoint，每隔 3 秒做一次 checkpoint。

然后, 对于数据库中的表 `products`, `orders`, `shipments`，使用 Flink SQL CLI 创建对应的表，用于同步这些底层数据库表的数据。

最后，创建 `enriched_orders` 表， 用来将关联后的订单数据写入 Elasticsearch 中。

四、关联订单数据并且将其写入 Elasticsearch 中
------------------------------

使用 Flink SQL 将订单表 `order` 与 商品表 `products`，物流信息表 `shipments` 关联，并将关联后的订单信息写入 Elasticsearch 中。

启动成功后，可以访问 [http://localhost](http://localhost/):8081/#/job/running 在 Flink Web UI 上看到正在运行的 Flink Streaming Job，如下图所示：

![](https://img.alicdn.com/imgextra/i2/O1CN01Jvp2c81pxcAmtQ3m2_!!6000000005427-0-tps-2029-1080.jpg)

现在，就可以在 Kibana 中看到包含商品和物流信息的订单数据。

首先访问 [http://localhost:5601/app/kibana#/management/kibana/index\_pattern](http://localhost:5601/app/kibana#/management/kibana/index_pattern) 创建 index pattern `enriched_orders`。

![](https://img.alicdn.com/imgextra/i2/O1CN01TaYAhI1rJQePTEvUs_!!6000000005610-0-tps-3357-1080.jpg)

然后就可以在 [http://localhost:5601/app/kibana#/discover](http://localhost:5601/app/kibana#/discover) 看到写入的数据了。

![](https://img.alicdn.com/imgextra/i2/O1CN01oD8vaR1WW9j5dUHO0_!!6000000002795-0-tps-3390-1080.jpg)

接下来，修改 MySQL 和 Postgres 数据库中表的数据，Kibana 中显示的订单数据也将实时更新。

1.  在 MySQL 的 `orders` 表中插入一条数据：
    
2.  在 Postgres 的 `shipment` 表中插入一条数据：
    
3.  在 MySQL 的 `orders` 表中更新订单的状态：
    
4.  在 Postgres 的 `shipment` 表中更新物流的状态：
    
5.  在 MYSQL 的 `orders` 表中删除一条数据：
    
    每执行一步就刷新一次 Kibana，可以看到 Kibana 中显示的订单数据将实时更新，如下所示：
    
    ![](https://img.alicdn.com/imgextra/i2/O1CN014X2GQJ291G1Nw65kc_!!6000000008007-1-tps-1200-308.gif)
    

五、环境清理
------

本教程结束后，在 `docker-compose.yml` 文件所在的目录下执行如下命令停止所有容器：

```null
docker-compose down
```

在 Flink 所在目录 `flink-1.13.2` 下执行如下命令停止 Flink 集群：

```null
./bin/stop-cluster.sh
```

六、总结
----

在本文中，我们以一个简单的业务场景展示了如何使用 Flink CDC 快速构建 Streaming ETL。希望通过本文，能够帮助读者快速上手 Flink CDC ，也希望 Flink CDC 能满足你的业务需求。

更多 Flink CDC 相关技术问题，可扫码加入社区钉钉交流群～

![](https://img.alicdn.com/imgextra/i4/O1CN01xXLZuk1zDURWgStVC_!!6000000006680-0-tps-859-1184.jpg)

* * *

**近期热点**

*   [Flink Forward Asia 2021 延期，线上相见](https://developer.aliyun.com/article/811093)
*   [奖金翻倍！Flink Forward Asia Hackathon 最新参赛指南请查收](https://developer.aliyun.com/article/808769)

![](https://img.alicdn.com/imgextra/i3/O1CN01ScFBh11VFpzwTvISH_!!6000000002624-2-tps-1080-319.png)

* * *

更多 Flink 相关技术问题，可扫码加入社区钉钉交流群  
第一时间获取最新技术文章和社区动态，请关注公众号～

![](https://ucc.alicdn.com/pic/developer-ecology/16ed69c2309147a387b946687259ba5f.png)

**活动推荐**

阿里云基于 Apache Flink 构建的企业级产品-实时计算Flink版现开启活动：  
99 元试用 [实时计算Flink版](https://www.aliyun.com/product/bigdata/sc)（包年包月、10CU）即有机会获得 Flink 独家定制卫衣；另包 3 个月及以上还有 85 折优惠！  
了解活动详情：[https://www.aliyun.com/product/bigdata/sc](https://www.aliyun.com/product/bigdata/sc)

![](https://img.alicdn.com/imgextra/i4/O1CN017JJwxq1U8gBGBXvx6_!!6000000002473-0-tps-5653-3144.jpg)