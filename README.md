# LearnWebRTC
收集一些关于WebRTC的资料
## 基础概念
* [Signaling server](#Signaling-server)
* [STUN/TURN server](#STUN/TURN-server)
* [offer/answer](#offer/answer)
* [IceCandidate](#IceCandidate)
* [SDP](#SDP)
* [NAT](#NAT)

### Signaling server
> https://html5rocks.com/en/tutorials/webrtc/infrastructure/#what-is-signaling

信令是协调通信的过程。为了让WebRTC应用建立一个呼叫，其客户端需要交换以下信息。

1. 用于打开或关闭通信的会话控制信息
1. 错误信息
1. 媒体元数据，如编解码器、编解码器设置、带宽和媒体类型
1. 用于建立安全连接的密钥数据
1. 网络数据，如外界看到的主机的IP地址和端口

这个信令过程需要一种方法让客户来回传递信息。这种机制不是由WebRTC APIs实现的。

**为什么WebRTC没有定义信令？**

为了避免冗余，并最大限度地与现有技术兼容，WebRTC标准没有指定信令方法和协议。

### STUN/TURN server

1. STUN

    NAT的会话穿越功能[Session Traversal Utilities for NAT (STUN)](https://en.wikipedia.org/wiki/STUN) (缩略语的最后一个字母是NAT的首字母)是一个允许位于NAT后的客户端找出自己的公网地址，判断出路由器阻止直连的限制方法的协议。

    客户端通过给公网的STUN服务器发送请求获得自己的公网地址信息，以及是否能够被（穿过路由器）访问。

1. TURN

    一些路由器使用一种“对称型NAT”的NAT模型。这意味着路由器只接受和对端先前建立的连接（就是下一次请求建立新的连接映射）。

    NAT的中继穿越方式[Traversal Using Relays around NAT (TURN)](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) 通过TURN服务器中继所有数据的方式来绕过“对称型NAT”。你需要在TURN服务器上创建一个连接，然后告诉所有对端设备发包到服务器上，TURN服务器再把包转发给你。很显然这种方式是开销很大的，所以只有在没得选择的情况下采用。

### offer/answer
>https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Connectivity#%E4%BC%9A%E8%AF%9D%E6%8F%8F%E8%BF%B0

当用户对另一个用户启动WebRTC调用时，将创建一个称为提议(offer)的特定描述。 该描述包括有关呼叫者建议的呼叫配置的所有信息。 接收者然后用应答(answer)进行响应，这是他们对呼叫结束的描述。 以这种方式，两个设备彼此共享以便交换媒体数据所需的信息。 该交换是使用交互式连接建立(ICE)(ICE处理的，这是一种协议，即使两个设备通过网络地址转换(NAT)。

然后，每个对等端保持两个描述：描述本身的本地描述和描述呼叫的远端的远程描述。

在首次建立呼叫时，还可以在呼叫格式或其他配置需要更改的任何时候执行提议/应答过程。 无论是新呼叫还是重新配置现有的呼叫，这>些都是交换提议和回答所必需的基本步骤，暂时忽略了ICE层：

1. 呼叫者通过 navigator.mediaDevices.getUserMedia() (en-US) 捕捉本地媒体。
1. 呼叫者创建一个RTCPeerConnection 并调用 RTCPeerConnection.addTrack() (注： addStream 已经过时。)
1. 呼叫者调用 ("RTCPeerConnection.createOffer()")来创建一个提议(offer).
1. 呼叫者调用 ("RTCPeerConnection.setLocalDescription()") 将提议(Offer)   设置为本地描述 (即，连接的本地描述).
1. setLocalDescription()之后, 呼叫者请求 STUN 服务创建ice候选(ice candidates)
1. 呼叫者通过信令服务器将提议(offer)传递至 本次呼叫的预期的接受者.
1. 接受者收到了提议(offer) 并调用 ("RTCPeerConnection.setRemoteDescription()") 将其记录为远程描述 (也就是连接的另一端的描述).
1. 接受者做一些可能需要的步骤结束本次呼叫：捕获本地媒体，然后通过RTCPeerConnection.addTrack()添加到连接中。
1. 接受者通过("RTCPeerConnection.createAnswer()")创建一个应答。
1. 接受者调用 ("RTCPeerConnection.setLocalDescription()") 将应答(answer)   设置为本地描述. 此时，接受者已经获知连接双方的配置了.
1. 接受者通过信令服务器将应答传递到呼叫者.
1. 呼叫者接受到应答.
1. 呼叫者调用 ("RTCPeerConnection.setRemoteDescription()") 将应答设定为远程描述. 如此，呼叫者已经获知连接双方的配置了.

### IceCandidate
[交互式连接建立](https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment)（ICE）是计算机网络中使用的一种技术，在点对点网络中为两台计算机找到尽可能直接对话的方法。

>除了交换关于媒体的信息(上面提到的Offer / Answer和SDP )中，对等体必须交换关于网络连接的信息。 这被称为ICE候选者，并详细说明了对等体能够直接或通过TURN服务器进行通信的可用方法。 通常，每个对点将优先提出最佳的ICE候选，逐次尝试到不佳的候选中。 理想情况下，候选地址是UDP(因为速度更快，媒体流能够相对容易地从中断恢复 )，但ICE标准也允许TCP候选。
https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Connectivity#%E4%BB%80%E4%B9%88%E6%98%AFice%E5%80%99%E9%80%89%E5%9C%B0%E5%9D%80%EF%BC%9F


### SDP
[会话描述协议](https://en.wikipedia.org/wiki/Session_Description_Protocol)（SDP）是一种用于描述多媒体通信会话的格式，以达到宣布和邀请的目的。 其主要用途是支持流媒体应用，如IP语音（VoIP）和视频会议。SDP本身并不提供任何媒体流，而是在端点之间用于协商网络指标、媒体类型和其他相关属性。这些属性和参数的集合被称为会话配置文件。

SDP是可扩展的，以支持新的媒体类型和格式。SDP最初是会话公告协议（SAP）的一个组成部分，但在与实时传输协议（RTP）、实时流媒体协议（RTSP）、会话发起协议（SIP）的结合中发现了其他用途，并作为描述组播会话的独立协议。

### NAT
[网络地址转换](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2)（英语：Network Address Translation，缩写：NAT；又称网络掩蔽、IP掩蔽）在计算机网络中是一种在IP数据包通过路由器或防火墙时重写来源IP地址或目的IP地址的技术。这种技术被普遍使用在有多台主机但只通过一个公有IP地址访问互联网的私有网络中。它是一个方便且得到了广泛应用的技术。当然，NAT也让主机之间的通信变得复杂，导致了通信效率的降低。


## 参考资料
1. [Getting Started](https://webrtc.github.io/webrtc-org/start/)
1. [Get Started with WebRTC](https://www.html5rocks.com/en/tutorials/webrtc/basics/#toc-disruptive)
1. [Real-time communication with WebRTC: Google I/O 2013](https://www.youtube.com/watch?v=p2HzZkd2A40)
1. [Why was SCTP Selected for WebRTC’s Data Channel?](https://bloggeek.me/sctp-data-channel/)
1. [Google: What's next for WebRTC?](https://www.youtube.com/watch?v=HCE3S1E5UwY)
1. [WebRTC connectivity](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Connectivity)
1. [WebRTC 协议介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API/Protocols)
1. [WebRTC权威指南](https://book.douban.com/subject/26915289/)
1. [WebRTC音视频实时互动技术 原理、实战与源码分析](https://www.amazon.cn/dp/B09B2BSMLG)

## 可以做什么
* https://webrtc.github.io/samples/
