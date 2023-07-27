# WebRTC 入门教程（一）| 搭建WebRTC信令服务器
> 作者：李超，音视频技术专家。本入门教程将分为三篇内容，分别讲述信令服务器的搭建、媒体服务器的搭建、Android 端的 WebRTC 应用实现，全文采用开源框架来搭建，适用于大多数入门的开发者。转载请注明：来自 WebRTC 中文网。
> 
> 如遇到问题，[请移步论坛与作者交流](https://rtcdeveloper.com/t/topic/13341)。

前言
--

我们在学习 WebRTC 时，首先要把实验环境搭建好，这样我们就可以在上面做各种实验了。

对于 WebRTC 来说，它有一整套规范，如怎样使用它的接口、使用SDP进行媒体协商、通过ICE收集地址并进行连通性检测等等。除此之外，WebRTC还需要房间服务器将多端聚集到一起管理，以及信令服务器进行信令数据交换（如媒体描述信息SDP的交换，连接地址的交抽换等），但在WebRTC的规范中没有对这部分内容进行规定，所以需要由用户自己处理。

你可以根据自己的喜好选择服务器（如 Apache，Nginx 或 Nodejs），我今天将介绍如何使用 Nodejs 来搭建信令服务器。

为什么选择 Nodejs
------------

Apache、Nginx和Nodejs都是非常成熟的Web服务器，Nginx 可以说是的性能是最好的Web服务器了。但从未来的发展来说，Nodejs可能会更有优势。

现在以Chrome为代表的浏览器的功能越来越强大，以前认为通过浏览器不可能完成的事儿，现在它都可以轻松实现。H5､ WebSocket的出现以及现在WebRTC的加入，让大家越来越觉得以后的浏览器可以说是“无所不能”。因此，推动 JavaScript 语言的发展越来越迅速。这可以从现在 JavaScript 技术的火爆，以及各种层叠不穷JS FrameWork的出现得以印证。

而 Nodejs 的最大优点即是可以使用 JS 语言开发服务器程序。这样使得大量的前端同学可以无缝的转到服务器开发，甚至有可能前后端使用同一套代码实现。对于这一点我想无论是对个人还是对于企业都是具大的诱惑。

*   一方面 JS 语言的简单性可以方便开发出各种各样功能的服务端程序。
*   更可贵的是 Nodejs 的生态链非常的完整，有各种各样的功能库。你可以根据自己的需要通过安装工具 NPM 快速的安装，这也使它也得到了广大开发者的喜欢。

Nodejs 现在是非常流行的 Web 服务器，它在服务器端使用 V8(JavaScript)引擎，通过它解析 JS 脚本来控制服务器的行为。这对于广大的 JS 同学来说真是太幸福了，在10年前还很难想像可以通过 JS 脚本语言来写服务器程序。

当然，如果你想对Nodejs作能力拓展的话，还是要写C/C++库，然后加载到 Nodejs 中去。

Nodejs的基本原理
-----------

[![](https://webrtc.org.cn/wp-content/uploads/2019/03/nodejs-1024x396.jpg)
](https://webrtc.org.cn/wp-content/uploads/2019/03/nodejs.jpg)

Nodejs的工作原理如上图所示， 其核心是 V8 引擎。通过该引擎，可以让 js 调用 C/C++方法 或 对象。相反，通过它也可能让 C/C++ 访问 javascript 方法和变量。

Nodejs 首先将 JavaScript 写好的应用程序交给 V8 引擎进行解析，V8理解应用程序的语义后，再调用 Nodejs 底层的 C/C++ API将服务启动起来。 所以 Nodejs 的强大就在于 js 可以直接调用 C/C++ 的方法，使其能力可以无限扩展。

以开发一个 HTTP 服务为例，Nodejs 打开侦听的服务端口后，底层会调用 libuv 处理该端口的所有 http 请求。其网络事件处理如下图所示：

[![](https://webrtc.org.cn/wp-content/uploads/2019/03/nodejs%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86-1024x596.jpg)
](https://webrtc.org.cn/wp-content/uploads/2019/03/nodejs%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86.jpg)

当有网络请求过来时，首先会被插入到一个事件处理队列中。libuv会监控该事件队列，当发现有事件时，先对请求做判断，如果是简单的请求，就直接返回响应了；如果是复杂请求，则从线程池中取一个线程进行异步处理；

线程处理完后，有两种可能：一种是已经处理完成，则向用户发送响应；另一种情况是还需要进一步处理，则再生成一个事件插入到事件队列中等待处理；事件处理就这样循环往复下去，永不停歇。

两个 V8 引擎
--------

[![](https://webrtc.org.cn/wp-content/uploads/2019/03/v8-1024x654.jpg)
](https://webrtc.org.cn/wp-content/uploads/2019/03/v8.jpg)

如上图所示，在我们使用 Nodejs之后实际存在了两个 V8 引擎。一个V8用于解析服务端的 JS 应用程序，它将服务启动起来。另一个 V8 是浏览器中的 V8 引擎，用于控制浏览器的行为。

对于使用 Nodejs 的新手来说，很容易出现思维混乱，因为在服务端至少要放两个 JS 脚本。其中一个是服务端程序，控制 Nodejs 的行为，它由 Nodejs 的V8引擎解析处理；另一个是客户端程序，它是要由浏览器请求后，下发到浏览器，由浏览器中的 V8 引擎进行解析处理。如果分不清这个，那麻烦就大了。

安装 Nodejs
---------

下面我们就来看看具体如何安装 Nodejs。

安装 Nodejs 非常的简单:

在Ubuntu系统下执行：

```
apt install nodejs
```

或在Mac 系统下执行：

```
brew install nodejs
```

通过上面的步骤我们就将 Nodejs 安装好了。我这里安装的 Nodejs版本为：**v8.10.0**。

安装NPM
-----

除了安装 Nodejs 之外，我们还要安装NPM(Node Package Manager），也就是 Nodejs 的包管理器。它就像Ubuntu下的 apt 或Mac 系统下的brew 命令类似，是专门用来管理各种依赖库的。

在它们没有出现之前，我们要安装个包特别麻烦。以Linux为例，假设要安装一个工具，其基本步骤是:

*   先将这个工具的源码下载下来。
*   执行./configure 生成Makefile 文件。
*   执行 make 命令对其进行编译。
*   最后，执行 make install 将其安装到指定目录下。
*   如果编译过程中发现有依赖的库，则要对依赖库执行前面的4步，也就是先将依赖库安装好，然后再来安装该工具。

大家可以看到，以前在Linux下安装个程序或工具是多么的麻烦。

Linux 有了apt 之后，一切都变得简单了。我们只要执行 apt install xxx 一条命令就好了，它会帮你完成上面的一堆操作。

对于 Nodejs的安装包也是如此，NPM 就是相当于 Linux 下的 apt，它的出现大大提高了人们的工作效率。

NPM 的安装像安装 Nodejs 一样简单:

在Ubuntu下执行:

```
apt install npm
```

或在Mac下执行：

```
brew install npm
```

socket.io
---------

此次，我们使用 Nodejs 下的 socket.io 库来实现 WebRTC 信令服务器。socket.io特别适合用来开发WebRTC的信令服务器，通过它来构建信令服务器特别的简单，这主要是因为它内置了**房间** 的概念。

[![](https://webrtc.org.cn/wp-content/uploads/2019/03/socket.io_-1024x397.jpg)
](https://webrtc.org.cn/wp-content/uploads/2019/03/socket.io_.jpg)

上图是 socket.io 与 Nodejs配合使用的逻辑关系图， 其逻辑非常简单。socket.io 分为服务端和客户端两部分。服务端由 Nodejs加载后侦听某个服务端口，客户端要想与服务端相连，首先要加载 socket.io 的客户端库，然后调用 \`io.connect();\`就与服务端连上了。

需要特别强调的是 socket.io 消息的发送与接收。socket.io 有很多种发送消息的方式，其中最常见的有下面几种，是我们必须要撑握的：

*   给本次连接发消息

```
socket.emit()
```

*   给某个房间内所有人发消息

```
io.in(room).emit()
```

*   除本连接外，给某个房间内所有人发消息

```
socket.to(room).emit()
```

*   除本连接外，给所以人发消息

```
socket.broadcast.emit()
```

消息又该如何接收呢？

*   发送 command 命令

C: socket.on('cmd',function(){...});

S: socket.emit('cmd’); C: socket.on('cmd',function(){...});

```
S: socket.emit('cmd’);
C: socket.on('cmd',function(){...});
```

*   送了一个 command 命令，带 data 数据

S: socket.emit('action', data);

C: socket.on('action',function(data){...});

S: socket.emit('action', data); C: socket.on('action',function(data){...});

```
S: socket.emit('action', data);
C: socket.on('action',function(data){...});
```

*   发送了command命令，还有两个数据

S: socket.emit(action,arg1,arg2);

C: socket.on('action',function(arg1,arg2){...});

S: socket.emit(action,arg1,arg2); C: socket.on('action',function(arg1,arg2){...});

```
S: socket.emit(action,arg1,arg2);
C: socket.on('action',function(arg1,arg2){...});
```

有了以上这些知识，我们就可以实现信令数据通讯了。

搭建信令服务器
-------

接下来我们来看一下，如何通过 Nodejs下的 socket.io 来构建的一个服务器：

这是客户端代码，也就是在浏览器里执行的代码。index.html：

<title>WebRTC client</title>

<script src='/socket.io/socket.io.js'></script>

<script src='js/client.js'></script>

<!DOCTYPE html> <html> <head> <title>WebRTC client</title> </head> <body> <script src='/socket.io/socket.io.js'></script> <script src='js/client.js'></script> </body> </html>

```
<!DOCTYPE html>
<html>
 <head>
 <title>WebRTC client</title>
 </head>
 <body>
 <script src='/socket.io/socket.io.js'></script>
 <script src='js/client.js'></script>
 </body>
</html>
```

该代码十分简单，就是在body里引入了两段 JS 代码。其中，socket.io.js 是用来与服务端建立 socket 连接的。client.js 的作用是做一些业务逻辑，并最终通过 socket 与服务端通讯。

首先，在 server.js 目录下创建  js  子目录，然后在 js目录下生成 client.js。

下面是client.js的代码：

room = prompt('Enter room name:'); //弹出一个输入窗口

const socket = io.connect(); //与服务端建立socket连接

if  (room !== '')  { //如果房间不空，则发送 "create or join" 消息

console.log('Joining room ' \+ room);

socket.emit('create or join', room);

socket.on('full', (room) =>  { //如果从服务端收到 "full" 消息

console.log('Room ' \+ room + ' is full');

socket.on('empty', (room) =>  { //如果从服务端收到 "empty" 消息

console.log('Room ' \+ room + ' is empty');

socket.on('join', (room) =>  { //如果从服务端收到 “join" 消息

console.log('Making request to join room ' \+ room);

console.log('You are the initiator!');

socket.on('log', (array) =>  {

console.log.apply(console, array);

var isInitiator; room = prompt('Enter room name:'); //弹出一个输入窗口 const socket = io.connect(); //与服务端建立socket连接 if (room !== '') { //如果房间不空，则发送 "create or join" 消息 console.log('Joining room ' + room); socket.emit('create or join', room); } socket.on('full', (room) => { //如果从服务端收到 "full" 消息 console.log('Room ' + room + ' is full'); }); socket.on('empty', (room) => { //如果从服务端收到 "empty" 消息 isInitiator = true; console.log('Room ' + room + ' is empty'); }); socket.on('join', (room) => { //如果从服务端收到 “join" 消息 console.log('Making request to join room ' + room); console.log('You are the initiator!'); }); socket.on('log', (array) => { console.log.apply(console, array); });

```
var isInitiator;

room = prompt('Enter room name:'); //弹出一个输入窗口

const socket = io.connect(); //与服务端建立socket连接

if (room !== '') { //如果房间不空，则发送 "create or join" 消息
 console.log('Joining room ' + room);
 socket.emit('create or join', room);
}

socket.on('full', (room) => { //如果从服务端收到 "full" 消息
 console.log('Room ' + room + ' is full');
});

socket.on('empty', (room) => { //如果从服务端收到 "empty" 消息
 isInitiator = true;
 console.log('Room ' + room + ' is empty');
});

socket.on('join', (room) => { //如果从服务端收到 “join" 消息
 console.log('Making request to join room ' + room);
 console.log('You are the initiator!');
});

socket.on('log', (array) => {
 console.log.apply(console, array);
});
```

在该代码中：

首先弹出一个输入框，要求用户写入要加入的房间。然后，通过 io.connect() 建立与服务端的连接，根据socket返回的消息做不同的处理：

*   当收到房间满”full”时的情况;
*   当收到房间空“empty”时的情况；
*   当收到加入“join”时的情况；

以上是客户端（也就是在浏览器）中执行的代码。下面我们来看一下服务端的处理逻辑：

服务器端代码，server.js：

const static = require('node-static');

const http = require('http');

const file = new(static.Server)();

const app = http.createServer(function  (req, res)  {

const io = require('socket.io').listen(app); //侦听 2013

io.sockets.on('connection', (socket) =>  {

// convenience function to log server messages to the client

const array = \['>>\> Message from server: '\];

for  (var i = 0; i < arguments.length; i++)  {

array.push(arguments\[i\]);

socket.emit('log', array);

socket.on('message', (message) =>  { //收到message时，进行广播

log('Got message:', message);

// for a real app, would be room only (not broadcast)

socket.broadcast.emit('message', message); //在真实的应用中，应该只在房间内广播

socket.on('create or join', (room) =>  { //收到 “create or join” 消息

var clientsInRoom = io.sockets.adapter.rooms\[room\];

var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length  :  0; //房间里的人数

log('Room ' \+ room + ' has ' \+ numClients + ' client(s)');

log('Request to create or join room ' \+ room);

if  (numClients === 0){ //如果房间里没人

socket.emit('created', room); //发送 "created" 消息

}  else  if  (numClients === 1)  { //如果房间里有一个人

io.sockets.in(room).emit('join', room);

socket.emit('joined', room); //发送 “joined”消息

}  else  { // max two clients

socket.emit('full', room); //发送 "full" 消息

socket.emit('emit(): client ' \+ socket.id +

socket.broadcast.emit('broadcast(): client ' \+ socket.id +

const static = require('node-static'); const http = require('http'); const file = new(static.Server)(); const app = http.createServer(function (req, res) { file.serve(req, res); }).listen(2013); const io = require('socket.io').listen(app); //侦听 2013 io.sockets.on('connection', (socket) => { // convenience function to log server messages to the client function log(){ const array = \['>>> Message from server: '\]; for (var i = 0; i < arguments.length; i++) { array.push(arguments\[i\]); } socket.emit('log', array); } socket.on('message', (message) => { //收到message时，进行广播 log('Got message:', message); // for a real app, would be room only (not broadcast) socket.broadcast.emit('message', message); //在真实的应用中，应该只在房间内广播 }); socket.on('create or join', (room) => { //收到 “create or join” 消息 var clientsInRoom = io.sockets.adapter.rooms\[room\]; var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length : 0; //房间里的人数 log('Room ' + room + ' has ' + numClients + ' client(s)'); log('Request to create or join room ' + room); if (numClients === 0){ //如果房间里没人 socket.join(room); socket.emit('created', room); //发送 "created" 消息 } else if (numClients === 1) { //如果房间里有一个人 io.sockets.in(room).emit('join', room); socket.join(room); socket.emit('joined', room); //发送 “joined”消息 } else { // max two clients socket.emit('full', room); //发送 "full" 消息 } socket.emit('emit(): client ' + socket.id + ' joined room ' + room); socket.broadcast.emit('broadcast(): client ' + socket.id + ' joined room ' + room); }); });

```
const static = require('node-static');
const http = require('http');
const file = new(static.Server)();
const app = http.createServer(function (req, res) {
  file.serve(req, res);
}).listen(2013);

const io = require('socket.io').listen(app); //侦听 2013

io.sockets.on('connection', (socket) => {

  // convenience function to log server messages to the client
  function log(){ 
    const array = \['>>> Message from server: '\]; 
    for (var i = 0; i < arguments.length; i++) {
      array.push(arguments\[i\]);
    } 
      socket.emit('log', array);
  }

  socket.on('message', (message) => { //收到message时，进行广播
    log('Got message:', message);
    // for a real app, would be room only (not broadcast)
    socket.broadcast.emit('message', message); //在真实的应用中，应该只在房间内广播
  });

  socket.on('create or join', (room) => { //收到 “create or join” 消息

    var clientsInRoom = io.sockets.adapter.rooms\[room\];
    var numClients = clientsInRoom ? Object.keys(clientsInRoom.sockets).length : 0; //房间里的人数

    log('Room ' + room + ' has ' + numClients + ' client(s)');
    log('Request to create or join room ' + room);

    if (numClients === 0){ //如果房间里没人
      socket.join(room);
      socket.emit('created', room); //发送 "created" 消息
    } else if (numClients === 1) { //如果房间里有一个人
      io.sockets.in(room).emit('join', room);
      socket.join(room);
      socket.emit('joined', room); //发送 “joined”消息
    } else { // max two clients
      socket.emit('full', room); //发送 "full" 消息
    }
    socket.emit('emit(): client ' + socket.id +
      ' joined room ' + room);
    socket.broadcast.emit('broadcast(): client ' + socket.id +
      ' joined room ' + room);

  });

});
```

在服务端引入了 **node-static** 库，使服务器具有发布静态文件的功能。服务器具有此功能后，当客户端（浏览器）向服务端发起请求时，服务器通过该模块获得客户端（浏览器）运行的代码，也就是上我面我们讲到的 index.html 和 client.js 并下发给客户端（浏览器）。

服务端侦听 2013 这个端口，对不同的消息做相应的处理：

– 服务器收到 message 消息时，它会直接进行广播，所有连接到该服务器的客户端都会收收广播的消息。  
– 服务端收到 “create or join”消息时，它会对房间里有人数进行统计，如果房间里没有人，则发送”created” 消息；如果房间里有一个人，发送”join”消息和“joined”消息；如果超过两个人，发送”full”消息。  
要运行该程序，需要使用 NPM 安装 socket.io 和 [node-static](https://github.com/cloudhead/node-static)，安装方法如下：

进入到 server.js 所在的目录，然后执行下面的命令。

npm install socket.io npm install node-static

```
npm install socket.io
npm install node-static
```

启动服务器并测试
--------

通过上面的步骤我们就使用 socket.io 构建好一个服务器，现在可以通过下面的命令将服务启动起来了：

```
node server.js
```

如果你是在本机上搭建的服务，则可以在浏览器中输入 localhost:2013 ，然后新建一个tab 在里边再次输入localhost:2013 。此时，打开控制台看看发生了什么?

在Chrome下你可以使用快捷键 Command-Option-J或Ctrl-Shift-J的DevTools访问控制台。

小结
--

以上我向大家介绍了 Nodejs 的工作原理、Nodejs的安装与布署，以及如何使用 要sokcet.io 构建 WebRTC 信令消息服务器。socket.io 由于有房间的概念所以与WebRTC非常匹配，用它开发WebRTC信令服务器非常方便。

另外，在本文中的例子只是一个简单例子并没有太多的实际价值。在后面的文章中我会以这个例子为基础，在其上面不断增加一些功能，最终你会看到一个完整的Demo程序。