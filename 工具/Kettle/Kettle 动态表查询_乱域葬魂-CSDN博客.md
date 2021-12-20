# Kettle 动态表查询_乱域葬魂-CSDN博客
前言

> 需求：  
> 数据库存在一个信息表，每天都在记录信息，数据量大，每天创建一张表存储。  
> 表名每天都在变，需要从当天的表拿到信息同步到指定数据库。

> 预定格式：  
> yyyyMMdd_tableName（ 20200202_abcd ）

本次测试环境：

| 系统 | Windows10 |
| 软件版本 | kettle 7.1.0.0-12 |
| MySQL 驱动 | mysql-connector-java-8.0.19.jar |

* * *

-   第一个转换设置表名为变量
-   第二关转换拿到变量带入 sql 查询出数据  
    ![](https://img-blog.csdnimg.cn/20200710120407664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

* * *

## 一、获取表名

![](https://img-blog.csdnimg.cn/20200710142937445.png)

实现步骤如下：

-   新建一个转换 getTableName，拖入`获取系统信息`，`字段选择`，`设置变量`  
    ![](https://img-blog.csdnimg.cn/20200710120701528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

1.  打开 `获取系统信息` 编辑界面，填写`名称`，点击`类型`选择要获取的信息类型  
    ![](https://img-blog.csdnimg.cn/20200710120911260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)
2.  打开`字段选择`，选择`元数据`，设置字段属性  
    ![](https://img-blog.csdnimg.cn/20200710142416517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)
3.  打开`设置变量`，填写相关信息，也可以点击`获取字段`自动填写字段信息  
    ![](https://img-blog.csdnimg.cn/20200710142728706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

`表名转换`到此结束。

* * *

## 二、根据表名更新数据

![](https://img-blog.csdnimg.cn/20200710143448403.png)

实现步骤如下：

-   新建一个转换 autoQuery，拖入`表输入`，`插入 / 更新`
-   表操作需要连接数据库，本次[Kettle 使用 8.x 版本的 MySQL 驱动](https://blog.csdn.net/u013023406/article/details/106861825)  
    ![](https://img-blog.csdnimg.cn/20200710144049504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

1.  打开`表输入`，选择数据库，填写 sql 语句，使用之前设置的`${TODAY}`变量，`替换 SQL 语句里的变量`勾选上，否则`变量`无法生效  
    ![](https://img-blog.csdnimg.cn/2020071014441571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)
2.  打开`插入 / 更新`，指定`目标表`，填写`查询关键字`，填写`更新字段`  
    ![](https://img-blog.csdnimg.cn/2020071014510771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

`根据表名更新数据转换`到此结束。

## 三、作业执行转换

![](https://img-blog.csdnimg.cn/20200710145515417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)

-   新建一个作业 autoTable，拖入`START`，先调用`获取表名`转换，然后调用`根据表名更新数据`转换，最后拖入`成功`

设置如下：

1.  拖入`START`
2.  拖入转换，编辑`获取表名转换`的作业项名称，选择要执行的转换文件 getTableName，其它默认  
    ![](https://img-blog.csdnimg.cn/20200710150023672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)
3.  拖入转换，编辑`根据表名更新数据转换`的作业项名称，选择要执行的转换文件 autoQuery，其它默认  
    ![](https://img-blog.csdnimg.cn/20200710150201216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMjM0MDY=,size_16,color_FFFFFF,t_70)
4.  拖入`成功`
5.  作业保存，运行。

* * *

到此，动态表查询结束。 
 [https://blog.csdn.net/u013023406/article/details/107247920](https://blog.csdn.net/u013023406/article/details/107247920)
