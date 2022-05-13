---
title: WebRTC实现屏幕共享
date: 2021-11-25 17:04
tags: [前端, vue, vuex]
---

WebRTC可以实现实时通信，他可以直接建立两个浏览器之间点对点的连接，在连接中实时传输媒体流和任何数据。这就非常适合进行屏幕共享。
<!--more-->

WebRTC使用起来还是挺复杂的，具体的介绍[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)非常详细，不过网上各种，信令服务器，ICE，SDP，NAT的介绍弄得我晕头转向，所以我就找到一个简单的[Demo](https://gist.github.com/WindStormrage/9bc117160fb29f535d29086d7874a8f4)，通过这个Demo来理解怎么使用。

# 一、创建RTCPeer

首先我们模拟创建两个pc端。直接通过RTCPeerConnection创建就好。

```
// 创建两个PeerConnection模拟两个客户端，pc1相当于本地，pc2相当于远端
const pc1 = new RTCPeerConnection();
const pc2 = new RTCPeerConnection();
```

# 二、交换ICE

WebRTC使用了ICE（交互式连接设施）的框架协议，在建立连接之前两个客户端需要互相交换，简单的说就是交换网络信息，告知两个客户端互相的地址。

```
// 交换 ICE 候选
// 可以理解为通知pc2连接pc1的地址
pc1.onicecandidate = e => {
	pc2.addIceCandidate(e.candidate);
};
// 可以理解为通知pc1接pc2地址
pc2.onicecandidate = e => {
	pc1.addIceCandidate(e.candidate);
};
```

# 三、交换SDP

WebRTC使用了SDP（会话描述协议）去交换两端的媒体信息，比如说分辨率格式等信息，主要是交换双方的Offer和Answer，这两个信息叫做提议（Offer）和应答（Answer）非常的形象。

```
// 设置offer和answer，可理解为通知两边另一边的编解码等媒体信息
// 1.pc1创建offer
pc1.createOffer(displayMediaOptions)
  .then(desc => {
  // 2.pc1设置offer
  pc1.setLocalDescription(desc)
  // 3.pc2设置offer
  pc2.setRemoteDescription(desc)
  // 4.pc2创建answer
  pc2.createAnswer()
    .then(answerDesc => {
    // 5.pc2设置answer
    pc2.setLocalDescription(answerDesc)
    // 6.pc1设置answer
    pc1.setRemoteDescription(answerDesc)
  })
})
```

这里双方做的事就是存下自己的信息，然后发送和接收对方的Offer或Answer。步骤如下图所示。

![](https://i.postimg.cc/g2dwLXSt/a1eb03b49c9047a2b44e3162a3bea7c4-tplv-k3u1fbpfcp-zoom-1.png)

# 四、添加媒体

最后还有两件简单的事情需要处理，一个是添加需要传输的媒体，还有就是监听到媒体后使用video去播放。

```
//  将需要传输的流添加给PeerConnection
stream.getTracks().forEach(track => pc1.addTrack(track, stream));

//  远端接收到流，交给video去播放
pc2.ontrack = event => {
  if (videoTag.srcObject !== event.streams[0]) {
    videoTag.srcObject = event.streams[0];
  }
};
```

通过上面的这四个简单步骤我们实现了WebRTC最简单的一个实例。然后有的同学就会有质疑了，你这是在一个浏览器页面下进行的传输，作为端对端的传输能不能在两个页面下分别创建pc1和pc2？

# 五、信令服务器

经过了解，WebRTC只会负责端对端的连接，中间的数据传输，控制传输给谁，和谁建立连接都是他不管的，所以需要我们自己去传输。

通过WebSocket来把pc1和pc2之间的信息传递一下。然后就有了以下伪代码。这里只演示交换ICE候选，交换SDP同理。

PC1的代码：

```
const pc1 = new RTCPeerConnection();
const socket = 你创建的socket连接
// 交换 ICE 候选
pc1.onicecandidate = e => {
  socket.emit('ice的信息', e.candidate)
};

socket.on('ice的信息', data => {
  pc1.addIceCandidate(data);
})
```

PC2的代码：

```
const pc2 = new RTCPeerConnection();
const socket = 你创建的socket连接
// 交换 ICE 候选
pc2.onicecandidate = e => {
  socket.emit('ice的信息', e.candidate)
};

socket.on('ice的信息', data => {
  pc2.addIceCandidate(data);
})
```

看起来很简单不过经过测试后发现连不通，打印出传输后的消息发现传输后的消息都变成了普通对象。

![](https://i.postimg.cc/BnnTbcQR/07dda0eaf5ef476dade7dcf804fc7c81-tplv-k3u1fbpfcp-zoom-1.png)

然而在传输前他是一个RTCIceCandidate的对象

![](https://i.postimg.cc/5ykx3VLB/c70d46c12d9c471f96ed6aa8b182785e-tplv-k3u1fbpfcp-zoom-1.png)

通过查阅API我们可以在传输后实例化这个对象就可以连接了。

```
new RTCIceCandidate(data)
```

下图就是我们所需要进行的所有信息的传输步骤。

![](https://i.postimg.cc/yNn29TF9/b2701955128a4ad9bf96fc422467cccc-tplv-k3u1fbpfcp-zoom-1.png)

其实我们WebSocket做的事情在WebRTC中被叫做信令服务器，信令就是双方发送的所有信息，你可以通过任何形式传输这些信息，WebSocket也好Http请求也可以，因为信令服务器不需要理解这些中间信息，也不需要做额外的处理，唯一要做的就是把信息带到另外一方。

# 六、一对多实现

WebRTC只是一对一两端之间的通信，但是我们存在一对多的需求，不过这种需求量挺小的，所以我们采取了最简单的办法就是同一个客户端创建多个WebRTC。但是在业内也有更加优秀的解决方案，使用SFU Server 让服务器去模拟一个客户端与每个客户端之间建立联系，帮客户端分担压力，这样就把通讯压力放在了我们可控的服务端上了。效果如下图所示。

![](https://i.postimg.cc/C1Q91H8N/c49375aa48a24c89aa4a6bb61cd5a661-tplv-k3u1fbpfcp-zoom-1.png)

在继续优化你可以使用MCU Server 的方案，在服务器合并多路连接。

![](https://i.postimg.cc/pLvVRRRd/b67340ee58b745e196c8b46ad58adf6d-tplv-k3u1fbpfcp-zoom-1.png)

具体内容可以看[这篇文章](https://zhuanlan.zhihu.com/p/56428846)的讲解。

# 七、NAT穿透

最后在项目提测前夕有发现一个大问题，之前开发两个浏览器端都是一台机测试，然后在两台设备上试了下发现一直连接不上，后来了解到是NAT和防火墙的原因导致双方无法通信。

WebRTC使用的ICE整合了STUN和TURN，STUN服务可以获取客户端的公网IP和端口，如果使用STUN依然无法建立连接就需要用到，TURN服务，可以中继所有的数据来绕过NAT。google提供了免费的STUN服务器，不过也可以自己搭建，搭建的话可以了解一下[coturn](https://github.com/coturn/coturn)。在WebRTC上配置对应的服务也非常简单。

```
const pc = new RTCPeerConnection({
  // 可以传入多个stun服务器或者turn服务器
  iceServers: [
    { url: 'stun:stun.l.google.com:19302' },
    { url: 'stun:stun1.l.google.com:19302' }
  ]
})
```

![image.png](https://i.postimg.cc/xCXntdy6/565176b02aad419a915ba6e4d464eb8d-tplv-k3u1fbpfcp-zoom-1.webp)