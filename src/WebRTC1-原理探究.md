---
title: WebRTC1-原理探究
tags: 
- FE
- HTML5
- WebRTC
categories: 前端技术
date: 2016-09-28 00:00:00
---
### 1.抛砖引玉   
`WebRTC (Web Real-Time Communications)` 是一项实时通讯技术，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流或/和音频流或者其他任意数据的传输      
实时查看WebRTC在浏览器中的支持情况： http://caniuse.com/#search=webRTC
FirFox 45+,Chrome 29+,Oprea 36+,Edge 14+,Android Brower 50+支持，其余支持情况有问题。   

备注：有的时候会使用`adapter.js`，这个js文件是为了提高兼容性，可以直接使用API 不用加前缀  
使用：  
- 下载  
- 引用 ``<script src="adapter.js">``
>https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/adapter.js  

**几个概念**   
- *SDP(Session Description Protocol)*    
SDP是一种会话描述协议，用来描述双方的IP地址和端口号，通信所使用的带宽，会话的名称、标识符、激活时间，双方所要传输的媒体类型（视频、音频、文本）、媒体格式等等。该协议仅包含所要传递的媒体的描述信息，而不直接传递媒体内容。             
- *ICE（Interactive Connectivity Establishment）*    
ICE是一种以UDP为基础用于实现穿越NAT网管或者防火墙的协议。              
- *TURN&&STUN*    
两种协议都是用来明确自己的外网地址的，差别是如果要服务器辅助进行数据交换则设置TURN服务器，不需要则设置STUN服务。           

**核心API**    
- *Navigator.getUserMedia*
用来获取视频和音频，在浏览器装有摄像头和麦克风的情况下使用.`navigator.getUserMedia(constraints, successCallback, errorCallback)`；constraints是用来控制视频和音频是否获取，一般设为{video:true,audio: true}，即视频和音频都获取。      
- *RTCPeerConnection*     
RTCPeerConnection是一个表示两个浏览器端的连接的对象，其含有关于这个连接的所有信息和相关方法，是WebRTC的核心API，负责制作建立连接的SDP、ICE等报文，管理连接状态等等。      
- *RTCDataChannel*    
RTCDataChannel由RTCPeerConnection创建，需要传递视频、音频以外的数据时使用，它代表浏览器两端间的一个数据通道，和这个数据通道有关的属性和方法都记录在这个对象里。 
    
### 2.按图索骥
 
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webRTC.png)    
 **过程:**    
1.  首先双方都建立一个RTCPeerConnection的实例，其中一方（称为offer方）用`RTCPeerConnection.createOffer()`创建一个会话描述`sessionDescription`，该会话描述包含SDP报文信息和该sessionDescription的类型(type)    
2.  接下来调用`RTCPeerConnection.setLocalDescription()`方法将本地的`localDescription`设置为刚才创建的`sessionDescription`。之后将创建的`sessionDescription`发送给对方（称为answer方），发送方式没有规定，可以通过服务器中转，可以通过IM软件发送(这里使用WebSocket信令服务器)。    
3. answer端接收到`sessionDescription`后调用`RTCPeerConnection. setRemoteDescription`方法设置,然后调用`RTCPeerConnection. createAnswer`方法产生自己的`sessionDescription`。    
4. 再将创建的`sessionDescription`发送给offfer方，同样发送方式没有规定。offer方接收到`sessionDescrip`后调用`RTCPeerConnection. setRemoteDescription`方法设置，这样双方的SDP信息就交换完成了。   
5. 在完成SDP的交换后双方还要交换ICE candidate信息。双方首先设置`RTCPeerConnection.onicecandidate`回调函数，当candidate可用时，双方中的一方将所有`icecandidate`发送给对方，发送方式同样没有规定，接收方调用`RTCPeerConnection.addIceCandidate`方法接收candidate信息。经过这些步骤后双方连接就建立完成了。   

### 3.纸上可谈兵   
#### 3.1.由简入繁   
html文件   
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta name="keywords" content="JavaScript, WebRTC" />
    <meta name="description" content="WebRTC" />
    <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1">
    <title>获取本地视频</title>
    <link rel="stylesheet" href="simpleVideo.css">
</head>

<body>
    <!-- <video /> -->
    <video></video>
    <p><br></p>
    <script src='../lib/adapter.js'></script>
    <script type="text/javascript" src="simpleVideo.js"></script>
</body>
</html>
```
js文件：   
```javascript
var constraints = {
    video: true,
    audio:true
};

function successCallback(stream) {
    window.stream = stream; // stream available to console
    console.log(stream);
    console.log(stream.getVideoTracks());
    console.log(stream.getAudioTracks());

    var video = document.querySelector("video");
    video.src = window.URL.createObjectURL(stream);
    video.play();
}

function errorCallback(error) {
    console.log("getUserMedia error: ", error);
}

getUserMedia(constraints, successCallback, errorCallback);
```

#### 3.2.顺藤摸瓜   
这是个完整的例子，参照`2.按图索骥`部分理解
```html
<!DOCTYPE html>
<html>

<head>
    <meta name="keywords" content="JavaScript, WebRTC" />
    <meta name="description" content="WebRTC codelab" />
    <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1">
    <title>WebRTC codelab: step 2</title>
    <link rel="stylesheet" href="css/index.css">
    <!-- css可以使用一些滤镜效果 -->
    <style>

    </style>
    <script src='js/lib/adapter.js'></script>
