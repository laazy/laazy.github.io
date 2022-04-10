---
layout: post
title: WebSocket 是什么以及如何在 Spring 中使用 WebSocket
tags: Spring kotlin WebSocket stomp
---

# WebSocket 是什么以及如何在 Spring 中使用 WebSocket

## 概述

我们一般在 Spring 框架中开发 Web 应用，往往使用**RESTful 风格**或**MVC 模式**，这两种方式都有一个共同点就是**只允许单向通信**，也就是客户端发送请求，服务端返回响应的方式。但是开发一个应用很自然会有双向通信的需求，而 IETF 在 2011 年在 `RFC 6455` 中提出了 WebSocket 协议，允许客户端和服务端之间进行**全双工通信**。[^websocket]

本文将介绍 WebSocket 以及在 Spring 中使用 Websocket 会遇到的一系列问题。

## WebSocket 协议介绍 [^websocket]

WebSocket 与 HTTP 协议一样，都属于 OSI 七层模型中的应用层，都依赖于 TCP 协议进行数据传输。由于为了浏览器使用，WebSocket 协议被设计为与 HTTP 协议兼容，都默认运行于 443 或者 80 端口，并且 WebSocket 协议使用 HTTP 协议进行握手，握手完成后，将连接转为 WebSocket 协议。

WebSocket 协议使用`ws://`或`wss://`作为 URI 的开头，其中`wss://`表示 WebSocket Secure，其建立连接后连接将会一直存在，客户端和服务端可以双向发送信息，任意一方断开连接则连接关闭。

WebSocket 总体来说比较简单，只是一个和 HTTP 协议同级的双全工协议，比较重要的是以什么格式在 WebSocket 上传输数据以及 Server 和 Client 如何处理 WebSocket 连接和信息。

WebSocket 并**不遵循 Same Origin Policy(SOP)**，因此当需要检查 SOP 时，需要自行编写代码。

## Spring WebSocket

Spring 提供了一篇[教程](https://Spring.io/guides/gs/messaging-stomp-WebSocket/)以说明如何在 Spring 中使用 WebSocket，但是这篇教程反而引入了更多问题，而这些问题更多的与 WebSocket 无关，只是这篇教程额外增加的复杂性。

- 什么是 STOMP？
- 什么是 SockJS？
- 为什么在浏览器/Postman 中用 WebSocket 无法连接 Spring WebSocket？

下面我将一一解释这些问题

### 什么是 STOMP[^stomp]

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
`Body`则以 UTF-8 编码，最后以`NULL`结尾标识`BODY`的结束。

### 什么是 SockJS[^sockjs]

SockJS 是一个浏览器端的 javascript 库，旨在提供一个 WebSocket-like 对象，允许不支持 WebSocket 协议的浏览器能够像支持 WebSocket 一样使用其提供的`SockJS`对象。SockJS 的目的是屏蔽双全工交流细节，使用上层应用封装 WebSocket、HTTP Streaming、HTTP Polling 等协议细节，使用户不需感知和考虑各种协议与浏览器以及服务端的兼容问题。

SockJS 客户端需要服务端也有对应的 SockJS 服务端，但服务端 SockJS 也支持原生 WebSocket 连接。

### 为什么在浏览器/Postman 中用 WebSocket 无法连接 Spring WebSocket？

当在浏览器或其他开发工具中使用类似`ws = new WebSocket("ws://localhost:8080/gs-guide-WebSocket")`的方式连接 Spring 后端时，会回报`HTTP 400`错误。
这正是由于按照教程中的

```java
registry.addEndpoint("/gs-guide-WebSocket").withSockJS();
```

实际上启用了`SockJSServiceRegistration`，使得在获得 Handler Mapping 时，将所有此处设置的路径全部映射使用了`SockJSHttpRequestHandler`来处理，进而使原生`WebSocket`对象无法连接至服务端给定端点。

修复方法有两条：

- 去掉`withSockJS()`，在服务端不使用 SockJS。
- 在连接 URI 后缀上`/WebSocket`，即`ws://localhost:8080/gs-guide-WebSocket/WebSocket`，告诉 Spring WebSocket 使用原生 WebSocket 连接。[^sockjs_spring]


[^websocket]: [wikipedia of WebSocket](https://en.wikipedia.org/wiki/WebSocket#:~:text=WebSocket%20is%20a,Mozilla%2C%20and%20Microsoft).)
[^stomp]: [STOMP Official Document](https://stomp.github.io/stomp-specification-1.2.html)
[^sockjs]: [SockJS github](https://github.com/SockJS/SockJS-client)
[^sockjs_spring]: [Spring SockJS overview](https://docs.spring.io/spring-framework/docs/5.3.x/reference/html/web.html#websocket-fallback-sockjs-overview)
