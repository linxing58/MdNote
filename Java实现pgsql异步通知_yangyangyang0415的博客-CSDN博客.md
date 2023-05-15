# Java实现pgsql异步通知_yangyangyang0415的博客-CSDN博客
```sql
create or replace function _table_update_notify_() returns trigger as $$
begin
    perform _pg_notify_('table_update',json_build_object('table',TG_TABLE_NAME,'timestamp',current_timestamp)::text);
    return new;
end;
$$ language plpgsql;

```

函数被调用会执行pg_notify()方法 实现异步通知

```sql
drop trigger if exists n_user_u on test.demo."user";
create trigger n_user_u after insert or update or delete on test.demo."user"
> for each statement execute procedure _table_update_notify_();

```

对应的数据表有 insert or update or delete 操作就会触发触发器，触发器调用上一步创建的方法

新建SpringBoot项目,  
Java中LISTEN / [NOTIFY](https://so.csdn.net/so/search?q=NOTIFY&spm=1001.2101.3001.7020)的JDBC驱动程序不支持真正的事件驱动通知。你必须经常轮询数据库以查看是否有新的通知。  
所以jdbc选用pgsql-ng，支持异步通知  
pgsql-ng文档地址 [http://impossibl.github.io/pgjdbc-ng/](http://impossibl.github.io/pgjdbc-ng/)

引入依赖

> _  
> <_dependency_>  
> <_groupId_>_com.impossibl.pgjdbc-ng_</_groupId_>  
> <_artifactId_>_pgjdbc-ng_</_artifactId_>  
> <_version_>_0.8.2_</_version_>  
> </_dependency_>_

编写监听器

```java
_@Configuration
public class Pglistener _{
_    _@Autowired
    DataSource dataSource;
    @Bean
    public void pgListen_() _throws SQLException _{
_        _System._out_.println_(_"初始化监听器"_)_;
        _
        _Connection connection = dataSource.getConnection_()_;
        _
        _PGConnection pgConnection = connection.unwrap_(_PGConnection.class_)_;
        _
        _pgConnection.addNotificationListener_(_new PGNotificationListener_() {
            _@Override
            public void notification_(_int processId, String channelName, String payload_) {
_                _System._out_.println_(_"Received Notification: " + processId + ", " + channelName + ", " + payload_)_;
            _}
_            _@Override
            public void closed_() {
                _PGNotificationListener.super.closed_()_;
            _}
        })_;
        _
        _Statement stmt = pgConnection.createStatement_()_;
        System._out_.println_(_"LISTEN table_update"_)_;
        stmt.executeUpdate_(_"LISTEN table_update"_)_;


    _}
}_

```

_  
此时修改监听的表，程序会收到pgsql的异步通知，打印日志_

![](https://img-blog.csdnimg.cn/d23f856efbfc4f34bd4f9555e2e3560c.png)