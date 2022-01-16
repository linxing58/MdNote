# Redis实现Mybatis的二级缓存
## 一、Mybatis 的缓存

通大多数 ORM 层框架一样，Mybatis 自然也提供了对一级缓存和二级缓存的支持。一下是一级缓存和二级缓存的作用于和定义。

      1、一级缓存是 SqlSession 级别的缓存。在操作数据库时需要构造 sqlSession 对象，在对象中有一个 (内存区域) 数据结构（HashMap）用于存储缓存数据。不同的 sqlSession 之间的缓存数据区域（HashMap）是互相不影响的。

二级缓存是 mapper 级别的缓存，多个 SqlSession 去操作同一个 Mapper 的 sql 语句，多个 SqlSession 去操作数据库得到数据会存在二级缓存区域，多个 SqlSession 可以共用二级缓存，二级缓存是跨 SqlSession 的。 

      2、一级缓存的作用域是同一个 SqlSession，在同一个 sqlSession 中两次执行相同的 sql 语句，第一次执行完毕会将数据库中查询的数据写 到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个 sqlSession 结束后该 sqlSession 中的一级缓存 也就不存在了。Mybatis 默认开启一级缓存。

      二级缓存是多个 SqlSession 共享的，其作用域是 mapper 的同一个 namespace，不同的 sqlSession 两次执行相同 namespace 下的 sql 语句且向 sql 中传递参数也相同即最终执行相同的 sql 语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二 次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis 默认没有开启二级缓存需要在 setting 全局参数中配置开启二级缓存。

      一般的我们将 Mybatis 和 Spring 整合时，mybatis-spring 包会自动分装 sqlSession，而 Spring 通过动态代理 sqlSessionProxy 使用一个模板方法封装了 select() 等操作，每一次 select() 查询都会自动先执行 openSession()， 执行完 close() 以后调用 close() 方法，相当于生成了一个新的 session 实例，所以我们无需手动的去关闭这个 session()，当然也无 法使用 mybatis 的一级缓存，也就是说 mybatis 的一级缓存在 spring 中是没有作用的。

      因此我们一般在项目中实现 Mybatis 的二级缓存，虽然 Mybatis 自带二级缓存功能，但是如果实在集群环境下，使用自带的二级缓存只是针对单个的节 点，所以我们采用分布式的二级缓存功能。一般的缓存 NoSql 数据库如 redis，Mancache 等，或者 EhCache 都可以实现，从而更好地服务 tomcat 集群中 ORM 的查询。

## 二、Mybatis 的二级缓存的实现

下面主要通过 Redis 实现 Mybatis 的二级缓存功能。

### 1、配置文件中开启二级缓存

1.  <setting name="cacheEnabled" value="true"/>  

   默认二级缓存是开启的。

### 2、实现 Mybatis 的 Cache 接口

     Mybatis 提供了第三方 Cache 实现的接口，我们自定义 MybatisRedisCache 实现 Cache 接口，代码如下：

10. public class MybatisRedisCache implements Cache {  

11.     private static final Logger LOG = Logger.getLogger(MybatisRedisCache.class);   

12.     private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock(true);  

13.     private RedisTemplate&lt;Serializable, Serializable> redisTemplate =  (RedisTemplate&lt;Serializable, Serializable>) SpringContextHolder.getBean("redisTemplate");   

14.     private String id;  

15.     private JdkSerializationRedisSerializer jdkSerializer = new JdkSerializationRedisSerializer();  

16.     public MybatisRedisCache(final String id){  

17.         if(id == null){  

18.             throw new IllegalArgumentException("Cache instances require an ID");  

19.         }  

20.         LOG.info("Redis Cache id" + id);  

21.         this.id = id;  

22.     }  

23.     @Override  

24.     public String getId() {  

25.         return this.id;  

26.     }  

27.     @Override  

28.     public void putObject(Object key, Object value) {  

29.         if(value != null){  

30.             redisTemplate.opsForValue().set(key.toString(), jdkSerializer.serialize(value), 2, TimeUnit.DAYS);  

31.         }  

32.     }  

33.     @Override  

34.     public Object getObject(Object key) {  

35.         try {  

36.             if(key != null){  

37.                 Object obj = redisTemplate.opsForValue().get(key.toString());  

38.                 return jdkSerializer.deserialize((byte\[])obj);   

39.             }  

40.         } catch (Exception e) {  

41.             LOG.error("redis");  

42.         }  

43.         return null;  

44.     }  

45.     @Override  

46.     public Object removeObject(Object key) {  

47.         try {  

48.             if(key != null){  

49.                 redisTemplate.expire(key.toString(), 1, TimeUnit.SECONDS);  

50.             }  

51.         } catch (Exception e) {  

52.         }  

53.         return null;  

54.     }  

55.     @Override  

56.     public void clear() {  

57.     }  

58.     @Override  

59.     public int getSize() {  

60.         Long size = redisTemplate.getMasterRedisTemplate().execute(new RedisCallback<Long>(){  

61.             @Override  

62.             public Long doInRedis(RedisConnection connection)  

63.                     throws DataAccessException {  

64.                 return connection.dbSize();  

65.             }  

66.         });  

67.         return size.intValue();  

68.     }  

69.     @Override  

70.     public ReadWriteLock getReadWriteLock() {  

71.         return this.readWriteLock;  

72.     }  

73. }  

### 3、二级缓存的实用

我们需要将所有的实体类进行序列化，然后在 Mapper 中添加自定义 cache 功能。

1.    &lt;cache  
2.      type="org.andy.shop.cache.MybatisRedisCache"  
3.  eviction="LRU"  
4.  flushInterval="6000000"  
5.  size="1024"  
6.  readOnly="false"  
7.  />  

### 

4、Redis 中的存储

     redis 会自动的将 Sql + 条件 + Hash 等当做 key 值，而将查询结果作为 value，只有请求中的所有参数都符合，那么就会使用 redis 中的二级缓存。其查询结果如下：

    ![](https://img-blog.csdn.net/20160125175605103?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 
 [https://www.cnblogs.com/lykxqhh/p/5690941.html](https://www.cnblogs.com/lykxqhh/p/5690941.html)
