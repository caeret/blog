---
title: "在 golang 中使用 redis cluster"
date: 2017-03-17T20:03:17+08:00
categories: ["go"]
tags: ["go", "redis"]
---

最近用 `Go` 来写部门的一组服务接口 ，其中使用到了 `redis`。部门内部使用的 `redis` 服务端由 `codis` 迁移到了 `redis cluster`，于是需要能支持 `redis` 官方集群模式的库。看了一下发现 `redis` 官方推荐的有 `redigo` 和 `radix` 两个库。经过简单比较后，还是选择了更简洁的 `radix`。

<!--more-->

## 连接集群

```go
addr := "192.168.10.1:6033"
password := "redispassword"
client, err = cluster.NewWithOpts(cluster.Opts{
    Addr:addr,
    Dialer:func(network, addr string) (*redis.Client, error) {
        conn, err := redis.DialTimeout(network, addr, 2 * time.Second)
        if err != nil {
            return nil, err
        }
        return conn, nil
    },
})
if err != nil {
    // do something with the error.
}
```

通过以上代码即可连接到 redis 集群。如果恰巧你的集群需要认证密码，则可在 Dialer 方法返回连接前增加：

```go
if conn.Cmd("AUTH", password).Err != nil {
    return nil, err
}
```

`radix` 会在连接服务端后根据服务端配置的服务器创建相同个数的连接池，连接池大小可以在 `cluster.Opts` 中配置，默认为 `10`。另外还有其他可配置参数，如连接超时等。

## 使用 redis 命

```go    
resp := client.Cmd("GET", "FOO")
```

`radix` 在 `Cluster` 的 `Cmd` 方法中封装了集群的 `key` 分布处理逻辑，直接调用即可。

## 遇到的问题

在使用过程中，发现过一段时间执行 Cmd 命令会有报错，提示 “Could not get cluster info: EOF” 或者直接报 “EOF”。经过测试确认服务端有 300 秒的超时配置，如果 300 秒内客户端连接都没有和服务端进行通信，则会服务端会断开和客户端连接。那么这个时候就需要一个类似心跳的功能，能持续告诉服务端「我还活着」。在创建Cluster 客户端的时候，我们也顺便启动一个来发心跳的 goroutine。

```go
func keepAlive(c *cluster.Cluster, interval time.Duration) {
    for {
        <-time.After(interval)
        // redis cluster heart beat.
        clients, err := c.GetEvery()
        if err != nil {
            // unable to do redis heart beat, log the error.
            continue
        }
        for _, c := range clients {
            if err := c.Cmd("PING").Err; err != nil {
                // unable to keep redis conn alive, log the error.
                c.Close()
                continue
            }
            c.Put(c)
        }
        // finished redis heart beat.
    }
}
```

