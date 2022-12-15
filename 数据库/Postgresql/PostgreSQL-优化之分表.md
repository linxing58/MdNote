# PostgreSQL-优化之分表 
## 分表概述

数据库分表，就是把一张表分成多张表，物理上虽然分开了，逻辑上彼此仍有联系。

分表有两种方式：水平分表，即按列分开；垂直分表，即按行分开

**优势**

1. 查询速度大幅提升

2. 删除数据速度更快

3. 可以将使用率低的数据通过表空间技术转移到低成本的存储介质上

**场景**

官方建议：当数据表大小超过数据库服务器内存时应该使用分表。

两种分表方式大致相同，下面以垂直分表为例进行介绍。

## 垂直分表

### 基本过程

1. 创建父表

2. 创建子表，子表必须继承父表，最好不要新加字段　　【加了以后如何，没试过】

　　// 可以给每个子表创建索引

3. 定义一个规则（rule） 或者触发器（trigger），把对父表的写入重定向到对应的分表

### 创建父表

父表无数据，无约束，无索引

![](https://common.cnblogs.com/images/copycode.gif)

CREATE TABLE tbl_partition
(
  date_key date,
  hour_key smallint,
  client_key integer,
  item_key integer,
  account integer,
  expense numeric
);

![](https://common.cnblogs.com/images/copycode.gif)

### 创建分表

分表必须继承父表

CREATE TABLE tbl_partition_2016_01() inherits (tbl_partition); CREATE TABLE tbl_partition_2016_02() inherits (tbl_partition); CREATE TABLE tbl_partition_2016_03() inherits (tbl_partition);

分表需要添加限制，这些限制决定了每张表允许保存的数据范围，每张表的限制范围不能有重叠。

![](https://common.cnblogs.com/images/copycode.gif)

ALTER TABLE tbl_partition_2016_01 ADD CONSTRAINT tbl_partition_2016_01_check_date_key CHECK (date_Key >= '2016-01-01'::date AND date_Key &lt;'2016-02-01'::date); ALTER TABLE tbl_partition_2016_02 ADD CONSTRAINT tbl_partition_2016_02_check_date_key CHECK (date_Key >= '2016-02-01'::date AND date_Key &lt;'2016-03-01'::date); ALTER TABLE tbl_partition_2016_03 ADD CONSTRAINT tbl_partition_2016_03_check_date_key CHECK (date_Key >= '2016-03-01'::date AND date_Key &lt;'2016-04-01'::date);

![](https://common.cnblogs.com/images/copycode.gif)

也可以建表和限制写在一起

create table t_sys_log_y2016m09
(CHECK (operation_time >= DATE '2016-09-01' AND operation_time&lt;DATE '2016-10-01'))
INHERITS (t_sys_log_main);

给分表添加索引

CREATE INDEX tbl_partition_date_key_2016_01 ON tbl_partition_2016_01 (date_key,client_key); CREATE INDEX tbl_partition_date_key_2016_02 ON tbl_partition_2016_02 (date_key,client_key); CREATE INDEX tbl_partition_date_key_2016_03 ON tbl_partition_2016_03 (date_key,client_key);

### 创建触发器

表建立完成后，就是写入的工作，如何将数据自动写入对应的分表呢？两种方法，分别是规则（rule）和触发器（trigger），相比 trigger，rule 开销更大，这里使用触发器。

trigger 通常会结合自定义函数来实现分区插入：Function 负责根据条件选择插入，trigger 负责自动调用 Function

**首先定义 Function**

![](https://common.cnblogs.com/images/copycode.gif)

CREATE OR REPLACE FUNCTION tbl_partition_trigger() RETURNS TRIGGER AS $$ BEGIN
  IF NEW.date_key >= DATE '2016-01-01' AND NEW.date_Key &lt; DATE '2016-02-01'
  THEN
    INSERT INTO tbl_partition_2016_01 VALUES (NEW.\*);
  ELSIF NEW.date_key >= DATE '2016-02-01' AND NEW.date_Key &lt; DATE '2016-03-01'
    THEN
      INSERT INTO tbl_partition_2016_02 VALUES (NEW.\*);
  ELSIF NEW.date_key >= DATE '2016-03-01' AND NEW.date_Key &lt; DATE '2016-04-01'
    THEN
      INSERT INTO tbl_partition_2016_03 VALUES (NEW.\*); END IF; RETURN NULL; END;
$$
LANGUAGE plpgsql;

![](https://common.cnblogs.com/images/copycode.gif)

**对父表创建触发器**

CREATE TRIGGER insert_tbl_partition_trigger
BEFORE INSERT ON tbl_partition FOR EACH ROW EXECUTE PROCEDURE tbl_partition_trigger();

至此分表成功

![](https://img2018.cnblogs.com/blog/1603920/201909/1603920-20190902135546337-185364096.png)

## 性能对比

为了对比分表与不分表的性能，我创建了一个全表。

![](https://common.cnblogs.com/images/copycode.gif)

CREATE TABLE tbl_partition_all
(
  date_key date,
  hour_key smallint,
  client_key integer,
  item_key integer,
  account integer,
  expense numeric
);

![](https://common.cnblogs.com/images/copycode.gif)

把数据先全部写到全表后，迁移到分表

INSERT INTO tbl_partition SELECT \* FROM tbl_partition_all;

全表 9w 条数据，分表每个 3w 条，测试如下

**1. 分表 - 查询单个分表内的数据**

EXPLAIN  ANALYZE select count(account) ,client_key  from tbl_partition  v where v.date_key >='2016-03-02'   and v.date_key &lt;='2016-03-07' group by client_key ;

3 月份的数据在一个表内，耗时约 18s 

全表查同样的数据

EXPLAIN  ANALYZE select count(account) ,client_key  from tbl_partition_all  v where v.date_key >='2016-03-02'   and v.date_key &lt;='2016-03-07' group by client_key ;

耗时约 30s

**2. 分表 - 查询跨分表的数据**

EXPLAIN  ANALYZE select count(account) ,client_key  from tbl_partition  v where v.date_key >='2016-01-02'   and v.date_key &lt;='2016-03-07' group by client_key ;

跨 3 个表，耗时约 65s

全表查同样的数据

EXPLAIN  ANALYZE select count(account) ,client_key  from tbl_partition_all  v where v.date_key >='2016-01-02'   and v.date_key &lt;='2016-03-07' group by client_key ;

耗时约 87s

**3. 有同事问为什么不直接分呢？不继承单纯按数据建多个表**

对此我也进行了测试，单独建立 3 个表，分别存放之前每个分表的数据，分别建立索引，然后查询同样的数据

![](https://common.cnblogs.com/images/copycode.gif)

EXPLAIN  ANALYZE select count(account) ,client_key  from test1  v where v.date_key >='2016-01-02'   and v.date_key &lt;='2016-01-28' group by client_key union
select count(account) ,client_key  from test2  v where v.date_key >='2016-02-01'   and v.date_key &lt;='2016-02-28' group by client_key union
select count(account) ,client_key  from test3  v where v.date_key >='2016-03-01'   and v.date_key &lt;='2016-03-07' group by client_key ;

![](https://common.cnblogs.com/images/copycode.gif)

耗时约 180s，效率更低

总结：分表效率很高，优于全表和多个单表，我这里只是用了少量的数据，性能并没有提升很大，如果数据量很大，性能应该提升明显。

## 分表其他操作

删除继承关系

ALTER TABLE tbl_partition_2016_01 NO INHERIT tbl_partition;

添加继承关系

ALTER TABLE test1 INHERIT tbl_partition;

参考资料：

[https://www.cnblogs.com/winkey4986/p/6824747.html](https://www.cnblogs.com/winkey4986/p/6824747.html)

[https://www.jb51.net/article/97937.htm](https://www.jb51.net/article/97937.htm)

[https://blog.csdn.net/imthemostshuaiin626/article/details/77318911](https://blog.csdn.net/imthemostshuaiin626/article/details/77318911)

[https://hacpai.com/article/1536655962119](https://hacpai.com/article/1536655962119) 
 [https://www.cnblogs.com/yanshw/p/11445780.html](https://www.cnblogs.com/yanshw/p/11445780.html)
