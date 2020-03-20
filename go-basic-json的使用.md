---
title: go-basic-json的使用
copyright: true
date: 2019-12-10 14:19:15
tags:
	- go
	- go-basic
	- json
categories:
	- go
description:

---

# Go的json包的使用============ヽ(✿ﾟ▽ﾟ)ノ============================

>PS：其它标准协议，如XML，ASN，Protocol Buffers都有类似的API接口

<!--more-->

## 直接来个栗子=================o(*^＠^*)o================================

>Go中将moive类似的结构体slice转换为json的过程叫编组（marshaling）
>
>对应的逆编码操作就是unmarshaling

```go
//json的使用
package main

import(
	"encoding/json"
	"fmt"
	"log"

)
//Movie 是电影的结构体抽象
type Movie struct{
	Title string
	Year int `json:"released"`
	Color bool `json:"color,omitempty"`
	Actors []string
}

func main() {
	var movies = []Movie{
		{Title: "movie1", Year:1942,Color:false,Actors:[]string{"actor1", "actor2"}},
		{Title: "movie2", Year:1943,Color:false,Actors:[]string{"actor1", "actor2"}},
	}	
	data, err := json.Marshal(movies)
	if err != nil {
		log.Fatalf("Json marshaling failed: %s", err)	
	}
	fmt.Printf("%s\n", data)
}
```

{%asset_img go-result.png slice转json%}

>结构会是一串，无法直视
>
>可以使用MarshalIndent,可以输出整齐的缩进

>Year int `json:"released"`  这个是结构体的成员Tag，通常是空格分割的key:"value"形式，用于控制Json包的编码和解码的行为，encoding下其它包也遵循相同的规则