# PostgreSQL的时间/日期函数使用 - David_Tang - 博客园
PostgreSQL 的常用时间函数使用整理如下：

**一、获取系统时间函数**

1.1 获取当前完整时间

select now();

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now();
              now -------------------------------
 2013-04-12 15:39:40.399711+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

current_timestamp 同 now() 函数等效。

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select current_timestamp;
              now -------------------------------
 2013-04-12 15:40:22.398709+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

1.2 获取当前日期

select current_date;

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select current_date;
    date ------------
 2013-04-12 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

1.3 获取当前时间

select current_time;

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select current_time;
       timetz --------------------
 15:43:31.101726+08 (1 row)

david\\=#

![](https://common.cnblogs.com/images/copycode.gif)

**二、时间的计算**

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now();
              now -------------------------------
 2013-04-12 15:47:13.244721+08 (1 row)

david\\=#

![](https://common.cnblogs.com/images/copycode.gif)

2.1 两年后

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now() + interval '2 years';
           ?column? -------------------------------
 2015-04-12 15:49:03.168851+08 (1 row)

david\\=# select now() + interval '2 year'; 
           ?column? -------------------------------
 2015-04-12 15:49:12.378727+08 (1 row)

david\\=# select now() + interval '2 y';  
           ?column? ------------------------------
 2015-04-12 15:49:25.46986+08 (1 row)

david\\=# select now() + interval '2 Y';
           ?column? -------------------------------
 2015-04-12 15:49:28.410853+08 (1 row)

david\\=# select now() + interval '2Y'; 
           ?column? -------------------------------
 2015-04-12 15:49:31.122831+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

2.2 一个月后

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now() + interval '1 month';  
           ?column? ------------------------------
 2013-05-12 15:51:22.24373+08 (1 row)

david\\=# select now() + interval 'one month';
ERROR:  invalid input syntax for type interval: "one month"
LINE 1: select now() + interval 'one month'; ^ david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

2.3 三周前

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now() - interval '3 week';
           ?column? -------------------------------
 2013-03-22 16:00:04.203735+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

2.4 十分钟后

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select now() + '10 min';  
           ?column? -------------------------------
 2013-04-12 16:12:47.445744+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

**说明：** 

interval 可以不写，其值可以是：

\| **Abbreviation** \| **Meaning** \|
| Y | Years |
| M | Months (in the date part) |
| W | Weeks |
| D | Days |
| H | Hours |
| M | Minutes (in the time part) |
| S | Seconds |

2.5 计算两个时间差

使用 age(timestamp, timestamp)

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select age(now(), timestamp '1989-02-05');
                  age ----------------------------------------
 24 years 2 mons 7 days 17:05:49.119848 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select age(timestamp '2007-09-15');  
          age ------------------------
 5 years 6 mons 27 days
(1 row)

david\\=#

![](https://common.cnblogs.com/images/copycode.gif)

**三、时间字段的截取**

在开发过程中，经常要取日期的年，月，日，小时等值，PostgreSQL 提供一个非常便利的 EXTRACT 函数。

EXTRACT(field FROM source)

field 表示取的时间对象，source 表示取的日期来源，类型为 timestamp、time 或 interval。

3.1 取年份

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select extract(year from now());
 date_part -----------
      2013 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

3.2 取月份

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select extract(month from now());  
 date_part -----------
         4 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select extract(day from timestamp '2013-04-13');
 date_part -----------
        13 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# SELECT EXTRACT(DAY FROM INTERVAL '40 days 1 minute');
 date_part -----------
        40 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

3.3 查看今天是一年中的第几天

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select extract(doy from now());
 date_part -----------
       102 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

3.4 查看现在距 1970-01-01 00:00:00 UTC 的秒数

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# select extract(epoch from now());
    date_part ------------------
 1365755907.94474 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

3.5 把 epoch 值转换回时间戳

![](https://common.cnblogs.com/images/copycode.gif)

david\\=# SELECT TIMESTAMP WITH TIME ZONE 'epoch' + 1369755555 \* INTERVAL '1 second'; 
        ?column? ------------------------
 2013-05-28 23:39:15+08 (1 row)

david\\=# 

![](https://common.cnblogs.com/images/copycode.gif)

以上是基本的 PG 时间 / 日期函数使用，可满足一般的开发运维应用。

详细用法请参考：

PostgreSQL 官方说明：[http://www.postgresql.org/docs/9.2/static/functions-datetime.html](http://www.postgresql.org/docs/9.2/static/functions-datetime.html) 
 [https://www.cnblogs.com/mchina/archive/2013/04/15/3010418.html](https://www.cnblogs.com/mchina/archive/2013/04/15/3010418.html)
