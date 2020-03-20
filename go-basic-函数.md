---
title: go-basic-函数
copyright: true
date: 2019-12-10 14:47:07
tags:
	- go
	- go-basic
	- go-basic-函数
categories:
	- go
description:
---



# 到函数了呢===========(～o￣▽￣)～o 。。。滚来滚去……o～(＿△＿o===============

<!--more-->

## 函数声明

>func name(parameter-list) (result-list){
>
>​	body..................
>
>}

>go的函数可以是一个类型,但不可以作为map的key，也不可以相互比较
>
```go
var add = func(a, b int) int {
return a+b
}

//调用
add(1,3)
```
>
>函数名称首字母大写就可以让其它包使用，和变量一样呢。。。。。ヾ(´∀`o)+

>go函数的递归所需的栈可以根据需要增加（初始时很小），所以不必考虑溢出和安全问题，（把机子内存撑爆另当别论===ㄟ( ▔, ▔ )ㄏ======）

## 返回值

>go的函数和c类似，但是可以多个返回值，型参是值拷贝，要改值需要传引用或指针
>
>go的返回值有个神奇的地方

```go
func add(a, b int) (c int){
    c = a+b
    return  //等同于 return c  ===>即在result-list指定返回值名称后，可以直接使用，在有很多返回值的时候很有用
}
```

## 匿名函数

>有点像java类的内部类，在某个函数中定义的匿名函数可以直接访问它的内部变量
>
>从结果可以看出，匿名函数保存着外部函数变量的引用，这就是函数值属于引用和不可以比较的原因
>
>Go实现函数值的方式是闭包

```go
//squares 
package main

import(
	"fmt"

)

func main() {
	f := squares()
	fmt.Println(f()) //1
	fmt.Println(f()) //4
	fmt.Println(f()) //9
}

func squares() func() int  {
	var x int
	return func() int{
		x++
		return x*x
	}	
}
```

### 匿名函数里词法作用域的陷阱====＜（＾－＾）＞============

>由于匿名函数会持有外部函数变量的引用（不是值），所以循环时如果直接使用d，那么最后rmdirs里所有匿名函数持有的都会是最后一次循环时d的值

```go
var rmdirs []func
for _,d := range tempDirs(){
    dir := d //这里必须有
    os.MkdirAll(dir, 0755)
    rmdirs = append(rmdirs, func(){
        os.RemoveAll(dir)
    })
}
```

## 可变参数

>可变参数使用...声明，将会被看作一个slice

```go
func add(vals...int)int{
    total := 0
    for _,val := range vals{
        total += val
    }
    return total
}
```

## Deferred函数

>defer声明的函数会在包含该defer语句的函数执行完后执行
>
>defer声明的函数一定会被执行，即使运行中出现panic导致程序异常，在程序退出前也会执行（除非，电脑突然没电这样的。。==ㄟ( ▔, ▔ )ㄏ======）
>
>PS：有那么一点点像java类的finallize=====(～￣(OO)￣)ブ

```go

func title(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err	
	}
	defer resp.Body.Close() //这里最后一定会被执行
	ct := resp.Header.Get("Content-Type")
	if ct != "text/html" && !strings.HasPrefix(ct,"text/html;") {
		return fmt.Errorf("%s has type %s, not text/html", url, ct)	
	}
	doc, err := html.Parse(resp.Body)
	if err != nil{
		return err
	}
	visitNode := func(n *html.Node){
		if n.Type == html.ElementNode && n.Data == "title" && n.FirstChild !=nil{
			fmt.Println(n.FirstChild.Data)
		}
	}
	forEachNode(doc, visitNode, nil)
	return nil
}


```



## Error和Panic

>这里不详细说
>
>Error，是一个interface，只有Error() string一个接口，用于返回错误信息
>
>Panic,异常，有可能会导致程序无法继续运行，可以recover处理，使程序更加健壮

>go对Error的处理风格是先处理一系列Error，然后再是正常的逻辑