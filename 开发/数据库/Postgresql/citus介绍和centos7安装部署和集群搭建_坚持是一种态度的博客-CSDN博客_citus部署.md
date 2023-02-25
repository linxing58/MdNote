# citus介绍和centos7安装部署和集群搭建_坚持是一种态度的博客-CSDN博客_citus部署
### 文章目录

*   *   [citus](#citus_1)
    *   *   [简介](#_2)
        *   [citus主要特性](#citus_8)
    *   [部署](#_17)
    *   *   [centos单节点版本部署启动](#centos_18)
        *   [centos集群部署启动](#centos_67)
        *   *   [要在所有节点上执行的步骤](#_70)
            *   [要在协调器节点上执行的步骤](#_117)
    *   [常用语句](#_136)
    *   [遇到的问题](#_183)
    *   [参考](#_201)

citus
-----

### 简介

*   Citus是PostgreSQL数据库的分布式中间件，用以解决PostgreSQL横向扩展问题，以支持更大的数据量、更大的写入和查询性能。
*   Citus由CitusData公司开发，目前已被微软收购,并在Azure上提供Citus Cloud服务。Citus为开源软件，经过约10年的发展，最近刚发布10.0版本，License为AGPL。
*   不像pg-xc的原生分布式方案，Citus是以Extension的方式扩展PostgreSQL能力，不侵入修改PostgreSQL内核代码，可以很容易和PostgreSQL的新版本配套适用，享受到内核版本演进带来的好处
*   citus是一款基于PostgreSQL的开源分布式数据库,自动继承了PostgreSQL强大的SQL支持能力和应用生态（不仅仅是客户端协议的兼容还包括服务端扩展和管理工具的完全兼容）

### citus主要特性

*   PostgreSQL兼容
*   水平扩展
*   实时并发查
*   快速数据加载
*   实时增删改查
*   持分布式事务
*   支持常用DDL

部署
--

### centos单节点版本部署启动

下载安装，参考：[https://docs.citusdata.com/en/stable/installation/single\_node\_rhel.html](https://docs.citusdata.com/en/stable/installation/single_node_rhel.html)

1.  安装 Citus 扩展（默认你已安装PostgreSQL 13，如果没安装PostgreSQL 13，需要先安装）

```
# 添加 Citus repository 这一步可能会因为网络原因失败，多来几次
curl https://install.citusdata.com/community/rpm.sh | sudo bash

# 安装 Citus
sudo yum install -y citus100_13

```

2.  初始化集群，让我们在磁盘上创建一个新数据库。为了方便使用 PostgreSQL Unix 域套接字连接，我们将使用`postgres`用户（安装PostgresQL时默认创建的）。

```
# 进入默认的postgres用户操作
sudo su - postgres

# include path to postgres binaries
export PATH=$PATH:/usr/pgsql-13/bin

cd ~
# 创建文件夹
mkdir citus
# 初始化数据库到citus文件夹
initdb -D citus

```

Citus 是 Postgres 的扩展。要告诉 Postgres 使用此扩展，您需要修改[PostgresQL](https://so.csdn.net/so/search?q=PostgresQL&spm=1001.2101.3001.7020)配置文件，将其添加到名为 `shared_preload_libraries` 的配置变量中:

```
echo "shared_preload_libraries = 'citus'" >> citus/postgresql.conf

```

3.  启动数据库服务器。最后，我们将为新目录启动一个 PostgreSQL 实例：

```
# 启动实例
pg_ctl -D citus -o "-p 9700" -l citus_logfile start
# 关闭实例
pg_ctl -D citus -o "-p 9700" -l citus_logfile stop
# 重启实例
pg_ctl -D citus -o "-p 9700" -l citus_logfile restart

```

在上面，您将 Citus 添加到`shared_preload_libraries`. 这让它可以连接到 Postgres 的一些深层部分，交换查询计划器和执行器。在这里，我们加载 Citus 面向用户的一面（例如您即将调用的函数）：

```
psql -p 9700 -c "CREATE EXTENSION citus;"

```

4.  验证安装是否成功

```
psql -p 9700 -c "select citus_version();"

```

### centos集群部署启动

集群版本和单节点差不多，都是先安装citus，改配置，增加extension，只是最后需要在cn节点增加cn节点和work节点配置。  
安装参考：[https://docs.citusdata.com/en/stable/installation/multi\_node\_rhel.html](https://docs.citusdata.com/en/stable/installation/multi_node_rhel.html)

#### 要在所有节点上执行的步骤

1.  添加仓库

```
# Add Citus repository for package manager
curl https://install.citusdata.com/community/rpm.sh | sudo bash

```

2.  安装Citus，并初始化一个数据库（默认你已安装PostgreSQL 13，如果没安装PostgreSQL 13，需要先安装）

```
# install PostgreSQL with Citus extension
sudo yum install -y citus100_13
# initialize system database (using RHEL 6 vs 7 method as necessary)
sudo service postgresql-13 initdb || sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
# preload citus extension
echo "shared_preload_libraries = 'citus'" | sudo tee -a /var/lib/pgsql/13/data/postgresql.conf

```

3.  配置连接和认证  
    在启动数据库之前，让我们更改其访问权限。默认情况下，数据库服务器仅侦听本地主机上的客户端。作为此步骤的一部分，我们指示它侦听所有 IP 接口，然后配置客户端身份验证文件以允许来自本地网络的所有传入连接。

```
sudo vi /var/lib/pgsql/13/data/postgresql.conf

# Uncomment listen_addresses for the changes to take effect
listen_addresses = '*'

```

```
sudo vi /var/lib/pgsql/13/data/pg_hba.conf

# Allow unrestricted access to nodes in the local network. The following ranges
# correspond to 24, 20, and 16-bit blocks in Private IPv4 address spaces.
# 10.0.0.0/8 是postgres集群服务器ip段
host    all             all             10.0.0.0/8              trust

# Also allow the host unrestricted access to connect to itself
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust

```

4.  启动数据库服务器，创建Citus扩展

```
# start the db server
sudo service postgresql-13 restart
# and make it start automatically when computer does
sudo chkconfig postgresql-13 on

```

您必须将 Citus 扩展添加到要在集群中使用的每个数据库(如果你有多个库)。以下示例将扩展添加到名为postgres的默认数据库。

```
sudo -i -u postgres psql -c "CREATE EXTENSION citus;"

```

#### 要在协调器节点上执行的步骤

以上步骤在所有postgres服务器执行，安装好citus，设置extension。下面的步骤，仅在协调器节点上执行。

1.  设置本节点为协调节点 cn

```
 # 设置协调节点cn
 SELECT citus_set_coordinator_host('192.168.1.76');

```

2.  把其他节点设置为工作节点 work

```
SELECT * from master_add_node('192.168.1.73', 5432);
SELECT * from master_add_node('192.168.1.74', 5432);

```

3.  验证集群是否成功

```
 # 查看所有节点
 select * from pg_dist_node;
 # 查看工作节点
 SELECT * FROM master_get_active_worker_nodes();

```

常用语句
----

```
 # shell命令
 # 将postgres的bin目录加入到path
 export PATH=$PATH:/usr/pgsql-13/bin
 # 重启 citus 扩展
 pg_ctl -D citus -o "-p 9700" -l citus_logfile restart
 # 查看postgres版本（-c执行SQL查看）
 psql -c "select version();"
 # 查看citus版本
 psql -p 9700 -c "select citus_version();"
 # 查看端口号占用
 netstat -lnp|grep 9700
 # 重启postgresql
 systemctl restart postgresql-13
 # 进入 postgres 用户
 sudo su - postgres 
 # 进入postgres命令窗口，可选：-p 端口号 -h ip地址
 psql 
 psql -h 192.168.1.74 -p 5430
 
 # sql 语句
 # 查看postgres版本
 select version();
 # 创建citus扩展
 CREATE EXTENSION citus;
 # 查看citus版本
 select citus_version();
 # 查看节点
 select * from pg_dist_node;
 # 查看工作节点
 SELECT * FROM master_get_active_worker_nodes();
 # 设置协调节点cn
 SELECT citus_set_coordinator_host('192.168.1.74');
 # 增加节点 
 SELECT * from master_add_node('192.168.1.73', 5432);
 # 删除节点 
 SELECT master_remove_node('192.168.1.74', '9700');
 # 查询有哪些表
 select tablename from pg_tables where tablename not like 'pg_%';
 # 查询有哪些表
 select tablename from pg_tables where tablename like 'ads%';
 # 删除extension
 drop extension citus cascade;
 

```

遇到的问题
-----

1.  citus无法yum安装源

```
https://repos.citusdata.com/community/el/7/x86_64/repodata/repomd.xml: [Errno 14] HTTPS Error 302 - Found
Trying other mirror.
failure: repodata/repomd.xml from citusdata_community: [Errno 256] No more mirrors to try.
https://repos.citusdata.com/community/el/7/x86_64/repodata/repomd.xml: [Errno 14] HTTPS Error 302 - Found

```

> 网络问题，多来几次

2.  pg\_ctl: command not found

> 将postgres的bin目录加入path：`export PATH=$PATH:/usr/pgsql-13/bin`，再去执行pg\_ctl

3.  extension 无法删除，或者database、table无法删除

```
RROR:  cannot drop extension citus because other objects depend on it
HINT:  Use DROP ... CASCADE to drop the dependent objects too.

```

> 确定不要的话，可以关联删除，全部删掉。根据提示直接加上 cascade `drop extension citus cascade;`

参考
--

*   使用文档：[多租户应用](http://docs.citusdata.com/en/v10.0/get_started/tutorial_multi_tenant.html)
*   安装地址： [单节点安装部署](https://docs.citusdata.com/en/stable/installation/single_node_rhel.html)
*   安装地址：[多节点安装部署](https://docs.citusdata.com/en/stable/installation/multi_node_rhel.html)