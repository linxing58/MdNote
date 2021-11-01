# mybatis plus 看这篇就够了，一发入魂 - 掘金
mybatis-plus 是一款 Mybatis 增强工具，用于简化开发，提高效率。下文使用缩写**mp**来简化表示**mybatis-plus**，本文主要介绍 mp 搭配 SpringBoot 的使用。

注：本文使用的 mp 版本是当前最新的 3.4.2，早期版本的差异请自行查阅文档

官方网站：[baomidou.com/](https://link.juejin.cn/?target=https%3A%2F%2Fbaomidou.com%2F "https&#x3A;//baomidou.com/")

## 快速入门

1.  创建一个 SpringBoot 项目
2.  导入依赖

    ```xml

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.3.4.RELEASE</version>
            <relativePath/> 
        </parent>
        <groupId>com.example</groupId>
        <artifactId>mybatis-plus</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <name>mybatis-plus</name>
        <properties>
            <java.version>1.8</java.version>
        </properties>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-configuration-processor</artifactId>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>3.4.2</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <scope>runtime</scope>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>
    ```
3.  配置数据库

    ```yaml

    spring:
      datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/yogurt?serverTimezone=Asia/Shanghai
        username: root
        password: root
        
    mybatis-plus:
      configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl 
    ```
4.  创建一个实体类

    ```java
    package com.example.mp.po;
    import lombok.Data;
    import java.time.LocalDateTime;
    @Data
    public class User {
    	private Long id;
    	private String name;
    	private Integer age;
    	private String email;
    	private Long managerId;
    	private LocalDateTime createTime;
    }
    ```
5.  创建一个 mapper 接口

    ```java
    package com.example.mp.mappers;
    import com.baomidou.mybatisplus.core.mapper.BaseMapper;
    import com.example.mp.po.User;
    public interface UserMapper extends BaseMapper<User> { }
    ```
6.  在 SpringBoot 启动类上配置 mapper 接口的扫描路径

    ```java
    package com.example.mp;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    @SpringBootApplication
    @MapperScan("com.example.mp.mappers")
    public class MybatisPlusApplication {
    	public static void main(String[] args) {
    		SpringApplication.run(MybatisPlusApplication.class, args);
    	}
    }
    ```
7.  在数据库中创建表

    ```sql
    DROP TABLE IF EXISTS user;
    CREATE TABLE user (
    id BIGINT(20) PRIMARY KEY NOT NULL COMMENT '主键',
    name VARCHAR(30) DEFAULT NULL COMMENT '姓名',
    age INT(11) DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
    manager_id BIGINT(20) DEFAULT NULL COMMENT '直属上级id',
    create_time DATETIME DEFAULT NULL COMMENT '创建时间',
    CONSTRAINT manager_fk FOREIGN KEY(manager_id) REFERENCES user (id)
    ) ENGINE=INNODB CHARSET=UTF8;

    INSERT INTO user (id, name, age ,email, manager_id, create_time) VALUES
    (1, '大BOSS', 40, 'boss@baomidou.com', NULL, '2021-03-22 09:48:00'),
    (2, '李经理', 40, 'boss@baomidou.com', 1, '2021-01-22 09:48:00'),
    (3, '黄主管', 40, 'boss@baomidou.com', 2, '2021-01-22 09:48:00'),
    (4, '吴组长', 40, 'boss@baomidou.com', 2, '2021-02-22 09:48:00'),
    (5, '小菜', 40, 'boss@baomidou.com', 2, '2021-02-22 09:48:00')
    ```
8.  编写一个 SpringBoot 测试类

    ```java
    package com.example.mp;
    import com.example.mp.mappers.UserMapper;
    import com.example.mp.po.User;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    import java.util.List;
    import static org.junit.Assert.*;
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class SampleTest {
    	@Autowired
    	private UserMapper mapper;
    	@Test
    	public void testSelect() {
    		List<User> list = mapper.selectList(null);
    		assertEquals(5, list.size());
    		list.forEach(System.out::println);
    	}
    }
    ```

准备工作完成

数据库情况如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4089e7904434d3f93d1d7bbe346ee93~tplv-k3u1fbpfcp-watermark.awebp)

项目目录如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0fa0aac38ae4e4097790f40543c9076~tplv-k3u1fbpfcp-watermark.awebp)

运行测试类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0d3be842a104119b89bb05c02cce739~tplv-k3u1fbpfcp-watermark.awebp)

可以看到，针对单表的基本 CRUD 操作，只需要创建好实体类，并创建一个继承自`BaseMapper`的接口即可，可谓非常简洁。并且，我们注意到，`User`类中的`managerId`，`createTime`属性，自动和数据库表中的`manager_id`，`create_time`对应了起来，这是因为 mp 自动做了数据库下划线命名，到 Java 类的驼峰命名之间的转化。

## 核心功能

### 注解

mp 一共提供了 8 个注解，这些注解是用在 Java 的实体类上面的。

-   `@TableName`

    注解在类上，指定类和数据库表的映射关系。**实体类的类名（转成小写后）和数据库表名相同时**，可以不指定该注解。
