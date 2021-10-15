# 【MyBatis】几种批量插入效率的比较 - 掘金
批处理数据主要有三种方式：

1.  反复执行单条插入语句
2.  `foreach` 拼接 `sql`
3.  批处理

### 一、前期准备

基于`Spring Boot` + `Mysql`，同时为了省略`get`/`set`，使用了`lombok`，详见`pom.xml`。

#### 1.1 表结构

> `id` 使用数据库自增。

```sql
DROP TABLE IF EXISTS `user_info_batch`;
CREATE TABLE `user_info_batch` (
                           `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
                           `user_name` varchar(100) NOT NULL COMMENT '账户名称',
                           `pass_word` varchar(100) NOT NULL COMMENT '登录密码',
                           `nick_name` varchar(30) NOT NULL COMMENT '昵称',
                           `mobile` varchar(30) NOT NULL COMMENT '手机号',
                           `email` varchar(100) DEFAULT NULL COMMENT '邮箱地址',
                           `gmt_create` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
                           `gmt_update` timestamp NULL DEFAULT NULL COMMENT '更新时间',
                           PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT 'Mybatis Batch';
```

#### 1.2 项目配置文件

细心的你可能已经发现，数据库`url` 后面跟了一段 `rewriteBatchedStatements=true`，有什么用呢？先不急，后面会介绍。

```xml
# 数据库配置
spring:
  datasource:
    url: jdbc:mysql://47.111.118.152:3306/mybatis?rewriteBatchedStatements=true
    username: mybatis
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
# mybatis
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: cn.van.mybatis.batch.entity
```

#### 1.3 实体类

    @Data
    @Accessors(chain = true)
    public class UserInfoBatchDO implements Serializable {
        private Long id;

        private String userName;

        private String passWord;

        private String nickName;

        private String mobile;

        private String email;

        private LocalDateTime gmtCreate;

        private LocalDateTime gmtUpdate;
    }
    复制代码

#### 1.4 `UserInfoBatchMapper`

```java
public interface UserInfoBatchMapper {

    
    int insert(UserInfoBatchDO info);

    
    int batchInsert(List<UserInfoBatchDO> list);
}
```

#### 1.5 `UserInfoBatchMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.van.mybatis.batch.mapper.UserInfoBatchMapper">

  <insert id="insert" parameterType="cn.van.mybatis.batch.entity.UserInfoBatchDO">
    insert into user_info_batch (user_name, pass_word, nick_name, mobile, email, gmt_create, gmt_update)
    values (#{userName,jdbcType=VARCHAR}, #{passWord,jdbcType=VARCHAR},#{nickName,jdbcType=VARCHAR}, #{mobile,jdbcType=VARCHAR}, #{email,jdbcType=VARCHAR}, #{gmtCreate,jdbcType=TIMESTAMP}, #{gmtUpdate,jdbcType=TIMESTAMP})
  </insert>

  <insert id="batchInsert">
    insert into user_info_batch (user_name, pass_word, nick_name, mobile, email, gmt_create, gmt_update)
    values
    <foreach collection="list" item="item" separator=",">
      (#{item.userName,jdbcType=VARCHAR}, #{item.passWord,jdbcType=VARCHAR}, #{item.nickName,jdbcType=VARCHAR}, #{item.mobile,jdbcType=VARCHAR}, #{item.email,jdbcType=VARCHAR}, #{item.gmtCreate,jdbcType=TIMESTAMP}, #{item.gmtUpdate,jdbcType=TIMESTAMP})
    </foreach>
  </insert>
</mapper>
```

#### 1.6 预备数据

> 为了方便测试，抽离了几个变量，并进行提前加载。

        private List<UserInfoBatchDO> list = new ArrayList<>();
        private List<UserInfoBatchDO> lessList = new ArrayList<>();
        private List<UserInfoBatchDO> lageList = new ArrayList<>();
        private List<UserInfoBatchDO> warmList = new ArrayList<>();
        // 计数工具
        private StopWatch sw = new StopWatch();
    复制代码

-   为了方便组装数据，抽出了一个公共方法。


        private List<UserInfoBatchDO> assemblyData(int count){
            List<UserInfoBatchDO> list = new ArrayList<>();
            UserInfoBatchDO userInfoDO;
            for (int i = 0;i < count;i++){
                userInfoDO = new UserInfoBatchDO()
                        .setUserName("Van")
                        .setNickName("风尘博客")
                        .setMobile("17098705205")
                        .setPassWord("password")
                        .setGmtUpdate(LocalDateTime.now());
                list.add(userInfoDO);
            }
            return list;
        }
    复制代码

-   预热数据


        @Before
        public void assemblyData() {
            list = assemblyData(200000);
            lessList = assemblyData(2000);
            lageList = assemblyData(1000000);
            warmList = assemblyData(5);
        }
    复制代码

### 二、反复执行单条插入语句

> 可能‘懒’的程序员会这么做，很简单，直接在原先单条`insert`语句上嵌套一个`for`循环。

#### 2.1 对应 mapper 接口

    int insert(UserInfoBatchDO info);
    复制代码

#### 2.2 测试方法

> 因为这种方法太慢，所以数据降低到 `2000` 条

    @Test
    public void insert() {
        log.info("【程序热身】");
        for (UserInfoBatchDO userInfoBatchDO : warmList) {
            userInfoBatchMapper.insert(userInfoBatchDO);
        }
        log.info("【热身结束】");
        sw.start("反复执行单条插入语句");
        // 这里插入 20w 条太慢了，所以我只插入了 2000 条
        for (UserInfoBatchDO userInfoBatchDO : lessList) {
            userInfoBatchMapper.insert(userInfoBatchDO);
        }
        sw.stop();
        log.info("all cost info:{}",sw.prettyPrint());
    }
    复制代码

#### 2.3 执行时间

-   第一次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
59887  100%  反复执行单条插入语句
```

-   第二次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
64853  100%  反复执行单条插入语句
```

-   第三次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
58235  100%  反复执行单条插入语句
```

该方式插入`2000` 条数据，执行三次的平均时间：`60991 ms`。

### 三、`foreach` 拼接`SQL`

#### 3.1 对应 mapper 接口

```java
int batchInsert(List<UserInfoBatchDO> list);
```

#### 3.2 测试方法

> 该方式和下一种方式都采用`20w`条数据测试。

    @Test
    public void batchInsert() {
        log.info("【程序热身】");
        for (UserInfoBatchDO userInfoBatchDO : warmList) {
            userInfoBatchMapper.insert(userInfoBatchDO);
        }
        log.info("【热身结束】");
        sw.start("foreach 拼接 sql");
        userInfoBatchMapper.batchInsert(list);
        sw.stop();
        log.info("all cost info:{}",sw.prettyPrint());
    }
    复制代码

#### 3.3 执行时间

-   第一次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
18835  100%  foreach 拼接 sql
```

-   第二次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
17895  100%  foreach 拼接 sql
```

-   第三次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
19827  100%  foreach 拼接 sql
```

该方式插入`20w` 条数据，执行三次的平均时间：`18852 ms`。

### 四、批处理

> 该方式 `mapper` 和`xml` 复用了 `2.1`。

#### 4.1 `rewriteBatchedStatements` 参数

> 我在测试一开始，发现改成 `Mybatis Batch`提交的方法都不起作用，实际上在插入的时候仍然是一条条记录的插，而且速度远不如原来 `foreach` 拼接`SQL`的方法，这是非常不科学的。

**后来才发现要批量执行的话，连接`URL`字符串中需要新增一个参数：`rewriteBatchedStatements=true`**

-   `rewriteBatchedStatements`参数介绍

`MySql`的`JDBC`连接的`url`中要加`rewriteBatchedStatements`参数，并保证`5.1.13`以上版本的驱动，才能实现高性能的批量插入。`MySql JDBC`驱动在默认情况下会无视`executeBatch()`语句，把我们期望批量执行的一组`sql`语句拆散，一条一条地发给`MySql`数据库，批量插入实际上是单条插入，直接造成较低的性能。只有把`rewriteBatchedStatements`参数置为`true`, 驱动才会帮你批量执行`SQL`。这个选项对`INSERT`/`UPDATE`/`DELETE`都有效。

#### 4.2 批处理准备

-   手动注入 `SqlSessionFactory`


        @Resource
        private SqlSessionFactory sqlSessionFactory;
    复制代码

-   测试代码


    @Test
    public void processInsert() {
        log.info("【程序热身】");
        for (UserInfoBatchDO userInfoBatchDO : warmList) {
            userInfoBatchMapper.insert(userInfoBatchDO);
        }
        log.info("【热身结束】");
        sw.start("批处理执行 插入");
        // 打开批处理
        SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH);
        UserInfoBatchMapper mapper = session.getMapper(UserInfoBatchMapper.class);
        for (int i = 0,length = list.size(); i < length; i++) {
            mapper.insert(list.get(i));
            //每20000条提交一次防止内存溢出
            if(i%20000==19999){
                session.commit();
                session.clearCache();
            }
        }
        session.commit();
        session.clearCache();
        sw.stop();
        log.info("all cost info:{}",sw.prettyPrint());
    }
    复制代码

#### 4.3 执行时间

-   第一次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
09346  100%  批处理执行 插入
```

-   第二次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
08890  100%  批处理执行 插入
```

-   第三次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
09042  100%  批处理执行 插入
```

该方式插入`20w` 条数据，执行三次的平均时间：`9092 ms`。

#### 4.4 如果数据更大

当我把数据扩大到 `100w` 时，`foreach` 拼接 `sql` 的方式已经无法完成插入了，所以我只能测试批处理的插入时间。

> 测试时，仅需将 【4.2】测试代码中的 `list` 切成 `lageList` 测试即可。

-   第一次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
32419  100%  批处理执行 插入
```

-   第二次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
31935  100%  批处理执行 插入
```

-   第三次

```xml
-----------------------------------------
ms     %     Task name
-----------------------------------------
33048  100%  批处理执行 插入
```

该方式插入`100w` 条数据，执行三次的平均时间：`32467 ms`。

### 五、总结

| 批量插入方式            | 数据量  | 执行三次的平均时间 |
| ----------------- | ---- | --------- |
| 循环插入单条数据          | 2000 | 60991 ms  |
| `foreach` 拼接`sql` | 20w  | 18852 ms  |
| 批处理               | 20w  | 9092 ms   |
| 批处理               | 100w | 32467 ms  |

1.  循环插入单条数据虽然效率极低，但是代码量极少，数据量较小时可以使用，但是数据量较大禁止使用，效率太低了；
2.  `foreach` 拼接`sql`的方式，使用时有大段的 xml 和 sql 语句要写，很容易出错，虽然效率尚可，但是真正应对大量数据的时候，依旧无法使用，所以不推荐使用；
3.  批处理执行是有大数据量插入时推荐的做法，使用起来也比较方便。

【[本文示例代码](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FvanDusty%2FFrame-Home%2Ftree%2Fmaster%2Fmybatis-case%2Fmybatis-batch "https&#x3A;//github.com/vanDusty/Frame-Home/tree/master/mybatis-case/mybatis-batch")】

    复制代码

 [https://juejin.cn/post/7007608714093920286#heading-16](https://juejin.cn/post/7007608714093920286#heading-16)
