# WebRTC 入门与实战教程
之前公司准备用 webRTC 来实现视频聊天，研究了几天，撸了个 demo 出来，(虽然最后并没有采用这项技术,囧)，听说掘金最近在搞 webRTC 有奖征文活动，自不量力一下，硬着头皮写个教程吧，顺便也给自己整理下思路。

WebRTC简单介绍
----------

> WebRTC (Web Real-Time Communication) 是一个可以用在视频聊天，音频聊天或P2P文件分享等Web App中的 API。-MDN

目前这项Web技术支持的浏览器有chrome, firefox和safari。

WebRTC 有三个主要的API

*   getUserMedia - 采集本地音频和视频流
*   RTCPeerConnection - 用来创建对端连接并传输音视频的API
*   RTCDataChannel - 用于传输二进制数据。

这三个API里面涉及了很多知识，要想全面细致的了解这项技术，建议去看官网，这篇教程只会介绍重要的知识点并且以此为前提，写一个聊天室和多人视频聊天的demo

直播软件(比如斗鱼直播，熊猫直播等)是在客户端采集和编码主播的音频和视频，传输给流媒体服务器，流媒体服务器将媒体数据转发出去，客户端收到视频流进行解码和播放 架构简化图

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/23/16a4a2cc0f0b8a11~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

WebRTC提供端对端的音视频通讯，不需要媒体服务器转发媒体数据，架构简化图如下，

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/23/16a4a2f210b33e25~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

其中每个相互连接的客户端叫做对等端

WebRTC采集和传输音视频数据的过程可以分为三步进行

1.  实时捕获本地的音视频流
2.  实时编码音视频并在网络中向对等端传输多媒体数据
3.  对等端接受发送者的音视频，实时解码播放

捕获本地媒体流
-------

这一步非常简单，调用navigator.getUserMedia这个api就可以，第一个参数是音视频的限制条件，第一个参数是捕获媒体流成功的回调，回调参数是stream, 第三个参数是捕获失败的回调，在成功的回调函数种将stream赋值给video的srcObject即可显示视频

示例代码如下:

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div>
        <button id="start">开始录制</button>
        <button id="stop">停止录制</button>
    </div>
    <div>
        <video autoplay controls id="stream"></video>
    </div>
    <script>
        
        let constraints = {audio: false, video: true}; 
        let startBtn = document.getElementById('start')
        let stopBtn = document.getElementById('stop')
        let video = document.getElementById('stream')
        startBtn.onclick = function() {
            navigator.getUserMedia(constraints, function(stream) {
                video.srcObject = stream;
                window.stream = stream;
            }, function(err) {
                console.log(err)
            })
        }
        stopBtn.onclick = function() {
            video.pause();
        }
    </script>
</body>
</html>

```

本地视频已经以采集和播放了，接下来要解决如何和对方进行视频会议，也就是实时传输音视频，WebRTC是提供端对端的音视频传输，也就是封装好的RTCPeerConnection这个API，调用这个API可以创建对等端连接并传输音视频，但是光有这个api还不够，**在传送音视频数据之前，我们需要先传输信令**。

什么是信令
-----

信令是协调通信的过程。为了使WebRTC应用程序能够建立一个"通话"，其客端户需要交换以下信息：

```markdown
      会话控制消息用于打开或关闭通信

      错误消息

      媒体元数据，如编解码器和编解码器设置，带宽和媒体类型

      密钥数据，用于建立安全的连接

      网络数据，如主机IP地址和端口


```

这个信令过程需要一个方式使客户端之间来回地进行消息传递， WebRTC标准并没有规定信令方法和协议，我们可以采用JavaScript会话建立协议JSEP来实现信令的交换，这里用一个例子来说明这个过程，假设A和B之间要建立RTCPeerConnection连接

```css
     1. A创建一个RTCPeerConnection对象。

     2. A使用RTCPeerConnection .createOffer()方法产生一个offer（一个SDP会话描述）。

     3. A用生成的offer调用setLocalDescription()，设置成自己的本地会话描述。

     4. A将offer通过信令机制发送给B。

     5. B用A的offer调用setRemoteDescription()，设置成自己的远端会话描述，以便他的RTCPeerConnection知道A的设置。

     6. B调用createAnswer()生成answer

     7. B通过调用setLocalDescription()将其answer设置为本地会话描述。

     8. B然后使用信令机制将他的answer发回给A。

     9. A使用setRemoteDescription()将B的应答设置为远端会话描述。

