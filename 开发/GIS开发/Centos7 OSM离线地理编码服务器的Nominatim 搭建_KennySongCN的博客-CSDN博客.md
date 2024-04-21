# Centos7 OSM离线地理编码服务器的Nominatim 搭建_KennySongCN的博客-CSDN博客
> 提示：文章写完后，目录可以自动生成，如何生成可参考右边的帮助文档

#### 文章目录

*   [一、Nominatim是什么？](#Nominatim_8)
*   [二、软硬件需求](#_11)
*   *   [1、硬件](#1_12)
    *   [2、软件版本](#2_14)
*   [三、安装](#_23)
*   *   [1、更新软件包](#1_24)
    *   [2、安装postgresql](#2postgresql_29)
    *   [3、安装postgis](#3postgis_35)
    *   [4、安装php](#4php_41)
    *   [4、安装其他支撑库](#4_49)
    *   [5、查看版本](#5_60)
*   [四、系统设置和配置](#_68)
*   [1、PostgreSQL数据库设置](#1PostgreSQL_72)

*   *   [2、系统设置和配置](#2_109)
*   [五、安装Nominatim](#Nominatim_120)
*   *   [1、下载安装环境](#1_121)
    *   [2、下载安装nominatim](#2nominatim_133)
    *   [3、设置Apache Webserver](#3Apache_Webserver_150)
    *   [4、添加SELinux安全设置](#4SELinux_169)
*   [总结](#_240)

* * *

一、Nominatim是什么？
---------------

Nominatim是一个可以按名称和地址来搜索[OSM](https://so.csdn.net/so/search?q=OSM&spm=1001.2101.3001.7020)中的数据，并生成OSM点的合成地址的工具（反向地理编码）。可用在http://nominatim.openstreetmap.org找到这个工具。Nominatim也用在OpenStreetMap首页的搜索工具栏中，同时也为MapQuest Open Initiative, PickPoint, OpenCage Geocoder, LocationIQ, Geocoding.ai 提供搜索支持。

二、软硬件需求
-------

### 1、硬件

至少需要 2GB 的 RAM，否则安装将失败。对于一个完整的版本，强烈建议使用32GB 或更多的 RAM。对于完整的安装，您将需要大约 500GB 的硬盘空间（截至 2016 年 6 月，考虑到 OSM 数据库正在快速增长）。SSD 磁盘将大大有助于加快导入和查询。

### 2、软件版本

PostgreSQL（9.1 或更高版本）  
PostGIS（2.0 或更高版本）  
PHP（5.4 或更高版本）  
PHP-pgsql  
PHP-intl（与 PHP 捆绑）  
网络服务器（推荐使用 apache 或 nginx）

三、安装
----

### 1、更新软件包

```
`sudo yum update -y
sudo yum install -y epel-release` 

*   1
*   2


```

### 2、安装postgresql

```
`sudo yum install postgresql-server
sudo yum install postgresql-contrib 
sudo yum install postgresql-devel` 

*   1
*   2
*   3


```

### 3、安装postgis

```
`sudo yum install postgis
sudo yum install postgis-utils
sudo yum install postgresql-devel` 

*   1
*   2
*   3


```

### 4、安装php

```
`sudo yum install php
sudo yum install php-pgsql
sudo yum install php-pear
sudo yum install php-pear-DB
sudo yum install php-intl` 

*   1
*   2
*   3
*   4
*   5


```

### 4、安装其他支撑库

```
`sudo yum install libpqxx-devel
sudo yum install proj-epsg bzip2-devel
sudo yum install proj-devel
sudo yum install geos-devel
sudo yum install libxml2-devel
sudo yum install boost-devel
sudo yum install expat-devel
sudo yum install zlib-devel` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8


```

### 5、查看版本

#查看PHP版本  
php -v  
#查看PostgreSQL版本  
psql --version  
#查看PostGIS版本  
rpm -qa | grep postgis

四、系统设置和配置
---------

Nominatim 将作为全局服务在您的机器上运行。因此，最好将其安装在自己单独的用户帐户下。在下面我们假设这个用户被调用nominatim并且安装将在/srv/nominatim. 创建用户和目录，本文章使用root用户进行演示。

* * *

1、[PostgreSQL数据库](https://so.csdn.net/so/search?q=PostgreSQL%E6%95%B0%E6%8D%AE%E5%BA%93&spm=1001.2101.3001.7020)设置
------------------------------------------------------------------------------------------------------------------

```
`# 1、 初始化
sudo postgresql-setup initdb
sudo systemctl enable postgresql

#2、修改PostgreSQL默认配置
vim /var/lib/pgsql/data/postgresql.conf

#添加或修改如下内容:
fsync = off
full_page_writes = off
shared_buffers =2GB
maintenance_work_mem =10GB
work_mem = 50MB
effective_cache_size =24GB
synchronous_commit = off
checkpoint_segments = 100 # only for postgresql <= 9.4
checkpoint_timeout = 10min
checkpoint_completion_target = 0.9

#3、重启PostgreSQL数据库
sudo systemctl restart postgresql

#4、创建三个PostgreSQL数据库用户：
#一个用于导入的用户；
#另一个用于访问数据库仅用于阅读的网络服务器：
#另一个用于作为PostgreSQL数据库角色的网站用户www-data
sudo -u postgres createuser -s $USERNAME
sudo -u postgres createuser apache
sudo -u postgres createuser -SDR www-data` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29


```

> 不要忘记fsync full\_page\_writes在初始导入后重新启用，否则您将面临数据库损坏的风险。不能关闭  
> Autovacuum，因为它可以确保经常分析表。

### 2、系统设置和配置

```
`#创建账号并设置目录
sudo useradd -d /srv/nominatim -s /bin/bash -m nominatim
export USERNAME=nominatim
export USERHOME=/srv/nominatim
#Make sure that system servers can read from the home directory:
chmod a+x $USERHOME` 

*   1
*   2
*   3
*   4
*   5
*   6


```

五、安装Nominatim
-------------

### 1、下载安装环境

```
`#安装gcc等必备程序包（已安装则略过此步）
yum install -y gcc gcc-c++ make automake 
#获取CMake源码包
wget https://github.com/Kitware/CMake/releases/download/v3.15.5/cmake-3.15.5.tar.gz
#解压CMake源码包
tar -zxvf cmake-3.15.5.tar.gz
cd cmake-3.15.5
#编译安装
./bootstrap && make -j4 && sudo make install` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

### 2、下载安装nominatim

```
`#进入/srv/nominatim目录，获取源码
cd $USERHOME
git clone --recursive git://github.com/openstreetmap/Nominatim.git
cd Nominatim
#When installing the latest source from github, you also need to download the country grid:
wget -O data/country_osm_grid.sql.gz http://www.nominatim.org/data/country_grid.sql.gz
#The code must be built in a separate directory.
mkdir build
cd build
cmake $USERHOME/Nominatim
make

#创建一个最小的配置文件，告诉 nominatim 您的网络服务器用户的名称和网站的 URL：` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13


```

### 3、设置Apache Webserver

您需要在 apache 配置中创建网站目录的别名。将单独的 nominatim 配置添加到您的网络服务器：

```
 `sudo tee /etc/httpd/conf.d/nominatim.conf << EOFAPACHECONF
<Directory "$USERHOME/nominatim-project/website">
  Options FollowSymLinks MultiViews
  AddType text/html   .php
  DirectoryIndex search.php
  Require all granted
</Directory>

Alias /nominatim $USERHOME/nominatim-project/website
EOFAPACHECONF

#重启Apache服务器
sudo systemctl enable httpd
sudo systemctl restart httpd` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15


```

### 4、添加SELinux安全设置

让 SELinux 保持启用和强制执行是一个好主意，尤其是在可从 Internet 访问的 Web 服务器的情况下。至少应为 Nominatim 做以下 SELinux 标签：

使用yum查询选项搜索  
yum provides semanage  
按照搜索结果安装对应包即可解决  
yum install -y policycoreutils-python  
![](https://img-blog.csdnimg.cn/cf2a21ecea954c67aea4be3bbb7fd110.png)

```
`sudo semanage fcontext -a -t httpd_sys_content_t "/usr/local/nominatim/lib/lib-php(/.*)?"
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/nominatim/nominatim-project/website(/.*)?"
sudo semanage fcontext -a -t lib_t "/srv/nominatim/nominatim-project/module/nominatim.so"
sudo restorecon -R -v /usr/local/lib/nominatim
sudo restorecon -R -v /srv/nominatim/nominatim-project

sudo semanage fcontext -a -t httpd_sys_content_t "$USERHOME/Nominatim/(website|lib|settings)(/.*)?"
sudo semanage fcontext -a -t lib_t "$USERHOME/Nominatim/module/nominatim.so"
sudo restorecon -R -v $USERHOME/Nominatim` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9


```

注意事项：  
问题1：  
CMake Error at /usr/local/share/cmake-3.15/Modules/FindPackageHandleStandardArgs.cmake:137 (message):  
Could NOT find PythonInterp: Found unsuitable version “2.7.5”, but required  
is at least “3.6” (found /usr/bin/python)

解决方案：升级Python

```
`yum groupinstall ‘Development Tools’
yum install centos-release-scl
yum install rh-python36
python --version
scl enable rh-python36 bash
yum -y install python-pip
#创建软连接
 mv /usr/bin/python /usr/bin/python_old
 ln -s /opt/rh/rh-python36/root/bin/python3 /usr/bin/python
 #修改yum指向
 vi /usr/bin/yum
 vi /usr/libexec/urlgrabber-ext-down
将第一个行#!/usr/bin/python” 改为 “#!/usr/bin/python_old”即可。` 



```

问题2：  
Target “osm2pgsql” requires the language dialect “CXX17” , but CMake does  
not know the compile flags to use to enable it.  
解决方案：升级gcc

```
`1、安装centos-release-scl
sudo yum install centos-release-scl
2、安装devtoolset，注意，如果想安装7.*版本的，就改成devtoolset-7-gcc*，以此类推
sudo yum install devtoolset-8-gcc*

3、激活对应的devtoolset，所以你可以一次安装多个版本的devtoolset，需要的时候用下面这条命令切换到对应的版本
scl enable devtoolset-8 bash

每个版本的目录下面都有个 enable 文件，如果需要启用某个版本，只需要执行
source ./enable
所以要想切换到某个版本，只需要执行
source /opt/rh/devtoolset-8/enable

可以将对应版本的切换命令写个shell文件放在配了环境变量的目录下，需要时随时切换，或者开机自启
长期有效
ln -s /opt/rh/devtoolset-7/root/bin/gcc /usr/bin/gcc
ln -s /opt/rh/devtoolset-7/root/bin/g++ /usr/bin/g++` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)



```
