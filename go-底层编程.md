---
title: go-底层编程
copyright: true
date: 2020-01-27 15:47:57
tags:
	- go
	- 底层编程
	- c
categories:
	- go
description:
---

# Go的底层编程

>使用CGO拥抱c，，，...(*￣０￣)ノ[等等我…]====================................

<!--more-->

>PS：真正需要使用复杂且性能高的底层接口时，才使用CGO
>
>没什么特殊需要，使用 os/exec调用子进程就行了，当然，会产生依赖

## unsafe包

>底层的包，和操作系统紧密相关

{%asset_img gosize.png go的数据类型所占的字节大小%}

>如果有一天需要丧心病狂的优化， 那么这个就有用了

## cgo

来个栗子。。。。(๑•̀ㅂ•́)و✧........................一个简易的压缩程序

```c
//bzip2.c
#include<bzlib.h>

int bz2compress(bz_stream *s, int action, char *in, unsigned *inlen, char *out, unsigned *outlen){
    s->next_in = in;
    s->avail_in = *inlen;
    s->next_out = out;
    s->avail_out = *outlen;
    int r = BZ2_bzCompress(s, action);
    *inlen -= s->avail_in;
    *outlen -= a->avail_out;
    s->next_in = s->next_out = NULL;
    return r;
}
```

然后是go的代码

```go
//下面是给cgo使用的特殊参数， 相应的会传给c编译器
/*
#cgo CFLAGS: -I/usr/include
#cgo LDFLAGS: -L/usr/lib -lbz2
#include <bzlib.h>
#include <stdlib.h>
bz_stream* bz2alloc() {return calloc(1, sizeof(bz_stream));}
int bz2compress(bz_stream *s, int action, char *in, unsigned *inlen, char *out, unsigned *outlen);
void bz2free(bz_stream* s){free(s);}
*/
import "C"

import(
	"os"
    "unsafe"
)

type writer struct{
    w io.Writer
    stream *C.bz_stream
    outbuf [64*1024]byte
}

func NewWriter(out io.Writer) io.WriteCloser {
    const blockSize = 9
    const verbosity = 0
    const workFactor = 30
    w := &writer{w: out, stream: C.bz2alloc()}
    C.BZ2_bzCompressInit(w.stream, blockSize, verbosity, workFactor)
    return w
}
```

```go
func (w *writer) Write(data []byte) (int, err) {
    if w.stream == nil {
        panic("closed")
    }
    var total int
    
    for len(data) > 0 {
        inlen, outlen := C.uint(len(data)), C.uint(cap(w.outbuf))
        C.bz2compress(w.stream, C.BZ_RUN, (*C.char)(unsafe.Pointer(&data[0])), &inlen, (*C.char)(unsafe.Pointer(&w.outbuf)), &outlen)
        total += int(inlen)
        data = data[inlen:]
        if _, err := w.w.Write(w.outbuf[:outlen]); err != nil {
            return total, err
        }
    }
    return total, nil
}
```

```go
func (w *writer) Close() error {
    if w.stream == nil {
        panic("closed")
    }
    
    defer func(){
        C.BZ2_bzCompressEnd(w.stream)
        C.bz2free(w.stream)
        w.stream = nil
    }()
    for {
        inlen, outlen := C.uint(0), C.uint(cap(w.outbuf))
        r := C.bz2compress(w.stream, C.BZ_FINISH, nil, &inlen, (*C.char)(unsafe.Pointer(&w.outbuf)), &outlen)
        if _, err := w.w.Write(w.outbuf[:outlen]); err != nil {
            return err
        }
        
        if r == C.BZ_STREAM_END {
            return nil
        }
    }
}
```

>PS：这里只有CGO的一小部分， 还有很多东西
>
>又：不是必要，不需要使用，不要过早优化