```

A和B除了交换会话信息，还需要交换网络信息。"查找候选项"是指使用**ICE框架**查找网络接口和端口的过程。

```css
     1. A使用onicecandidate创建一个RTCPeerConnection对象，注册一个处理器函数。

     2. 处理器在网络候选变得可用时被调用。

     3. 在处理器中，A通过信令通道将候选数据发送给B。

     4. 当B从A那里获得候选消息时，调用addIceCandidate()，将候选项添加到远端对等描述中。

```

稳住，别晕，我们用代码来说明这个过程吧，交换信令需要信令服务器，这个我们晚点讨论，我们现在考虑同一个页面上的两个RTCPeerConnection对象，一个代表本地，一个代表远端， 来说明信令交换和网络信息交换的过程和细节。 这个是官方文档的demo,[第五个实验室教程](https://link.juejin.cn/?target=https%3A%2F%2Fcodelabs.developers.google.com%2Fcodelabs%2Fwebrtc-web%2F%234 "https://codelabs.developers.google.com/codelabs/webrtc-web/#4"), 能够科学上网的小伙伴建议直接去看教程，由浅入深地教你搭建WebRTC项目，在这里借用这个demo来解释建立RTCPeerConnection连接建立的过程

首先建立index.html

html结构如下：

```xml
 <!DOCTYPE html>
 <html>

     <head>
         <title>Realtime communication with WebRTC</title>
         <style>
             body {
                 font-family: sans-serif;
             }
     
             video {
                 max-width: 100%;
                 width: 320px;
             }
         </style>
     </head>

     <body>
         <h1>Realtime communication with WebRTC</h1>
     
         <video id="localVideo" autoplay playsinline></video>
         <video id="remoteVideo" autoplay playsinline></video>
     
         <div>
             <button id="startButton">开始</button>
             <button id="callButton">调用</button>
             <button id="hangupButton">挂断</button>
         </div>
         <script src="./main.js">
         </script>
 </body>

</html>

```

一个video元素显示通过getUserMedia()方法获取的本地视频流，另一个显示通过 RTCPeerconnection传输的视频流，这里是同样的视频流，但是在真实的应用中，一个video显示本地视频流，另一个显示远端视频流

main.js代码如下

```javascript
    'use strict';
    
    const mediaStreamConstraints = {
      video: true,
      audio: false
    };
    
    
    const offerOptions = {
      offerToReceiveVideo: 1,
    };
    
    let startTime = null;
    
    
    const localVideo = document.getElementById('localVideo');
    const remoteVideo = document.getElementById('remoteVideo');
    
    let localStream;
    let remoteStream;
    
    let localPeerConnection;
    let remotePeerConnection;



function gotLocalMediaStream(mediaStream) {
  localVideo.srcObject = mediaStream;
  localStream = mediaStream;
  trace('Received local stream.');
  callButton.disabled = false; 
}

function handleLocalMediaStreamError(error) {
  trace(`navigator.getUserMedia error: ${error.toString()}.`);
}

function gotRemoteMediaStream(event) {
  const mediaStream = event.stream;
  remoteVideo.srcObject = mediaStream;
  remoteStream = mediaStream;
  trace('Remote peer connection received remote stream.');
}

function logVideoLoaded(event) {
  const video = event.target;
  trace(`${video.id} videoWidth: ${video.videoWidth}px, ` +
        `videoHeight: ${video.videoHeight}px.`);
}

function logResizedVideo(event) {
  logVideoLoaded(event);
  if (startTime) {
    const elapsedTime = window.performance.now() - startTime;
    startTime = null;
    trace(`Setup time: ${elapsedTime.toFixed(3)}ms.`);
  }
}

localVideo.addEventListener('loadedmetadata', logVideoLoaded);
remoteVideo.addEventListener('loadedmetadata', logVideoLoaded);
remoteVideo.addEventListener('onresize', logResizedVideo);


function handleConnection(event) {
  const peerConnection = event.target;
  const iceCandidate = event.candidate;

  if (iceCandidate) {
    const newIceCandidate = new RTCIceCandidate(iceCandidate);
    const otherPeer = getOtherPeer(peerConnection);

    otherPeer.addIceCandidate(newIceCandidate)
      .then(() => {
        handleConnectionSuccess(peerConnection);
      }).catch((error) => {
        handleConnectionFailure(peerConnection, error);
      });

    trace(`${getPeerName(peerConnection)} ICE candidate:\n` +
          `${event.candidate.candidate}.`);
  }
}

