# 终于理解Spring Boot 为什么青睐HikariCP了，图解的太透彻了！ - 掘金
## 前言

现在已经有很多公司在使用 HikariCP 了，HikariCP 还成为了 SpringBoot 默认的连接池，伴随着 SpringBoot 和微服务，HikariCP 必将迎来广泛的普及。

下面陈某带大家从源码角度分析一下 HikariCP 为什么能够被 Spring Boot 青睐，文章目录如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b912ede303204277b55aa7e76809898c~tplv-k3u1fbpfcp-watermark.awebp)

## 零、类图和流程图

开始前先来了解下 HikariCP 获取一个连接时类间的交互流程，方便下面详细流程的阅读。

获取连接时的类间交互：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-1.png?imageView2/0/w/1024)

## 一、主流程 1：获取连接流程

HikariCP 获取连接时的入口是`HikariDataSource`里的`getConnection`方法，现在来看下该方法的具体流程：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-2.png?imageView2/0/w/1024)

上述为 HikariCP 获取连接时的流程图，由图 1 可知，每个`datasource`对象里都会持有一个`HikariPool`对象，记为 pool，初始化后的 datasource 对象 pool 是空的，所以第一次`getConnection`的时候会进行`实例化 pool`属性（参考`主流程 1`），初始化的时候需要将当前 datasource 里的`config 属性`传过去，用于 pool 的初始化，最终标记`sealed`，然后根据 pool 对象调用`getConnection`方法（参考`流程 1.1`），获取成功后返回连接对象。

## 二、主流程 2：初始化池对象

