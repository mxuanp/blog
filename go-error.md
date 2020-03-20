---
title: go-error
copyright: true
date: 2019-12-13 14:11:10
tags:
	- go
	- go-error
categories:
	- go
description:

---

# Go的Error

<!--more-->

>直接来看下go error的定义

```go
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}
```

>go的error就是一个interface,只有一个需要实现的方法Error(),用于返回一个string,一般用于返回错误信息

## 创建一个error最简单的方法=======￣ω￣===============￣ω￣=======

>使用errors.New()方法

```go
// Copyright 2011 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}

```

>现在go的error就是这么简单
>
>errors包的Error()方法使用指针接收的原因是每次分配新的地址，以避免可能出现的与已有的错误撞车，，<( ‵□′)───C＜─___-)||。。。。。。。。。

```go
//栗子
fmt.Println(erros.New("EOF") == errors.New("EOF")) // false
```

# Go的panic的一个坑

>recover()必须在defer函数中运行

>recover捕获的是祖父级调用时的异常， 直接调用是无效的， 必须是祖父级， 且只隔着一个栈帧

```go
func main(){
    recover() //没用， 捕获不了下面的panic
    panic(1)
}
```

>直接defer recover也是没用的

```go
func main(){
    defer recover() //没用
    panic(1)
}
```

>defer里的recover如果嵌套了， 也是没用的

```go
func main(){
    defer func(){
        func(){
            recover() // 嵌套了， 捕获不了main的panic
        }()
    }()
    panic(1)
}
```

>必须直接在defer中recover

```go
func main() {
    defer func(){
        recover() //这样就有效了
    }()
    panic(1)
}
```