function handleConnectionSuccess(peerConnection) {
  trace(`${getPeerName(peerConnection)} addIceCandidate success.`);
};

function handleConnectionFailure(peerConnection, error) {
  trace(`${getPeerName(peerConnection)} failed to add ICE Candidate:\n`+
        `${error.toString()}.`);
}

function handleConnectionChange(event) {
  const peerConnection = event.target;
  console.log('ICE state change event: ', event);
  trace(`${getPeerName(peerConnection)} ICE state: ` +
        `${peerConnection.iceConnectionState}.`);
}

function setSessionDescriptionError(error) {
  trace(`Failed to create session description: ${error.toString()}.`);
}

function setDescriptionSuccess(peerConnection, functionName) {
  const peerName = getPeerName(peerConnection);
  trace(`${peerName} ${functionName} complete.`);
}

function setLocalDescriptionSuccess(peerConnection) {
  setDescriptionSuccess(peerConnection, 'setLocalDescription');
}

function setRemoteDescriptionSuccess(peerConnection) {
  setDescriptionSuccess(peerConnection, 'setRemoteDescription');
}

function createdOffer(description) {
  trace(`Offer from localPeerConnection:\n${description.sdp}`);

  trace('localPeerConnection setLocalDescription start.');
  localPeerConnection.setLocalDescription(description)
    .then(() => {
      setLocalDescriptionSuccess(localPeerConnection);
    }).catch(setSessionDescriptionError);

  trace('remotePeerConnection setRemoteDescription start.');
  remotePeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);

  trace('remotePeerConnection createAnswer start.');
  remotePeerConnection.createAnswer()
    .then(createdAnswer)
    .catch(setSessionDescriptionError);
}

function createdAnswer(description) {
  trace(`Answer from remotePeerConnection:\n${description.sdp}.`);

  trace('remotePeerConnection setLocalDescription start.');
  remotePeerConnection.setLocalDescription(description)
    .then(() => {
      setLocalDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);

  trace('localPeerConnection setRemoteDescription start.');
  localPeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(localPeerConnection);
    }).catch(setSessionDescriptionError);
}

const startButton = document.getElementById('startButton');
const callButton = document.getElementById('callButton');
const hangupButton = document.getElementById('hangupButton');
callButton.disabled = true;
hangupButton.disabled = true;

function startAction() {
  startButton.disabled = true;
  navigator.getUserMedia(mediaStreamConstraints, gotLocalMediaStream, handleLocalMediaStreamError)
  trace('Requesting local stream.');
}

function callAction() {
  callButton.disabled = true;
  hangupButton.disabled = false;

  trace('Starting call.');
  startTime = window.performance.now();

  const videoTracks = localStream.getVideoTracks();
  const audioTracks = localStream.getAudioTracks();
  if (videoTracks.length > 0) {
    trace(`Using video device: ${videoTracks[0].label}.`);
  }
  if (audioTracks.length > 0) {
    trace(`Using audio device: ${audioTracks[0].label}.`);
  }
  
  const servers = null; 

  localPeerConnection = new RTCPeerConnection(servers);
  trace('Created local peer connection object localPeerConnection.');

  localPeerConnection.addEventListener('icecandidate', handleConnection);
  localPeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);

  remotePeerConnection = new RTCPeerConnection(servers);
  trace('Created remote peer connection object remotePeerConnection.');

  remotePeerConnection.addEventListener('icecandidate', handleConnection);
  remotePeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);
  remotePeerConnection.addEventListener('addstream', gotRemoteMediaStream);

  localPeerConnection.addStream(localStream);
  trace('Added local stream to localPeerConnection.');

  trace('localPeerConnection createOffer start.');
  localPeerConnection.createOffer(offerOptions)
    .then(createdOffer).catch(setSessionDescriptionError);
}
function hangupAction() {
  localPeerConnection.close();
  remotePeerConnection.close();
  localPeerConnection = null;
  remotePeerConnection = null;
  hangupButton.disabled = true;
  callButton.disabled = false;
  trace('Ending call.');
}

