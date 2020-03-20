---
title: RPC入门
copyright: true
date: 2020-01-29 15:03:24
tags:
	- go
	- RPC
	- gRPC
categories:
	- go
description:
---

# RPC入门=========φ(≧ω≦*)♪==========

>RPC:远程过程调用，分布式系统中不同节点间流行的通信方式

<!--more-->

先来个Hello World

```go
type HelloService struct{}

//Hello 这个方法必须满足Go的RPC的规则：方法只能有两个可序列化的参数，其中第二个是指针类型， 必须是公开的方法，即可导出
func (h *HelloService) Hello(request string, reply *string) error{
    *reply = "Hello World"
    return nil
}

//服务端代码
func main(){
    // 注册RPC服务
    rpc.RegisterName("HelloService", new(HelloService))
	
    listener, err := net.Listen("tcp", ":1234")
    if err != nil {
        log.Fatal("ListenTCP error:", err)
    }
    
    conn, err := listener.Accept()
    if err != nil {
        log.Fatal("Accept error:", err)
    }
    
    rpc.ServeConn(conn)
}
```

```go
//客户端代码

func main() {
    client, err := rpc.Dial("tcp", "localhost:1234")
    if err != nil {
        log.Fatal("dialing:", err)
    }
    
    var reply string
    err = client.Call("HelloService.Hello", &reply)
    if err != nil{
        log.Fatal(err)
    }
    
    fmt.Println(reply)
}
```

.....TODO