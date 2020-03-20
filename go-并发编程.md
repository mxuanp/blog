---
title: go-并发编程
copyright: true
date: 2020-01-23 17:09:50
tags:
	- go
	- 并发编程
categories:
	- go
description:

---

# 并发编程

<!--more-->

## 竞争条件

举个栗子吧=======================(* ￣3)(ε￣ *)======================

```go
type note struct{
    content string
}

type person struct{
    name string
}

func (p person) WriteNote(n note){
    content = p.name
}

var p1, p2 person
var n note //note只有一本

//p1想在note写写自己的名字
go func(){
    p1.WriteNote(n)
}()
//但是此时，p2也想在note上写写自己的名字
go func(){
    p2.WriteNote(n)
}()

//两个goroutine的执行顺序没法保证
//所以p1，p2会争起来，而 n 就是竞争条件
```

## 解决方法1 足够多的note

>如果不是只有一个n，而是两个，三个，甚至更多，那么就没有争的必要了
>
>(￣△￣；) ========== 废话=======================
>
>现实是一般没有那么多的资源

## 解决方法2 绑定变量，顺序传递

>就像工厂的流水线一样，要加工的东西在第一个工人处完成第一步骤，传递到第二个工人处， 如此循环往复，中间不会出现步骤混乱的情况

```go
//就像下面加工cake的栗子============Σ(っ °Д °;)っ===================
type Cake struct{
    state string
}

func baker(cooked chan<- *Cake){
    for{
        cake := new(Cake)
        cake.state = "cooked"
        cooked <- cake //烤蛋糕的人绝不会再碰这个蛋糕
    }
}

func icer(iced chan<- *Cake, cooked chan<- *cake){
    for cake := range cooked{
        cake.state = "Iced"
        iced <- cake //加奶油的也不会再碰这个蛋糕
    }
}

//就像这样，一个cake虽然会在多个goroutine走过，但每次只有一个goroutine会拿到cake
```

## 解决方法3 互斥，加锁

>然而很多情况下没办法像流水线那样一步步的来，就像p1和p2， 谁来决定谁先写呢，这个时候就需要互斥，加锁了

```go
var mu sync.Mutex
//就像这样，谁先抢到谁先写，写的过程中，别人无法进入
func (p person) Write(n note){
    mu.Lock()
    defer mu.Unlock()
    n.content = p.name
}


```

>PS：注意这里mu.Lock()保护的Write方法只能被一个Goroutine访问，而不是保护note n
>
>PS：Go不能重入锁，例如这里Write有mu.Lock()了，不能在里面再次调用mu.Lock(), 调用的方法里也不可以有mu.Lock()

### 读写锁

>有些时候，我们可能需要大量的读取，而不需要写入，或者有时只需要写入，不需要读取，那么就可以用读写锁

```go
var mu sync.RWMutex
//这里多个读取操作不会有什么问题，但是写操作会互斥
func (p person) Read(n note){
    mu.RLock()
    defer mu.RUnlock()
    return n.content
}

```

>RWMutex只有当获得锁的大部分goroutine都是读操作，而锁在竞争条件下， 也就是groutine必须等待才能获取锁的时候，RWMutex才能带来好处。一般情况下，因为RWMutex需要更加复杂的内部记录，因而比一般的无竞争锁的mutex更慢

## sync.Once

>有些时候，我们需要初始化共享变量，但只需要初始化一遍，这时，就可以使用sync.Once

```go
var load sync.Once
var note map[string]string

func GetNote(string k) string {
    load.Do(LoadNote(&note)) // 只会执行一次
    return note[k]
}
```

## 竞争条件检测

>go 的一个工具
>
>在go build， go run， go test命令后加上 -race的flag，编译器就会创建一个程序的修改版，能记录所有运行时期对共享变量访问工具的test
