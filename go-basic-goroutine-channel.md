---
title: go-basic-goroutine-channel
copyright: true
tags:
  - go
  - go-basic
  - goroutine
  - go-channel
categories:
  - go
date: 2019-12-17 14:03:00
description:
---

# Goroutines 和 Channel

>这里只是记录go的goroutine和channel的基础用法,多线程有风险，用之需谨慎。。。。ヾ(＠⌒ー⌒＠)ノ。。。==========

<!--more-->

# Goroutine

## 使用方法，，，，很简单的，，，，(＠￣ー￣＠)

```go
func test(){
	......
}
//用关键字go 后跟一个函数或方法，这个函数就会在新的goroutine中运行
go test()
```

来个小栗子

```go
//弄个动画小图标
package main

import (
	"fmt"
	"time"
)

func main() {
	go spinner(100 * time.Millisecond)
	const n = 45
	fibN := fib(n)
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x-1) + fib(x-2)
}

```

>PS : main函数结束后，所有goroutine都会被直接打断，程序退出。除了主函数退出，或程序直接被终止，以及goroutine自行停止，没有其它编程方法能停止一个goroutine

## 例子：并发的Clock

```go
//clock
package main

import(
	"net"
	"time"
	"io"
	"log"

)

func main() {
	listener, err := net.Listen("tcp", "localhost:8000")	
	if err != nil {
		log.Fatal(err)	
	}
	for{
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)	
			continue
		}
		handleConn(conn)
	}
}

func handleConn(conn net.Conn)  {
	defer conn.Close()
	for{
		_, err := io.WriteString(conn, time.Now().Format("15:04:05\n"))
		if err != nil {
			return	
		}
		time.Sleep(1 * time.Second)
	}	
}
```

>可以看出，这个服务器一次只能接受一个连接，，
>
>可以用nc之类的工具连接看结果

```go
//并发的clock
for{
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)	
			continue
		}
    	//只要这里加个go就行了
		go handleConn(conn)
	}

```

## 例子：并发的Echo

```go
//直接来个简单的echo
func handleConn(c net.Conn){
    io.Copy(c, c)
    c.Close()
}
```

```go
//来个复杂点的
//回响echo，，大写到小写。。。
func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		echo(c, input.Text(), 1*time.Second)
	}
}

func echo(c net.Conn, shout string, delay time.Duration) {
	fmt.Fprintln(c, "\t", strings.ToUpper(shout))
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", shout)
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

```

# Channel

>Channel实在goroutine之间的通信机制，她可以一个goroutine通过她向另一个goroutine发送消息

```go
ch := make(chan int)
//这样，一个可以发送int的channel就创建好了

//发送消息到channel
ch <- x

//从channel接收消息
x := <-ch
//关闭channel
close(ch)
```

### 不带缓存的Channel

>ch := make(chan int) 这样就声明初始化了一个不带缓存的Channel，不带缓存的Channel在执行发送操作后会堵塞发送者的goroutine， 直到发送的值被接收，相反，如果有goroutine在执行接收操作，也会堵塞，直到发送方向Channel发送数据

## 例子

>用channel来同步goroutine的工作，让主goroutine等待后台goroutine完成工作

```go
//netcat第一版，就是网络版cat
package main

import "net"

import "log"

import "os"

import "io"

func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	done := make(chan struct{})
	go func(){
		io.Copy(os.Stdout, conn)
		log.Println("done")
		done <- struct{}{}
	}()
	mustCopy(conn, os.Stdin)
	conn.Close()
	<- done
}
func mustCopy(dst io.Writer, src io.Reader) {
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

## 串联的Channel（Pipeline）。。。就是管道。。。━━(￣ー￣*|||━━

>用Channel将多个goroutine连接起来

{%asset_img go-pipe.png 管道%}

```go
//管道
package main

import (
	"fmt"
)
//串串
func main() {
	naturals := make(chan int)
	squares := make(chan int)

	go func() {
		for x := 0; x<100 ; x++ {
			naturals <- x
		}
        
	}()

	go func() {
		for {
			x, ok := <-naturals
			if !ok {
				break	
			}
			squares <- x * x
		}
		close(squares)
	}()

	for {
		fmt.Println(<-squares)
	}
}

```

>Channel在没有被引用后会被Go的gc回收，所以可以不用手动关闭
>
>和文件不同，文件不用后一定要手动调用对应的Close方法，否则后造成内存泄漏，资源的耗费

## 单方向Channel

>即只能接收，或只能发送的Channel
>
>双方向的Channel可以转换为单方向的Channel，但不可以将单方向的Channel转换为双方向的或转换为另一个方向

```go
ch := make(in <-chan int)//这样声明一个只能接收int的Channel