-   `@TableId`

    注解在实体类的某一字段上，**表示这个字段对应数据库表的主键**。当主键名为 id 时（表中列名为 id，实体类中字段名为 id），无需使用该注解显式指定主键，mp 会自动关联。若类的字段名和表的列名不一致，可用`value`属性指定表的列名。另，这个注解有个重要的属性`type`，用于指定主键策略，参见[主键策略小节](#idType "\#idType")
-   `@TableField`

    注解在某一字段上，指定 Java 实体类的字段和数据库表的列的映射关系。这个注解有如下几个应用场景。

    -   **排除非表字段**

        若 Java 实体类中某个字段，不对应表中的任何列，它只是用于保存一些额外的，或组装后的数据，则可以设置`exist`属性为`false`，这样在对实体对象进行插入时，会忽略这个字段。排除非表字段也可以通过其他方式完成，如使用`static`或`transient`关键字，但个人觉得不是很合理，不做赘述
    -   **字段验证策略**

        通过`insertStrategy`，`updateStrategy`，`whereStrategy`属性进行配置，可以控制在实体对象进行插入，更新，或作为 WHERE 条件时，对象中的字段要如何组装到 SQL 语句中。参见[配置小节](#fieldStrategy "\#fieldStrategy")
    -   **字段填充策略**

        通过`fill`属性指定，字段为空时会进行自动填充
-   `@Version`

    乐观锁注解，参见[乐观锁小节](#version "\#version")
-   `@EnumValue`

    注解在枚举字段上
-   `@TableLogic`

    逻辑删除，参见[逻辑删除小节](#logicDel "\#logicDel")
-   `KeySequence`

    序列主键策略（`oracle`）
-   `InterceptorIgnore`

    插件过滤规则

### CRUD 接口

mp 封装了一些最基础的 CRUD 方法，只需要直接继承 mp 提供的接口，无需编写任何 SQL，即可食用。mp 提供了两套接口，分别是 Mapper CRUD 接口和 Service CRUD 接口。并且 mp 还提供了条件构造器`Wrapper`，可以方便地组装 SQL 语句中的 WHERE 条件，参见[条件构造器小节](#wrapper "\#wrapper")

#### Mapper CRUD 接口

只需定义好实体类，然后创建一个接口，继承 mp 提供的`BaseMapper`，即可食用。mp 会在 mybatis 启动时，自动解析实体类和表的映射关系，并注入带有通用 CRUD 方法的 mapper。`BaseMapper`里提供的方法，部分列举如下：

-   `insert(T entity)` 插入一条记录
-   `deleteById(Serializable id)` 根据主键 id 删除一条记录
-   `delete(Wrapper<T> wrapper)` 根据条件构造器 wrapper 进行删除
-   `selectById(Serializable id)` 根据主键 id 进行查找
-   `selectBatchIds(Collection idList)` 根据主键 id 进行批量查找
-   `selectByMap(Map<String,Object> map)` 根据 map 中指定的列名和列值进行**等值匹配**查找
-   `selectMaps(Wrapper<T> wrapper)` 根据 wrapper 条件，查询记录，将查询结果封装为一个 Map，Map 的 key 为结果的列，value 为值
-   `selectList(Wrapper<T> wrapper)` 根据条件构造器`wrapper`进行查询
-   `update(T entity, Wrapper<T> wrapper)` 根据条件构造器`wrapper`进行更新
-   `updateById(T entity)`
-   ...

简单的食用示例如前文[快速入门小节](#quickStart "\#quickStart")，下面讲解几个比较特别的方法

###### selectMaps

`BaseMapper`接口还提供了一个`selectMaps`方法，这个方法会将查询结果封装为一个 Map，Map 的 key 为结果的列，value 为值

该方法的使用场景如下：

-   **只查部分列**

    当某个表的列特别多，而 SELECT 的时候只需要选取个别列，查询出的结果也没必要封装成 Java 实体类对象时（只查部分列时，封装成实体后，实体对象中的很多属性会是 null），则可以用`selectMaps`，获取到指定的列后，再自行进行处理即可

    比如

    ```java
    	@Test
    	public void test3() {
    		QueryWrapper<User> wrapper = new QueryWrapper<>();
    		wrapper.select("id","name","email").likeRight("name","黄");
    		List<Map<String, Object>> maps = userMapper.selectMaps(wrapper);
    		maps.forEach(System.out::println);
    	}
    ```

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57e49bf8aaeb4f4f8535b85aa9f0f58b~tplv-k3u1fbpfcp-watermark.awebp)
-   **进行数据统计**

    比如

    ```java



    @Test
    public void test3() {
    	QueryWrapper<User> wrapper = new QueryWrapper<>();
    	wrapper.select("manager_id", "avg(age) avg_age", "min(age) min_age", "max(age) max_age")
    			.groupBy("manager_id").having("sum(age) < {0}", 500);
    	List<Map<String, Object>> maps = userMapper.selectMaps(wrapper);
    	maps.forEach(System.out::println);
    }
    ```

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db7d3148e6324d80ad2478018e22c299~tplv-k3u1fbpfcp-watermark.awebp)

###### selectObjs

只会返回第一个字段（第一列）的值，其他字段会被舍弃

比如

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		wrapper.select("id", "name").like("name", "黄");
		List<Object> objects = userMapper.selectObjs(wrapper);
		objects.forEach(System.out::println);
	}
```

得到的结果，只封装了第一列的 id

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79eb199104744f86814d3297bf84625c~tplv-k3u1fbpfcp-watermark.awebp)

###### selectCount

查询满足条件的总数，注意，使用这个方法，不能调用`QueryWrapper`的`select`方法设置要查询的列了。这个方法会自动添加`select count(1)`

比如

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		wrapper.like("name", "黄");

		Integer count = userMapper.selectCount(wrapper);
		System.out.println(count);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c84c94aa9c5c40719e499b2d6f4a61d9~tplv-k3u1fbpfcp-watermark.awebp)

#### Service CRUD 接口

另外一套 CRUD 是 Service 层的，只需要编写一个接口，继承`IService`，并创建一个接口实现类，即可食用。（这个接口提供的 CRUD 方法，和 Mapper 接口提供的功能大同小异，**比较明显的区别在于`IService`支持了更多的批量化操作**，如`saveBatch`，`saveOrUpdateBatch`等方法。

食用示例如下

1.  首先，新建一个接口，继承`IService`

    ```java
    package com.example.mp.service;

    import com.baomidou.mybatisplus.extension.service.IService;
    import com.example.mp.po.User;

    public interface UserService extends IService<User> {
    }
    ```
2.  创建这个接口的实现类，并继承`ServiceImpl`，最后打上`@Service`注解，注册到 Spring 容器中，即可食用

    ```java
    package com.example.mp.service.impl;

    import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
    import com.example.mp.mappers.UserMapper;
    import com.example.mp.po.User;
    import com.example.mp.service.UserService;
    import org.springframework.stereotype.Service;

    @Service
    public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService { }
    ```
3.  测试代码

    ```java
    package com.example.mp;

    import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
    import com.baomidou.mybatisplus.core.toolkit.Wrappers;
    import com.example.mp.po.User;
    import com.example.mp.service.UserService;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class ServiceTest {
    	@Autowired
    	private UserService userService;
    	@Test
    	public void testGetOne() {
    		LambdaQueryWrapper<User> wrapper = Wrappers.<User>lambdaQuery();
    		wrapper.gt(User::getAge, 28);
    		User one = userService.getOne(wrapper, false); 
    		System.out.println(one);
    	}
    }
    ```
4.  结果

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aa7114c01424c368f70f1fa7d30b51a~tplv-k3u1fbpfcp-watermark.awebp)

另，`IService`也支持链式调用，代码写起来非常简洁，查询示例如下

```java
	@Test
	public void testChain() {
		List<User> list = userService.lambdaQuery()
				.gt(User::getAge, 39)
				.likeRight(User::getName, "王")
				.list();
		list.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca85bb9ee8144265a1014e97becb3d7d~tplv-k3u1fbpfcp-watermark.awebp)

更新示例如下

```java
	@Test
	public void testChain() {
		userService.lambdaUpdate()
				.gt(User::getAge, 39)
				.likeRight(User::getName, "王")
				.set(User::getEmail, "w39@baomidou.com")
				.update();
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8879182f2b8742268b53926264e4f23d~tplv-k3u1fbpfcp-watermark.awebp)

删除示例如下

```java
	@Test
	public void testChain() {
		userService.lambdaUpdate()
				.like(User::getName, "青蛙")
				.remove();
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bd05bd172044a89958a92bd19d01bc1~tplv-k3u1fbpfcp-watermark.awebp)

### 条件构造器

mp 让我觉得极其方便的一点在于其提供了强大的条件构造器`Wrapper`，可以非常方便的构造 WHERE 条件。条件构造器主要涉及到 3 个类，`AbstractWrapper`。`QueryWrapper`，`UpdateWrapper`，它们的类关系如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c76a5a69cf94a768cb1c9b9a20527f9~tplv-k3u1fbpfcp-watermark.awebp)

在`AbstractWrapper`中提供了非常多的方法用于构建 WHERE 条件，而`QueryWrapper`针对`SELECT`语句，提供了`select()`方法，可自定义需要查询的列，而`UpdateWrapper`针对`UPDATE`语句，提供了`set()`方法，用于构造`set`语句。条件构造器也支持 lambda 表达式，写起来非常舒爽。

下面对`AbstractWrapper`中用于构建 SQL 语句中的 WHERE 条件的方法进行部分列举

-   `eq`：equals，等于
-   `allEq`：all equals，全等于
-   `ne`：not equals，不等于
-   `gt`：greater than ，大于 `>`
-   `ge`：greater than or equals，大于等于`≥`
-   `lt`：less than，小于`<`
-   `le`：less than or equals，小于等于`≤`
-   `between`：相当于 SQL 中的 BETWEEN
-   `notBetween`
-   `like`：模糊匹配。`like("name","黄")`，相当于 SQL 的`name like '% 黄 %'`
-   `likeRight`：模糊匹配右半边。`likeRight("name","黄")`，相当于 SQL 的`name like '黄 %'`
-   `likeLeft`：模糊匹配左半边。`likeLeft("name","黄")`，相当于 SQL 的`name like '% 黄'`
-   `notLike`：`notLike("name","黄")`，相当于 SQL 的`name not like '% 黄 %'`
-   `isNull`
-   `isNotNull`
-   `in`
-   `and`：SQL 连接符 AND
-   `or`：SQL 连接符 OR
-   `apply`：用于拼接 SQL，该方法可用于数据库函数，并可以动态传参
-   .......

#### 使用示例

下面通过一些具体的案例来练习条件构造器的使用。（使用前文创建的`user`表）

```java




QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.like("name", "佳").lt("age", 25);
List<User> users = userMapper.selectList(wrapper);




wrapper.likeRight("name","黄").between("age", 20, 40).isNotNull("email");



wrapper.likeRight("name","黄").or().ge("age",40).orderByDesc("age").orderByAsc("id");



wrapper.apply("date_format(create_time, '%Y-%m-%d') = {0}", "2021-03-22")  
				.inSql("manager_id", "SELECT id FROM user WHERE name like '李%'");

wrapper.apply("date_format(create_time, '%Y-%m-%d') = '2021-03-22'");



wrapper.likeRight("name", "王").and(q -> q.lt("age", 40).or().isNotNull("email"));



wrapper.likeRight("name", "王").or(
				q -> q.lt("age",40)
						.gt("age",20)
						.isNotNull("email")
		);



wrapper.nested(q -> q.lt("age", 40).or().isNotNull("email"))
				.likeRight("name", "王");



wrapper.in("age", Arrays.asList(30,31,34,35));

wrapper.inSql("age","30,31,34,35");



wrapper.in("age", Arrays.asList(30,31,34,35)).last("LIMIT 1");



wrapper.select("id", "name");



wrapper.select(User.class, info -> {
			String columnName = info.getColumn();
			return !"create_time".equals(columnName) && !"manager_id".equals(columnName);
		});
```

#### Condition

条件构造器的诸多方法中，均可以指定一个`boolean`类型的参数`condition`，用来决定该条件是否加入最后生成的 WHERE 语句中，比如

```java
String name = "黄"; 
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.like(StringUtils.hasText(name), "name", name);


if (StringUtils.hasText(name)) {
	wrapper.like("name", name);
}
```

#### 实体对象作为条件

调用构造函数创建一个`Wrapper`对象时，可以传入一个实体对象。后续使用这个`Wrapper`时，会以实体对象中的非空属性，构建 WHERE 条件（默认构建**等值匹配**的 WHERE 条件，这个行为可以通过实体类里各个字段上的`@TableField`注解中的`condition`属性进行改变）

示例如下

```java
	@Test
	public void test3() {
		User user = new User();
		user.setName("黄主管");
		user.setAge(28);
		QueryWrapper<User> wrapper = new QueryWrapper<>(user);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

执行结果如下。可以看到，是根据实体对象中的非空属性，进行了**等值匹配查询**。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bdffbad3acb4061b16d54e8acb37c9c~tplv-k3u1fbpfcp-watermark.awebp)

若希望针对某些属性，改变**等值匹配**的行为，则可以在实体类中用`@TableField`注解进行配置，示例如下

```java
package com.example.mp.po;
import com.baomidou.mybatisplus.annotation.SqlCondition;
import com.baomidou.mybatisplus.annotation.TableField;
import lombok.Data;
import java.time.LocalDateTime;
@Data
public class User {
	private Long id;
	@TableField(condition = SqlCondition.LIKE)   
	private String name;
	private Integer age;
	private String email;
	private Long managerId;
	private LocalDateTime createTime;
}
```

运行下面的测试代码

```java
	@Test
	public void test3() {
		User user = new User();
		user.setName("黄");
		QueryWrapper<User> wrapper = new QueryWrapper<>(user);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

从下图得到的结果来看，对于实体对象中的`name`字段，采用了`like`进行拼接

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/016db90924ac41468413a5d795660fd3~tplv-k3u1fbpfcp-watermark.awebp)

`@TableField`中配置的`condition`属性实则是一个字符串，`SqlCondition`类中预定义了一些字符串以供选择

```java
package com.baomidou.mybatisplus.annotation;

public class SqlCondition {
    
    public static final String EQUAL = "%s=#{%s}";
    public static final String NOT_EQUAL = "%s&lt;&gt;#{%s}";
    public static final String LIKE = "%s LIKE CONCAT('%%',#{%s},'%%')";
    public static final String LIKE_LEFT = "%s LIKE CONCAT('%%',#{%s})";
    public static final String LIKE_RIGHT = "%s LIKE CONCAT(#{%s},'%%')";
}
```

`SqlCondition`中提供的配置比较有限，当我们需要`<`或`>`等拼接方式，则需要自己定义。比如

```java
package com.example.mp.po;
import com.baomidou.mybatisplus.annotation.SqlCondition;
import com.baomidou.mybatisplus.annotation.TableField;
import lombok.Data;
import java.time.LocalDateTime;
@Data
public class User {
	private Long id;
	@TableField(condition = SqlCondition.LIKE)
	private String name;
    @TableField(condition = "%s &gt; #{%s}")   
	private Integer age;
	private String email;
	private Long managerId;
	private LocalDateTime createTime;
}
```

测试如下

```java
	@Test
	public void test3() {
		User user = new User();
		user.setName("黄");
        user.setAge(30);
		QueryWrapper<User> wrapper = new QueryWrapper<>(user);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

从下图得到的结果，可以看出，`name`属性是用`like`拼接的，而`age`属性是用`>`拼接的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/116ea0d2cd9e4afab5b8c1c0796cbbd7~tplv-k3u1fbpfcp-watermark.awebp)

#### allEq 方法

allEq 方法传入一个`map`，用来做等值匹配

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		Map<String, Object> param = new HashMap<>();
		param.put("age", 40);
		param.put("name", "黄飞飞");
		wrapper.allEq(param);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/389ad865105f47afa408fe02ce4c37dc~tplv-k3u1fbpfcp-watermark.awebp)

当 allEq 方法传入的 Map 中有 value 为`null`的元素时，默认会设置为`is null`

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		Map<String, Object> param = new HashMap<>();
		param.put("age", 40);
		param.put("name", null);
		wrapper.allEq(param);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/131f77c3158c446483220f24c59aa9bb~tplv-k3u1fbpfcp-watermark.awebp)

若想忽略 map 中 value 为`null`的元素，可以在调用 allEq 时，设置参数`boolean null2IsNull`为`false`

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		Map<String, Object> param = new HashMap<>();
		param.put("age", 40);
		param.put("name", null);
		wrapper.allEq(param, false);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/729448eb09be4152b5d2edbcf04ca26d~tplv-k3u1fbpfcp-watermark.awebp)

若想要在执行 allEq 时，过滤掉 Map 中的某些元素，可以调用 allEq 的重载方法`allEq(BiPredicate<R, V> filter, Map<R, V> params)`

```java
	@Test
	public void test3() {
		QueryWrapper<User> wrapper = new QueryWrapper<>();
		Map<String, Object> param = new HashMap<>();
		param.put("age", 40);
		param.put("name", "黄飞飞");
		wrapper.allEq((k,v) -> !"name".equals(k), param); 
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ca69fe3b1df43fabad6561b73fef454~tplv-k3u1fbpfcp-watermark.awebp)

#### lambda 条件构造器

lambda 条件构造器，支持 lambda 表达式，可以不必像普通条件构造器一样，以字符串形式指定列名，它可以直接以实体类的**方法引用**来指定列。示例如下

```java
	@Test
	public void testLambda() {
		LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
		wrapper.like(User::getName, "黄").lt(User::getAge, 30);
		List<User> users = userMapper.selectList(wrapper);
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4de34693032b4475b9bbb9e2e7f7047a~tplv-k3u1fbpfcp-watermark.awebp)

像普通的条件构造器，列名是用字符串的形式指定，无法在编译期进行列名合法性的检查，这就不如 lambda 条件构造器来的优雅。

另外，还有个**链式 lambda 条件构造器**，使用示例如下

```java
	@Test
	public void testLambda() {
		LambdaQueryChainWrapper<User> chainWrapper = new LambdaQueryChainWrapper<>(userMapper);
		List<User> users = chainWrapper.like(User::getName, "黄").gt(User::getAge, 30).list();
		users.forEach(System.out::println);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/953528e18c224debbfcd3aa1d142f42c~tplv-k3u1fbpfcp-watermark.awebp)

### 更新操作

上面介绍的都是查询操作, 现在来讲更新和删除操作。

`BaseMapper`中提供了 2 个更新方法

-   `updateById(T entity)`

    根据入参`entity`的`id`（主键）进行更新，对于`entity`中非空的属性，会出现在 UPDATE 语句的 SET 后面，即`entity`中非空的属性，会被更新到数据库，示例如下

    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class UpdateTest {
    	@Autowired
    	private UserMapper userMapper;
    	@Test
    	public void testUpdate() {
    		User user = new User();
    		user.setId(2L);
    		user.setAge(18);
    		userMapper.updateById(user);
    	}
    }
    ```

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2961355a9344df492bd33931893ccd0~tplv-k3u1fbpfcp-watermark.awebp)
-   `update(T entity, Wrapper<T> wrapper)`

    根据实体`entity`和条件构造器`wrapper`进行更新，示例如下

    ```java
    	@Test
    	public void testUpdate2() {
    		User user = new User();
    		user.setName("王三蛋");
    		LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
    		wrapper.between(User::getAge, 26,31).likeRight(User::getName,"吴");
    		userMapper.update(user, wrapper);
    	}
    ```

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27f6c97894b84c9992291ba7a992934d~tplv-k3u1fbpfcp-watermark.awebp)

    额外演示一下，把实体对象传入`Wrapper`，即用实体对象构造 WHERE 条件的案例

    ```java
    	@Test
    	public void testUpdate3() {
    		User whereUser = new User();
    		whereUser.setAge(40);
    		whereUser.setName("王");

    		LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>(whereUser);
    		User user = new User();
    		user.setEmail("share@baomidou.com");
    		user.setManagerId(10L);

    		userMapper.update(user, wrapper);
    	}
    ```

    注意到我们的 User 类中，对`name`属性和`age`属性进行了如下的设置

    ```java
    @Data
    public class User {
    	private Long id;
    	@TableField(condition = SqlCondition.LIKE)
    	private String name;
    	@TableField(condition = "%s &gt; #{%s}")
    	private Integer age;
    	private String email;
    	private Long managerId;
    	private LocalDateTime createTime;
    }
    ```

    执行结果

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ef2c7235be04cc69f47b508ffde3397~tplv-k3u1fbpfcp-watermark.awebp)

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc91cbe05ae440839cc5c416cf0ea714~tplv-k3u1fbpfcp-watermark.awebp)

    再额外演示一下，链式 lambda 条件构造器的使用

    ```java
    	@Test
    	public void testUpdate5() {
    		LambdaUpdateChainWrapper<User> wrapper = new LambdaUpdateChainWrapper<>(userMapper);
    		wrapper.likeRight(User::getEmail, "share")
    				.like(User::getName, "飞飞")
    				.set(User::getEmail, "ff@baomidou.com")
    				.update();
    	}
    ```

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91607128bfcd48508db78dd47b7b541b~tplv-k3u1fbpfcp-watermark.awebp)

**反思**

由于`BaseMapper`提供的 2 个更新方法都是传入一个实体对象去执行更新，这**在需要更新的列比较多时还好**，若想要更新的只有那么一列，或者两列，则创建一个实体对象就显得有点麻烦。针对这种情况，`UpdateWrapper`提供有`set`方法，可以手动拼接 SQL 中的 SET 语句，此时可以不必传入实体对象，示例如下

```java
	@Test
	public void testUpdate4() {
		LambdaUpdateWrapper<User> wrapper = new LambdaUpdateWrapper<>();
		wrapper.likeRight(User::getEmail, "share").set(User::getManagerId, 9L);
		userMapper.update(null, wrapper);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ac50fdf867740cc9612f085af26014d~tplv-k3u1fbpfcp-watermark.awebp)

### 删除操作

`BaseMapper`一共提供了如下几个用于删除的方法

-   `deleteById` 根据主键 id 进行删除
-   `deleteBatchIds` 根据主键 id 进行批量删除
-   `deleteByMap` 根据 Map 进行删除（Map 中的 key 为列名，value 为值，根据列和值进行等值匹配）
-   `delete(Wrapper<T> wrapper)` 根据条件构造器`Wrapper`进行删除

与前面查询和更新的操作大同小异，不做赘述

### 自定义 SQL

当 mp 提供的方法还不能满足需求时，则可以自定义 SQL。

#### 原生 mybatis

示例如下

-   注解方式

```java
package com.example.mp.mappers;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mp.po.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;


public interface UserMapper extends BaseMapper<User> {
	
	@Select("select * from user")
	List<User> selectRaw();
}
```

-   xml 方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mp.mappers.UserMapper">
	<select id="selectRaw" resultType="com.example.mp.po.User">
        SELECT * FROM user
    </select>
</mapper>
```

```java
package com.example.mp.mappers;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mp.po.User;
import org.apache.ibatis.annotations.Select;
import java.util.List;

public interface UserMapper extends BaseMapper<User> {
	List<User> selectRaw();
}
```

使用 xml 时，**若 xml 文件与 mapper 接口文件不在同一目录下**，则需要在`application.yml`中配置 mapper.xml 的存放路径

```yaml
mybatis-plus:
  mapper-locations: /mappers/*
```

若有多个地方存放 mapper，则用数组形式进行配置

```yaml
mybatis-plus:
  mapper-locations: 
  - /mappers/*
  - /com/example/mp/*
```

测试代码如下

```java
	@Test
	public void testCustomRawSql() {
		List<User> users = userMapper.selectRaw();
		users.forEach(System.out::println);
	}
```

结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aba4a4b899af4c8ba8467ef3b5186dc6~tplv-k3u1fbpfcp-watermark.awebp)

#### mybatis-plus

也可以使用 mp 提供的 Wrapper 条件构造器，来自定义 SQL

示例如下

-   注解方式

```java
package com.example.mp.mappers;
import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Constants;
import com.example.mp.po.User;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import java.util.List;

public interface UserMapper extends BaseMapper<User> {

    
	@Select("select * from user ${ew.customSqlSegment}")
	List<User> findAll(@Param(Constants.WRAPPER)Wrapper<User> wrapper);
}
```

-   xml 方式

```java
package com.example.mp.mappers;
import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mp.po.User;
import java.util.List;

public interface UserMapper extends BaseMapper<User> {
	List<User> findAll(Wrapper<User> wrapper);
}
```

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mp.mappers.UserMapper">

    <select id="findAll" resultType="com.example.mp.po.User">
        SELECT * FROM user ${ew.customSqlSegment}
    </select>
</mapper>
```

### 分页查询

`BaseMapper`中提供了 2 个方法进行分页查询，分别是`selectPage`和`selectMapsPage`，前者会将查询的结果封装成 Java 实体对象，后者会封装成`Map<String,Object>`。分页查询的食用示例如下

1.  创建 mp 的分页拦截器，注册到 Spring 容器中

    ```java
    package com.example.mp.config;
    import com.baomidou.mybatisplus.annotation.DbType;
    import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
    import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class MybatisPlusConfig {

        
    	@Bean
    	public MybatisPlusInterceptor mybatisPlusInterceptor() {
    		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    		interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    		return interceptor;
    	}
        
    }
    ```
2.  执行分页查询

    ```java
    	@Test
    	public void testPage() {
    		LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    		wrapper.ge(User::getAge, 28);
            
    		Page<User> page = new Page<>(3, 2);
            
    		Page<User> userPage = userMapper.selectPage(page, wrapper);
    		System.out.println("总记录数 = " + userPage.getTotal());
    		System.out.println("总页数 = " + userPage.getPages());
    		System.out.println("当前页码 = " + userPage.getCurrent());
            
    		List<User> records = userPage.getRecords();
    		records.forEach(System.out::println);
    	}
    ```
3.  结果

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78d2c34e2a8c475590daa8c8806f0231~tplv-k3u1fbpfcp-watermark.awebp)
4.  其他

    -   注意到，分页查询总共发出了 2 次 SQL，一次查总记录数，一次查具体数据。**若希望不查总记录数，仅查分页结果**。可以通过`Page`的重载构造函数，指定`isSearchCount`为`false`即可

        ```java
        public Page(long current, long size, boolean isSearchCount)
        ```
    -   在实际开发中，可能遇到**多表联查**的场景，此时`BaseMapper`中提供的单表分页查询的方法无法满足需求，需要**自定义 SQL**，示例如下（使用单表查询的 SQL 进行演示，实际进行多表联查时，修改 SQL 语句即可）

        1.  在 mapper 接口中定义一个函数，接收一个 Page 对象为参数，并编写自定义 SQL

            ```java

            @Select("SELECT * FROM user ${ew.customSqlSegment}")
            Page<User> selectUserPage(Page<User> page, @Param(Constants.WRAPPER) Wrapper<User> wrapper);
            ```
        2.  执行查询

            ```java
            	@Test
            	public void testPage2() {
            		LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
            		wrapper.ge(User::getAge, 28).likeRight(User::getName, "王");
            		Page<User> page = new Page<>(3,2);
            		Page<User> userPage = userMapper.selectUserPage(page, wrapper);
            		System.out.println("总记录数 = " + userPage.getTotal());
            		System.out.println("总页数 = " + userPage.getPages());
            		userPage.getRecords().forEach(System.out::println);
            	}
            ```
        3.  结果

            ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01f1ee0a1cb343eb8169f8d63fe8d987~tplv-k3u1fbpfcp-watermark.awebp)

### AR 模式

ActiveRecord 模式，通过操作实体对象，直接操作数据库表。与 ORM 有点类似。

示例如下

1.  让实体类`User`继承自`Model`

    ```java
    package com.example.mp.po;

    import com.baomidou.mybatisplus.annotation.SqlCondition;
    import com.baomidou.mybatisplus.annotation.TableField;
    import com.baomidou.mybatisplus.extension.activerecord.Model;
    import lombok.Data;
    import lombok.EqualsAndHashCode;
    import java.time.LocalDateTime;

    @EqualsAndHashCode(callSuper = false)
    @Data
    public class User extends Model<User> {
    	private Long id;
    	@TableField(condition = SqlCondition.LIKE)
    	private String name;
    	@TableField(condition = "%s &gt; #{%s}")
    	private Integer age;
    	private String email;
    	private Long managerId;
    	private LocalDateTime createTime;
    }

    ```
2.  直接调用实体对象上的方法

    ```java
    	@Test
    	public void insertAr() {
    		User user = new User();
    		user.setId(15L);
    		user.setName("我是AR猪");
    		user.setAge(1);
    		user.setEmail("ar@baomidou.com");
    		user.setManagerId(1L);
    		boolean success = user.insert(); 
    		System.out.println(success);
    	}
    ```
3.  结果

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b80cfc9a1cb4642a124c7ab499ef898~tplv-k3u1fbpfcp-watermark.awebp)

其他示例

```java
	
	@Test
	public void selectAr() {
		User user = new User();
        user.setId(15L);
		User result = user.selectById();
		System.out.println(result);
	}
	
	@Test
	public void updateAr() {
		User user = new User();
		user.setId(15L);
		user.setName("王全蛋");
		user.updateById();
	}
	
	@Test
	public void deleteAr() {
		User user = new User();
		user.setId(15L);
		user.deleteById();
	}
```

### 主键策略

在定义实体类时，用`@TableId`指定主键，而其`type`属性，可以指定主键策略。

mp 支持多种主键策略，默认的策略是基于雪花算法的自增 id。全部主键策略定义在了枚举类`IdType`中，`IdType`有如下的取值

-   `AUTO`

    数据库 ID 自增，**依赖于数据库**。在插入操作生成 SQL 语句时，不会插入主键这一列
-   `NONE`

    未设置主键类型。若在代码中没有手动设置主键，则会根据**主键的全局策略**自动生成（默认的主键全局策略是基于雪花算法的自增 ID）
-   `INPUT`

    需要手动设置主键，若不设置。插入操作生成 SQL 语句时，主键这一列的值会是`null`。oracle 的序列主键需要使用这种方式
-   `ASSIGN_ID`

    当没有手动设置主键，即实体类中的主键属性为空时，才会自动填充，使用雪花算法
-   `ASSIGN_UUID`

    当实体类的主键属性为空时，才会自动填充，使用 UUID
-   ....（还有几种是已过时的，就不再列举）

可以针对每个实体类，使用`@TableId`注解指定该实体类的主键策略，这可以理解为**局部策略**。若希望对所有的实体类，都采用同一种主键策略，挨个在每个实体类上进行配置，则太麻烦了，此时可以用主键的**全局策略**。只需要在`application.yml`进行配置即可。比如，配置了全局采用自增主键策略

```yaml

mybatis-plus:
  global-config:
    db-config:
      id-type: auto
```

下面对不同主键策略的行为进行演示

-   `AUTO`

    在`User`上对`id`属性加上注解，然后将 MYSQL 的`user`表修改其主键为自增。

    ```java
    @EqualsAndHashCode(callSuper = false)
    @Data
    public class User extends Model<User> {
    	@TableId(type = IdType.AUTO)
    	private Long id;
    	@TableField(condition = SqlCondition.LIKE)
    	private String name;
    	@TableField(condition = "%s &gt; #{%s}")
    	private Integer age;
    	private String email;
    	private Long managerId;
    	private LocalDateTime createTime;
    }
    ```

    测试

    ```java
    	@Test
    	public void testAuto() {
    		User user = new User();
    		user.setName("我是青蛙呱呱");
    		user.setAge(99);
    		user.setEmail("frog@baomidou.com");
    		user.setCreateTime(LocalDateTime.now());
    		userMapper.insert(user);
            System.out.println(user.getId());
    	}
    ```

    结果

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b406358f82b49c2937e15b3663b1966~tplv-k3u1fbpfcp-watermark.awebp)

    可以看到，代码中没有设置主键 ID，发出的 SQL 语句中也没有设置主键 ID，并且插入结束后，主键 ID 会被写回到实体对象。
-   `NONE`

    在 MYSQL 的`user`表中，去掉主键自增。然后修改`User`类（若不配置`@TableId`注解，默认主键策略也是`NONE`）

    ```java
    @TableId(type = IdType.NONE)
    private Long id;
    ```

    插入时，若实体类的主键 ID 有值，则使用之；若主键 ID 为空，则使用主键全局策略，来生成一个 ID。
-   其余的策略类似，不赘述

**小结**

`AUTO`依赖于数据库的自增主键，插入时，实体对象无需设置主键，插入成功后，主键会被写回实体对象。

`INPUT`完全依赖于用户输入。实体对象中主键 ID 是什么，插入到数据库时就设置什么。若有值便设置值，若为`null`则设置`null`

其余的几个策略，都是在实体对象中主键 ID 为空时，才会自动生成。

`NONE`会跟随全局策略，`ASSIGN_ID`采用雪花算法，`ASSIGN_UUID`采用 UUID

全局配置，在`application.yml`中进行即可；针对单个实体类的局部配置，使用`@TableId`即可。对于某个实体类，若它有局部主键策略，则采用之，否则，跟随全局策略。

### 配置

mybatis plus 有许多可配置项，可在`application.yml`中进行配置，如上面的全局主键策略。下面列举部分配置项

#### 基本配置

-   `configLocation`：若有单独的 mybatis 配置，用这个注解指定 mybatis 的配置文件（mybatis 的全局配置文件）
-   `mapperLocations`：mybatis mapper 所对应的 xml 文件的位置
-   `typeAliasesPackage`：mybatis 的别名包扫描路径
-   .....

#### 进阶配置

-   `mapUnderscoreToCamelCase`：是否开启自动驼峰命名规则映射。（默认开启）
-   `dbTpe`：数据库类型。一般不用配，会根据数据库连接 url 自动识别
-   `fieldStrategy`：[（已过时）字段验证策略。**该配置项在最新版的 mp 文档中已经找不到了**，被细分成了`insertStrategy`，`updateStrategy`，`selectStrategy`。默认值是`NOT_NULL`，即对于实体对象中非空的字段，才会组装到最终的 SQL 语句中。](https://link.juejin.cn/?target=undefined)
        [

        有如下几种可选配置

        *   `IGNORED`：忽略校验。即，不做校验。实体对象中的全部字段，无论值是什么，都如实地被组装到SQL语句中（为`NULL`的字段在SQL语句中就组装为`NULL`）。
            
        *   `NOT_NULL`：非`NULL`校验。只会将非`NULL`的字段组装到SQL语句中
            
        *   `NOT_EMPTY`：非空校验。当有字段是字符串类型时，只组装非空字符串；对其他类型的字段，等同于`NOT_NULL`
            
        *   `NEVER`：不加入SQL。所有字段不加入到SQL语句
            

        这个配置项，可在`application.yml`中进行**全局配置**，也可以在某一实体类中，对某一字段用`@TableField`注解进行**局部配置**

        这个字段验证策略有什么用呢？在UPDATE操作中能够体现出来，若用一个`User`对象执行UPDATE操作，我们希望只对`User`对象中非空的属性，更新到数据库中，其他属性不做更新，则`NOT_NULL`可以满足需求。而若`updateStrategy`配置为`IGNORED`，则不会进行非空判断，会将实体对象中的全部属性如实组装到SQL中，这样，执行UPDATE时，可能就将一些不想更新的字段，设置为了`NULL`。

        ](https://link.juejin.cn/?target=undefined)
    \[\*   `tablePrefix`：添加表名前缀
        比如

        ```yaml
        mybatis-plus:
          global-config:
            db-config:
              table-prefix: xx_
        ```

        然后将MYSQL中的表做一下修改。但Java实体类保持不变（仍然为`User`）。

        ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e55c4cfc839e42e5ab01e42168ec31e4~tplv-k3u1fbpfcp-watermark.awebp)

        测试

        ```java
        	@Test
        	public void test3() {
        		QueryWrapper<User> wrapper = new QueryWrapper<>();
        		wrapper.like("name", "黄");
        		Integer count = userMapper.selectCount(wrapper);
        		System.out.println(count);
        	}
        ```

        可以看到拼接出来的SQL，在表名前面添加了前缀

        ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f55785c4c1b146cbb31084f6fd08833d~tplv-k3u1fbpfcp-watermark.awebp)
        ](https://link.juejin.cn/?target=undefined)

[完整的配置可以参考 mp 的官网 ==>](https://link.juejin.cn/?target=undefined) [传送门](https://link.juejin.cn/?target=https%3A%2F%2Fbaomidou.com%2Fconfig%2F%23mapperlocations "https&#x3A;//baomidou.com/config/#mapperlocations")

### 代码生成器

mp 提供一个生成器，可快速生成 Entity 实体类，Mapper 接口，Service，Controller 等全套代码。

示例如下

```java
public class GeneratorTest {
	@Test
	public void generate() {
		AutoGenerator generator = new AutoGenerator();

		
		GlobalConfig config = new GlobalConfig();
		String projectPath = System.getProperty("user.dir");
		
		config.setOutputDir(projectPath + "/src/main/java");
		config.setAuthor("yogurt");
		
		config.setOpen(false);

		
		generator.setGlobalConfig(config);

		
		DataSourceConfig dataSourceConfig = new DataSourceConfig();
		dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/yogurt?serverTimezone=Asia/Shanghai");
		dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
		dataSourceConfig.setUsername("root");
		dataSourceConfig.setPassword("root");

		
		generator.setDataSource(dataSourceConfig);

		
		PackageConfig packageConfig = new PackageConfig();
		packageConfig.setParent("com.example.mp.generator");

		
		generator.setPackageInfo(packageConfig);

		
		StrategyConfig strategyConfig = new StrategyConfig();
		
		strategyConfig.setNaming(NamingStrategy.underline_to_camel);
		strategyConfig.setColumnNaming(NamingStrategy.underline_to_camel);
		
		strategyConfig.setEntityLombokModel(true);
		
		strategyConfig.setRestControllerStyle(true);
		generator.setStrategy(strategyConfig);
		generator.setTemplateEngine(new FreemarkerTemplateEngine());

        
		generator.execute();
	}
}
```

运行后，可以看到生成了如下图所示的全套代码

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9466749d3e7c48249cc76d6a0d297628~tplv-k3u1fbpfcp-watermark.awebp)

## 高级功能

高级功能的演示需要用到一张新的表`user2`

```sql
DROP TABLE IF EXISTS user2;
CREATE TABLE user2 (
id BIGINT(20) PRIMARY KEY NOT NULL COMMENT '主键id',
name VARCHAR(30) DEFAULT NULL COMMENT '姓名',
age INT(11) DEFAULT NULL COMMENT '年龄',
email VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
manager_id BIGINT(20) DEFAULT NULL COMMENT '直属上级id',
create_time DATETIME DEFAULT NULL COMMENT '创建时间',
update_time DATETIME DEFAULT NULL COMMENT '修改时间',
version INT(11) DEFAULT '1' COMMENT '版本',
deleted INT(1) DEFAULT '0' COMMENT '逻辑删除标识,0-未删除,1-已删除',
CONSTRAINT manager_fk FOREIGN KEY(manager_id) REFERENCES user2(id)
) ENGINE = INNODB CHARSET=UTF8;

INSERT INTO user2(id, name, age, email, manager_id, create_time)
VALUES
(1, '老板', 40 ,'boss@baomidou.com' ,NULL, '2021-03-28 13:12:40'),
(2, '王狗蛋', 40 ,'gd@baomidou.com' ,1, '2021-03-28 13:12:40'),
(3, '王鸡蛋', 40 ,'jd@baomidou.com' ,2, '2021-03-28 13:12:40'),
(4, '王鸭蛋', 40 ,'yd@baomidou.com' ,2, '2021-03-28 13:12:40'),
(5, '王猪蛋', 40 ,'zd@baomidou.com' ,2, '2021-03-28 13:12:40'),
(6, '王软蛋', 40 ,'rd@baomidou.com' ,2, '2021-03-28 13:12:40'),
(7, '王铁蛋', 40 ,'td@baomidou.com' ,2, '2021-03-28 13:12:40')
```

并创建对应的实体类`User2`

```java
package com.example.mp.po;
import lombok.Data;
import java.time.LocalDateTime;
@Data
public class User2 {
	private Long id;
	private String name;
	private Integer age;
	private String email;
	private Long managerId;
	private LocalDateTime createTime;
	private LocalDateTime updateTime;
	private Integer version;
	private Integer deleted;
}
```

以及 Mapper 接口

```java
package com.example.mp.mappers;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.mp.po.User2;
public interface User2Mapper extends BaseMapper<User2> { }
```

### 逻辑删除

首先，为什么要有逻辑删除呢？直接删掉不行吗？当然可以，但日后若想要恢复，或者需要查看这些数据，就做不到了。**逻辑删除是为了方便数据恢复，和保护数据本身价值的一种方案**。

日常中，我们在电脑中删除一个文件后，也仅仅是把该文件放入了回收站，日后若有需要还能进行查看或恢复。当我们确定不再需要某个文件，可以将其从回收站中彻底删除。这也是类似的道理。

mp 提供的逻辑删除实现起来非常简单

只需要在`application.yml`中进行逻辑删除的相关配置即可

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted  
      logic-delete-value: 1 
      logic-not-delete-value: 0 
      
```

测试代码

```java
package com.example.mp;
import com.example.mp.mappers.User2Mapper;
import com.example.mp.po.User2;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import java.util.List;
@RunWith(SpringRunner.class)
@SpringBootTest
public class LogicDeleteTest {
	@Autowired
	private User2Mapper mapper;
	@Test
	public void testLogicDel() {
		int i = mapper.deleteById(6);
		System.out.println("rowAffected = " + i);
	}
}
```

结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dc09ad6ded347ae9f4299e2b26147a9~tplv-k3u1fbpfcp-watermark.awebp)

可以看到，发出的 SQL 不再是`DELETE`，而是`UPDATE`

此时我们再执行一次`SELECT`

```java
	@Test
	public void testSelect() {
		List<User2> users = mapper.selectList(null);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c2495022b8143548449e4cef24a3ba9~tplv-k3u1fbpfcp-watermark.awebp)

可以看到，发出的 SQL 语句，会自动在 WHERE 后面拼接逻辑未删除的条件。查询出来的结果中，没有了 id 为 6 的王软蛋。

若想要 SELECT 的列，不包括逻辑删除的那一列，则可以在实体类中通过`@TableField`进行配置

```java
@TableField(select = false)
private Integer deleted;
```

可以看到下图的执行结果中，SELECT 中已经不包含 deleted 这一列了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/219e174e679842119514d16ace049f46~tplv-k3u1fbpfcp-watermark.awebp)

前面在`application.yml`中做的配置，是全局的。通常来说，对于多个表，我们也会统一逻辑删除字段的名称，统一逻辑已删除和未删除的值，所以全局配置即可。当然，若要对某些表进行单独配置，在实体类的对应字段上使用`@TableLogic`即可

```java
@TableLogic(value = "0", delval = "1")
private Integer deleted;
```

**小结**

开启 mp 的逻辑删除后，会对 SQL 产生如下的影响

-   INSERT 语句：没有影响
-   SELECT 语句：追加 WHERE 条件，过滤掉已删除的数据
-   UPDATE 语句：追加 WHERE 条件，防止更新到已删除的数据
-   DELETE 语句：转变为 UPDATE 语句

\*\*注意，上述的影响，只针对 mp 自动注入的 SQL 生效。\*\*如果是自己手动添加的自定义 SQL，则不会生效。比如

```java
public interface User2Mapper extends BaseMapper<User2> {
	@Select("select * from user2")
	List<User2> selectRaw();
}
```

调用这个`selectRaw`，则 mp 的逻辑删除不会生效。

另，逻辑删除可在`application.yml`中进行全局配置，也可在实体类中用`@TableLogic`进行局部配置。

### 自动填充

表中常常会有 “新增时间”，“修改时间”，“操作人” 等字段。比较原始的方式，是每次插入或更新时，手动进行设置。mp 可以通过配置，对某些字段进行自动填充，食用示例如下

1.  在实体类中的某些字段上，通过`@TableField`设置自动填充

    ```java
    public class User2 {
    	private Long id;
    	private String name;
    	private Integer age;
    	private String email;
    	private Long managerId;
    	@TableField(fill = FieldFill.INSERT) 
    	private LocalDateTime createTime;
    	@TableField(fill = FieldFill.UPDATE) 
    	private LocalDateTime updateTime;
    	private Integer version;
    	private Integer deleted;
    }
    ```
2.  实现自动填充处理器

    ```java
    package com.example.mp.component;
    import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
    import org.apache.ibatis.reflection.MetaObject;
    import org.springframework.stereotype.Component;
    import java.time.LocalDateTime;

    @Component 
    public class MyMetaObjectHandler implements MetaObjectHandler {

    	@Override
    	public void insertFill(MetaObject metaObject) {
            
            
    		strictFillStrategy(metaObject, "createTime", LocalDateTime::now);
    	}

    	@Override
    	public void updateFill(MetaObject metaObject) {
            
    		strictFillStrategy(metaObject, "updateTime", LocalDateTime::now);
    	}
    }
    ```

测试

```java
	@Test
	public void test() {
		User2 user = new User2();
		user.setId(8L);
		user.setName("王一蛋");
		user.setAge(29);
		user.setEmail("yd@baomidou.com");
		user.setManagerId(2L);
		mapper.insert(user);
	}
```

根据下图结果，可以看到对 createTime 进行了自动填充

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48c0d98bc67a4826abf8e3b5b61d6c94~tplv-k3u1fbpfcp-watermark.awebp)

注意，自动填充仅在该字段为空时会生效，若该字段不为空，则直接使用已有的值。如下

```java
	@Test
	public void test() {
		User2 user = new User2();
		user.setId(8L);
		user.setName("王一蛋");
		user.setAge(29);
		user.setEmail("yd@baomidou.com");
		user.setManagerId(2L);
		user.setCreateTime(LocalDateTime.of(2000,1,1,8,0,0));
		mapper.insert(user);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53518f87b1f542949d2e871e6e73fae8~tplv-k3u1fbpfcp-watermark.awebp)

更新时的自动填充，测试如下

```java
	@Test
	public void test() {
		User2 user = new User2();
		user.setId(8L);
		user.setName("王一蛋");
		user.setAge(99);
		mapper.updateById(user);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9657626a8d2405dafc9689c4d36cd79~tplv-k3u1fbpfcp-watermark.awebp)

### 乐观锁插件

当出现并发操作时，需要确保各个用户对数据的操作不产生冲突，此时需要一种并发控制手段。悲观锁的方法是，在对数据库的一条记录进行修改时，先直接加锁（数据库的锁机制），锁定这条数据，然后再进行操作；而乐观锁，正如其名，它先假设不存在冲突情况，而在实际进行数据操作时，再检查是否冲突。乐观锁的一种通常实现是**版本号**，在 MySQL 中也有名为 MVCC 的基于版本号的并发事务控制。

在读多写少的场景下，乐观锁比较适用，能够减少加锁操作导致的性能开销，提高系统吞吐量。

在写多读少的场景下，悲观锁比较使用，否则会因为乐观锁不断失败重试，反而导致性能下降。

乐观锁的实现如下：

1.  取出记录时，获取当前 version
2.  更新时，带上这个 version
3.  执行更新时， set version = newVersion where version = oldVersion
4.  如果 oldVersion 与数据库中的 version 不一致，就更新失败

这种思想和 CAS（Compare And Swap）非常相似。

乐观锁的实现步骤如下

1.  配置乐观锁插件

    ```java
    package com.example.mp.config;

    import com.baomidou.mybatisplus.extension.plugins.inner.OptimisticLockerInnerInterceptor;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class MybatisPlusConfig {
        
    	@Bean
    	public MybatisPlusInterceptor mybatisPlusInterceptor() {
    		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    		interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    		return interceptor;
    	}
        
        
    }
    ```
2.  在实体类中表示版本的字段上添加注解`@Version`

    ```java
    @Data
    public class User2 {
    	private Long id;
    	private String name;
    	private Integer age;
    	private String email;
    	private Long managerId;
    	private LocalDateTime createTime;
    	private LocalDateTime updateTime;
    	@Version
    	private Integer version;
    	private Integer deleted;
    }
    ```

测试代码

```java
	@Test
	public void testOpLocker() {
		int version = 1; 
		User2 user = new User2();
		user.setId(8L);
		user.setEmail("version@baomidou.com");
		user.setVersion(version);
		int i = mapper.updateById(user);
	}
```

执行之前先看一下数据库的情况

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cefca48254c84576b78f540f351c2eec~tplv-k3u1fbpfcp-watermark.awebp)

根据下图执行结果，可以看到 SQL 语句中添加了 version 相关的操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3849f4e1b06e47e8a6cba4fc639095a8~tplv-k3u1fbpfcp-watermark.awebp)

当 UPDATE 返回了 1，表示影响行数为 1，则更新成功。反之，由于 WHERE 后面的 version 与数据库中的不一致，匹配不到任何记录，则影响行数为 0，表示更新失败。更新成功后，新的 version 会被封装回实体对象中。

实体类中 version 字段，类型只支持 int，long，Date，Timestamp，LocalDateTime

**注意，乐观锁插件仅支持`updateById(id)`与`update(entity, wrapper)`方法**

\*\*注意：如果使用`wrapper`，则`wrapper`不能复用！\*\*示例如下

```java
	@Test
	public void testOpLocker() {
		User2 user = new User2();
		user.setId(8L);
		user.setVersion(1);
		user.setAge(2);

		
		LambdaQueryWrapper<User2> wrapper = new LambdaQueryWrapper<>();
		wrapper.eq(User2::getName, "王一蛋");
		mapper.update(user, wrapper);

		
		user.setAge(3);
		mapper.update(user, wrapper);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d131591999c840d888301a9d8b891aa2~tplv-k3u1fbpfcp-watermark.awebp)

可以看到在第二次复用`wrapper`时，拼接出的 SQL 中，后面 WHERE 语句中出现了 2 次 version，是有问题的。

### 性能分析插件

该插件会输出 SQL 语句的执行时间，以便做 SQL 语句的性能分析和调优。

注：3.2.0 版本之后，mp 自带的性能分析插件被官方移除了，而推荐食用第三方性能分析插件

食用步骤

1.  引入 maven 依赖

    ```java
    <dependency>
        <groupId>p6spy</groupId>
        <artifactId>p6spy</artifactId>
        <version>3.9.1</version>
    </dependency>
    ```
2.  修改`application.yml`

    ```yaml
    spring:
      datasource:
        driver-class-name: com.p6spy.engine.spy.P6SpyDriver 
        url: jdbc:p6spy:mysql://localhost:3306/yogurt?serverTimezone=Asia/Shanghai 
        username: root
        password: root
    ```
3.  在`src/main/resources`资源目录下添加`spy.properties`

    ```properties


    modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory



    logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger

    appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger





    deregisterdrivers=true

    useprefix=true

    excludecategories=info,debug,result,commit,resultset

    dateformat=yyyy-MM-dd HH:mm:ss

    outagedetection=true

    outagedetectioninterval=2

    executionThreshold=10
    ```

随便运行一个测试用例，可以看到该 SQL 的执行时长被记录了下来

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/815f2088a83143e8bb49c6a737cc7c90~tplv-k3u1fbpfcp-watermark.awebp)

### 多租户 SQL 解析器

多租户的概念：多个用户共用一套系统，但他们的数据有需要相对的独立，保持一定的隔离性。

多租户的数据隔离一般有如下的方式：

-   不同租户使用不同的数据库服务器

    优点是：不同租户有不同的独立数据库，有助于扩展，以及对不同租户提供更好的个性化，出现故障时恢复数据较为简单。

    缺点是：增加了数据库数量，购置成本，维护成本更高
-   不同租户使用相同的数据库服务器，但使用不同的数据库（不同的 schema）

    优点是购置和维护成本低了一些，缺点是数据恢复较为困难，因为不同租户的数据都放在了一起
-   不同租户使用相同的数据库服务器，使用相同的数据库，共享数据表，在表中增加租户 id 来做区分

    优点是，购置和维护成本最低，支持用户最多，缺点是隔离性最低，安全性最低

食用实例如下

添加多租户拦截器配置。添加配置后，在执行 CRUD 的时候，会自动在 SQL 语句最后拼接租户 id 的条件

```java
package com.example.mp.config;

import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.handler.TenantLineHandler;
import com.baomidou.mybatisplus.extension.plugins.inner.TenantLineInnerInterceptor;
import net.sf.jsqlparser.expression.Expression;
import net.sf.jsqlparser.expression.LongValue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisPlusConfig {

	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
		interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(new TenantLineHandler() {
			@Override
			public Expression getTenantId() {
				
                
				return new LongValue(1);
			}

            
			@Override
			public String getTenantIdColumn() {
				
				return "manager_id";
			}

			@Override
			public boolean ignoreTable(String tableName) {
				
				return !"user2".equals(tableName);
			}
		}));
        
        
        
		return interceptor;
	}

}
```

测试代码

```java
	@Test
	public void testTenant() {
		LambdaQueryWrapper<User2> wrapper = new LambdaQueryWrapper<>();
		wrapper.likeRight(User2::getName, "王")
				.select(User2::getName, User2::getAge, User2::getEmail, User2::getManagerId);
		user2Mapper.selectList(wrapper);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c34ec3a62d58423b8fe664e334bbabf8~tplv-k3u1fbpfcp-watermark.awebp)

### 动态表名 SQL 解析器

当数据量特别大的时候，我们通常会采用分库分表。这时，可能就会有多张表，其表结构相同，但表名不同。例如`order_1`，`order_2`，`order_3`，查询时，我们可能需要动态设置要查的表名。mp 提供了动态表名 SQL 解析器，食用示例如下

先在 mysql 中拷贝一下`user2`表

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07265365b5e64b0faa29a80d3854b343~tplv-k3u1fbpfcp-watermark.awebp)

配置动态表名拦截器

```java
package com.example.mp.config;

import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.handler.TableNameHandler;
import com.baomidou.mybatisplus.extension.plugins.inner.DynamicTableNameInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Random;

@Configuration
public class MybatisPlusConfig {

	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
		DynamicTableNameInnerInterceptor dynamicTableNameInnerInterceptor = new DynamicTableNameInnerInterceptor();
		HashMap<String, TableNameHandler> map = new HashMap<>();
        
		map.put("user2", (sql, tableName) -> {
			String _ = "_";
			int random = new Random().nextInt(2) + 1;
			return tableName + _ + random; 
		});
		dynamicTableNameInnerInterceptor.setTableNameHandlerMap(map);
		interceptor.addInnerInterceptor(dynamicTableNameInnerInterceptor);
		return interceptor;
	}

}

```

测试

```java
	@Test
	public void testDynamicTable() {
		user2Mapper.selectList(null);
	}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8c924f7259e496da5331ac234a6d997~tplv-k3u1fbpfcp-watermark.awebp)

## 总结

-   条件构造器`AbstractWrapper`中提供了多个方法用于构造 SQL 语句中的 WHERE 条件，而其子类`QueryWrapper`额外提供了`select`方法，可以只选取特定的列，子类`UpdateWrapper`额外提供了`set`方法，用于设置 SQL 中的 SET 语句。除了普通的`Wrapper`，还有基于 lambda 表达式的`Wrapper`，如`LambdaQueryWrapper`，`LambdaUpdateWrapper`，它们在构造 WHERE 条件时，直接以**方法引用**来指定 WHERE 条件中的列，比普通`Wrapper`通过字符串来指定要更加优雅。另，还有**链式 Wrapper**，如`LambdaQueryChainWrapper`，它封装了`BaseMapper`，可以更方便地获取结果。
-   条件构造器采用**链式调用**来拼接多个条件，条件之间默认以`AND`连接
-   当`AND`或`OR`后面的条件需要被括号包裹时，将括号中的条件以 lambda 表达式形式，作为参数传入`and()`或`or()`

    特别的，当`()`需要放在 WHERE 语句的最开头时，可以使用`nested()`方法
-   条件表达式时当需要传入自定义的 SQL 语句，或者需要调用数据库函数时，可用`apply()`方法进行 SQL 拼接
-   条件构造器中的各个方法可以通过一个`boolean`类型的变量`condition`，来根据需要灵活拼接 WHERE 条件（仅当`condition`为`true`时会拼接 SQL 语句）
-   使用 lambda 条件构造器，可以通过 lambda 表达式，直接使用实体类中的属性进行条件构造，比普通的条件构造器更加优雅
-   若 mp 提供的方法不够用，可以通过**自定义 SQL**（原生 mybatis）的形式进行扩展开发
-   使用 mp 进行分页查询时，需要创建一个分页拦截器（Interceptor），注册到 Spring 容器中，随后查询时，通过传入一个分页对象（Page 对象）进行查询即可。单表查询时，可以使用`BaseMapper`提供的`selectPage`或`selectMapsPage`方法。复杂场景下（如多表联查），使用自定义 SQL。
-   AR 模式可以直接通过操作实体类来操作数据库。让实体类继承自`Model`即可

（完） 
 [https://juejin.cn/post/6961721367846715428](https://juejin.cn/post/6961721367846715428)
