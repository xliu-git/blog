---
date: '2025-04-09T19:33:43+08:00'
draft: false       
title: 'WebRTC小结'
tags: ["WebRTC"]
---

### WebRTC
一种应用在Web应用上的流式传输协议
* RTCPeerConnection：WebRTC的连接。
    * `addTrack()`：添加媒体流。
    * `createOffer()`：创建一个信息SDP offer，包含必要的如ICE信息、媒体流信息，来启动与peer的WebRTC连接。
* MediaStream：媒体流，提供了用于处理媒体流及其组成轨道的接口和方法，e.g. 由`getUserMedia()`创建的MediaStream就是本地用户麦克风和相机的原输入，可以向RTC连接中添加。
* RTCDataChannel：双向数据通道，有`onopen`, `onmessage`, `onerror`等事件处理器，可以向RTC连接中添加。
* SDP：会话描述协议，描述P2P连接的标准，音视频的编解码器。
* ICE：交互式连接建立，在NAT的情况下依然能建立P2P连接
    * STUN服务器：提供了一种寻找NAT设备后的IP以及端口的方案，公网上有一些免费的STUN服务器。
    * TURN服务器：复杂防火墙的情况下，一种对等连接的解决方案，有开源的TURN服务器项目。
#### 信令
不同网络上的两个设备建立连接的过程需要有发现机制和媒体协商，以及两个设备连接到第三个共同商定的服务器，这个过程就是信令。信令可以通过WebSocket或者Http建立。
* 交换SDP：请求方会创建一个offer并发送到接收者那里，接收者返回一个应答，双方此时都知道使用什么样的代码和参数进行通信。
* 交换ICE候选：通过交换ICE Candidates建立具体连接路径，媒体流通过该连接开始运转。


### References
1. https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API
2. https://webrtc.org/getting-started/peer-connections?hl=zh-cn