![](http://myblog.sharemer.com/2019/08/28/20190828-1-3.png?imageView2/0/w/1024)

该流程用于`初始化整个连接池`，这个流程会给连接池内所有的属性做初始化的工作，其中比较主要的几个流程上图已经指出，简单概括一下：

1.  利用`config`初始化各种连接池属性，并且产生一个用于`生产物理连接`的数据源`DriverDataSource`
2.  初始化存放连接对象的核心类`connectionBag`
3.  初始化一个延时任务线程池类型的对象`houseKeepingExecutorService`，用于后续执行一些延时 / 定时类任务（比如连接泄漏检查延时任务，参考`流程 2.2`以及`主流程 4`，除此之外`maxLifeTime`后主动回收关闭连接也是交由该对象来执行的，这个过程可以参考`主流程 3`）
4.  预热连接池，HikariCP 会在该流程的`checkFailFast`里初始化好一个连接对象放进池子内，当然触发该流程得保证`initializationTimeout > 0`时（默认值 1），这个配置属性表示留给预热操作的时间（默认值 1 在预热失败时不会发生重试）。与`Druid`通过`initialSize`控制预热连接对象数不一样的是，HikariCP 仅预热进池一个连接对象。
5.  初始化一个线程池对象`addConnectionExecutor`，用于后续扩充连接对象
6.  初始化一个线程池对象`closeConnectionExecutor`，用于关闭一些连接对象，怎么触发关闭任务呢？可以参考`流程 1.1.2`

## 三、流程 1.1：通过 HikariPool 获取连接对象

![](http://myblog.sharemer.com/2019/08/28/20190828-1-4.png?imageView2/0/w/1024)

从最开始的结构图可知，每个`HikariPool`里都维护一个`ConcurrentBag`对象，用于存放连接对象，由上图可以看到，实际上`HikariPool`的`getConnection`就是从`ConcurrentBag`里获取连接的（调用其`borrow`方法获得，对应`ConnectionBag 主流程`），在长连接检查这块，与之前说的`Druid`不同，这里的长连接判活检查在连接对象没有被标记为 “`已丢弃`” 时，只要距离上次使用超过`500ms`每次取出都会进行检查（500ms 是默认值，可通过配置`com.zaxxer.hikari.aliveBypassWindowMs`的系统参数来控制），emmmm，也就是说`HikariCP`对长连接的活性检查很频繁，但是其并发性能依旧优于`Druid`，说明频繁的长连接检查并不是导致连接池性能高低的关键所在。

> 作者的 Spring Boot 专栏、Mybatis 专栏已经完成，关注公众号【码猿技术专栏】回复关键词`Spring Boot 进阶`、`Mybatis 进阶`获取。

这个其实是由于 HikariCP 的`无锁`实现，在高并发时对 CPU 的负载没有其他连接池那么高而产生的并发性能差异，后面会说 HikariCP 的具体做法，即使是`Druid`，在`获取连接`、`生成连接`、`归还连接`时都进行了`锁控制`，因为通过上篇解析`Druid`的文章可以知道，`Druid`里的连接池资源是多线程共享的，不可避免的会有锁竞争，有锁竞争意味着线程状态的变化会很频繁，线程状态变化频繁意味着 CPU 上下文切换也将会很频繁。

回到`流程 1.1`，如果拿到的连接为空，直接报错，不为空则进行相应的检查，如果检查通过，则包装成`ConnectionProxy`对象返回给业务方，不通过则调用`closeConnection`方法关闭连接（对应`流程 1.1.2`，该流程会触发`ConcurrentBag`的`remove`方法丢弃该连接，然后把实际的驱动连接交给`closeConnectionExecutor`线程池，异步关闭驱动连接）。

## 四、流程 1.1.1：连接判活

![](http://myblog.sharemer.com/2019/08/28/20190828-1-5.png?imageView2/0/w/1024)

承接上面的`流程 1.1`里的判活流程，来看下判活是如何做的，首先说验证方法（注意这里该方法接受的这个`connection`对象不是`poolEntry`，而是`poolEntry`持有的实际驱动的连接对象），在之前介绍 Druid 的时候就知道，Druid 是根据驱动程序里是否存在`ping 方法`来判断是否启用 ping 的方式判断连接是否存活，但是到了 HikariCP 则更加简单粗暴，仅根据是否配置了`connectionTestQuery`觉定是否启用 ping：

```java
this.isUseJdbc4Validation = config.getConnectionTestQuery() == null;
```

所以一般驱动如果不是特别低的版本，**不建议配置该项**，否则便会走`createStatement+excute`的方式，相比`ping`简单发送心跳数据，这种方式显然更低效。

此外，这里在刚进来还会通过驱动的连接对象重新给它设置一遍`networkTimeout`的值，使之变成`validationTimeout`，表示一次验证的超时时间，为啥这里要重新设置这个属性呢？因为在使用 ping 方法校验时，是没办法通过类似`statement`那样可以`setQueryTimeout`的，所以只能由网络通信的超时时间来控制，这个时间可以通过`jdbc`的连接参数`socketTimeout`来控制：

    jdbc:mysql://127.0.0.1:3306/xxx?socketTimeout=250
    复制代码

这个值最终会被赋值给 HikariCP 的`networkTimeout`字段，这就是为什么最后那一步使用这个字段来还原驱动连接超时属性的原因；说到这里，最后那里为啥要再次还原呢？这就很容易理解了，因为验证结束了，连接对象还存活的情况下，它的`networkTimeout`的值这时仍然等于`validationTimeout`（不合预期），显然在拿出去用之前，需要恢复成本来的值，也就是 HikariCP 里的`networkTimeout`属性。

## 五、流程 1.1.2：关闭连接对象

![](http://myblog.sharemer.com/2019/08/28/20190828-1-6.png?imageView2/0/w/1024)

这个流程简单来说就是把`流程 1.1.1`中验证不通过的死连接，主动关闭的一个流程，首先会把这个连接对象从`ConnectionBag`里`移除`，然后把实际的物理连接交给一个线程池去异步执行，这个线程池就是在`主流程 2`里初始化池的时候初始化的线程池`closeConnectionExecutor`，然后异步任务内开始实际的关连接操作，因为主动关闭了一个连接相当于少了一个连接，所以还会触发一次扩充连接池（参考`主流程 5`）操作。

## 六、流程 2.1：HikariCP 监控设置

不同于 Druid 那样监控指标那么多，HikariCP 会把我们非常关心的几项指标暴露给我们，比如当前连接池内闲置连接数、总连接数、一个连接被用了多久归还、创建一个物理连接花费多久等，HikariCP 的连接池的监控我们这一节专门详细的分解一下，首先找到 HikariCP 下面的`metrics`文件夹，这下面放置了一些规范实现的监控接口等，还有一些现成的实现（比如 HikariCP 自带对`prometheus`、`micrometer`、`dropwizard`的支持，不太了解后面两个，`prometheus`下文直接称为`普罗米修斯`）：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-7.png?imageView2/0/w/500)

下面，来着重看下接口的定义：

```java

public interface IMetricsTracker extends AutoCloseable
{
    
    default void recordConnectionCreatedMillis(long connectionCreatedMillis) {}

    
    default void recordConnectionAcquiredNanos(final long elapsedAcquiredNanos) {}

    
    default void recordConnectionUsageMillis(final long elapsedBorrowedMillis) {}

    
    default void recordConnectionTimeout() {}

    @Override
    default void close() {}
}
```

触发点都了解清楚后，再来看看`MetricsTrackerFactory`的接口定义：

```java

public interface MetricsTrackerFactory
{
    
    IMetricsTracker create(String poolName, PoolStats poolStats);
}
```

上面的接口用法见注释，针对新出现的`PoolStats`类，我们来看看它做了什么：

```java
public abstract class PoolStats {
    private final AtomicLong reloadAt; 
    private final long timeoutMs; 

    
    protected volatile int totalConnections;
    
    protected volatile int idleConnections;
    
    protected volatile int activeConnections;
    
    protected volatile int pendingThreads;
    
    protected volatile int maxConnections;
    
    protected volatile int minConnections;

    public PoolStats(final long timeoutMs) {
        this.timeoutMs = timeoutMs;
        this.reloadAt = new AtomicLong();
    }

    
    public int getMaxConnections() {
        if (shouldLoad()) { 
            update(); 
        }

        return maxConnections;
    }
    
    protected abstract void update(); 

    private boolean shouldLoad() { 
        for (; ; ) {
            final long now = currentTime();
            final long reloadTime = reloadAt.get();
            if (reloadTime > now) {
                return false;
            } else if (reloadAt.compareAndSet(reloadTime, plusMillis(now, timeoutMs))) {
                return true;
            }
        }
    }
}
```

实际上这里就是这些属性获取和触发刷新的地方，那么这个对象是在哪里被生成并且丢给`MetricsTrackerFactory`的`create`方法的呢？这就是本节所需要讲述的要点：`主流程 2`里的设置监控器的流程，来看看那里发生了什么事吧：

```java

public void setMetricsTrackerFactory(MetricsTrackerFactory metricsTrackerFactory) {
    if (metricsTrackerFactory != null) {
        
        
        this.metricsTracker = new MetricsTrackerDelegate(metricsTrackerFactory.create(config.getPoolName(), getPoolStats()));
    } else {
        
        this.metricsTracker = new NopMetricsTrackerDelegate();
    }
}

private PoolStats getPoolStats() {
    
    return new PoolStats(SECONDS.toMillis(1)) {
        @Override
        protected void update() {
            
            this.pendingThreads = HikariPool.this.getThreadsAwaitingConnection();
            this.idleConnections = HikariPool.this.getIdleConnections();
            this.totalConnections = HikariPool.this.getTotalConnections();
            this.activeConnections = HikariPool.this.getActiveConnections();
            this.maxConnections = config.getMaximumPoolSize();
            this.minConnections = config.getMinimumIdle();
        }
    };
}
```

到这里 HikariCP 的监控器就算是注册进去了，所以要想实现自己的监控器拿到上面的指标，要经过如下步骤：

1.  新建一个类实现`IMetricsTracker`接口，我们这里将该类记为`IMetricsTrackerImpl`
2.  新建一个类实现`MetricsTrackerFactory`接口，我们这里将该类记为`MetricsTrackerFactoryImpl`，并且将上面的`IMetricsTrackerImpl`在其`create 方法`内实例化
3.  将`MetricsTrackerFactoryImpl`实例化后调用 HikariPool 的`setMetricsTrackerFactory`方法注册到 Hikari 连接池。

上面没有提到`PoolStats`里的属性怎么监控，这里来说下，由于`create 方法`是调用一次就没了，`create 方法`只是接收了`PoolStats`对象的实例，如果不处理，那么随着 create 调用的结束，这个实例针对监控模块来说就失去持有了，所以这里如果想要拿到`PoolStats`里的属性，就需要开启一个`守护线程`，让其持有`PoolStats`对象实例，并且定时获取其内部属性值，然后`push`给监控系统，如果是普罗米修斯等使用`pull 方式`获取监控数据的监控系统，可以效仿 HikariCP 原生普罗米修斯监控的实现，自定义一个`Collector`对象来接收`PoolStats`实例，这样普罗米修斯就可以定期拉取了，比如 HikariCP 根据普罗米修斯监控系统自己定义的`MetricsTrackerFactory`实现（对应`图 2`里的`PrometheusMetricsTrackerFactory`类）：

```java
@Override
public IMetricsTracker create(String poolName, PoolStats poolStats) {
    getCollector().add(poolName, poolStats); 
    return new PrometheusMetricsTracker(poolName, this.collectorRegistry); 
}


private HikariCPCollector getCollector() {
    if (collector == null) {
        
        collector = new HikariCPCollector().register(this.collectorRegistry);
    }
    return collector;
```

通过上面的解释可以知道在 HikariCP 中如何自定义一个自己的监控器，以及相比 Druid 的监控，有什么区别。 工作中很多时候都是需要自定义的，我司虽然也是用的普罗米修斯监控，但是因为 HikariCP 原生的普罗米修斯收集器里面对监控指标的命名并`不符合我司的规范`，所以就`自定义`了一个，有类似问题的不妨也试一试。

🍁 这一节没有画图，纯代码，因为画图不太好解释这部分的东西，这部分内容与连接池整体流程关系也不大，充其量获取了连接池本身的一些属性，在连接池里的触发点也在上面代码段的注释里说清楚了，看代码定义可能更好理解一些。

## 七、流程 2.2：连接泄漏的检测与告警

本节对应`主流程 2`里的`子流程 2.2`，在初始化池对象时，初始化了一个叫做`leakTaskFactory`的属性，本节来看下它具体是用来做什么的。

### 7.1：它是做什么的？

一个连接被拿出去使用时间超过`leakDetectionThreshold`（可配置，默认 0）未归还的，会触发一个连接泄漏警告，通知业务方目前存在连接泄漏的问题。

### 7.2：过程详解

该属性是`ProxyLeakTaskFactory`类型对象，且它还会持有`houseKeepingExecutorService`这个线程池对象，用于生产`ProxyLeakTask`对象，然后利用上面的`houseKeepingExecutorService`延时运行该对象里的`run`方法。该流程的触发点在上面的`流程 1.1`最后包装成`ProxyConnection`对象的那一步，来看看具体的流程图：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-8.png?imageView2/0/w/1024)

每次在`流程 1.1`那里生成`ProxyConnection`对象时，都会触发上面的流程，由流程图可以知道，`ProxyConnection`对象持有`PoolEntry`和`ProxyLeakTask`的对象，其中初始化`ProxyLeakTask`对象时就用到了`leakTaskFactory`对象，通过其`schedule`方法可以进行`ProxyLeakTask`的初始化，并将其实例传递给`ProxyConnection`进行初始化赋值（ps：由图知`ProxyConnection`在触发回收事件时，会主动取消这个泄漏检查任务，这也是`ProxyConnection`需要持有`ProxyLeakTask`对象的原因）。

在上面的流程图中可以知道，只有在`leakDetectionThreshold`不等于 0 的时候才会生成一个带有实际延时任务的`ProxyLeakTask`对象，否则返回无实际意义的空对象。所以要想启用连接泄漏检查，首先要把`leakDetectionThreshold`配置设置上，这个属性表示经过该时间后借出去的连接仍未归还，则触发连接泄漏告警。

`ProxyConnection`之所以要持有`ProxyLeakTask`对象，是因为它可以监听到连接是否触发归还操作，如果触发，则调用`cancel`方法取消延时任务，防止误告。

由此流程可以知道，跟 Druid 一样，HikariCP 也有连接对象泄漏检查，与 Druid 主动回收连接相比，HikariCP 实现更加简单，仅仅是在触发时打印警告日志，不会采取具体的强制回收的措施。

与 Druid 一样，默认也是关闭这个流程的，因为实际开发中一般使用第三方框架，框架本身会保证及时的 close 连接，防止连接对象泄漏，开启与否还是取决于业务是否需要，如果一定要开启，如何设置`leakDetectionThreshold`的大小也是需要考虑的一件事。作者的 Spring Boot 专栏、Mybatis 专栏已经完成，关注公众号【码猿技术专栏】回复关键词`Spring Boot 进阶`、`Mybatis 进阶`获取。

## 八、主流程 3：生成连接对象

本节来讲下`主流程 2`里的`createEntry`方法，这个方法利用 PoolBase 里的`DriverDataSource`对象生成一个实际的连接对象（如果忘记`DriverDatasource`是哪里初始化的了，可以看下`主流程 2`里`PoolBase`的`initializeDataSource`方法的作用），然后用`PoolEntry`类包装成`PoolEntry`对象，现在来看下这个包装类有哪些主要属性：

```java
final class PoolEntry implements IConcurrentBagEntry {
    private static final Logger LOGGER = LoggerFactory.getLogger(PoolEntry.class);
    
    private static final AtomicIntegerFieldUpdater stateUpdater;

    Connection connection; 
    long lastAccessed; 
    long lastBorrowed; 

    @SuppressWarnings("FieldCanBeLocal")
    private volatile int state = 0; 
    private volatile boolean evict; 

    private volatile ScheduledFuture<?> endOfLife; 

    private final FastList openStatements; 
    private final HikariPool hikariPool; 

    private final boolean isReadOnly; 
    private final boolean isAutoCommit; 
}
```

上面就是整个`PoolEntry`对象里所有的属性，这里再说下`endOfLife`对象，它是一个利用`houseKeepingExecutorService`这个线程池对象做的延时任务，这个延时任务一般在创建好连接对象后`maxLifeTime`左右的时间触发，具体来看下`createEntry`代码：

```java
private PoolEntry createPoolEntry() {

        final PoolEntry poolEntry = newPoolEntry(); 

        final long maxLifetime = config.getMaxLifetime(); 
        if (maxLifetime > 0) { 
            
            
            final long variance = maxLifetime > 10_000 ? ThreadLocalRandom.current().nextLong(maxLifetime / 40) : 0;
            final long lifetime = maxLifetime - variance; 
            poolEntry.setFutureEol(houseKeepingExecutorService.schedule(
                    () -> { 
                        if (softEvictConnection(poolEntry, "(connection has passed maxLifetime)", false )) {
                            addBagItem(connectionBag.getWaitingThreadCount()); 
                        }
                    },
                    lifetime, MILLISECONDS)); 
        }

        return poolEntry;
    }

    
    public void addBagItem(final int waiting) {
        

        
        final boolean shouldAdd = waiting - addConnectionQueue.size() >= 0; 
        if (shouldAdd) {
            
            addConnectionExecutor.submit(poolEntryCreator);
        }
    }
```

通过上面的流程，可以知道，HikariCP 一般通过`createEntry`方法来新增一个连接入池，每个连接被包装成 PoolEntry 对象，在创建好对象时，同时会提交一个延时任务来关闭废弃该连接，这个时间就是我们配置的`maxLifeTime`，为了保证不在同一时间失效，HikariCP 还会利用`maxLifeTime`减去一个随机数作为最终的延时任务延迟时间，然后在触发废弃任务时，还会触发`addBagItem`，进行连接添加任务（因为废弃了一个连接，需要往池子里补充一个），该任务则交给由`主流程 2`里定义好的`addConnectionExecutor`线程池执行，那么，现在来看下这个异步添加连接对象的任务流程：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-9.png?imageView2/0/w/1024)

这个流程就是往连接池里加连接用的，跟`createEntry`结合起来说是因为这俩流程是紧密相关的，除此之外，`主流程 5`（`fillPool`，扩充连接池）也会触发该任务。

## 九、主流程 4：连接池缩容

HikariCP 会按照`minIdle`定时清理闲置过久的连接，这个定时任务在`主流程 2`初始化连接池对象时被启用，跟上面的流程一样，也是利用`houseKeepingExecutorService`这个线程池对象做该定时任务的执行器。

来看下`主流程 2`里是怎么启用该任务的：

```java

this.houseKeeperTask = houseKeepingExecutorService.scheduleWithFixedDelay(new HouseKeeper(), 100L, housekeepingPeriodMs, MILLISECONDS);
```

那么本节主要来说下`HouseKeeper`这个类，该类实现了`Runnable`接口，回收逻辑主要在其`run`方法内，来看看`run`方法的逻辑流程图：

![](http://myblog.sharemer.com/2019/08/28/20190828-1-10.png?imageView2/0/w/1024)

上面的流程就是`HouseKeeper`的 run 方法里具体做的事情，由于系统时间回拨会导致该定时任务回收一些连接时产生误差，因此存在如下判断：

```java




if (plusMillis(now, 128) < plusMillis(previous, housekeepingPeriodMs))
```

这是 hikariCP 在解决系统时钟被回拨时做出的一种措施，通过流程图可以看到，它是直接把池子里所有的连接对象取出来挨个儿的标记成废弃，并且尝试把状态值修改为`STATE_RESERVED`（后面会说明这些状态，这里先不深究）。如果系统时钟没有发生改变（绝大多数情况会命中这一块的逻辑），由图知，会把当前池内所有处于闲置状态（`STATE_NOT_IN_USE`）的连接拿出来，然后计算需要检查的范围，然后循环着修改连接的状态：

```java

final List notInUse = connectionBag.values(STATE_NOT_IN_USE);

int toRemove = notInUse.size() - config.getMinIdle();
for (PoolEntry entry : notInUse) {
  
  if (toRemove > 0 && elapsedMillis(entry.lastAccessed, now) > idleTimeout && connectionBag.reserve(entry)) {
    closeConnection(entry, "(connection has passed idleTimeout)"); 
    toRemove--;
  }
}
fillPool(); 
```

上面的代码就是流程图里对应的没有回拨系统时间时的流程逻辑。该流程在`idleTimeout`大于 0（默认等于 0）并且`minIdle`小于`maxPoolSize`的时候才会启用，默认是不启用的，若需要启用，可以按照条件来配置。

## 十、主流程 5：扩充连接池

这个流程主要依附 HikariPool 里的`fillPool`方法，这个方法已经在上面很多流程里出现过了，它的作用就是在触发连接废弃、连接池连接不够用时，发起扩充连接数的操作，这是个很简单的过程，下面看下源码（为了使代码结构更加清晰，对源码做了细微改动）：

```java


private final PoolEntryCreator poolEntryCreator = new PoolEntryCreator(null);
private final PoolEntryCreator postFillPoolEntryCreator = new PoolEntryCreator("After adding ");

private synchronized void fillPool() {
  
  
  int needAdd = Math.min(maxPoolSize - connectionBag.size(),
  minIdle - connectionBag.getCount(STATE_NOT_IN_USE));

  
  final int connectionsToAdd = needAdd - addConnectionQueue.size();
  for (int i = 0; i < connectionsToAdd; i++) {
    
    addConnectionExecutor.submit((i < connectionsToAdd - 1) ? poolEntryCreator : postFillPoolEntryCreator);
  }
}
```

由该过程可以知道，最终这个新增连接的任务也是交由`addConnectionExecutor`线程池来处理的，而任务的主题也是`PoolEntryCreator`，这个流程可以参考`主流程 3.`

然后`needAdd`的推算：

```java
Math.min(最大连接数 - 池内当前连接总数, 最小连接数 - 池内闲置的连接数)
```

根据这个方式判断，可以保证池内的连接数永远不会超过`maxPoolSize`，也永远不会低于`minIdle`。在连接吃紧的时候，可以保证每次触发都以`minIdle`的数量扩容。因此如果在`maxPoolSize`跟`minIdle`配置的值一样的话，在池内连接吃紧的时候，就不会发生任何扩容了。

## 十一、主流程 6：连接回收

最开始说过，最终真实的物理连接对象会被包装成`PoolEntry`对象，存放进`ConcurrentBag`，然后获取时，PoolEntry 对象又会被再次包装成`ProxyConnection`对象暴露给使用方的，那么触发连接回收，实际上就是触发 ProxyConnection 里的 close 方法：

```java
public final void close() throws SQLException {
  
  closeStatements(); 
  if (delegate != ClosedConnection.CLOSED_CONNECTION) {
    leakTask.cancel(); 
    try {
      if (isCommitStateDirty && !isAutoCommit) { 
        delegate.rollback(); 
        lastAccess = currentTime(); 
      }
    } finally {
      delegate = ClosedConnection.CLOSED_CONNECTION;
      poolEntry.recycle(lastAccess); 
    }
  }
}
```

这个就是 ProxyConnection 里的 close 方法，可以看到它最终会调用 PoolEntry 的 recycle 方法进行回收，除此之外，连接对象的最后一次使用时间也是在这个时候刷新的，该时间是个很重要的属性，可以用来判断一个连接对象的闲置时间，来看下 PoolEntry 的`recycle`方法：

```java
void recycle(final long lastAccessed) {
  if (connection != null) {
    this.lastAccessed = lastAccessed; 
    hikariPool.recycle(this); 
  }
}
```

之前有说过，每个 PoolEntry 对象都持有 HikariPool 的对象，方便触发连接池的一些操作，由上述代码可以看到，最终还是会触发 HikariPool 里的 recycle 方法，再来看下 HikariPool 的 recycle 方法：

```java
void recycle(final PoolEntry poolEntry) {
  metricsTracker.recordConnectionUsage(poolEntry); 
  connectionBag.requite(poolEntry); 
}
```

以上就是连接回收部分的逻辑，相比其他流程，还是比较简洁的。

## 十二、ConcurrentBag 主流程

这个类用来存放最终的 PoolEntry 类型的连接对象，提供了基本的增删查的功能，被 HikariPool 持有，上面那么多的操作，几乎都是在 HikariPool 中完成的，HikariPool 用来管理实际的连接生产动作和回收动作，实际操作的却是 ConcurrentBag 类，梳理下上面所有流程的触发点：

-   主流程 2：初始化 HikariPool 时初始化`ConcurrentBag（构造方法）`，预热时通过`createEntry`拿到连接对象，调用`ConcurrentBag.add`添加连接到 ConcurrentBag。
-   流程 1.1：通过 HikariPool 获取连接时，通过调用`ConcurrentBag.borrow`拿到一个连接对象。
-   主流程 6：通过`ConcurrentBag.requite`归还一个连接。
-   流程 1.1.2：触发关闭连接时，会通过`ConcurrentBag.remove`移除连接对象，由前面的流程可知关闭连接触发点为：连接超过最大生命周期 maxLifeTime 主动废弃、健康检查不通过主动废弃、连接池缩容。
-   主流程 3：通过异步添加连接时，通过调用`ConcurrentBag.add`添加连接到 ConcurrentBag，由前面的流程可知添加连接触发点为：连接超过最大生命周期 maxLifeTime 主动废弃连接后、连接池扩容。
-   主流程 4：连接池缩容任务，通过调用`ConcurrentBag.values`筛选出需要的做操作的连接对象，然后再通过`ConcurrentBag.reserve`完成对连接对象状态的修改，然后会通过`流程 1.1.2`触发关闭和移除连接操作。

通过触发点整理，可以知道该结构里的主要方法，就是上面触发点里标记为`标签色`的部分，然后来具体看下该类的基本定义和主要方法：

```java
public class ConcurrentBag<T extends IConcurrentBagEntry> implements AutoCloseable {

    private final CopyOnWriteArrayList<T> sharedList; 
    private final boolean weakThreadLocals; 

    private final ThreadLocal<List<Object>> threadList; 
    private final IBagStateListener listener; 
    private final AtomicInteger waiters; 
    private volatile boolean closed; 

    private final SynchronousQueue<T> handoffQueue; 

    
    public interface IConcurrentBagEntry {

        
        int STATE_NOT_IN_USE = 0; 
        int STATE_IN_USE = 1; 
        int STATE_REMOVED = -1; 
        int STATE_RESERVED = -2; 

        boolean compareAndSet(int expectState, int newState); 

        void setState(int newState); 

        int getState(); 
    }

    
    public interface IBagStateListener {
        void addBagItem(int waiting);
    }

    
    public T borrow(long timeout, final TimeUnit timeUnit) {
        
    }

    
    public void requite(final T bagEntry) {
        
    }

    
    public void add(final T bagEntry) {
        
    }

    
    public boolean remove(final T bagEntry) {
        
    }

    
    public List values(final int state) {
        
    }

    
    public List values() {
        
    }

    
    public boolean reserve(final T bagEntry) {
        
    }

    
    public int getCount(final int state) {
        
    }
}
```

从这个基本结构就可以稍微看出 HikariCP 是如何优化传统连接池实现的了，相比 Druid 来说，HikariCP 更加偏向无锁实现，尽量避免锁竞争的发生。

### 12.1：borrow

这个方法用来获取一个可用的连接对象，触发点为`流程 1.1`，HikariPool 就是利用该方法获取连接的，下面来看下该方法做了什么：

```java
public T borrow(long timeout, final TimeUnit timeUnit) throws InterruptedException {
    
    final List<Object> list = threadList.get(); 
    for (int i = list.size() - 1; i >= 0; i--) {
        final Object entry = list.remove(i); 
        final T bagEntry = weakThreadLocals ? ((WeakReference<T>) entry).get() : (T) entry; 
        
        
        if (bagEntry != null && bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
            return bagEntry; 
        }
    }

    
    final int waiting = waiters.incrementAndGet(); 
    try {
        for (T bagEntry : sharedList) {
            
            if (bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
                
                if (waiting > 1) { 
                    listener.addBagItem(waiting - 1);
                }
                return bagEntry; 
            }
        }

        
        listener.addBagItem(waiting);

        timeout = timeUnit.toNanos(timeout); 
        do {
            final long start = currentTime();
            
            final T bagEntry = handoffQueue.poll(timeout, NANOSECONDS);
            
            if (bagEntry == null || bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_IN_USE)) {
                return bagEntry;
            }
            
            timeout -= elapsedNanos(start); 
        } while (timeout > 10_000); 

        return null; 
    } finally {
        waiters.decrementAndGet(); 
    }
}
```

仔细看下注释，该过程大致分成三个主要步骤：

1.  从线程缓存获取连接
2.  获取不到再从`sharedList`里获取
3.  都获取不到则触发添加连接逻辑，并尝试从队列里获取新生成的连接对象

### 12.2：add

这个流程会添加一个连接对象进入 bag，通常由`主流程 3`里的`addBagItem`方法通过`addConnectionExecutor`异步任务触发添加操作，该方法主流程如下：

```java
public void add(final T bagEntry) {

    sharedList.add(bagEntry); 

    
    
    while (waiters.get() > 0 && bagEntry.getState() == STATE_NOT_IN_USE && !handoffQueue.offer(bagEntry)) { 
        yield();
    }
}
```

结合`borrow`来理解的话，这里在存在等待线程时会添加一个连接对象入队列，可以让`borrow`里发生等待的地方更容易 poll 到这个连接对象。

### 12.3：requite

这个流程会回收一个连接，该方法的触发点在`主流程 6`，具体代码如下：

```java
public void requite(final T bagEntry) {
    bagEntry.setState(STATE_NOT_IN_USE); 

    for (int i = 0; waiters.get() > 0; i++) { 
        if (bagEntry.getState() != STATE_NOT_IN_USE || handoffQueue.offer(bagEntry)) {
            return;
        }
        else if ((i & 0xff) == 0xff) {
            parkNanos(MICROSECONDS.toNanos(10));
        }
        else {
            yield();
        }
    }

    final List<Object> threadLocalList = threadList.get();
    if (threadLocalList.size() < 50) { 
        threadLocalList.add(weakThreadLocals ? new WeakReference<>(bagEntry) : bagEntry); 
    }
}
```

### 12.4：remove

这个负责从池子里移除一个连接对象，触发点在`流程 1.1.2`，代码如下：

```java
public boolean remove(final T bagEntry) {
    
    if (!bagEntry.compareAndSet(STATE_IN_USE, STATE_REMOVED) && !bagEntry.compareAndSet(STATE_RESERVED, STATE_REMOVED) && !closed) {
        LOGGER.warn("Attempt to remove an object from the bag that was not borrowed or reserved: {}", bagEntry);
        return false;
    }

    
    final boolean removed = sharedList.remove(bagEntry);
    if (!removed && !closed) {
        LOGGER.warn("Attempt to remove an object from the bag that does not exist: {}", bagEntry);
    }

    return removed;
}
```

这里需要注意的是，移除时仅仅移除了`sharedList`里的对象，各个线程内缓存的那一份集合里对应的对象并没有被移除，这个时候会不会存在该连接再次从缓存里拿到呢？会的，但是不会返回出去，而是直接`remove`掉了，仔细看`borrow`的代码发现状态不是闲置状态的时候，取出来时就会`remove`掉，然后也拿不出去，自然也不会触发回收方法。

### 12.5：values

该方法存在重载方法，用于返回当前池子内连接对象的集合，触发点在`主流程 4`，代码如下：

```java
public List values(final int state) {
    
    final List list = sharedList.stream().filter(e -> e.getState() == state).collect(Collectors.toList());
    Collections.reverse(list);
    return list;
}

public List values() {
    
    return (List) sharedList.clone();
}
```

### 12.6：reserve

该方法单纯将连接对象的状态值由`STATE_NOT_IN_USE`修改为`STATE_RESERVED`，触发点仍然是`主流程 4`，缩容时使用，代码如下：

```java
public boolean reserve(final T bagEntry){
   return bagEntry.compareAndSet(STATE_NOT_IN_USE, STATE_RESERVED);
}
```

### 12.7：getCount

该方法用于返回池内符合某个状态值的连接的总数量，触发点为`主流程 5`，扩充连接池时用于获取闲置连接总数，代码如下：

```java
public int getCount(final int state){
   int count = 0;
   for (IConcurrentBagEntry e : sharedList) {
      if (e.getState() == state) {
         count++;
      }
   }
   return count;
}
```

以上就是`ConcurrentBag`的主要方法和处理连接对象的主要流程。

## 十三、总结

到这里基本上一个连接的生产到获取到回收到废弃一整个生命周期在 HikariCP 内是如何管理的就说完了，相比之前的 Druid 的实现，有很大的不同，主要是 HikariCP 的`无锁`获取连接，本篇没有涉及`FastList`的说明，因为从连接管理这个角度确实很少用到该结构，用到`FastList`的地方主要在存储连接对象生成的`statement 对象`以及用于存储线程内缓存起来的连接对象；作者的 Spring Boot 专栏、Mybatis 专栏已经完成，关注公众号【码猿技术专栏】回复关键词`Spring Boot 进阶`、`Mybatis 进阶`获取。

除此之外 HikariCP 还利用`javassist`技术编译期生成了`ProxyConnection`的初始化，这里也没有相关说明，网上有关 HikariCP 的优化有很多文章，大多数都提到了`字节码优化`、`fastList`、`concurrentBag`的实现，本篇主要通过深入解析`HikariPool`和`ConcurrentBag`的实现，来说明 HikariCP 相比 Druid 具体做了哪些不一样的操作。 
 [https://juejin.cn/post/6986812265357901860](https://juejin.cn/post/6986812265357901860)
