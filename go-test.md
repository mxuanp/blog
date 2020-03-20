---
title: go-test
copyright: true
date: 2020-01-24 15:31:01
tags:
	- go
	- test
categories:
	- go
description:

---

# Go 测试

>美好生活从测试开始==============(๑•̀ㅂ•́)و✧========================

<!--more-->

>go的测试文件以*_test.go结尾，以go test运行， 不会参与go build的构建
>
>测试函数名以Test开头， 例如
>
```go
func TestWriteNote(n note){}
```

>go的测试函数分为3类， 测试函数，基准测试函数， 示例函数

## 测试函数

>测试函数一定有一下签名

```go
func TestName(t *testing.T){/*...*/}
```

栗子。。。

```go
//word.go
package word

func IsPalindrome(s string) bool {
    for i := range s{
        if s[i] != s[len(s) - 1 - i]{
            return false
        }
    }
    return true
}

//word_test.go
import "testing"
func TestIsPalindrome(t *testing.T){
    if !IsPalindrome("aksjdncas"){
        t.Error(`IsPalindrome("aksjdncas) = false"`)
    }
}

```

>go test =======直接这样会出错，go version 1.13.5
>
>go test word_test.go word.go -v 这样就好了，或者把test文件放到另一个文件夹，比如test
>
>这是go的一些问题

## 测试覆盖率

>go test -run=Coverage -coverprofile=c.out projectName
>
>-coverprofile 这个标志是在代码中插入钩子来统计覆盖率（就是bool）， 
>
>如果使用-covermode=count 那么就不是bool， 而是计数，用来衡量哪些代码频繁执行

## 基准测试

>测量程序固定负载下的性能

```go
//必须以Benchmark为前缀
func BenchmarkFunc(b *testing.B){
    ....
}
```

>go test -bench=.
>
>使用-bench标志，如果没有就是普通的go test

## 剖析，优化，（这里就比较高深了，很多时候也用不到。。。。但是真的需要优化程序的时候就很有用了。(～￣(OO)￣)ブ）

>go test -cpuprofile=cpu.out  -cpuprofile= 分析函数执行所需要的cpu时间
>
>go test -blockprofile=block.out  -blockprofile 记录goroutine阻塞情况
>
>go test -memprofile=mem.out   -memprofile= 记录内存使用情况



