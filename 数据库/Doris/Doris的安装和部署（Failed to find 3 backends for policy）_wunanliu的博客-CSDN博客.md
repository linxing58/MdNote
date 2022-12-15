# Doris的安装和部署（Failed to find 3 backends for policy）_wunanliu的博客-CSDN博客
> **一、硬件环境准备**

          Doris作为开源MPP架构OLAP数据库，可以运行在大部分主流商业服务器上。为了充分利用 MPP 架构的并发优势和 Doris 的高可用特性，我们建议 Doris 的部署遵循以下要求：

**硬件环境**

| Linux System | Version |
| --- | --- |
| Centos | 7.1 and above |
| Ubuntu | 16.04 and above |

**软件环境**[​](https://doris.incubator.apache.org/docs/install/install-deploy#software-requirements "​")

| Soft | Version |
| --- | --- |
| Java | 1.8 and above |
| GCC | 

4.8.2 and above（Linux系统已经带了）

 |
| Doris | 1.1.1 |

当前Doris的下载安装包，由于挂在apache官网上，官网地址CDN在境外，网络下载速度极慢，可以使用挂载VPN下载，速度会极快，本博主已经下载完毕上传在百度云盘，地址为：

**Doris1.1.1 下载网盘地址：** 

> 链接：https://pan.baidu.com/s/1ciD0f8YE5YxLkAcjFhoLtw   
> 提取码：boyp

**JDK1.8 下载网盘下载地址：** 

> 链接：https://pan.baidu.com/s/1lsQPkl41oroR60NN0nMvNw   
> 提取码：yzjw

> **二、组件分配**

| 服务器IP         | 服务组件 | 备注说明（测试环境） |
| 192.168.1.1 | FE，BE | 测试环境，所以FE和BE在一台服务器 |
| 192.168.1.2 | BE | 多台FE组件切勿放在，会存在问题         |
| 192.168.1.3 | BE | 同时建设表的时候，OLAP引擎时候，是最少需要3台FE组件 |

> **重点说明**：（如果只有一个节点，FE和BE只在一台服务器，单节点，在建表OLAP时候，会出现）ERROR 1105 (HY000): errCode = 2, detailMessage = Failed to find 3 backends for policy: cluster|query|load|schedule|tags|medium: default\_cluster|false|false|true|\[{"location" : "default"}\]|HDD

> **三、Linxu 系统环境设置（所有服务器）**

```null
vi /etc/security/limits.conf
```

重启服务器：reboot  
查           看：ulimit -n     (如果显示65536即为正确)

> **三、Linxu 系统环境设置（时钟同步|所有服务器）**

       Doris 的元数据要求时间精度小于 5000ms，所以集群中的所有机器都需要同步时钟，避免时钟问题导致元数据不一致导致服务异常。说明：分布式，对服务器之间时间由很好要求，最好安装Ntpd服务）

> **四、Linxu 系统环境设置（关闭交换分区（swap ）所有服务器）**

Linux swap分区会对Doris造成严重的性能问题，安装前需要禁用  
swap分区      :    **swapoff -a**  
查看是否关闭：  **free -h**

> **五、硬件环境要求**
> 
> ![](https://img-blog.csdnimg.cn/afe320c257024e10a0bae5c70555b1e8.png)

**解压Doris 压缩包(重命名)**

```null
tar -zxvf apache-doris-1.1.1-bin-x86.tar.gzmv apache-doris-1.1.1-bin-x86.tar.gz doris
```

查看服务器Ip Addr 将对应IP地址 填写在**Fe.conf**配置文件，去掉注释添加下列配置

**`priority_networks=192.168.1.1/24`**

> 由于存在多个网卡，或者安装docker等环境导致的虚拟网卡存在，同一主机可能有多个不同的ip。目前 Doris 不会自动识别可用 IP。所以当你在部署主机上遇到多个IP时，必须通过priority\_networks配置项强制指定正确的IP 。

将上述解压包，拷贝到对应的Be服务器，只修改**Be.conf**,将服务器对应IP地址修改在  
**priority\_networks=192.168.1.2/24**

**其余配置文件，元数据存储，实际数据存储测试环境先默认不书写。** 

> **运行Fe服务**
> 
> ```null
> cd /Doris/apache-doris-1.1.1-bin-x86/fe/bin
> ```
> 
> **运行Be服务,先需要将Be注册到Fe，在Fe服务器解压，此处用到Mysql-Client5.7**
> 
> ```null
> tar Mysql-5.7.1xxxxx.tar.gz
> ```
> 
> 使用Mysql-Client登录（默认无密码）
> 
> ```null
> /usr/local/mysql/bin/mysql -h 192.168.1.1 -P 9030 -uroot
> ```
> 
> 将BE的IP地址和端口添加注册到FE
> 
> ```null
> ALTER SYSTEM ADD BACKEND "192.168.1.1:9050";ALTER SYSTEM ADD BACKEND "192.168.1.2:9050";ALTER SYSTEM ADD BACKEND "192.168.1.3:9050";
> ```

>  **运行Be服务**
> 
> ```null
> cd /Doris/apache-doris-1.1.1-bin-x86/be/bin/
> ```

**登录WEB页面**查看FE运行情况，同时可以看到日志  
Fe的Ip:8030 用户名：root 密码没修改，默认为空（密码可以修改，生产环境建议修改，包括Root密码）

![](https://img-blog.csdnimg.cn/b3a1186e79f548868515f56efa3d6d27.jpeg)

**内部登录界面：** 

 ![](https://img-blog.csdnimg.cn/ec177731afbb4a9ca8dadec9d7fe2c43.jpeg)

> 修改 admin 和 root 用户密码  
> 因为web界面的admin用户和 数据库root都没有密码，所以需要设置一下，方式都相同  
> SET PASSWORD FOR 'admin' = PASSWORD('666666');  
> SET PASSWORD FOR 'root' = PASSWORD('666666');  
> \# 这样访问web界面和数据库登录时都需要输入密码了

**登录数据库查看FE和BE对应的状态**

![](https://img-blog.csdnimg.cn/824cc8dccc2f4a949938dd1d39371414.jpeg)

 使用mysql > Help create table; 获取操作指南，数据库建表语句。![](https://img-blog.csdnimg.cn/5030b2833ae94435aa43eca379b2d026.jpeg)

**上述异常为，没找到三个节点，节点异常，所以需要修改。各节点详细信息查看对应节点的错误日志**

```null
| :----- | :----------------------------------------------------------- || 1005   | 创建表格失败，在返回错误信息中给出具体原因                   || 1007   | 数据库已经存在，不能创建同名的数据库                         || 1044   | 数据库对用户未授权，不能访问                                 || 1045   | 用户名及密码不匹配，不能访问系统                             || 1052   | 指定的列名有歧义，不能唯一确定对应列                         || 1053   | 为Semi-Join/Anti-Join查询指定了非法的数据列                  || 1058   | 查询语句中选择的列数目与查询结果的列数目不一致               || 1064   | 没有存活的Backend节点                                        || 1066   | 查询语句中出现了重复的表别名                                 || 1095   | 非线程的拥有者不能终止线程的运行                             || 1096   | 查询语句没有指定要查询或操作的数据表                         || 1111   | 在Where从句中非法使用聚合函数                                || 1130   | 客户端使用了未被授权的IP地址来访问系统                       || 1141   | 撤销用户权限时指定了用户不具备的权限                         || 1203   | 用户使用的活跃连接数超过了限制                               || 1227   | 拒绝访问，用户执行了无权限的操作                             || 1228   | 会话变量不能通过SET GLOBAL指令来修改                         || 1229   | 全局变量应通过SET GLOBAL指令来修改                           || 1232   | 给某系统变量设置了错误数据类型的值                           || 1251   | 客户端不支持服务器请求的身份验证协议；请升级MySQL客户端      || 1353   | SELECT和视图的字段列表具有不同的列数                         || 1364   | 字段不允许NULL值，但是没有设置缺省值                         || 1507   | 删除不存在的分区，且没有指定如果存在才删除的条件             || 1508   | 无法删除所有分区，请改用DROP TABLE                           || 1748   | 不能将数据插入具有空分区的表中使用“ SHOW PARTITIONS FROM  tbl”来查看此表的当前分区 || 5003   | Key列应排在Value列之前                                       || 5008   | 数据插入提示：仅适用于有分区的数据表                         || 5009   | PARTITION子句对于INSERT到未分区表中无效                      || 5010   | 列数不等于SELECT语句的选择列表数                             || 5026   | 通过SELECT语句创建表时使用了不支持的数据类型                 || 5037   | 删除集群之前，必须删除集群中的所有数据库                     || 5037   | 集群中不存在这个ID的BE节点                                   || 5051   | 应先将源数据库连接到目标数据库                               || 5052   | 集群内部错误：BE节点错误信息                                 || 5053   | 没有从源数据库到目标数据库的迁移任务                         || 5054   | 指定数据库已经连接到目标数据库，或正在迁移数据               || 5055   | 数据连接或者数据迁移不能在同一集群内执行                     || 5056   | 不能删除数据库：它被关联至其它数据库或正在迁移数据           || 5056   | 不能重命名数据库：它被关联至其它数据库或正在迁移数据         || 5056   | 集群内已存在指定数目的BE节点                                 || 5059   | 集群中存在处于下线状态的BE节点                               || 5062   | 不正确的群集名称（名称'default_cluster'是保留名称）          || 5063   | Colocate功能已被管理员禁用                                   || 5063   | colocate数据表不存在                                         || 5063   | Colocate表必须是OLAP表                                       || 5063   | Colocate表应该具有同样的副本数目                             || 5063   | Colocate表应该具有同样的分桶数目                             || 5063   | Colocate表的分区列数目必须一致                               || 5063   | Colocate表的分区列的数据类型必须一致                         || 5064   | 指定表不是colocate表                                         || 5065   | 指定的时间单位是非法的，正确的单位包括：HOUR / DAY / WEEK / MONTH || 5066   | 动态分区起始值不是有效的数字                                 || 5066   | 动态分区结束值不是有效的数字                                 || 5067   | 动态分区分桶值不是有效的数字                                 || 5068   | 是否允许动态分区的值不是有效的布尔值：true或者false          || 5069   | 指定的动态分区名前缀是非法的                                 || 5072   | 动态分区副本值不是有效的数字                                 || 5074   | 创建历史动态分区参数：create_history_partition无效，期望的是：true或者false |
```

官方文档：[https://doris.incubator.apache.org/docs/install/install-deploy](https://doris.incubator.apache.org/docs/install/install-deploy "https://doris.incubator.apache.org/docs/install/install-deploy")