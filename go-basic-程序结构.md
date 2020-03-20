---
title: go-basic-程序结构
copyright: true
date: 2019-12-09 15:07:31
tags:
	- go
	- go-basic
	- go-basic-程序结构
categories:
	- go
description:
---

# Gopher，开始程序结构啦，go，go，go============<(￣︶￣)>==========

<!--more-->

## go内置的关键字

{%asset_img go-keyword1.png go关键字%}

{%asset_img go-keyword2.png go关键字%}

## 声明变量

>var 变量名字 类型 = 表达式
>
>声明const不需要变量类型，数据变量可以简短声明，形式如下
>
>变量名字 := 表达式
>
>简短声明的变量名必须有一个是没有声明过

```go
//boiling
package main

import (
	"fmt")

//命名常量
//使用const
//在外面声明的是全局变量
const boilingF = 212.0

func main() {
	//局部变量, 局部变量必须使用
	var f = boilingF
    //简短声明，c必须是没有声明过的
    c := (f - 32) * 5 / 9
	fmt.Printf("boiling point = %gF or %gC\n", f, c)
}


```

## 来个小栗子=========o(*^＠^*)o=======================

```go
//echo第四版本
package main

import(
    //flag，好用的命令行参数构建包，=======(≧∇≦)ﾉ=========
	"flag"
	"strings"
	"fmt"

)
var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "separator")
func main() {
	flag.Parse()
	fmt.Print(strings.Join(flag.Args(),*sep))
	if !*n {
		fmt.Println()	
	}
}
```

## go的变量的生命周期

* 全局变量：在程序运行期间有效
* 局部变量：从创建开始，直到该变量不再被应用，并不是函数返回就没了，而是指针不再可达就不再需要此变量

## type重定义类型

>type 类型名字 底层类型
>
>类型转换只有底层类型相同才可以互相转换

```go
package tempconv

type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15 //绝对零度
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func CToF(c Celsius) Fahrenheit {
	return Fahrenheit(c*9/5 + 32)
}

func FToC(f Fahrenheit) Celsius {
	return Celsius((f - 32) * 5 / 9)
}

```

## go的包和文件

>包内的包级别变量可以使用init()初始化，这个init不能被调用或引用

### 有一个栗子============￣ω￣===================================

```go
package popcount

var pc [256]byte
//也可以使用匿名函数初始化
/*
var pc [256]byte=func(pc [256]byte){
	for i := range pc{
		pc[i] = pc[i/2] + byte(i&1)
	}
}()
*/

func init() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
}

//PopCount 返回x的总数
func PopCount(x uint64) int {
	return int(pc[byte(x>>(0*8))] +
		pc[byte(x>>(1*8))] +
		pc[byte(x>>(2*8))] +
		pc[byte(x>>(3*8))] +
		pc[byte(x>>(4*8))] +
		pc[byte(x>>(5*8))] +
		pc[byte(x>>(6*8))] +
		pc[byte(x>>(7*8))])
}


```

## go作用域

>和c类似

程序结构完结了。。================～(￣▽￣～)(～￣▽￣)～=============