</head>

<body>
    <video id="localVideo" autoplay muted></video>
    <video id="remoteVideo" autoplay muted></video>
    <div>
        <button id="startButton">Start</button>
        <button id="callButton">Call</button>
        <button id="hangupButton">Hang Up</button>
    </div>
    <script src="index.js">
</body>

</html>

```
`index.js`
```javascript
	<script>
    var localStream, localPeerConnection, remotePeerConnection;

    var localVideo = document.getElementById("localVideo");
    var remoteVideo = document.getElementById("remoteVideo");

    var startButton = document.getElementById("startButton");
    var callButton = document.getElementById("callButton");
    var hangupButton = document.getElementById("hangupButton");
    startButton.disabled = false;
    callButton.disabled = true;
    hangupButton.disabled = true;
    startButton.onclick = start;
    callButton.onclick = call;
    hangupButton.onclick = hangup;

    function trace(text) {
        console.log((performance.now() / 1000).toFixed(3) + ": " + text);
    }

    function gotStream(stream) {
        trace("Received local stream");
        localVideo.src = URL.createObjectURL(stream);
        localStream = stream;
        callButton.disabled = false;
    }

    function start() {
        trace("Requesting local stream");
        startButton.disabled = true;
        getUserMedia({
                audio: true,
                video: true
            }, gotStream,
            function(error) {
                trace("getUserMedia error: ", error);
            });
    }

    function call() {
        callButton.disabled = true;
        hangupButton.disabled = false;
        trace("Starting call");

        if (localStream.getVideoTracks().length > 0) {
            trace('Using video device: ' + localStream.getVideoTracks()[0].label);
        }
        if (localStream.getAudioTracks().length > 0) {
            trace('Using audio device: ' + localStream.getAudioTracks()[0].label);
        }

        var servers = null;//本机测试不用其他服务器

        localPeerConnection = new RTCPeerConnection(servers);//offer方
        trace("Created local peer connection object localPeerConnection");
        localPeerConnection.onicecandidate = gotLocalIceCandidate;//offer方发送ICE
        remotePeerConnection = new RTCPeerConnection(servers);//answe方
        trace("Created remote peer connection object remotePeerConnection");
        remotePeerConnection.onicecandidate = gotRemoteIceCandidate;//answer方发送ICE
        remotePeerConnection.onaddstream = gotRemoteStream;//设置视频流

        localPeerConnection.addStream(localStream);
        trace("Added localStream to localPeerConnection");
        localPeerConnection.createOffer(gotLocalDescription, handleError);//作为offer，产生自己的SessionDescription【SD】信息
    }

    function gotLocalDescription(description) {//description是offer方的SD
        localPeerConnection.setLocalDescription(description);
        trace("Offer from localPeerConnection: \n" + description.sdp);
        remotePeerConnection.setRemoteDescription(description);//answer方接收offer的SD
        remotePeerConnection.createAnswer(gotRemoteDescription, handleError);//answer方发送自己的SD
    }

    function gotRemoteDescription(description) {
        remotePeerConnection.setLocalDescription(description);//anwer方设置本身自己的SD
        trace("Answer from remotePeerConnection: \n" + description.sdp);
        localPeerConnection.setRemoteDescription(description);//offer接收answer方的SD
    }

    function hangup() {
        trace("Ending call");
        localPeerConnection.close();
        remotePeerConnection.close();
        localPeerConnection = null;
        remotePeerConnection = null;
        hangupButton.disabled = true;
        callButton.disabled = false;
    }

    function gotRemoteStream(event) {
        remoteVideo.src = URL.createObjectURL(event.stream);
        trace("Received remote stream");
    }

    function gotLocalIceCandidate(event) {
        if (event.candidate) {
            remotePeerConnection.addIceCandidate(new RTCIceCandidate(event.candidate));//answer方接收ICE
            trace("Local ICE candidate: \n" + event.candidate.candidate);
        }
    }

    function gotRemoteIceCandidate(event) {
        if (event.candidate) {
            localPeerConnection.addIceCandidate(new RTCIceCandidate(event.candidate));//offer方接收ICE
            trace("Remote ICE candidate: \n " + event.candidate.candidate);
        }
    }

    function handleError() {}
    </script>
```
另外加一些CSS可以很容易实现滤镜效果
```css
	video {
    filter: hue-rotate(180deg) saturate(200%);
    -moz-filter: hue-rotate(180deg) saturate(200%);
    -webkit-filter: hue-rotate(180deg) saturate(200%);
}
```
运行这两段代码，就可以调出来本地视频窗口   

### 参考资料
[强烈推荐的WebRTC入门教程](https://bitbucket.org/webrtc/codelab)   
[google webRTC](https://sites.google.com/site/webrtc/)  
[WebRTC官网](https://webrtc.org/)  
[WebRTC-W3School](http://w3c.github.io/webrtc-pc/)  
[MDN-WebRTC](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)

[WebRTC实践教程](https://segmentfault.com/a/1190000000436544)  
[使用WebRTC搭建前端视频聊天室——信令篇](http://www.tuicool.com/articles/eYJvee)  
[可以用WebRTC来做视频直播吗？-知乎](https://www.zhihu.com/question/25497090)  
[实时猫--WebRTC服务商](https://shishimao.com/)  