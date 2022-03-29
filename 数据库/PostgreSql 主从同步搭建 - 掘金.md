# PostgreSql 主从同步搭建 - 掘金
这是我参与 8 月更文挑战的第 24 天，活动详情查看：[8 月更文挑战](https://juejin.cn/post/6987962113788493831 "https&#x3A;//juejin.cn/post/6987962113788493831")

### 环境

#### 操作系统：

CentOS Linux release 7.6.1810 (Core)

#### 数据库版本：

PostgreSQL 12.4

#### IP:

192.168.100.170 主库  
192.168.100.202 从库

### 实施步骤

#### 主库创建账号同步数据

    postgres=# CREATE ROLE replica login replication encrypted password 'replica';
    CREATE ROLE
    复制代码

#### 主库 pg_hba.conf 文件增加备库访问控制

    host    replication     replica         192.168.100.202/32      trust
    复制代码

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b9ce9124b844e25a36722ff0eccd634~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

#### 主库 postgresql.conf 文件添加主从同步参数

    wal_level = hot_standby 
    max_wal_senders = 8 
    wal_keep_segments = 64 
    wal_sender_timeout = 60s
    max_connections = 100 
    复制代码

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ab8dd4ae566475ab9bf9400e9609692~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

#### 主库重启

    [postgres@yuan data]$ pg_ctl restart -D $PGDATA -l $PGLOG
    waiting for server to shut down.... done
    server stopped
    waiting for server to start.... done
    server started
    复制代码

#### 从库验证可访问主库

返回输入密码即表示可访问

    [postgres@dj ~]$ psql -h 192.168.100.170 -U postgres
    Password for user postgres:
    复制代码

#### 停止从库

    [postgres@dj ~]$ pg_ctl stop -D $PGDATA -l $PGLOG
    waiting for server to shut down.... done
    server stopped
    复制代码

#### 清空从库数据文件

    [postgres@dj data]$ rm -rf  /app/pgsql/data/*
    [postgres@dj data]$ ll
    total 0
    复制代码

#### 拉取主库数据文件

    [postgres@dj data]$ pg_basebackup -h 192.168.100.170 -D /app/pgsql/data -p 5432 -U replica -Fp -Xs -Pv -R --checkpoint=fast
    Password:
    pg_basebackup: initiating base backup, waiting for checkpoint to complete
    pg_basebackup: checkpoint completed
    pg_basebackup: write-ahead log start point: 0/D000028 on timeline 1
    pg_basebackup: starting background WAL receiver
    pg_basebackup: created temporary replication slot "pg_basebackup_17064"
    50729/50729 kB (100%), 1/1 tablespace
    pg_basebackup: write-ahead log end point: 0/D000100
    pg_basebackup: waiting for background process to finish streaming ...
    pg_basebackup: syncing data to disk ...
    pg_basebackup: base backup completed
    [postgres@dj data]$ ll
    total 128
    -rw-------. 1 postgres postgres   224 Jul 12 03:43 backup_label
    drwx------. 7 postgres postgres  4096 Jul 12 03:43 base
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 global
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_commit_ts
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_dynshmem
    -rw-------. 1 postgres postgres  4886 Jul 12 03:43 pg_hba.conf
    -rw-------. 1 postgres postgres  1636 Jul 12 03:43 pg_ident.conf
    drwx------. 4 postgres postgres  4096 Jul 12 03:43 pg_logical
    drwx------. 4 postgres postgres  4096 Jul 12 03:43 pg_multixact
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_notify
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_replslot
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_serial
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_snapshots
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_stat
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_stat_tmp
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_subtrans
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_tblspc
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_twophase
    -rw-------. 1 postgres postgres     3 Jul 12 03:43 PG_VERSION
    drwx------. 3 postgres postgres  4096 Jul 12 03:43 pg_wal
    drwx------. 5 postgres postgres  4096 Jul 12 03:43 pg_walminer
    drwx------. 2 postgres postgres  4096 Jul 12 03:43 pg_xact
    -rw-------. 1 postgres postgres   267 Jul 12 03:43 postgresql.auto.conf
    -rw-------. 1 postgres postgres 27115 Jul 12 03:43 postgresql.conf
    -rw-------. 1 postgres postgres    30 Jul 12 03:43 postmaster.opts.bak
    -rw-------. 1 postgres postgres     0 Jul 12 03:43 standby.signal
    复制代码

#### 从库 postgresql.conf 文件修改主从同步参数

删掉主库添加的同步参数，添加如下参数：

    primary_conninfo = 'host=192.168.100.170 port=5432 user=replica password=replica'
    recovery_target_timeline = latest 
    hot_standby = on
    max_standby_streaming_delay = 30s
    wal_receiver_status_interval = 10s
    hot_standby_feedback = on
    max_connections = 200  #大于主节点
    max_worker_processes = 20
    复制代码

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6678985f0af48728340c673d666e3ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

#### 启动从库

    [postgres@dj data]$ pg_ctl start -D $PGDATA -l $PGLOG
    waiting for server to start.... done
    server started
    复制代码

### 主从同步验证

    --主库查询
    postgres=# select client_addr,usename,backend_start,application_name,sync_state,sync_priority FROM pg_stat_replication;
       client_addr   | usename |         backend_start         | application_name | sync_state | sync_priority
    -----------------+---------+-------------------------------+------------------+------------+---------------
     192.168.100.202 | replica | 2021-08-24 18:03:32.089937+08 | walreceiver      | async      |             0


    --测试创建删除数据库观察从库是否同步
    create database test;
    drop database test;
    复制代码

 [https://juejin.cn/post/6999935606738403342](https://juejin.cn/post/6999935606738403342)