startButton.addEventListener('click', startAction);
callButton.addEventListener('click', callAction);
hangupButton.addEventListener('click', hangupAction);

function getOtherPeer(peerConnection) {
  return (peerConnection === localPeerConnection) ?
      remotePeerConnection : localPeerConnection;
}

function getPeerName(peerConnection) {
  return (peerConnection === localPeerConnection) ?
      'localPeerConnection' : 'remotePeerConnection';
}

function trace(text) {
  text = text.trim();
  const now = (window.performance.now() / 1000).toFixed(3);
  console.log(now, text);
}

```

好吧，很长，我们来一起看看main.js到底做了什么

WebRTC使用RTCPeerConnection这个API来建立连接，在客户端（也就是我们所说的端对端中的端）之间传输媒体流,在这个例子中，两个RTCPeerConnection对象在同一个页面,这个例子没有什么实际的用处，但是却可以很好地说明这个API的工作流程

建议两个WebRTC用户之间的连接涉及三个任务：

```null
每一个调用需要为每个端创建一个RTCPeerConnection对象
获取和分享网络信息：可能的连接端点，也就是ICE候选
获取和分享本地和远端的描述：SDP格式的本地媒体的元信息

```

假设A和B想通过RTCPeerConnection建立视频一个聊天室

首先，A和B交换网络信息，查找候选项就是使用ICE框架查找网络接口和端口的过程

A创建一个RTCPeerConnection对象，绑定一个onicecandidate事件的处理器(addEventListener('icecandidate'))，这个过程对应main.js下面的代码：

```reasonml
    let localPeerConnection;
    
    localPeerConnection = new RTCPeerConnection(servers);
    localPeerConnection.addEventListener('icecandidate', handleConnection);
    localPeerConnection.addEventListener('iceconnectionstatechange', handleConnectionChange);

```

WebRTC是为了直接端对端而设计的，让用户可以使用最直接的路由进行连接，而不需要经过服务器，然而，WebRTC 需要处理真实世界中的网络：客户端应用需要穿透NAT和防火墙，端对端直接连接失败的时候需要回退处理，在这个过程，WebRTC使用STUN服务器来获取IP,TURN服务器来作为回退服务器以免端对端连接失败

A 调用getUserMedia()获取视频流，然后addStream(localPeerConnection.addStream):

```reasonml
navigator.getUserMedia(mediaStreamConstraints, gotLocalMediaStream, handleLocalMediaStreamError).
function gotLocalMediaStream(mediaStream) {
  localVideo.srcObject = mediaStream;
  localStream = mediaStream;
  trace('Received local stream.');
  callButton.disabled = false;  
}
localPeerConnection.addStream(localStream);
trace('Added local stream to localPeerConnection.');

```

当网络候选项(candidate)变得可用的时候，onicecandidate的处理器会被调用 A发送序列化的数据给B, 在真实的应用中，这个过程(交换信令)需要一个信令服务器， 当然，在我们这个例子中， 两个RTCPeerConnection在同一个页面中， 可以直接沟通，不需要额外的信令传输 当B获取到A的candidate信息的时候, 他会调用addIceCandidate, 将candidate添加到候选中

```javascript
function handleConnection(event) {
  const peerConnection = event.target;
  const iceCandidate = event.candidate;

  if (iceCandidate) {
    const newIceCandidate = new RTCIceCandidate(iceCandidate);
    const otherPeer = getOtherPeer(peerConnection);

    otherPeer.addIceCandidate(newIceCandidate)
      .then(() => {
        handleConnectionSuccess(peerConnection);
      }).catch((error) => {
        handleConnectionFailure(peerConnection, error);
      });

    trace(`${getPeerName(peerConnection)} ICE candidate:\n` +
          `${event.candidate.candidate}.`);
  }
}

```

WebRTC的对等端也需要找出并交换本地和远程的音视频媒体信息， 比如编解码能力，使用会话描述协议也就是SDP，通过交换元数据，也就是offer和answer来达到交换媒体配置信息的目的

A运行RTCPeerConnection的createOffer()方法，返回一个promise，提供了一个RTCSessionDescription： A的本地会话描述

```scss
trace('localPeerConnection createOffer start.');
localPeerConnection.createOffer(offerOptions)
  .then(createdOffer).catch(setSessionDescriptionError);

