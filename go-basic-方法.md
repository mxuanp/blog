---
title: go-basic-方法
copyright: true
date: 2019-12-12 10:35:40
tags:
	- go
	- go-basic
	- go-basic-方法
categories:
	- go
description:

---

# 到方法了呢===(๑•̀ㅂ•́)و✧==============(๑•̀ㅂ•́)و✧===============

<!--more-->

>方法和函数相比，多个接收器（receiver）

```go
package geometry

import "math"

//Point 是对二维平面的点的抽象
type Point struct{
	X,Y float64
}

//Distance 计算两个点的距离
func Distance(p,q Point) float64 {
	return math.Hypot(q.X-p.X,q.Y-p.Y)
}

//Distance 计算两个点的距离
func (p Point) Distance(q Point) float64{
	return math.Hypot(q.X-p.X,q.Y-p.Y)
}
```

## 如果接收器是指针调用方法

>###### 和普通struct的调用没什么区别，go编译器会自行优化====～(￣▽￣～)(～￣▽￣)～=========

```go
r := &Point{1,2}
b := &Point{2,4}
d := r.Distance(b)
```

## 接收器允许是nil

```go
type List struct{
    Value int
    Tail *List
}
//Sum 返回list的所有value的和
func (list *List) Sum() int{
    if list == nil{
        return 0
    }
    return list.Value + list.Tail.Sum()
}
```

## 可以嵌套结构体来继承方法

>嵌套的结构体的方法没有二义性，就可以直接调用,有的话需要详细到某个成员

```go
type ColorPoint struct{
    Point
    Color color.RGBA
}

p := ColorPoint{Point{1,2},color.RGBA{23,34,45,100}}
d := p.Distance(p)
```

## 方法和函数一样可以是函数类型

```go
DistanceFromP := p.Distance
//调用
DistanceFromP(q)
```

==========いざ、ここまで==========*´∀`)´∀`)*´∀`)*´∀`)======================