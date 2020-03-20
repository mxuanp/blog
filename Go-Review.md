---
title: Go-Review
copyright: true
date: 2020-01-28 15:36:52
tags:
	- go
	- review
categories:
	- go
description:
---

# Gopher, 查漏补缺， 进阶学习啦。。。。。

>Go的基础过了， 复习，以及补缺， 进阶

<!--more-->

## Go程序的入口

>首先， Go程序的初始化和执行总是从main.main开始。
>
>但是， 如果有import， 则会先import包， 按顺序导入， 一个包只会导入一次 ，如果包里有init(), 则会执行init(), 如果一个文件有多个init(), 则会按顺序执行
>
>PS：init和普通函数不同， 可以有多个

{%asset_img pro_order.png Go程序的执行顺序%}

>PS：要注意的是， 在main.main执行之前， 所有代码都在一个Goroutine， 即程序主系统线程里， 所以如果某个init()里使用 go 开启了goroutine， 新的goroutine也只能在main.main之后才能执行到

## Go并不强调堆，栈

>Go的栈是动态栈，会自行调整大小， 所以普通程序员不再需要关心栈的大小问题， 在Go的语言规范中甚至没有讲到栈，堆的概念

>在Go1.4后，Go的动态栈是连续动态栈，类似go的slice， 好处不提， 这里有一个问题， 动态栈增长后， 需要移到新的内存空间， 这就使得栈中所有变量的内存地址改变， 也就是说，Go的指针不再是固定不变（在使用CGO的时候，c代码中不可以长期持有Go的变量的指针， 因为指针指向的地址可能已经改变）

