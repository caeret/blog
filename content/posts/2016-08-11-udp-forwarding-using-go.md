---
title: "使用 Go 进行 UDP 转发"
date: 2016-08-11T22:21:20+08:00
categories: ["go"]
tags: ["go", "udp"]
---
最近开始接手处理一个新项目，这个项目中有一部分服务是用 `UDP` 进行通信的，并且只有线上环境。大家都是在线上直接进行开发的，但是对于我个人来说总觉得有点别扭。我还是喜欢有一套本地开发环境，方便调试，这样也不会对线上服务造成太大影响。

我尝试写一个可以进行 `udp` 转发的小功能。这样我在本地进行开发，将 `udp` 请求转发到服务器的 `A` 端口，然后在服务器进行转发，这样就可以愉快地在本地进行开发了。

<!--more-->

于是，就有了下面这段代码。

```go
package main

import (
    "net"
    "time"
    "log"
    "os"
)

const LocalAddr = ":25001" //本地监听地址
const AllowedIp = "111.203.228.26" //允许的请求IP
const TargetAddr = "192.168.200.1:8877" //转发地址
const BufferSize = 65535
const ErrorMessage = "{\"message\": \"error\"}" //未授权请求IP响应内容

var logger = log.New(os.Stdout, "udp => ", log.Ldate | log.Ltime)

func main() {
    addr, err := net.ResolveUDPAddr("udp", LocalAddr)
    if err != nil {
        panic(err)
    }

    logger.Println("try to listen on:", addr)

    // 监听
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        panic(err)
    }

    defer conn.Close()

    logger.Println("try to handle request on:", addr)

    for {
        handleConn(conn)
    }
}

func handleConn(conn *net.UDPConn) {
    // 读取请求
    request := make([]byte, BufferSize)
    n, remoteAddr, err := conn.ReadFromUDP(request)
    if err != nil {
        logger.Println("fail to read from UDP:", err)
        return
    }

    logger.Printf("%d bytes read from %s: \"%s\"", n, remoteAddr.String(), string(request))

    // 转发
    if remoteAddr.IP.String() == AllowedIp {
        logger.Println("try to forward request")
        go forward(conn, remoteAddr, request[:n])
    } else {
        logger.Println("invalid request from ip", remoteAddr)
        go sendError(conn, remoteAddr)
    }
}

func forward(conn *net.UDPConn, remoteAddr *net.UDPAddr, request []byte) {
    // 转发请求到目标地址
    logger.Println("try to connect target service")
    targetConn, err := net.DialTimeout("udp", TargetAddr, time.Second * 1)
    if err != nil {
        logger.Println("unable to connect target service with error:", err)
        return
    }

    defer targetConn.Close()

    targetConn.SetWriteDeadline(time.Now().Add(time.Second * 1))

    logger.Printf("try to write request %s to target\n", string(request))
    n, err := targetConn.Write(request)
    if err != nil {
        logger.Printf("unable to write request to target: %s\n", err.Error())
        return
    }
    logger.Printf("write %d bytes of request to target\n", n)

    data := make([]byte, BufferSize)
    n, err = targetConn.Read(data)
    if err != nil {
        logger.Printf("unable to read from target: %s\n", err.Error())
        return
    }

    logger.Printf("read %d bytes from target: \"%s\"\n", n, string(data))
    n, err = conn.WriteToUDP(data[:n], remoteAddr)
    if err != nil {
        logger.Println("fail to write to remote addr:", remoteAddr, string(request), err)
        return
    }
    logger.Println(n, "bytes write to remote addr:", remoteAddr)
}

func sendError(conn *net.UDPConn, remoteAddr *net.UDPAddr) {
    conn.WriteToUDP([]byte(ErrorMessage), remoteAddr)
}
```
