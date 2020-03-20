---
title: go-basic-接口
copyright: true
date: 2019-12-12 14:43:47
tags:
	- go
	- go-basic
	- go-basic-接口
categories:
	- go
description:

---

# 终于到接口啦，，啦啦啦啦了========(￣▽￣)～■干杯□～(￣▽￣)====================

<!--more-->

>所谓接口，就是合约======ο(=•ω＜=)ρ⌒☆=======================

```go
//来个栗子
package fmt
//只要实现Wirter接口，都可以调用Fprintf()
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf) //p.buf这个buf就是[]byte
	p.free()
	return
}
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}

// Sprintf formats according to a format specifier and returns the resulting string.
func Sprintf(format string, a ...interface{}) string {
	..........
	return s
}

package io

type Wirter interface{
    Write(p []byte)(n int,err error)
}
```

## 接口类型就是一系列方法的集合

```go
package io
type Reader interface{
    Read(p []byte) (n int, err error)
}
type Closer interface{
    Close() error
}

```

## 实现接口的条件

>一个类型如果实现了一个接口所有的方法，就实现了这个接口

>这里Intset就实现了Stringer接口

```go
//Intset int的集合
type Intset struct {
	words []uint64
}
//fmt.Stringer
type Stringer interface {
	String() string
}
//String 返回intset的string
func (s *Intset) String() string {
	var buf bytes.Buffer
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			if word&(1<<uint(j)) != 0 {
				if buf.Len() > len("}") {
					buf.WriteByte('}')
				}
				fmt.Fprintf(&buf, "%d", 64*i+j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

```

## 空接口 interface{}

>空接口因为没有任何要求，所以可以接受任意值

## 接口的初始化

>一个接口只声明没有值初始化，那么它的type和value都是nil

```go
var w io.Writer
```

{% asset_img go-interface-nil.png go接口初始化 %}

>初始化后，type和value将会变化

```go
w = os.Stdout
```

{% asset_img go-interface-non.png go接口初始化后的type和值 %}

>改变w的指向，w的type也会跟着改变

```go
w = new(bytes.Buffer)
```

{% asset_img go-interface-change.png go接口值变化 %}

## 空接口和包含nil的非空接口====坑====o(≧口≦)o======================

```go
const debug = true

func main(){
    var buf *bytes.Buffer
    if debug{
        buf = new(bytes.Buffer)
    }
    f(buf)
    if debug{
        .....
    }
}
func f(out io.Writer){
    //do something .....
    if out != nil{
        out.Write([]byte("hello"))
    }
}
```

>这个栗子里，如果吧debug置为false，将发生panic
>
>为什么，看上面接口初始化，这里把nil的buf传进来后，out被初始化成type为io.Writer,而value为nil
>
>所以if out！= nil为真，但调用out.Write是却是调用nil的value的方法

{% asset_img go-interface-func.png go函数参数为接口 %}

```go
//do something .....
if out != nil{
    out.Write([]byte("hello")) //panic:nil pointer
}
```

## 类型断言

>简单来说就是检查某个操作对象是否和断言的类型匹配，
>
>1. 如果T是具体类型，则检查x的动态类型是否和T相同，成功返回x的动态值，类型为T
>2. 如果T是接口，检查x的动态类型是否满足T，成功，不会获取到动态值，而是知晓了x满足T接口，它仍是x类型，相当于x的可获取的方法集合扩大了，有了接口T的方法
>
>格式：x.(T), x是接口，T是类型

```go
var w io.Writer
w = os.Stdout
f := w.(*os.File) //success, os.Stdout是file类型===ε = = (づ′▽`)づ
c := w.(*bytes.Buffer) //panic, 
```

```go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // success, *io.File 实现了ReadWriter接口,里面其实就两个方法==(～o￣3￣)～。。。。。
w = new(ByteCounter)
rw = w.(io.ReadWriter) //panic, *ByteCounter没有实现ReadWriter接口
```

### 类型断言的一个用法

```go
//来个栗子=====n(*≧▽≦*)n====================
//io包下的Writer接口有一个Write方法，需要一个byte切片，如果直接将stirng
//转换为byte切片再写入，会生成一个临时byte切片，在数据写入后这个临时的
//byte切片就没用了，会造成内存重新分配，使得性能降低，但是大多数对于实现了
//Writer接口的类型，还实现了WriteString方法，这个方法会避免分配临时的内
//存拷贝，所以这里使用类型断言判断w是不是更加具体的StringWriter接口
type StringWriter interface {
	WriteString(s string) (n int, err error)
}

func WriteString(w Writer, s string) (n int, err error) {
	if sw, ok := w.(StringWriter); ok {
		return sw.WriteString(s)
	}
	return w.Write([]byte(s))
}

```



## 关于接口======...=====（*゜ー゜*）====......==========

>不要滥用接口，当有两个甚至更多类型需要实现同一个接口的时候才使用接口。 
>
>ask only for what you need.............━━(￣ー￣*|||━━================