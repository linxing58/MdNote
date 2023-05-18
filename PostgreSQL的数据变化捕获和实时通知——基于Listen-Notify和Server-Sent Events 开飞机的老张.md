# PostgreSQL的数据变化捕获和实时通知——基于Listen-Notify和Server-Sent Events | 开飞机的老张
有这样一个需求：监听某个PostgreSQL的业务数据库，当特定表中的数据发生变化时，实时通知用户。

[](#数据库→后端 "数据库→后端")数据库→后端
--------------------------

### [](#方案1：轮询 "方案1：轮询")方案1：轮询

监听数据表，最容易想到、实现最简单的，就是`轮询`。

但是，考虑到实时性，高频率的轮询必然会对数据库造成一定压力。

随着业务扩展，监听的表增多，这种模式必然是`不可持续`的。

### [](#方案2：基于日志的CDC "方案2：基于日志的CDC")方案2：基于日志的CDC

类似于MySQL的数据变化捕获（CDC），PostgreSQL也有基于日志的`CDC`方案。

但这种方式更多是用于库-库的`数据同步`，还引入了较重的依赖（例如`debezium`）。

### [](#方案3：触发器 "方案3：触发器")方案3：触发器

从通信方向来看，与常规的数据请求正好相反：

`库→后端→前端`

而且又要求实时性，我很快想到了PostgreSQL的`触发器`。

但从触发器到后端，仍然存在一道鸿沟。

#### [](#方案3-1：pgsql-http "方案3.1：pgsql-http")方案3.1：pgsql-http

从触发器到后端的事件传递，找到一个在PostgreSQL存储过程中发起http请求的插件：

[pgsql-http - GitHub](https://github.com/pramsey/pgsql-http)

这样就打通了从触发器到后端的通信：

`库表→触发器→pgsql-http→后端`

但后来发现了更优的解决方案。

#### [](#方案3-2：Listen-Notify "方案3.2：Listen-Notify")方案3.2：Listen-Notify

PostgreSQL从7.2版本开始支持Listen-Notify通知机制，这是一个语法非常简洁的API。

在SQL中执行：

```sql

LISTEN channelA;
```

```sql

NOTIFY channelA, 'test-message';
```

在存储过程（pl/pgsql）中执行：

```sql

PERFORM pg_listen('channelA');
```

```sql

PERFORM pg_notify('channelA', 'test-message');
```

在Java中监听（伪代码）：

```java

Class.forName("org.postgresql.Driver");
String url = "jdbc:postgresql://localhost:5432/test";
Connection conn = DriverManager.getConnection(url,"test","");
org.postgresql.PGConnection pgconn = conn.unwrap(org.postgresql.PGConnection.class);

Statement stmt = conn.createStatement();
stmt.execute("LISTEN channelA");
stmt.close();

while (true)
{
  org.postgresql.PGNotification notifications[] = pgconn.getNotifications();
  if (notifications != null)
  {
    for (int i=0; i < notifications.length; i++)
      System.out.println("Got notification: " + notifications[i].getName());
  }
  Thread.sleep(500);
}
```

具体见官方文档：

[Listen-Notify - PostgreSQL官方文档](https://jdbc.postgresql.org/documentation/head/listennotify.html)

[](#后端→前端 "后端→前端")后端→前端
-----------------------

### [](#WebSocket-vs-Server-Sent-Events "WebSocket vs Server-Sent Events")WebSocket vs Server-Sent Events

[WebSocket 教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/05/websocket.html)

[Server-Sent Events 教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)

### [](#选择SSE的原因 "选择SSE的原因")选择SSE的原因

1.  无双工通信需求。只需要从后端向前端发通知，不需要双向通信。
    
2.  轻量级。SSE基于http，WebSocket需要支持ws协议，而公司开发环境和部分小型云厂商，出于各种考虑或技术限制，不开放ws协议。
    
3.  实现简单。SSE有自动重连机制，不需要手动处理连接；实现代码简单。
    

[](#实现代码 "实现代码")实现代码
--------------------

### [](#触发器 "触发器")触发器

信息流的起点——触发器。

#### [](#触发器函数 "触发器函数")触发器函数

首先创建一个触发器函数`notify_global_data_change`。

```sql
CREATE OR REPLACE FUNCTION "public"."notify_global_data_change"()
  RETURNS "pg_catalog"."trigger" AS $BODY$
BEGIN
  // 表发生数据变化时，把表名、操作名以JSON形式通知到`change_data_capture`频道
  // 其中`tg_table_name`是表名，`tg_op`是`INSERT`或`UPDATE`或`DELETE`
  PERFORM pg_notify('change_data_capture','{"table":"'||tg_table_name||'","operation":"'||tg_op||'"}');
  RETURN null;
END
$BODY$
LANGUAGE plpgsql VOLATILE
COST 100;
```

项目需求只想知道哪张表发生了哪种类型的变化（增删改），因此所有表使用同一个触发器函数即可。

如果需要更多细节，例如哪个字段发生了变化，变化前后的值是多少，就需要为每张表，单独写一个触发器函数。

#### [](#触发器定义 "触发器定义")触发器定义

有这样一个名为`table_name`的数据表。

将前文的触发器函数`notify_global_data_change`，挂载到目标表的触发器上。

```sql
CREATE TRIGGER "trigger_table_name" AFTER INSERT OR UPDATE OR DELETE ON "public"."table_name"
FOR EACH ROW
EXECUTE PROCEDURE "public"."notify_global_data_change"();
```

### [](#后端实现（Java） "后端实现（Java）")后端实现（Java）

参考GitHub的demo项目：[https://github.com/aliakh/demo-spring-sse](https://github.com/aliakh/demo-spring-sse)

```java
package demo.sse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

class SseEmitters {

    private static final Logger logger = LoggerFactory.getLogger(SseEmitters.class);

    private final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    SseEmitter add() {
        return add(new SseEmitter(-1L));
    }

    SseEmitter add(SseEmitter emitter) {
        this.emitters.add(emitter);

        emitter.onCompletion(() -> {
            logger.info("Emitter completed: {}", emitter);
            this.emitters.remove(emitter);
        });
        emitter.onTimeout(() -> {
            logger.info("Emitter timed out: {}", emitter);
            emitter.complete();
            this.emitters.remove(emitter);
        });

        return emitter;
    }

    void send(Object obj) {
        send(emitter -> emitter.send(obj));
    }

    void send(SseEmitter.SseEventBuilder builder) {
        send(emitter -> emitter.send(builder));
    }

    private void send(SseEmitterConsumer<SseEmitter> consumer) {
        List<SseEmitter> failedEmitters = new ArrayList<>();

        this.emitters.forEach(emitter -> {
            try {
                consumer.accept(emitter);
            } catch (Exception e) {
                emitter.completeWithError(e);
                failedEmitters.add(emitter);
                logger.error("Emitter failed: {}", emitter, e);
            }
        });

        this.emitters.removeAll(failedEmitters);
    }

    @FunctionalInterface
    private interface SseEmitterConsumer<T> {

        void accept(T t) throws IOException;
    }
}

```

```java
package demo.sse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.sql.*;
import org.postgresql.PGConnection;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

import javax.sql.DataSource;

@RestController
@RequestMapping("/datainterface")
public class SseController {

  private final SseEmitters emitters = new SseEmitters();

  private ArrayList<Map<String,String>> events = new ArrayList<Map<String,String>>();
  
  private final ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(1);
  
  private Map<String, PGConnection> pgConns = new HashMap<String, PGConnection>();
  
  private Boolean initStatus = true;

  
  @RequestMapping(value = "/sse/listen", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
  public SseEmitter listenToBroadcast(@RequestParam String url) throws SQLException {
    String dbUrl = "jdbc:postgresql://localhost:5432/test";
    if (url != null && url != '') {
      dbUrl = url;
    }
    if (this.initStatus) {
      this.initStatus = false;
      broadcast();
    }
    if (dbUrl != "" && dbUrl != null) {
      this.initDbListener(dbUrl);
    }
      return emitters.add();
  }
  
  public Boolean initDbListener(String url) throws SQLException {
    
    if (this.pgConns.get(url) != null) {
      return false;
    }
    Class.forName("org.postgresql.Driver");
    Connection conn = DriverManager.getConnection(url,"test","");
    org.postgresql.PGConnection pgConn = conn.unwrap(org.postgresql.PGConnection.class);
    
    this.pgConns.put(url, pgConn);
    
    Statement stmt = conn.createStatement();
    stmt.execute("LISTEN change_data_capture");
    stmt.close();
    return true;
  }

  
  @RequestMapping(value = "/sse/broadcast")
  public Boolean addToBroadcast(@RequestParam String type, @RequestParam String message) {
    if (message != null && message != "") {
      Map<String,String> typeMessage = new HashMap<String,String>();
      typeMessage.put("type", type);
      typeMessage.put("message", message);
      events.add(typeMessage);
      return true;
    } else {
      return false;
    }
  }

  
  private void broadcast() throws SQLException {
    scheduledThreadPool.scheduleAtFixedRate(() -> {
      try {
        
        this.pgConns.forEach( (dbid,pgConn) -> {
          org.postgresql.PGNotification notifications[] = null;
          try {
            notifications = pgConn.getNotifications();
          } catch (SQLException e) {
            e.printStackTrace();
          }
          if (notifications != null) {
            for (int i = 0; i < notifications.length; i++) {
              String message = notifications[i].getParameter();
              
              if (i > 0) {
                String previousMessage = notifications[i-1].getParameter();
                if (message == previousMessage) {
                  continue;
                }
              }
              this.addToBroadcast(dbid, notifications[i].getParameter());
            }
          }
        });
        
        if (events.size() > 0) {
            Map<String,String> firstEvent = events.remove(0);
            String type = firstEvent.get("type");
            String message = firstEvent.get("message");
            emitters.send(
              SseEmitter.event()
                .id(UUID.randomUUID().toString())
                .name(type)
                .data(message)
            );
        }
      } catch (Exception e) {}
    }, 0, 1, TimeUnit.SECONDS);
  }
}
```

在浏览器输入以下地址，回车：

`http://localhost/datainterface/sse/listen?url=jdbc:postgresql://localhost:5432/test`

会发现浏览器小图标一直在转，Network一直没有response。这就是SSE的本质——长连接。

如果这时在`table_name`表中插入一条数据，SSE会返回一个消息：

```json
{"table":"table_name","operation":"INSERT"}
```

然而此时连接并没有结束/中断，而是等待下一个消息。

### [](#前端实现 "前端实现")前端实现

SSE本质是一种长连接，目前浏览器对连接数有一定限制（Chrome 6个）。

采用SharedWorker接口，实现全域只保持一个SSE连接。

```js
const notificationWorker = new SharedWorker('./notification.js');
notificationWorker.port.onmessage = function(e) {
  console.log(e);
}
notificationWorker.port.start();
```

建立SSE连接，监听数据库。

接收到表数据变化的事件时，通过BroadcastChannel接口，向各页面/组件分发通知。

```js
let source = null;
let broadcastChannel = null;
let events = [];


function listen(dbid) {
  source = new EventSource('/datainterface/sse/listen?url=' + url);
  source.addEventListener(dbid, function(event) {
    events.push(event.data);
    throttle(broadcast, 1000)();
  });
  source.onopen = function(e) {};
  source.onerror = function(e) {};
}


function broadcast() {
  events = Array.from(new Set(events));
  if (events.length > 0) {
    const event = events.splice(0, 1);
    let message;
    try {
      message = JSON.parse(event);
    } catch (e) {
      message = event;
    }
    if (message?.table && message?.operation) {
      broadcastChannel = new BroadcastChannel(message.table);
      broadcastChannel.postMessage(message.operation);
    }
  }
}


function throttle(func, wait) {
  let timeout;
  return function() {
    const context = this;
    const args = arguments;
    if (!timeout) {
      timeout = setTimeout(() => {
        timeout = null;
        func.apply(context, args);
      }, wait);
    }
  };
}

listen('jdbc:postgresql://localhost:5432/test');
```

页面/组件接收事件通知，做相应的操作。

```js
const broadcastChannel = new BroadcastChannel('table_name');
broadcastChannel.onmessage = function(e) {
  if (e.data === 'INSERT') {
    ...
  } else if (e.data === 'UPDATE') {
    ...
  } else if (e.data === 'DELETE') {
    ...
  }
}
```

[](#总结 "总结")总结
--------------

### [](#全流程 "全流程")全流程

文章看起来虽然很长，但全流程还是很清晰的：

`数据库表→触发器Notify→后端Listen→SSE广播→主页监听→分发给各页面/组件`

### [](#适用范围 "适用范围")适用范围

现代Web架构中，服务间通信已经有非常成熟的解决方案。

我想要知道发生了什么，完全可以要求事件的发起人，通过服务间通信告诉我。

而非吭哧吭哧扒着数据库监听，这是下策，是不得不走的弯路。

比如：

1、老旧系统，已无人维护，没人做服务间通信接口。

2、合作方，只会玩数据库，或坚持不做服务间通信。

3、涉密或内外网隔离，只能通过数据库通信。

### [](#扩展 "扩展")扩展

经过调研，Oracle、MySQL、SQL Server等主流关系数据库，都有类似`Listen-Notify`的接口。

简单调整，其他主流关系数据库也可以采用这种模式，实现数据变化的实时通知。