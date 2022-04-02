---
layout: post
title: WebSocket 是什么以及如何在 Spring 中使用 WebSocket
tags: Spring kotlin WebSocket stomp
---

# WebSocket 是什么以及如何在 Spring 中使用 WebSocket

## 概述

我们一般在 Spring 框架中开发 Web 应用，往往使用**RESTful 风格**或**MVC 模式**，这两种方式都有一个共同点就是**只允许单向通信**，也就是客户端发送请求，服务端返回响应的方式。但是开发一个应用很自然会有双向通信的需求，而 IETF 在 2011 年在 `RFC 6455` 中提出了 WebSocket 协议，允许客户端和服务端之间进行**全双工通信**。[^WebSocket]

## WebSocket 协议介绍 [^WebSocket]

WebSocket 与 HTTP 协议一样，都属于 OSI 七层模型中的应用层，都依赖于 TCP 协议进行数据传输。由于为了浏览器使用，WebSocket 协议被设计为与 HTTP 协议兼容，都默认运行于 443 或者 80 端口，并且 WebSocket 协议使用 HTTP 协议进行握手，握手完成后，将连接转为 WebSocket 协议。

WebSocket 协议使用`ws://`或`wss://`作为 URI 的开头，其中`wss://`表示 WebSocket Secure，其建立连接后连接将会一直存在，客户端和服务端可以双向发送信息，任意一方断开连接则连接关闭。

WebSocket 总体来说比较简单，只是一个和 HTTP 协议同级的双全工协议，比较重要的是以什么格式在 WebSocket 上传输数据以及 Server 和 Client 如何处理 WebSocket 连接和信息。

## Spring WebSocket

Spring 提供了一篇[教程](https://Spring.io/guides/gs/messaging-stomp-WebSocket/)以说明如何在 Spring 中使用 WebSocket，但是这篇教程反而引入了更多问题，而这些问题更多的与 WebSocket 无关，只是 Spring WebSocket 增加的额外的复杂性。

- 什么是 STOMP？
- 什么是 SockJS？
- 为什么在浏览器/Postman 中用 WebSocket 无法连接 Spring WebSocket？

下面我将一一解释这些问题

### 什么是 STOMP[^STOMP]

STOMP 是一个简单的文本流协议，其全名为 Simple(or Streaming) Text Oriented Message Protocol，提供了一种可互操作的连接格式，允许任何 STOMP 客户端与任何支持该协议的消息代理交互。

STOMP 协议是一种以帧为基础的协议，每个帧的构成如下。

```
COMMAND
header1:value1
header2:value2

Body^@
```

其中`COMMAND`指当前帧的作用，区分客户端和服务端，两侧支持的`COMMAND`各不相同，客户端支持：`SEND SUBSCRIBE UNSUBSCRIBE BEGIN COMMIT ABORT ACK NACK DISCONNECT`，服务端支持：`MESSAGE RECEIPT ERROR`。
`header:value`随着`COMMAND`的不同而不同，对应不同的`COMMAND`均有必须填入的`header`，其格式为`<key>:<value>`。
`Body`则以UTF-8编码，最后以`NULL`结尾标识`BODY`的结束。





[^WebSocket]: [wiki pedia of WebSocket](https://en.wikipedia.org/wiki/WebSocket#:~:text=WebSocket%20is%20a,Mozilla%2C%20and%20Microsoft).)
[^STOMP]: [STOMP Official Document](https://stomp.github.io/stomp-specification-1.2.html)
