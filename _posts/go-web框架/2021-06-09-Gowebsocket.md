---
title: Go-websocket
date: 2022-01-12 10:34:00 +0800
categories: [Go]
tags: [Go网络]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../../libits.github.io
math: false
mermaid: true
---

##  socket图解

>Socket是应用层与TCP/IP协议族通信的中间软件抽象层。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket后面，对用户来说只需要调用Socket规定的相关函数，让Socket去组织符合指定的协议数据然后进行通信。

![socket图解](/assets/blog_res/2021-06-09-Gowebsocket.assets/3.png)

##  websocket

- WebSocket是一种在单个TCP连接上进行全双工通信的协议
- WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据
- 在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输
- 需要安装第三方包：go get -u -v github.com/gorilla/websocket
