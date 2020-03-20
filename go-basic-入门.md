---
title: go-basic-入门
copyright: true
date: 2019-12-09 09:18:42
tags:
	- go
	- go-basic
	- go-baisc-入门
categories:
    - go
description:
---

# Gopher第一天

<!--more-->

## hello world

```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello world\n")
}
```

==================萌新学习中=====================================

==================(>▽<)=====================(>▽<)================

## 命令行参数

> 可以使用os.Args获取命令行参数，os.Args是一个string的切片

===================萌新的分界线======┗|｀O′|┛ 嗷~~=================

举个栗子：

```go
//echo第一版本
package main

import "os"

import "fmt"

func main() {
	var s,sep string
    //go的for循环
    //for initialization;condition;post{
    //括号一定要这样。。。。。
    //}
	for i:=1;i<len(os.Args);i++{
		s += sep + os.Args[i]
		sep = " "
	}
	fmt.Println(s)
}
```

```go
//echo第二版本
package main

import "os"

import "fmt"

func main() {
	s,sep := "", ""
    //for的另一种形式，range， 在区间访问，range每次返回index，和值
    //go定义的变量必须使用，如果不适用可以使用_代替
	for _, arg := range os.Args[1:]{
		s += sep + arg
		sep = " "
	}
	fmt.Println(s)
}
```

```go
//echo第三版本，高效，简洁
package main

import "fmt"

import "strings"

import "os"

func main() {
	fmt.Println(strings.Join(os.Args[1:], " "))	
}
```

做个小练习===========( *￣▽￣)((≧︶≦*)===============================

```go
//dup1, 打印重复行
package main

import(
	"bufio"
	"fmt"
	"os"

)

func main() {
	counts := make(map[string]int)	
	input := bufio.NewScanner(os.Stdin)
	for input.Scan(){
        if input.Text() == "end" {
			break	
		}
		//没有处理错误
		counts[input.Text()]++
	}

	for line, n := range counts{
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)	
		}
	}
}
```

## 一些常用的格式化参数

{% asset_img go-format.png fmt.Printf 参数%}



## 小栗子===ヾ(≧O≦)〃嗷~===

```go
//fetch 第一版 读取html网页的内容
package main

import (
	"os"
	"io/ioutil"
	"fmt"
	"net/http"
)

func main() {
	for _, url := range os.Args[1:] {
		resp, err := http.Get(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)	
			os.Exit(1)
		}
		b, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)	
		}
		fmt.Printf("%s", b)
	}
}
```

## 试试go的goroutine，channel===go， go， go，，，，┗|｀O′|┛ 嗷~~

```go
//fetch all 第一版
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"os"
	"time"
)

func main() {
	start := time.Now()
	ch := make(chan string)
	for _, url := range os.Args[1:] {
		go fetch(url, ch)
	}
	for range os.Args[1:]{
		fmt.Println(<-ch) //接受channel的信息
	}
	fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
	start := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		ch <- fmt.Sprint(err) //发送信息到channel
		return
	}
	nbytes, err := io.Copy(ioutil.Discard, resp.Body)
	resp.Body.Close()
	if err != nil {
		ch <- fmt.Sprintf("while reading %s: %v\n", url, err)
		return
	}
	secs := time.Since(start).Seconds()
	ch <- fmt.Sprintf("%.2fs %7d %s", secs, nbytes, url)
}

```

## 一个好玩的小服务器======(๑•̀ㅂ•́)و✧==================

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 21.

// Server3 is an "echo" server that displays request parameters.
package main

import (
	"fmt"
	"strconv"
	"image"
	"image/color"
	"image/gif"
	"io"
	"log"
	"math"
	"math/rand"
	"net/http"
)

func main() {
	http.HandleFunc("/", handler)
	http.HandleFunc("/lissajous", func(w http.ResponseWriter, r *http.Request) {
		cycles, _:= strconv.Atoi(r.URL.Query().Get("cycles"))
		lissajous(w,float64(cycles))
	})
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

//!+handler
// handler echoes the HTTP request.
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "%s %s %s\n", r.Method, r.URL, r.Proto)
	for k, v := range r.Header {
		fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
	}
	fmt.Fprintf(w, "Host = %q\n", r.Host)
	fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
	if err := r.ParseForm(); err != nil {
		log.Print(err)
	}
	for k, v := range r.Form {
		fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
	}
}

var palette = []color.Color{color.White, color.RGBA{234, 134, 53, 100}}

const (
	whiteIndex = 0 //palette第一个颜色
	blackIndex = 1 //palette第二个颜色
)

//!-handler
func lissajous(out io.Writer, cycles float64) {
	const (
		res     = 0.001
		size    = 100
		nframes = 64
		delay   = 8
	)
	freq := rand.Float64() * 3.0
	anim := gif.GIF{LoopCount: nframes}
	phase := 0.0
	for i := 0; i < nframes; i++ {
		rect := image.Rect(0, 0, 2*size+1, 2*size+1)
		img := image.NewPaletted(rect, palette)
		for t := 0.0; t < cycles*2*math.Pi; t += res {
			x := math.Sin(t)
			y := math.Sin(t*freq + phase)
			img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5), blackIndex)
		}
		phase += 0.1
		anim.Delay = append(anim.Delay, delay)
		anim.Image = append(anim.Image, img)
	}
	gif.EncodeAll(out, &anim)
}

```



## 第一章总结

>go从c继承了一些东西，但也有很多属于自己的东西