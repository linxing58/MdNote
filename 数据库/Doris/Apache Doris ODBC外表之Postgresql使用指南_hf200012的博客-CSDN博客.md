# Apache Doris ODBC外表之Postgresql使用指南_hf200012的博客-CSDN博客
今天是2022年阳历新年第一天，祝大家元旦快乐，这也是2022年第一篇文章，后续会给大家带来更多关于Doris的文章，同时也希望Doris 2022年起飞，顺利从[Apache](https://so.csdn.net/so/search?q=Apache&spm=1001.2101.3001.7020) 孵化器毕业成顶级项目，给大家带来更快、更稳定、生态更丰富的MPP OLAP分析型数据库产品。

Apache Doris 社区 2022 年的总体规划，包括待开展或已开展、以及已完成但需要持续优化的功能、文档、社区建设等多方面，我们期待有更多的小伙伴参与进来讨论。同时也希望多多关注Doris，给Doris加Star

[Apache Doris 2022 Roadmap](https://link.zhihu.com/?target=https%3A//github.com/apache/incubator-doris/issues/7502 "Apache Doris 2022 Roadmap")

该使用指南之针对Ubuntu环境来进行测试的，Centos环境可以参考，但是不确保一定能成功。

**1.软件环境**
----------

1.  操作系统：ubuntu 18.04
2.  Apache Doris ：0.15
3.  Postgresql数据库：PostgreSQL 12.9
4.  UnixODBC：2.3.4
5.  Mysql Connector ODBC ：5.3.13、8.0.11、8.0.26

**2.安装ODBC驱动**
--------------

首先我们安装unixODBC驱动、这里直接给出驱动的下载地址及安装命令

```null
tar -xvzf unixODBC-2.3.4.tar.gz sudo ./configure --prefix=/usr/local/unixODBC-2.3.7 --includedir=/usr/include --libdir=/usr/lib -bindir=/usr/bin --sysconfdir=/etc
```

安装成功后，unixODBC所需的头文件都被安装到了/usr/inlucde下，编译好的库文件安装到了/usr/lib下，与unixODBC相关的可执行文件安装到了/usr/bin下，配置文件放到了/etc下。

验证安装是否成功

```null
DRIVERS............: /etc/odbcinst.iniSYSTEM DATA SOURCES: /etc/odbc.iniFILE DATA SOURCES..: /etc/ODBCDataSourcesUSER DATA SOURCES..: /root/.odbc.ini
```

**3.安装Postgresql数据库**
---------------------

Ubuntu的默认存储库包含Postgres软件包，因此您可以使用`apt`安装这些软件包。

安装之前先用`apt`更新一下本地软件包，然后，安装`Postgres`包和一个附加实用程序和功能的`- managed`包:

```null
$ sudo apt install postgresql postgresql-contrib
```

现在已经安装了该软件，我们可以了解它的工作原理以及它与您可能使用的类似数据库管理系统的不同之处。

### **3.1 使用PostgreSQL roles和数据库**

默认情况下，Postgres使用称为“roles”的概念来处理身份验证和授权。在某些方面，这些类似于常规的Unix风格帐户，但Postgres不区分用户和组，而是更喜欢更灵活的术语“roles”。

安装后，Postgres设置为使用_ident身份_验证，这意味着它将Postgresroles与匹配的Unix / Linux系统帐户相关联。如果Postgres中存在roles，则具有相同名称的Unix / Linux用户名可以作为该roles登录。

安装过程创建了一个名为**postgres**的用户帐户，该帐户与默认的Postgresroles相关联。要使用Postgres，您可以登录该帐户。

有几种方法可以使用此帐户访问Postgres。

### **3.2 切换到postgres帐户**

输入以下内容切换到服务器上的**postgres**帐户：

```null
$ sudo -i -u postgres

```

您现在可以通过输入以下内容立即访问Postgres提示：

```null
$ psql

```

这将使您进入[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020)提示符，从此处您可以立即与数据库管理系统进行交互。

输入以下命令退出PostgreSQL提示符：

```null
postgres=

```

这将带您回到`postgres`Linux命令提示符。

### **3.3 在不切换帐户的情况下访问Postgres**

您也可以让**postgres**帐户用`sudo`运行您想要的命令。

例如，在最后一个示例中，您被指示通过首先切换到**postgres**用户然后运行`psql`以打开Postgres提示来进入Postgres提示。您可以通过`psql`以**postgres**用户身份运行单个命令来一步完成此操作`sudo`，如下所示：

```null
$ sudo -u postgres psql

```

这将直接登录到Postgres，中间没有中间`bash`shell。

同样，您可以通过输入以下内容退出交互式Postgres会话：

```null
postgres=

```

许多用例需要多个Postgresroles。继续阅读以了解如何配置这些

### **3.4 创建用户，数据库及表**

使用默认用户登录postgresql创建用户、创建数据库及完成授权

```null
$ sudo -u postgres psql
```

创建数据库新用户，如 dbuser：

```null
postgres=# CREATE USER dbuser WITH PASSWORD 'zhangfeng';
```

注意：

语句要以分号结尾。 密码要用单引号括起来。 创建用户数据库，你也可以通过你创建的用户登录进去以后创建数据库，如demo：

```null
postgres=
```

将demo数据库的所有权限都赋予dbuser：

```null
postgres=# GRANT ALL PRIVILEGES ON DATABASE demo TO dbuser;
```

使用命令 \\q 退出psql：

```null
postgres=
```

创建Linux普通用户，与刚才新建的数据库用户同名，如 dbuser：

以dbuser的身份连接数据库exampledb：

```null
Last login: Wed Mar 1 11:52:07 CST 2017 on pts/
```

用我们创建的用户（dbuser）登录psql

```null
could not change directory to "/root": Permission deniedpsql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1)) Schema |           Name            |   Type   |  Owner--------+---------------------------+----------+---------- public | playground                | table    | postgres public | playground_1              | table    | dbuser public | playground_1_equip_id_seq | sequence | dbuser public | playground_equip_id_seq   | sequence | postgres
```

创建表及插入数据

```null
CREATE TABLE playground_test_odbc (    equip_id serial PRIMARY KEY,    type varchar (50) NOT NULL,    color varchar (25) NOT NULL,
```

示例数据

```null
INSERT INTO playground_test_odbc (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');INSERT INTO playground_test_odbc (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');
```

执行结果

```null
demo=> CREATE TABLE playground_test_odbc (demo(>     equip_id serial PRIMARY KEY,demo(>     type varchar (50) NOT NULL,demo(>     color varchar (25) NOT NULL,demo(>     location varchar(25) ,demo=> INSERT INTO playground_test_odbc (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');demo=> INSERT INTO playground_test_odbc (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');
```

**4.安装Postgresql ODBC驱动**
-------------------------

这里我们下载是和数据版本相对于的驱动程序

Postgresql ODBC驱动下载地址：[https://www.postgresql.org/ftp/odbc/versions/src/](https://link.zhihu.com/?target=https%3A//www.postgresql.org/ftp/odbc/versions/src/ "https://www.postgresql.org/ftp/odbc/versions/src/")

```null
wget https://ftp.postgresql.org/pub/odbc/versions/src/psqlodbc-12.02.0000.tar.gztar zxvf psqlodbc-12.02.0000.tar.gz./configure --without-libpq   (注：由于本机未安装postgresql，故使用without-libpq选项) 
```

如果在编译过程中出现下面的错误

```null
configure: error: libpq library version >= 9.2 is required
```

这是因为缺少libpq的包，需要进行安装，执行下面的命令

```null
apt-get install libpq-dev
```

安装成功，默认驱动放在/usr/local/lib/psqlodbcw.so下

**5.验证ODBC驱动是否成功**
------------------

### **5.1 配置注册Postgresql ODBC驱动**

编辑/etc/odbcinst.ini，加入下面的内容

```null
Description = ODBC for PostgreSQLDriver = /usr/local/lib/psqlodbcw.soDriver64 = /usr/local/lib/psqlodbcw.soSetup = /usr/lib/libodbc.so    ##注意这里是在第二节安装的unixODBC的so文件路径Setup64 = /usr/lib/libodbc.so
```

### **5.2 配置PG 数据源**

编辑/etc/odbc.ini

加入下面内容

```null
Driver = PostgreSQL    ###这里的名称和odbcinst.ini里配置的名称一致Description = Postgres DSN
```

其他的是你的Postgresql地址及刚才创建的用户、密码、数据库、端口等

### **5.3 验证是否成功**

```null
isql -v PostgresDB dbuser zhangfeng
```

注意这里的PostgresDB是我们在odbc.ini里定义的名称，这里显示ODBC正常

**6.Apache Doris PG外表验证**
-------------------------

### **6.1 修改配置**

修改BE节点conf/odbcinst.ini文件,加入刚才/etc/odbcinst.ini添加的一样内容，并删除原先的PostgreSQL配置

```null
Description = ODBC for PostgreSQLDriver = /usr/local/lib/psqlodbcw.soDriver64 = /usr/local/lib/psqlodbcw.soSetup = /usr/lib/libodbc.soSetup64 = /usr/lib/libodbc.so
```

### **6.2 验证**

创建PG ODBC Resource

```null
CREATE EXTERNAL RESOURCE `pg_12` "password" = "zhangfeng", "table" = "playground_test_odbc", "odbc_type" = "postgresql",
```

创建ODBC外表

```null
CREATE EXTERNAL TABLE `playground_odbc_12` (    type varchar (50) NOT NULL,    color varchar (25) NOT NULL,"odbc_catalog_resource" = "pg_12", "table" = "playground_test_odbc"
```

在Doris下执行查询：

```null
mysql> select * from playground_odbc_12;| equip_id | type  | color  | location  | install_date ||        1 | slide | blue   | south     | 2017-04-28   ||        2 | swing | yellow | northwest | 2018-08-16   |
```

OK，一切正常，相对Mysql PG的ODBC驱动更简单一些，只要你的PG版本和ODBC驱动版本对应上问题都不大。