ch := make(out chan<- int)//这样声明一个只能发送int的Channel
```

举个栗子。。。。。。o(*^▽^*)┛。。。。。。。。。。。。。。=======

```go
//管道
package main

import (
	"fmt"
)

func main() {
	naturals := make(chan int)
	squares := make(chan int)
    //包含了隐式转换
	go counter(naturals)
	go squarer(squares, naturals)
	printer(squares)
}
func counter(out chan<- int) {
	for x := 0; x <= 100; x++ {
		out <- x
	}
	close(out)
}

func squarer(out chan<- int, in <-chan int) {
	for v := range in {
		out <- v * v
	}
	close(out)
}
func printer(in <-chan int) {
	for v := range in {
		fmt.Println(v)
	}
}

```

## 带缓存的Channel

>带缓存的Channel内部持有一个元素的队列，队列的最大容量在make时指定

```go
ch := make(chan string, 3)//这样就声明初始化了一个带缓存的Channel，缓存容量为3
```

>向有缓存的Channel发送数据，直到Channel的缓存占满为止，发送操作不会阻塞goroutine
>
>可以使用内置函数cap查看Channel的容量，内置函数len查看Channel的缓存区的数据量

```go
fmt.Println(cap(ch))//查看容量

fmt.Println(len(ch))//查看数据量
```

## 看一个栗子：。。。。(。・∀・)ノ。。。。。========

>这个栗子请求反应最快的镜像，并打印
>
>如果使用无缓存的Channel，那么将有两个groutines没有接收而被卡住，这种情况称为goroutines泄漏，BUG。和有垃圾变量不一样，泄漏的groutine是无法被gc回收的，因此确保每个不再需要的goroutine能正常退出是很重要的 

```go
func mirroredQuery() string{
    resp := make(chan string, 3)
    go func(){resp <- request("asia.gopl.io")}()
    go func(){resp <- request("europe.gopl.io")}()
    go func(){resp <- request("americas.gopl.io")}()
}
func request(){
    .........
}
```

>关于goroutines和Channel,可以想象一下工厂生产线

## 来弄个爬虫。。。。====━━(￣ー￣*|||━━。。。。。。。。。。。。

```go
package main

import (
	"fmt"
	"log"
	"os"

	"gopl.io/ch5/links"
)

func main() {
	worklist := make(chan []string)
	go func(){worklist <- os.Args[1:]}()
	var n int
	n++
	seen := make(map[string]bool)
	for ;n>0;n--{
		list := <- worklist
		for _,link := range list{
			if(!seen[link]){
				seen[link]=true
				n++
				go func(link string){
					worklist <- crawl(link)
				}(link)
			}
		}
	}
}
var tokens = make(chan struct{}, 20)
func crawl(url string) []string {
	fmt.Println(url)
	tokens <- struct{}{}
	list, err := links.Extract(url)
	<-tokens
	if err != nil {
		log.Print(err)
	}
	return list
}

```

## select

>go的select和switch类似，但是每个case必须是通信操作，发送或接收
>
>PS：在select中是nil的channel永远不会被select到，因而可以使用nil激活或禁用case

```go
select{
    case <-ch1:
    //.........
    case x:=<-ch2:
    //.........
    case ch3<-y:
    //.........
    default:
    //.........
}
```



## 轮询Channel

>有时候需要从Channel接收值或发送，但又不能堵塞，特别是Channel没有准备好读写时，select就可以实现这样的功能，select的default可以用来设置当其它的操作都不能马上被处理时，执行的逻辑

```go
select{
    case <-abort:
    fmt.Println("Launch abort")
    return
    default:
    	........
}
```

## Goroutines和线程

### 动态栈

>每个OS线程都有一个固定的内存块(一般是2MB)作为栈， 但是2MB对一个Goroutine实在有些大，且固定大小的栈不方便修改。Goroutine以一个小的栈开始，一般只需要2KB， 且会根据需要动态地伸缩，最大值为1GB，当然，大多数时候不需要这么的栈

### Goroutine调度

>Go有自己的运行时调度，例如n：m调度，这种调度，不是系统级的，因而Goroutine的调度代价比调度线程小的多

#### GOMAXPROCS

>Go的调度器使用GOMAXPROCS的环境变量来决定会有多少个操作系统的线程来调度
>
>所以可以使用GOMACPROCS这个环境变量来显示地控制

## Goroutine没有ID号

>大多数支持多线程的操作系统和语言中，当前线程会有一个独特的身份（id），但是Groutine是没有的，且是设计上故意没有的，怕thread-locale storage 被滥用

