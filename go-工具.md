---
title: go-工具
copyright: true
tags: 
	- go
	- go-tool
	- vim
categories: 
	- go
date: 2020-03-15 20:02:00
description:
---

# Go的工具

<!--more-->

>go本身带的工具，如果在goland这样的ide下，没有必要，但是有些时候只是需要vim之类的写个小项目，不想开ide，那么go自带的工具就有用了

## Go vet

>go vet会捕获一些语法错误

* Printf类函数调用时，类型匹配占位符错误
* 定义常用的方法时，方法签名的错误
* 错误的结构体标签
* 没有指定字段名的结构体字面量

## Go fmt

>将源代码格式化成Go原码类似的风格

## Go doc

>查看包的信息

```bash
go doc fmt # 这样就可以查看fmt包的信息
```

## godoc 

>go doc 的浏览器版本

```bash
godoc -http=:8080 #本地端口8080的服务器
```

>对于自己的包只要遵守规范，也可以生成到godoc，具体查看golang documention

more ...........

