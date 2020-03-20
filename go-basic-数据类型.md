---
title: go-basic-数据类型
copyright: true
date: 2019-12-10 09:35:23
tags:
	- go
	- go-basic
	- go-basic-数据类型
categories:
	- go
description:

---

# 到了数据类型的时间了呢，，，===<(￣ˇ￣)/============================

<!--more-->

## 基础数据类型

* int8,int16,int32,int64分别是8位，16位，32位，64位的有符号整型
* uint8,uint16,uint32,uint64与上面的整型对应，是无符号的整型
* int,uint对应特定CPU平台，可能32位，可能64位  PS:int并不是int32，如果都是32位，需要int为int32时，也需要强制转换
* float32,float64浮点数类型 PS：可以使用科学计数法表示
* bool布尔型
* string字符串  PS：字符串是不可变的字节序列，不是字符序列
* rune 和int32等价，通常用于表示Unicode码点，可以当成是字符
* complex64,complex128复数，对应float32和float64
* 常量，使用const表示

## 复合数据类型

### 数组

>固定长度的特定类型元素序列，长度固定

## slice切片

>不定长数组，用法和数组类型，以后单独谈论slice ===(✿◡‿◡)==========
>
>内置函数len，cap分别返回slice的长度和容量
>
>增加元素使用内置的append函数
>
>slice有好用的切片操作

```go
//arr在这里是一个引用
arr := []int{}
//arr是一个引用，为什么还要append返回，留在以后讨论===（小声：是由于底层数组容量问题，数组容量重新分配。。。(＠￣ー￣＠)）
arr = append(arr,1)
arr = append(arr,2)
//切片
arr2 := arr[1:] //arr2的数据为arr下标1到最后
arr2 = arr[0:2]
...
```



## Map

>哈希表，无序键值对集合，以后单独讨论===(✿◡‿◡)==================
>
>可是使用内建函数make构建map,也可以直接用字面值创建
>
>删除某个key使用内置的delete函数

```go
ages := make(map[string]int)

ages := map[string]int{
    //可以指定一些值
    "alice":13,
    "kiyoko":14,
}
```

## 结构体

>聚合类型
>
>结构体里面可以嵌套结构体 PS：有点像继承
>
>聚合类型不可以包含自身，但可以包含自身的指针。。(略略略，不然怎么实现树，，，（〃｀ 3′〃）)

```go
type Person struct{
    Name string
    Age int
    Gender string
}

type Worker struct{
    Person //嵌套
    job string
}

//使用
var alice Person
//初始化的时候可以直接指定面值
var alice Person = Person{Name:"alice",Age:13,Gender:"女"}
//嵌套的struct初始化面值必须写全
var w Worker = Worker{{"alice",16,"女"},"jk"}
alice.Name = "alice"
//可以使用指针取值
var p *Person = &alice
p.Name = "alice" //相当于 (*p).Name = "alice"
//也可以去内部的成员的指针
p1 := &alice.Name

```

## 弄几个好玩的小栗子======ヾ(≧O≦)〃嗷~======ヾ(≧O≦)〃嗷~=======================

```go
//生成svg图像
package main

import (
	"fmt"
	"math"
)

const (
	width, height = 600, 320
	cells         = 100
	xyrange       = 30.0
	xyscale       = width / 2 / xyrange
	zscale        = height * 0.4
	angle         = math.Pi / 6
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle)

func main() {
	fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' style='stroke: orange; fill: white; stroke-width: 0.7' width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i+1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j+1)
			dx, dy := corner(i+1, j+1)
			fmt.Printf("<polygon points='%g,%g,%g,%g,%g,%g,%g,%g'/>\n", ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Printf("</svg>")
}
func corner(i, j int) (float64, float64) {
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)
	z := f(x, y)
	sx := width/2 + (x-y)*cos30*xyscale
	sy := height/2 + (x+y)*sin30*xyscale - z*zscale
	return sx, sy
}
func f(x, y float64) float64 {
	r := math.Hypot(x, y)
	return math.Sin(r) / r
}

```

```go
//弄个二叉树排序======o(*^＠^*)o=========================================
package main

import (
	"fmt")

type tree struct {
	value       int
	left, right *tree
}

func main() {
	arr := []int{1,3,5,2,5,7}
	sort(arr)
	fmt.Printf("%v\n", arr)
}

func sort(values []int) {
	var root *tree
	for _, v := range values {
		root = add(root, v)
	}
    //将排序后的数据重新写入values
	appendValues(values[:0], root)
}
func appendValues(values []int, t *tree) []int {
	if t != nil {
		values = appendValues(values, t.left)
		values = append(values, t.value)
		values = appendValues(values, t.right)
	}
	return values
}
func add(t *tree, value int) *tree {
	if t == nil {
		t = new(tree)
		t.value = value
		return t
	}
	if value < t.value {
		t.left = add(t.left, value)
	} else {
		t.right = add(t.right, value)
	}
	return t
}

```

## Error类型

```go
type error interface {
	Error() string
}
```



