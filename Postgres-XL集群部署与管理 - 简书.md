# Postgres-XL集群部署与管理 - 简书
[![](https://upload.jianshu.io/users/upload_avatars/68979/ade680e57161?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/2cbf98ef6104)

0.2472016.08.27 16:50:29 字数 2,458 阅读 23,672

Postgres 的 - XL 是一个基于 PostgreSQL 数据库的横向扩展开源 SQL 数据库集群，具有足够的灵活性来处理不同的数据库工作负载:

-   **完全 ACID，保持事务一致性**
-   **OLTP 写频繁的业务**
-   **需要 MPP 并行性商业智能 / 大数据分析**
-   **操作数据存储**
-   **Key-value 存储**
-   **GIS 的地理空间**
-   **混合业务工作环境**
-   **多租户服务提供商托管环境**
-   **Web 2.0**  

    ![](https://upload-images.jianshu.io/upload_images/68979-45bfbc62d3671b5f.png)

    Postgres-XL 架构


-   **Global Transaction Monitor (GTM)**  
    全局事务管理器，确保群集范围内的事务一致性。 GTM 负责发放事务 ID 和快照作为其多版本并发控制的一部分。  
    集群可选地配置一个备用 GTM，以改进可用性。此外，可以在协调器间配置代理 GTM， 可用于改善可扩展性，减少 GTM 的通信量。
-   **GTM Standby**  
    GTM 的备节点，在 pgxc,pgxl 中，GTM 控制所有的全局事务分配，如果出现问题，就会导致整个集群不可用，为了增加可用性，增加该备用节点。当 GTM 出现问题时，GTM Standby 可以升级为 GTM，保证集群正常工作。
-   **GTM-Proxy**  
    GTM 需要与所有的 Coordinators 通信，为了降低压力，可以在每个 Coordinator 机器上部署一个 GTM-Proxy。
-   **Coordinator**  
    协调员管理用户会话，并与 GTM 和数据节点进行交互。协调员解析，并计划查询，并给语句中的每一个组件发送下一个序列化的全局性计划。  
    为节省机器，通常此服务和数据节点部署在一起。
-   **Data Node**  
    数据节点是数据实际存储的地方。数据的分布可以由 DBA 来配置。为了提高可用性，可以配置数据节点的热备以便进行故障转移准备。

总结：gtm 是负责 ACID 的，保证分布式数据库全局事务一致性。得益于此，就算数据节点是分布的，但是你在主节点操作增删改查事务时，就如同只操作一个数据库一样简单。Coordinator 是调度的，将操作指令发送到各个数据节点。datanodes 是数据节点，分布式存储数据。

更多介绍参考：[《Postgres-XL：基于 PostgreSQL 的开源分布式实现》](https://www.biaodianfu.com/postgres-xl.html)

## 3.1 集群规划

准备三台 Centos7 服务器（或者虚拟机），集群规划如下：

| 主机名       | IP            | 角色             | 端口    | nodename    | 数据目录             |
| --------- | ------------- | -------------- | ----- | ----------- | ---------------- |
| gtm       | 192.168.0.125 | GTM            | 6666  | gtm         | /nodes/gtm       |
|           |               | GTM Slave      | 20001 | gtmSlave    | /nodes/gtmSlave  |
| datanode1 | 192.168.0.127 | Coordinator    | 5432  | coord1      | /nodes/coord     |
|           |               | Datanode       | 5433  | node1       | /nodes/dn_master |
|           |               | Datanode Slave | 15433 | node1_slave | /nodes/dn_slave  |
|           |               | GTM Proxy      | 6666  | gtm_pxy1    | /nodes/gtm_pxy   |
| datanode2 | 192.168.0.128 | Coordinator    | 5432  | coord2      | /nodes/coord     |
|           |               | Datanode       | 5433  | node2       | nodes/dn_master  |
|           |               | Datanode Slave | 15433 | node2_slave | /nodes/dn_slave  |
|           |               | GTM Proxy      | 6666  | gtm_pxy2    | /nodes/gtm_pxy   |

在每台机器的 /etc/hosts 中加入以下内容：

    192.168.0.125 gtm
    192.168.0.126 datanode1
    192.168.0.127 datanode2 

gtm 上部署 gtm，gtm_sandby 测试环境暂未部署。  
Coordinator 与 Datanode 节点一般部署在同一台机器上。实际上，GTM-proxy,Coordinator 与 Datanode 节点一般都在同一个机器上，使用时避免端口号与连接池端口号重叠！规划 datanode1,datanode2 作为协调节点与数据节点。

## 3.2 系统环境设置

以下操作，对每个服务器节点都适用。  
关闭防火墙：

    [root@localhost ~]# systemctl stop firewalld.service
    [root@localhost ~]# systemctl disable firewalld.service 

selinux 设置:

    [root@localhost ~]#vim /etc/selinux/config 

设置 SELINUX=disabled，保存退出。

    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=disabled
    # SELINUXTYPE= can take one of three two values:
    #     targeted - Targeted processes are protected,
    #     minimum - Modification of targeted policy. Only selected processes are protected.
    #     mls - Multi Level Security protection. 

安装依赖包：

    [root@localhost ~]# yum install -y flex bison readline-devel zlib-devel openjade docbook-style-dsssl 

同步系统时间

    [root@localhost ~]# ntpdate asia.pool.ntp.org 

重启服务器！一定要重启！

## 3.3 新建用户

每个节点都建立用户 postgres，并且建立. ssh 目录，并配置相应的权限：

    [root@localhost ~]# useradd postgres
    [root@localhost ~]# passwd postgres
    [root@localhost ~]# su - postgres
    [root@localhost ~]# mkdir ~/.ssh
    [root@localhost ~]# chmod 700 ~/.ssh 

## 3.4 ssh 免密码登录

仅仅在 gtm 节点配置如下操作：

    [root@localhost ~]# su - postgres
    [postgres@localhost ~]# ssh-keygen -t rsa
    [postgres@localhost ~]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    [postgres@localhost ~]# chmod 600 ~/.ssh/authorized_keys 

将刚生成的认证文件拷贝到 datanode1 到 datanode2 中，使得 gtm 节点可以免密码登录 datanode1~datanode2 的任意一个节点：

    [postgres@localhost ~]# scp ~/.ssh/authorized_keys postgres@datanode1:~/.ssh/
    [postgres@localhost ~]# scp ~/.ssh/authorized_keys postgres@datanode2:~/.ssh/ 

对所有提示都不要输入，直接 enter 下一步。直到最后，因为第一次要求输入目标机器的用户密码，输入即可。

## 3.5 Postgres-XL 安装

pg1-pg3 每个节点都需安装配置。切换回 root 用户下，执行如下步骤安装

    [root@localhost ~]#  cd /opt
    [root@localhost ~]# git clone git://git.postgresql.org/git/postgres-xl.git
    [root@localhost ~]# cd postgres-xl
    [root@localhost ~postgres-xl]# ./configure --prefix=/home/postgres/pgxl/
    [root@localhost ~postgres-xl]# make
    [root@localhost ~postgres-xl]# make install
    [root@localhost ~postgres-xl]# cd contrib/  
    [root@localhost ~contrib]# make
    [root@localhost ~contrib]# make install 

cortrib 中有很多 postgres 很牛的工具，一般要装上。如 ltree,uuid,postgres_fdw 等等。

## 3.6 配置环境变量

进入 postgres 用户，修改其环境变量，开始编辑

    [root@localhost ~]#su - postgres
    [postgres@localhost ~]#vi .bashrc 

在打开的文件末尾，新增如下变量配置：

    export PGHOME=/home/postgres/pgxl
    export LD_LIBRARY_PATH=$PGHOME/lib:$LD_LIBRARY_PATH
    export PATH=$PGHOME/bin:$PATH 

按住 esc，然后输入: wq! 保存退出。输入以下命令对更改重启生效。

    [root@localhost ~]# source .bashrc
    #输入以下语句，如果输出变量结果，代表生效
    [root@localhost ~]# echo $PGHOME
    #应该输出/home/postgres/pgxl代表生效 

如上操作，除特别强调以外，是 datanode1-datanode2 节点都要配置安装的。

## 4.1 生成 pgxc_ctl 配置文件

    [postgres@localhost ~]# pgxc_ctl

    PGXC prepare ---执行该命令将会生成一份配置文件模板
    PGXC   ---按ctrl c退出。 

## 4.2 配置 pgxc_ctl.conf

在 pgxc_ctl 文件夹中存在一个 pgxc_ctl.conf 文件，编辑如下：

    pgxcInstallDir=$PGHOME
    pgxlDATA=$PGHOME/data 

    pgxcOwner=postgres

    #---- GTM Master -----------------------------------------
    gtmName=gtm
    gtmMasterServer=gtm
    gtmMasterPort=6666
    gtmMasterDir=$pgxlDATA/nodes/gtm

    gtmSlave=y                  # Specify y if you configure GTM Slave.   Otherwise, GTM slave will not be configured and
                                # all the following variables will be reset.
    gtmSlaveName=gtmSlave
    gtmSlaveServer=gtm      # value none means GTM slave is not available.  Give none if you don't configure GTM Slave.
    gtmSlavePort=20001          # Not used if you don't configure GTM slave.
    gtmSlaveDir=$pgxlDATA/nodes/gtmSlave    # Not used if you don't configure GTM slave.

    #---- GTM-Proxy Master -------
    gtmProxyDir=$pgxlDATA/nodes/gtm_proxy
    gtmProxy=y                              
    gtmProxyNames=(gtm_pxy1 gtm_pxy2)   
    gtmProxyServers=(datanode1 datanode2)           
    gtmProxyPorts=(6666 6666)               
    gtmProxyDirs=($gtmProxyDir $gtmProxyDir)            

    #---- Coordinators ---------
    coordMasterDir=$pgxlDATA/nodes/coord
    coordNames=(coord1 coord2)      
    coordPorts=(5432 5432)          
    poolerPorts=(6667 6667)         
    coordPgHbaEntries=(0.0.0.0/0)

    coordMasterServers=(datanode1 datanode2)    
    coordMasterDirs=($coordMasterDir $coordMasterDir)
    coordMaxWALsernder=0    #没设置备份节点，设置为0
    coordMaxWALSenders=($coordMaxWALsernder $coordMaxWALsernder) #数量保持和coordMasterServers一致

    coordSlave=n

    #---- Datanodes ----------
    datanodeMasterDir=$pgxlDATA/nodes/dn_master
    primaryDatanode=node1               # 主数据节点
    datanodeNames=(node1 node2)
    datanodePorts=(5433 5433)   
    datanodePoolerPorts=(6668 6668) 
    datanodePgHbaEntries=(0.0.0.0/0)

    datanodeMasterServers=(datanode1 datanode2)
    datanodeMasterDirs=($datanodeMasterDir $datanodeMasterDir)
    datanodeMaxWalSender=4
    datanodeMaxWALSenders=($datanodeMaxWalSender $datanodeMaxWalSender)

    datanodeSlave=n
    #将datanode1节点的slave做到了datanode2服务器上，交叉做了备份
    datanodeSlaveServers=(datanode2 datanode1)  # value none means this slave is not available
    datanodeSlavePorts=(15433 15433)    # value none means this slave is not available
    datanodeSlavePoolerPorts=(20012 20012)  # value none means this slave is not available
    datanodeSlaveSync=y     # If datanode slave is connected in synchronized mode
    datanodeSlaveDirs=($datanodeSlaveDir $datanodeSlaveDir)
    datanodeArchLogDirs=( $datanodeArchLogDir $datanodeArchLogDir) 

如上配置，都没有配置 slave，具体生产环境，请阅读配置文件，自行配置。

## 4.3 集群初始化，启动，停止

第一次启动集群，需要初始化，初始化如下：

    [postgres@pg1 pgxc_ctl]$ pgxc_ctl -c /home/postgres/pgxc_ctl/pgxc_ctl.conf init all 

初始化后会直接启动集群。

    /bin/bash
    Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
    Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
    Reading configuration using /home/postgres/pgxc_ctl/pgxc_ctl_bash --home /home/postgres/pgxc_ctl --configuration /home/postgres/pgxc_ctl/pgxc_ctl.conf
    Finished reading configuration.
       ******** PGXC_CTL START ***************

    Current directory: /home/postgres/pgxc_ctl
    Initialize GTM master
    ERROR: target directory (/home/postgres/pgxl/data/nodes/gtm) exists and not empty. Skip GTM initilialization
    1:1430554432:2017-07-11 23:31:14.737 PDT -FATAL:  lock file "gtm.pid" already exists
    2:1430554432:2017-07-11 23:31:14.737 PDT -HINT:  Is another GTM (PID 2823) running in data directory "/home/postgres/pgxl/data/nodes/gtm"?
    LOCATION:  CreateLockFile, main.c:2099
    waiting for server to shut down.... done
    server stopped
    Done.
    Start GTM master
    server starting
    Initialize all the gtm proxies.
    Initializing gtm proxy gtm_pxy1.
    Initializing gtm proxy gtm_pxy2.
    The files belonging to this GTM system will be owned by user "postgres".
    This user must also own the server process.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/gtm_pxy ... ok
    creating configuration files ... ok

    Success.
    The files belonging to this GTM system will be owned by user "postgres".
    This user must also own the server process.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/gtm_pxy ... ok
    creating configuration files ... ok

    Success.
    Done.
    Starting all the gtm proxies.
    Starting gtm proxy gtm_pxy1.
    Starting gtm proxy gtm_pxy2.
    server starting
    server starting
    Done.
    Initialize all the coordinator masters.
    Initialize coordinator master coord1.
    Initialize coordinator master coord2.
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/coord ... ok
    creating subdirectories ... ok
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting dynamic shared memory implementation ... posix
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... creating cluster information ... ok
    syncing data to disk ... ok
    freezing database template0 ... ok
    freezing database template1 ... ok
    freezing database postgres ... ok

    WARNING: enabling "trust" authentication for local connections
    You can change this by editing pg_hba.conf or using the option -A, or
    --auth-local and --auth-host, the next time you run initdb.

    Success.
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/coord ... ok
    creating subdirectories ... ok
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting dynamic shared memory implementation ... posix
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... creating cluster information ... ok
    syncing data to disk ... ok
    freezing database template0 ... ok
    freezing database template1 ... ok
    freezing database postgres ... ok

    WARNING: enabling "trust" authentication for local connections
    You can change this by editing pg_hba.conf or using the option -A, or
    --auth-local and --auth-host, the next time you run initdb.

    Success.
    Done.
    Starting coordinator master.
    Starting coordinator master coord1
    Starting coordinator master coord2
    2017-07-11 23:31:31.116 PDT [3650] LOG:  listening on IPv4 address "0.0.0.0", port 5432
    2017-07-11 23:31:31.116 PDT [3650] LOG:  listening on IPv6 address "::", port 5432
    2017-07-11 23:31:31.118 PDT [3650] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
    2017-07-11 23:31:31.126 PDT [3650] LOG:  redirecting log output to logging collector process
    2017-07-11 23:31:31.126 PDT [3650] HINT:  Future log output will appear in directory "pg_log".
    2017-07-11 23:31:31.122 PDT [3613] LOG:  listening on IPv4 address "0.0.0.0", port 5432
    2017-07-11 23:31:31.122 PDT [3613] LOG:  listening on IPv6 address "::", port 5432
    2017-07-11 23:31:31.124 PDT [3613] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
    2017-07-11 23:31:31.132 PDT [3613] LOG:  redirecting log output to logging collector process
    2017-07-11 23:31:31.132 PDT [3613] HINT:  Future log output will appear in directory "pg_log".
    Done.
    Initialize all the datanode masters.
    Initialize the datanode master datanode1.
    Initialize the datanode master datanode2.
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/dn_master ... ok
    creating subdirectories ... ok
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting dynamic shared memory implementation ... posix
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... creating cluster information ... ok
    syncing data to disk ... ok
    freezing database template0 ... ok
    freezing database template1 ... ok
    freezing database postgres ... ok

    WARNING: enabling "trust" authentication for local connections
    You can change this by editing pg_hba.conf or using the option -A, or
    --auth-local and --auth-host, the next time you run initdb.

    Success.
    The files belonging to this database system will be owned by user "postgres".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_US.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /home/postgres/pgxl/data/nodes/dn_master ... ok
    creating subdirectories ... ok
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting dynamic shared memory implementation ... posix
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... creating cluster information ... ok
    syncing data to disk ... ok
    freezing database template0 ... ok
    freezing database template1 ... ok
    freezing database postgres ... ok

    WARNING: enabling "trust" authentication for local connections
    You can change this by editing pg_hba.conf or using the option -A, or
    --auth-local and --auth-host, the next time you run initdb.

    Success.
    Done.
    Starting all the datanode masters.
    Starting datanode master datanode1.
    Starting datanode master datanode2.
    2017-07-11 23:31:37.013 PDT [3995] LOG:  listening on IPv4 address "0.0.0.0", port 5433
    2017-07-11 23:31:37.013 PDT [3995] LOG:  listening on IPv6 address "::", port 5433
    2017-07-11 23:31:37.014 PDT [3995] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"
    2017-07-11 23:31:37.021 PDT [3995] LOG:  redirecting log output to logging collector process
    2017-07-11 23:31:37.021 PDT [3995] HINT:  Future log output will appear in directory "pg_log".
    2017-07-11 23:31:37.008 PDT [3958] LOG:  listening on IPv4 address "0.0.0.0", port 5433
    2017-07-11 23:31:37.008 PDT [3958] LOG:  listening on IPv6 address "::", port 5433
    2017-07-11 23:31:37.009 PDT [3958] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"
    2017-07-11 23:31:37.017 PDT [3958] LOG:  redirecting log output to logging collector process
    2017-07-11 23:31:37.017 PDT [3958] HINT:  Future log output will appear in directory "pg_log".
    Done.
    ALTER NODE coord1 WITH (HOST='datanode1', PORT=5432);
    ALTER NODE
    CREATE NODE coord2 WITH (TYPE='coordinator', HOST='datanode2', PORT=5432);
    CREATE NODE
    CREATE NODE datanode1 WITH (TYPE='datanode', HOST='datanode1', PORT=5433, PRIMARY, PREFERRED);
    CREATE NODE
    CREATE NODE datanode2 WITH (TYPE='datanode', HOST='datanode2', PORT=5433);
    CREATE NODE
    SELECT pgxc_pool_reload();
     pgxc_pool_reload 
    ------------------
     t
    (1 row)

    CREATE NODE coord1 WITH (TYPE='coordinator', HOST='datanode1', PORT=5432);
    CREATE NODE
    ALTER NODE coord2 WITH (HOST='datanode2', PORT=5432);
    ALTER NODE
    CREATE NODE datanode1 WITH (TYPE='datanode', HOST='datanode1', PORT=5433, PRIMARY);
    CREATE NODE
    CREATE NODE datanode2 WITH (TYPE='datanode', HOST='datanode2', PORT=5433, PREFERRED);
    CREATE NODE
    SELECT pgxc_pool_reload();
     pgxc_pool_reload 
    ------------------
     t
    (1 row)

    Done.
    EXECUTE DIRECT ON (datanode1) 'CREATE NODE coord1 WITH (TYPE=''coordinator'', HOST=''datanode1'', PORT=5432)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode1) 'CREATE NODE coord2 WITH (TYPE=''coordinator'', HOST=''datanode2'', PORT=5432)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode1) 'ALTER NODE datanode1 WITH (TYPE=''datanode'', HOST=''datanode1'', PORT=5433, PRIMARY, PREFERRED)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode1) 'CREATE NODE datanode2 WITH (TYPE=''datanode'', HOST=''datanode2'', PORT=5433, PREFERRED)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode1) 'SELECT pgxc_pool_reload()';
     pgxc_pool_reload 
    ------------------
     t
    (1 row)

    EXECUTE DIRECT ON (datanode2) 'CREATE NODE coord1 WITH (TYPE=''coordinator'', HOST=''datanode1'', PORT=5432)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode2) 'CREATE NODE coord2 WITH (TYPE=''coordinator'', HOST=''datanode2'', PORT=5432)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode2) 'CREATE NODE datanode1 WITH (TYPE=''datanode'', HOST=''datanode1'', PORT=5433, PRIMARY, PREFERRED)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode2) 'ALTER NODE datanode2 WITH (TYPE=''datanode'', HOST=''datanode2'', PORT=5433, PREFERRED)';
    EXECUTE DIRECT
    EXECUTE DIRECT ON (datanode2) 'SELECT pgxc_pool_reload()';
     pgxc_pool_reload 
    ------------------
     t
    (1 row)

    Done. 

以后启动，直接执行如下命令：

    [postgres@pg1 pgxc_ctl]$ pgxc_ctl -c /home/postgres/pgxc_ctl/pgxc_ctl.conf start all 

停止集群如下：

    [postgres@pg1 pgxc_ctl]$ pgxc_ctl -c /home/postgres/pgxc_ctl/pgxc_ctl.conf stop all 

这几个主要命令暂时这么多，更多请从 pgxc_ctl --help 中获取更多信息。

## 5.1 插入数据

在 datanode1 节点，执行 psql -p 5432 进入数据库操作。

    [postgres@localhost]$ psql -p 5432
    psql (PGXL 10alpha1, based on PG 10beta1 (Postgres-XL 10alpha1))
    Type "help" for help.

    postgres=
     node_name | node_type | node_port | node_host | nodeis_primary | nodeis_preferred |   node_id   
    -----------+-----------+-----------+-----------+----------------+------------------+-------------
     coord1    | C         |      5432 | datanode1 | f              | f                |  1885696643
     coord2    | C         |      5432 | datanode2 | f              | f                | -1197102633
     datanode1 | D         |      5433 | datanode1 | t              | t                |   888802358
     datanode2 | D         |      5433 | datanode2 | f              | f                |  -905831925
    (4 rows)
    postgres=
    postgres= 

## 5.2 查看数据分布

在 datanode1 节点上查看数据

    [postgres@bogon ~]$ psql -p 5433
    psql (PGXL 10alpha1, based on PG 10beta1 (Postgres-XL 10alpha1))
    Type "help" for help.

    postgres=
     id | name 
    ----+------
      1 | 测试
      2 | 测试
      5 | 测试
      6 | 测试
      8 | 测试
    (5 rows) 

在 datanode2 节点上查看数据

    postgres=
     id | name 
    ----+------
      3 | 测试
      4 | 测试
      7 | 测试
    (3 rows) 

注意：由于所有的数据节点组成了完整的数据视图，所以一个数据节点 down 机，整个 pgxl 都启动不了了，所以实际生产中，为了提高可用性，一定要配置数据节点的热备以便进行故障转移准备。

## 6.1 建表说明

-   REPLICATION 表：各个 datanode 节点中，表的数据完全相同，也就是说，插入数据时，会分别在每个 datanode 节点插入相同数据。读数据时，只需要读任意一个 datanode 节点上的数据。  
    建表语法：


    postgres=#  CREATE TABLE repltab (col1 int, col2 int) DISTRIBUTE BY REPLICATION; 

-   DISTRIBUTE ：会将插入的数据，按照拆分规则，分配到不同的 datanode 节点中存储，也就是 sharding 技术。每个 datanode 节点只保存了部分数据，通过 coordinate 节点可以查询完整的数据视图。


    postgres=#  CREATE TABLE disttab(col1 int, col2 int, col3 text) DISTRIBUTE BY HASH(col1); 

模拟部分数据，插入测试数据：

     [postgres@gtm ~]$  psql -h  datanode1 -p 5432 -U postgres
    postgres=
    INSERT 0 100
    postgres=
    INSERT 0 100 

查看数据分布结果：

    #DISTRIBUTE表分布结果
    postgres=# SELECT xc_node_id, count(*) FROM disttab GROUP BY xc_node_id;
     xc_node_id | count 
    ------------+-------
     1148549230 |    42
     -927910690 |    58
    (2 rows)
    #REPLICATION表分布结果
    postgres=# SELECT count(*) FROM repltab;
     count 
    -------
       100
    (1 row) 

查看另一个 datanode2 中 repltab 表结果

    [postgres@datanode2 pgxl9.5]$ psql -p 5433
    psql (PGXL 9.5r1.3, based on PG 9.5.4 (Postgres-XL 9.5r1.3))
    Type "help" for help.

    postgres=# SELECT count(*) FROM repltab;
     count 
    -------
       100
    (1 row) 

结论：REPLICATION 表中，datanode1,datanode2 中表是全部数据，一模一样。而 DISTRIBUTE 表，数据散落近乎平均分配到了 datanode1,datanode2 节点中。

## 6.2 新增 datanode 节点与数据重分布

### 6.2.1 新增 datanode 节点

在 gtm 集群管理节点上执行 pgxc_ctl 命令

    [postgres@gtm ~]$ pgxc_ctl
    /bin/bash
    Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
    Installing pgxc_ctl_bash script as /home/postgres/pgxc_ctl/pgxc_ctl_bash.
    Reading configuration using /home/postgres/pgxc_ctl/pgxc_ctl_bash --home /home/postgres/pgxc_ctl --configuration /home/postgres/pgxc_ctl/pgxc_ctl.conf
    Finished reading configuration.
       ******** PGXC_CTL START ***************

    Current directory: /home/postgres/pgxc_ctl
    PGXC 

在 PGXC 后面执行新增数据节点命令:

    Current directory: /home/postgres/pgxc_ctl
    # 在服务器datanode1上，新增一个master角色的datanode节点，名称是dn3
    # 端口号暂定5430，pool master暂定6669 ，指定好数据目录位置，从两个节点升级到3个节点，之后要写3个none
    # none应该是datanodeSpecificExtraConfig或者datanodeSpecificExtraPgHba配置
    PGXC add datanode master dn3 datanode1 5430 6669 /home/postgres/pgxl9.5/data/nodes/dn_master3 none none none 

等待新增完成后，查询集群节点状态：

    [postgres@gtm ~]$ psql -h datanode1 -p 5432 -U postgres
    psql (PGXL 9.5r1.3, based on PG 9.5.4 (Postgres-XL 9.5r1.3))
    Type "help" for help.

    postgres=
     node_name | node_type | node_port | node_host | nodeis_primary | nodeis_preferred |   node_id   
    -----------+-----------+-----------+-----------+----------------+------------------+-------------
     coord1    | C         |      5432 | datanode1 | f              | f                |  1885696643
     coord2    | C         |      5432 | datanode2 | f              | f                | -1197102633
     node1     | D         |      5433 | datanode1 | f              | t                |  1148549230
     node2     | D         |      5433 | datanode2 | f              | f                |  -927910690
     dn3       | D         |      5430 | datanode1 | f              | f                |  -700122826
    (5 rows) 

可以发现节点新增完毕。

### 6.2.2 数据重分布

之前我们的 DISTRIBUTE 表分布在了 node1,node2 节点上，如下：

    postgres=
     xc_node_id | count 
    ------------+-------
     1148549230 |    42
     -927910690 |    58
    (2 rows) 

新增一个节点后，将 sharding 表数据重新分配到三个节点上，将 repl 表复制到新节点：

    # 重分布sharding表
    postgres=# ALTER TABLE disttab ADD NODE (dn3);
    ALTER TABLE
    # 复制数据到新节点
    postgres=#  ALTER TABLE repltab ADD NODE (dn3);
    ALTER TABLE 

查看新的数据分布：

    postgres=
     xc_node_id | count 
    ------------+-------
     -700122826 |    36
     -927910690 |    32
     1148549230 |    32
    (3 rows) 

登录 dn3(新增的时候，放在了 datanode1 服务器上，端口 5430) 节点查看数据：

    [postgres@gtm ~]$ psql -h datanode1 -p 5430 -U postgres
    psql (PGXL 9.5r1.3, based on PG 9.5.4 (Postgres-XL 9.5r1.3))
    Type "help" for help.
    postgres=# select count(*) from repltab;
     count 
    -------
       100
    (1 row) 

很明显, 通过 ALTER TABLE tt ADD NODE (dn) 命令，可以将 DISTRIBUTE 表数据重新分布到新节点，重分布过程中会中断所有事务。可以将 REPLICATION 表数据复制到新节点。

### 6.2.3 从 datanode 节点中回收数据

    postgres=# ALTER TABLE disttab DELETE NODE (dn3);
    ALTER TABLE
    postgres=# ALTER TABLE repltab DELETE NODE (dn3);
    ALTER TABLE 

## 6.3 删除数据节点

Postgresql-XL 并没有检查将被删除的 datanode 节点是否有 replicated/distributed 表的数据，为了数据安全，在删除之前需要检查下被删除节点上的数据，有数据的话，要回收掉分配到其他节点，然后才能安全删除。删除数据节点分为四步骤：

-   查询要删除节点 dn3 的 oid


    postgres=
      oid  | node_name | node_type | node_port | node_host | nodeis_primary | nodeis_preferred |   node_id   
    -------+-----------+-----------+-----------+-----------+----------------+------------------+-------------
     11819 | coord1    | C         |      5432 | datanode1 | f              | f                |  1885696643
     16384 | coord2    | C         |      5432 | datanode2 | f              | f                | -1197102633
     16385 | node1     | D         |      5433 | datanode1 | f              | t                |  1148549230
     16386 | node2     | D         |      5433 | datanode2 | f              | f                |  -927910690
     16397 | dn3       | D         |      5430 | datanode1 | f              | f                |  -700122826
    (5 rows) 

-   查询 dn3 对应的 oid 中是否有数据


    testdb=
     pcrelid | pclocatortype | pcattnum | pchashalgorithm | pchashbuckets |     nodeoids      
    ---------+---------------+----------+-----------------+---------------+-------------------
       16388 | H             |        1 |               1 |          4096 | 16397 16385 16386
       16394 | R             |        0 |               0 |             0 | 16397 16385 16386
    (2 rows) 

-   有数据的先回收数据


    postgres=
    ALTER TABLE
    postgres=
    ALTER TABLE
    postgres=
     pcrelid | pclocatortype | pcattnum | pchashalgorithm | pchashbuckets | nodeoids 
    ---------+---------------+----------+-----------------+---------------+----------
    (0 rows) 

-   安全删除 dn3


    PGXC$  remove datanode master dn3 clean 

## 6.4 coordinate 节点管理

同 datanode 节点相似，列出语句不做测试了：

     PGXC$  add coordinator master coord3 localhost 30003 30013 $dataDirRoot/coord_master.3 none none none

    PGXC$  remove coordinator master coord3 clean 

## 6.5 故障切换

-   查看当前数据集群


    postgres=
      oid  | node_name | node_type | node_port | node_host | nodeis_primary | nodeis_preferred |   node_id   
    -------+-----------+-----------+-----------+-----------+----------------+------------------+-------------
     11819 | coord1    | C         |      5432 | datanode1 | f              | f                |  1885696643
     16384 | coord2    | C         |      5432 | datanode2 | f              | f                | -1197102633
     16385 | node1     | D         |      5433 | datanode1 | f              | t                |  1148549230
     16386 | node2     | D         |      5433 | datanode2 | f              | f                |  -927910690
    (4 rows) 

-   模拟 node1 节点故障


    PGXC$  stop -m immediate datanode master node1
    Stopping datanode master node1.
    Done. 

-   测试集群查询


    postgres=
    ERROR:  Failed to get pooled connections
    postgres=
     xc_node_id | col1 | col2 | col3 
    ------------+------+------+------
     -927910690 |    3 |  103 | foo
    (1 row) 

测试发现，查询范围如果涉及到故障的 node1 节点，会报错，而查询的数据范围不在 node1 上的话，仍然可以查询。

-   手动切换 node1 的 slave


    PGXC$  failover datanode node1

    postgres=
      oid  | node_name | node_type | node_port | node_host | nodeis_primary | nodeis_preferred |   node_id   
    -------+-----------+-----------+-----------+-----------+----------------+------------------+-------------
     11819 | coord1    | C         |      5432 | datanode1 | f              | f                |  1885696643
     16384 | coord2    | C         |      5432 | datanode2 | f              | f                | -1197102633
     16386 | node2     | D         |      5433 | datanode2 | f              | f                |  -927910690
     16385 | node1     | D         |     15433 | datanode2 | f              | t                |  1148549230
    (4 rows) 

发现 node1 节点的 ip 和端口都已经替换为配置的 slave 了。

在配置的时候一定要细心，避免端口号之类的配置冲突等错误。  
错误一：

    postgres=# create table test1(id integer,name varchar(20));
    LOG:  failed to connect to node, connection string (host=192.168.0.125 port=1925 dbname=postgres user=postgres application_name=pgxc sslmode=disable options='-c remotetype=coordinator -c parentnode=coord1  -c DateStyle=iso,mdy -c timezone=prc -c geqo=on -c intervalstyle=postgres -c lc_monetary=C'), connection error (fe_sendauth: no password supplied
            )
    WARNING:  can not connect to node 16384
    WARNING:  Health map updated to reflect DOWN node (16384)
    LOG:  Pooler could not open a connection to node 16384
    LOG:  failed to acquire connections
    STATEMENT:  create table test1(id integer,name varchar(20));
    ERROR:  Failed to get pooled connections
    HINT:  This may happen because one or more nodes are currently unreachable, either because of node or network failure.
             Its also possible that the target node may have hit the connection limit or the pooler is configured with low connections.
             Please check if all nodes are running fine and also review max_connections and max_pool_size configuration parameters
    STATEMENT:  create table test1(id integer,name varchar(20)); 

原因：这个是由于某些环境或配置出了问题，我的就是 pg_hba.conf 配置出了问题，Ipv4 要改成 0:0:0:0/0 trust 才行。  
但这仅仅是一个问题，开发者搭建环境遇到这个错误，一定要检查如下:

-   \*\* 各个机器的防火墙是否关闭？\*\*
-   \*\* 各个机器的 SELINUX 状态是否是 disabled？\*\*
-   \*\* 各个机器的 ssh 免密登录是否成功？\*\*
-   \*\* 各个节点的 pg_hba.conf,postgresql.conf 是否配置为信任登录？是否有 IP 限制？\*\*
-   **超过某些节点的最大连接数？（对于我们测试环境来说，肯定不会是这个问题）**

作者搭建 pgxl 是为地理大数据做技术预研的，使用 postgis 作为空间数据，欢迎 postgis 开发者参与交流。

更多精彩内容，就在简书 APP

"小礼物走一走，来简书关注我"

[![](https://cdn2.jianshu.io/assets/default_avatar/2-9636b13945b9ccf345bc98d0d81074eb.jpg)
](https://www.jianshu.com/u/ce4d7c510323)![](http://wx.qlogo.cn/mmopen/EcOKiaZLsUY9nSMv21nG3SYlJM7d6mYPfkfeC4LKeNM7tyGWsIfh3INTZOyA7E1uf2Y5Ig0G7K4Ek17FicjUzovn2tLX5hJ0le/0)
共 2 人赞赏

[![](https://upload.jianshu.io/users/upload_avatars/68979/ade680e57161?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)
](https://www.jianshu.com/u/2cbf98ef6104)

[遥想公瑾当年](https://www.jianshu.com/u/2cbf98ef6104 "遥想公瑾当年")专注于 OpenSource GIS 开发，从事开源 gis 开发与架构设计。<br>1 相关专业开发...

总资产 193 共写了 4.4W 字获得 619 个赞共 856 个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

-   1. 数据库安装与配置步骤 安装环境准备操作系统： Oracle Linux Server 6.5IP 地址...

    [![](https://upload-images.jianshu.io/upload_images/2066703-d571c3306afe56cf.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/ba02513d4a24)
-   说明：以下命令前面的 "./" 是在 root 用户下调用 oracle 查询信息才使用，如果在 oracle 或者 grid 用户下...

    [![](https://cdn2.jianshu.io/assets/default_avatar/15-a7ac401939dd4df837e3bbf82abaa2a8.jpg)
    十野早望](https://www.jianshu.com/u/2280f0d2f234)阅读 2,114 评论 0 赞 0


-   Hadoop 部署方式 本地模式 伪分布模式 (在一台机器中模拟，让所有进程在一台机器上运行) 集群模式 服务器只是一...

    [![](https://upload.jianshu.io/users/upload_avatars/1936544/b9f274c61823.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)
    陈半仙儿](https://www.jianshu.com/u/cc5920f691e0)阅读 1,161 评论 0 赞 9

    [![](https://upload-images.jianshu.io/upload_images/1936544-3ead3f0ff522317c?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/5a360bc38a5a)
-   匆匆離開了夢想所在的城市，來不及告別，也不想告別，因爲我知道我終有一天要回去。 我生活在一個郊區小鎮，生活設施算是...

    [![](https://upload-images.jianshu.io/upload_images/3227390-af063efb256aab2a.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/3bbef29c46c8)
-   今日出差，意料中的早起，宁静的早晨，随手拍几张照片，心底难自禁的涌现着美好。早起，不堵车，不匆忙，多了一丝淡定和从...

    [![](https://upload-images.jianshu.io/upload_images/4893014-e6653a3526bfcfda.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)
    ](https://www.jianshu.com/p/c129247203ab)
-   关于开发新客户 1. 和利时 2. 警视达 3. 星网锐捷 ## 测试标题


-   雪九凤琳的作业还没有做完，一会就要交了。哎！怎么办呀，我的作业还没有写完老师肯定要批评我了。 怎么办呢？雪九凤... 
    [https://www.jianshu.com/p/82aaf352b772](https://www.jianshu.com/p/82aaf352b772)
