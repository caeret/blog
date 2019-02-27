---
title: "关于 HTTP 消息体长度的确定"
date: 2016-08-20T15:15:19+08:00
categories: ["HTTP"]
tags: ["HTTP"]
---
最近在面试中聊到关于 `HTTP` 首部的 `Content-Length` 的一个问题。

> `Content-Length` 字段是必须的吗？

之前只了解过一部分相关知识点，只能确定在使用长连接通讯时可以靠 `Content-Length` 来确定消息体的长度。如果是在传输时不能确定总长度是多少，应该是在传送消息体的部分定义了一种协议，可以来确定消息体长度。之前也经常在头部看到 `Transfer-Encoding: chunked` 这个字段，但是没有太多了解，查了一部分资料，做个补充。

<!--more-->

*HTTP 通讯过程中可以没有 Content-Length 字段。*

如果客户端和服务端是通过短连接方式进行通讯，则可以通过服务端关闭连接来确定消息体长度。

如果服务端和客户端支持 `Connection: Keep-Alive` ([HTTP持久连接] [2]) 保持连接，则可通过 `Transfer-Encoding` 和 `Content-Length` 确定消息体长度。

  * 优先通过 `Transfer-Encoding` (比如 `chunked` [分块传输编码] [1]) 来确定消息体长度。这样可以在不确定消息体长度的情况下发送数据给客户端。比如发送大量数据时可先进行 `gzip` 压缩，在压缩的同时即可开始发送数据，而不必等待压缩完成后再进行数据发送。
  * 如果有 `Content-Length`， 则 `Content-Length` 必须等于消息体长度。服务端或者客户端可通过该字段来判断是否接受完数据。

另外，之前在请求头部与响应头部也会看到这俩个字段，`Accept-Encoding`，`Content-Encoding`。
`Accept-Encoding` 字段用户请求时告诉服务端客户端这边可以接受的消息体编码方式，比如 `Accept-Encoding: gzip` 等，这样服务端通过该字段可以知道客户端支持 `gzip` 方式来编码消息体，就可以将消息体使用 `gzip` 压缩后发送，发送前需要在首部告知客户端 `Content-Encoding: gzip`。
  

[1]: https://zh.wikipedia.org/wiki/%E5%88%86%E5%9D%97%E4%BC%A0%E8%BE%93%E7%BC%96%E7%A0%81 "分块传输编码"
[2]: https://zh.wikipedia.org/wiki/HTTP%E6%8C%81%E4%B9%85%E8%BF%9E%E6%8E%A5 "HTTP持久链接"