```

如果Promise成功了，A通过setLocalDescription方法将promise成功返回的描述设置成自己的本地描述，并且将描述通过信令通道发送给B, B使用setRemoteDescription将A发送给他的描述设置成自己的远端描述，B运行RTCPeerConnection的createAnswer()方法，可以产生一个相匹配的描述，B将自己产生的描述设置为本地描述并且发送给A， A将其设置成自己的远端描述

```coffeescript
function createdOffer(description) {
  trace(`Offer from localPeerConnection:\n${description.sdp}`);

  trace('localPeerConnection setLocalDescription start.');
  localPeerConnection.setLocalDescription(description)
    .then(() => {
      setLocalDescriptionSuccess(localPeerConnection);
    }).catch(setSessionDescriptionError);

  trace('remotePeerConnection setRemoteDescription start.');
  remotePeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);

  trace('remotePeerConnection createAnswer start.');
  remotePeerConnection.createAnswer()
    .then(createdAnswer)
    .catch(setSessionDescriptionError);
}

function createdAnswer(description) {
  trace(`Answer from remotePeerConnection:\n${description.sdp}.`);

  trace('remotePeerConnection setLocalDescription start.');
  remotePeerConnection.setLocalDescription(description)
    .then(() => {
      setLocalDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);

  trace('localPeerConnection setRemoteDescription start.');
  localPeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(localPeerConnection);
    }).catch(setSessionDescriptionError);
}

```

搞定!

最终demo
------

### 信令服务器

信令服务器也是web服务器，只是传输的不是普通的数据，而是信令，你可以根据自己的喜好选择服务器(如 Apache，Nginx 或 Nodejs), 这里使用node和搭配socket.io来搭建信令服务器

#### socket.io

[官网](https://link.juejin.cn/?target=https%3A%2F%2Fsocket.io%2Fdocs%2F "https://socket.io/docs/")，使用socekt.io可以非常容易的创建聊天室 用socket.io当信令服务器，后端是node, 前端是vue框架,搭建了一个聊天室，并且可以向聊天室的人发起视频聊天的demo,STUN是用的谷歌提供的免费服务器，TURN是我们公司的TURN服务器，这只是为了让demo可以运行,真实的项目需要自己搭建STUN和TRUN服务器，[github项目地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwxiaoshuang%2Fwebrtc "https://github.com/wxiaoshuang/webrtc")

*   克隆项目
*   npm install 安装依赖
*   node run start在本地的9000端口启动后端服务器
*   npm run dev在8080端口开启前端项目
*   访问http://localhost:8080访问项目，输入用户名和密码，进入聊天室，在用户列表里面可以选择其他用户进行视频聊天(可以通过再开一个窗口，再输入用户名和密码来模拟多人)

图示说明：打开谷歌浏览器，开三个标签页，分别创建A，B, C三个用户，点击加入聊天室后点击开始采集本地视频， A, B, C的页面如下，

*   A![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a458d31aab324f~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)
    

* * *

A页面切换到用户列表，可以看到有A,B和C三个用户,

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a4591729d6d49a~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

点击B后面的互动按钮，给B发送视频互动请求

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a4590830c6eda7~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

切换到B页面，发现B收到了A的互动邀请，点击接受

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a457d3e03ab4e3~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

回到A页面，可以看到本人A和和A互动的用户B

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a457f01c1a6cb9~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

再点击和C互动，C接受互动邀请，那么A现在就可以看到，A,B,C三路视频了,同理，B现在看到A和B两路视频，C看到C和A两路视频，你也可以让B用户和C用户进行互动，这样就是一个三人的视频会议了

此时， A页面如下

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/22/16a45822176a01b6~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

注意： 这个项目启动了两个本地node服务器，socket.io的访问地址是后端服务器也就是http://localhost:9000, 如果需要部署到线上，需要将src/APP.vue的joinRoom的方法中的socket的url替换成线上服务器的地址，

*   src/APP.vue的data的pcConfig里面配置为了stun服务器和turn服务器，为了测试我在项目里面使用的是google的stun服务器和我们公司的turn服务器(之前测试是可以使用的，现在不知道还能不能用)，到时候需要替换成线上的stun服务器和turn服务器
    
    **[Agora SDK 使用体验征文大赛 | 掘金技术征文，征文活动正在进行中](https://juejin.cn/post/6844903811249618958 "https://juejin.cn/post/6844903811249618958")**