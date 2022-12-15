# PostgreSQL 中文全文检索 （使用zhparser） - ldp.im - 博客园
前言：  
PostgreSQL 默认分词是按照空格及各种标点符号来分词，但是对于国内更多的是中文文章，按照默认分词方式不符合中文的分词方式。检索了网上很多文章，发现使用最多的是 zhparser，并且是开源的，完成能够满足检索需求。

前置：  
centOS7  
PostgreSQL11  
SCWS(下载地址：[http://www.xunsearch.com/scws/down/scws-1.2.2.tar.bz2](http://www.xunsearch.com/scws/down/scws-1.2.2.tar.bz2))  
zhparser(GitHub 地址：[https://github.com/amutu/zhparser](https://github.com/amutu/zhparser))  
postgresql-devel

```null
zhparser支持PostgreSQL 9.2及以上版本，请确保你的PG版本符合要求。 对于REDHAT/CentOS Linux系统，
请确保安装了相关的库和头文件，一般它们在postgresql-devel软件包中。
```

-   1
-   2

安装：  
1、首先需要安装 postgresql-devel

```null
yum install postgresql-devel 或者 yum install postgresql11-devel (我这里使用的第一种，没有指定版本号)
```

-   1

![](https://img-blog.csdnimg.cn/20190430102225815.png)

如果不安装此软件包的话，在编译 zhparser 文件时会报 pg_config: Command not found 错误  
![](https://img-blog.csdnimg.cn/20190430102316936.png)

2、安装 SCWS  
因为 zhparser 是基于 SCWS(简易中文分词系统) 开发的。所以必须首先安装 SCWS。

·2.1 创建一个文件夹 (这里不详细写创建步骤，这里我统一将 SCWS 和 zhparser 放在了 / home/pgsql / 下面)  
·2.2 进入创建的文件夹  
![](https://img-blog.csdnimg.cn/20190430102813169.png)

·2.3 下载 SCWS

```null
	wget http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2
```

-   1

![](https://img-blog.csdnimg.cn/20190430122711456.png)

·2.3 安装

```null
[root@pg1 opt]#  tar -jxvf scws-1.2.3.tar.bz2
[root@ecs-f49b-0003 pgsql]# cd scws-1.2.3 ; ./configure ; make install

注意:在FreeBSD release 10及以上版本上运行configure时，需要增加--with-pic选项。
如果是从github上下载的scws源码需要先运行以下命令生成configure文件： 
touch README;aclocal;autoconf;autoheader;libtoolize;automake --add-missing
```

-   1
-   2
-   3
-   4
-   5
-   6

3、安装 zhparser

·3.1 这里可以参考 GitHub 上的步骤，[https://github.com/amutu/zhparser](https://github.com/amutu/zhparser) ，忽略创建 extension 步骤，这里需要按照下面步骤检查和创建 extension ，与 GitHub 上写的略有不同 (建议编译和安装 zhparser 时采用指定 PG_CONFIG 的方式指定版本编译扩展，避免直接使用 make && make install 编译结果与安装的数据库版本不一致)。

```null
ps:因为在安装PostgreSQL时，我是用YUM方式安装的，clang并没有被加入，所以在指定 PG_CONFIG 
版本编译扩展时可能会报--新增插件异常(clang: Command not found)，或者直接编译不通过。
这里可以参考（https:

vim	 /usr/pgsql-11/lib/pgxs/src/makefiles/pgxs.mk
注释所有的with_llvm相关
```

-   1
-   2
-   3
-   4
-   5
-   6

![](https://img-blog.csdnimg.cn/20190430104045944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

下面是我编译后的结果，编译的文件没有直接放到 pgsql 中，下面图片中标注中的是最后编辑后文件的放置路径。(如果未出现相同的问题，那么就可以跳过文件整理步骤直接去创建 extension。)  
![](https://img-blog.csdnimg.cn/20190430104651648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

因为编译后的文件不在应该在的文件夹中，这里就需要手动去将文件移动至指定目录：  
/usr/lib64/pgsql/zhparser.so → /usr/pgsql-11/lib (/usr/pgsql-11 / 是 PostgreSQL 安装后路径，如果 PostgreSQL 是使用的 YUM 安装方式，路径应该是一样)  
![](https://img-blog.csdnimg.cn/20190430105845118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

/usr/share/pgsql/extension/ → /usr/pgsql-11/share/extension  
![](https://img-blog.csdnimg.cn/20190430105937421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

/usr/share/pgsql/tsearch_data/ → /usr/pgsql-11/share/tsearch_data  
![](https://img-blog.csdnimg.cn/2019043011004358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

·3.2 创建 extension  
整理好这些文件后，就可以继续创建 extension 了。  
·3.2.1 切换到 postgres 账户

```null
	su - postgres
```

-   1

![](https://img-blog.csdnimg.cn/20190430111146655.png)

·3.2.2 安装扩展 (每新建一个数据库都需要执行这一步；这里我直接使用的已有的一个测试数据库)

```null

knowledge=
```

-   1
-   2

![](https://img-blog.csdnimg.cn/20190430112010907.png)

这里可以看到就只有一个默认的解析器。下面开始创建扩展：

```null
knowledge=
```

-   1

![](https://img-blog.csdnimg.cn/20190430112135103.png)

这里再来看一次已有的解析器：  
![](https://img-blog.csdnimg.cn/20190430112224789.png)

这样解析器就添加到当前数据库了。但是此时还是不能用，还需要创建使用 zhparser 作为解析器的全文搜索的配置，也就是需要给 zhparser 解析器取一个在 sql 里面可以使用的名字。这里测试取的是 (‘zh’)。

![](https://img-blog.csdnimg.cn/20190430112611453.png)

再继续向全文搜索配置中增加 token 映射：

```null
knowledge=
```

-   1

![](https://img-blog.csdnimg.cn/20190430113305723.png)

这样就算是安装配置完成了。下面来测试一下效果：  
![](https://img-blog.csdnimg.cn/20190430113452339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

#### 追加问题解决

最近使用中遇到一个查询时的问题:  
例如一篇文章中有 “… 合江县…”，分词分出来的就是合江县，如果查询的时候输入的是合江，那么是匹配不到的，所以这里需要在继续设置一下全文检索的复合等级；在命令行中使用上一节中介绍的 scws 命令测试分词配置，如我认为复合等级为 7 时分词结果最好，则我在 postgresql.conf 添加配置：

```null
zhparser.multi_short = true 
zhparser.multi_duality = true  
zhparser.multi_zmain = true  
zhparser.multi_zall = false  
```

-   1
-   2
-   3
-   4

![](https://img-blog.csdnimg.cn/20190626110429826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2019062611051287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxOTQ4OTUx,size_16,color_FFFFFF,t_70)

然后重启数据库，将之前的分词重新全部生成一遍，在来查询，问题解决。

OK！！！这样就可以满足当前需求了，完工！！！

### SQL

查询中我们可以使用最简单的 `SELECT * FROM table WHERE to_tsvector('parser_name', field) @@ 'word'` 来查询 field 字段分词中带有 word 一词的数据；

使用 `to_tsquery()` 方法将句子解析成各个词的组合向量，如 `国家大剧院` 的返回结果为 `'国家' & '大剧院' & '大剧' & '剧院'` ，当然我们也可以使用 `& |` 符号拼接自己需要的向量；在查询 长句 时，可以使用 `SELECT * FROM table WHERE to_tsvector('parser_name', field) @@ to_tsquery('parser_name','words')`；

有时候我们想像 MySQL 的 `SQL_CALC_FOUND_ROWS` 语句一样同步返回结果条数，则可以使用 `SELECT COUNT(*) OVER() AS score FROM table WHERE ...`，PgSQL 会在每一行数据添加 score 字段存储查询到的总结果条数；

到这里，普通的全文检索需求已经实现了。

* * *

我们接着对分词效果和效率进行优化：

### 存储分词结果

我们可以使用一个字段来存储分词向量，并在此字段上创建索引来更优地使用分词索引：

````null
ALTER TABLE table ADD COLUMN tsv_column tsvector;           // 添加一个分词字段
UPDATE table SET tsv_column = to_tsvector('parser_name', coalesce(field,''));   // 将字段的分词向量更新到新字段中
CREATE INDEX idx_gin_zhcn ON table USING GIN(tsv_column);   // 在新字段上创建索引
CREATE TRIGGER trigger_name BEFORE INSERT OR UPDATE  ON table FOR EACH ROW EXECUTE PROCEDURE
tsvector_update_trigger(tsv_column, 'parser_name', field); // 创建一个更新分词触发器```

这样，再进行查询时就可以直接使用 `SELECT * FROM table WHERE tsv_column @@ 'keyword'` 了。

这里需要注意，这时候在往表内插入数据的时候，可能会报错，提示指定 parser\_name 的 schema， 这时候可以使用 `\dF` 命令查看所有 text search configuration 的参数：

```null
               List of text search configurations
   Schema   |    Name    |              Description
------------+------------+---------------------------------------
 pg_catalog | english    | configuration for english language
 public     | myparser   |```

注意 schema 参数，在创建 trigger 时需要指定 schema， 如上面，就需要使用 `public.myparser`。

### 添加自定义词典

我们可以在网上下载 xdb 格式的词库来替代默认词典，词库放在 `share/tsearch_data/` 文件夹下才能被 PgSQL 读取到，默认使用的词库是 `dict.utf8.xdb`。要使用自定义词库，可以将词库放在词库文件夹后，在 `postgresql.conf` 配置 `zhparser.extra_dict="mydict.xdb"` 参数；

当我们只有 txt 的词库，想把这个词库作为默认词库该怎么办呢？使用 scws 带的`scwe-gen-dict` 工具或网上找的脚本生成 xdb 后放入词库文件夹后，在 PgSQL 中分词一直报错，读取词库文件失败。我经过多次实验，总结出了一套制作一个词典文件的方法：

1.  准备词库源文件 mydict.txt：词库文件的内容每一行的格式为`词 TF IDF 词性`，词是必须的，而 `TF 词频(Term Frequency)、IDF 反文档频率(Inverse Document Frequency) 和 词性` 都是可选的，除非确定自己的词典资料是对的且符合 scws 的配置，不然最好还是留空，让 scws 自已确定；
2.  在 `postgresql.conf` 中设置 `zhparser.extra_dicts = "mydict.txt"` 同时设置 `zhparser.dict_in_memory = true`；
3.  命令行进入 PgSQL，执行一条分词语句 `select to_tsquery('parser', '随便一个词')` ，分词会极慢，请耐心(请保证此时只有一个分词语句在执行)；
4.  分词成功后，在/tmp/目录下找到生成的 `scws-xxxx.xdb` 替换掉 `share/tsearch_data/dict.utf8.xdb`；
5.  删除刚加入的 `extra_dicts dict_in_memory` 配置，重启服务器。

### 扩展

由于查询的是 POI 的名称，一般较短，且很多词并无语义，又考虑到用户的输入习惯，一般会输入 POI 名称的前几个字符，而且 scws 的分词准确率也不能达到100%，于是我添加了名称的前缀查询来提高查询的准确率，即使用 B树索引 实现 `LIKE '关键词%'` 的查询。这里需

这里要注意的是，创建索引时要根据字段类型配置 `操作符类`，不然索引可能会不生效，如在 字段类型为 varchar 的字段上创建索引需要使用语句`CREATE INDEX idx_name ON table(COLUMN varchar_pattern_ops)`，这里的 varchar_pattern_ops 就是操作符类，操作符类的介绍和选择可以查看文档：[11.9. 操作符类和操作符族](http://www.postgres.cn/docs/9.4/indexes-opclass.html)。

自此，一个良好的全文检索系统就完成了。

* * *

简单的数据迁移并不是终点，后续要做的还有很多，如整个系统的数据同步、查询效率优化、查询功能优化（添加拼音搜索、模糊搜索）等。特别是查询效率，不知道是不是我配置有问题，完全达不到那种 E级毫秒 的速度，1kw 的数据效率在进行大结果返回时就大幅下降（200ms），只好老老实实地提前进行了分表，目前百万级查询速度在 20ms 以内，优化还有一段路要走。

不过这次倒是对 技术的“生态”有了个更深的体会，这方面 PgSQL 确实和 MySQL 差远了，使用 MySQL 时再奇葩的问题都能在网上快速找到答案，而 PgSQL 就尴尬了，入门级的问题搜索 stackoverflow 来来回回就那么几个对不上的回答。虽然也有阿里的“德哥”一样的大神在辛苦布道，但用户的数量才是根本。不过，随着 PgSQL 越来越完善，使用它的人一定会越来越多的，我这篇文章也算是为 PgSQL 加温了吧，哈哈~希望能帮到后来的使用者。 
 [https://www.cnblogs.com/Amos-Turing/p/14147576.html](https://www.cnblogs.com/Amos-Turing/p/14147576.html)
````
