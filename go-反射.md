---
title: go-反射
copyright: true
date: 2020-01-26 12:33:14
tags:
	- go
	- 反射
	- reflect
categories:
	- go
description:
---

# Go 反射

<!--more-->

>反射：在运行时更新变量和检查它们的值，调用它们的方法和支持的内在操作，但在编译时不知道这些变量的具体类型。

## reflect.Type 和 reflect.Value

>这应该是go的反射最重要的东西了。。。。w(ﾟДﾟ)w..........

### reflect.Type, reflect.TypeOf()

>reflect.TypeOf 接受interface{},并返回reflect.Type

```go
t := reflect.TypeOf(3) // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t) // "int"
```

>PS：reflect.TypeOf 返回的是接口值，而不是接口类型，可以参考 [https://yaya.life/2019/12/12/go-basic-%E6%8E%A5%E5%8F%A3/](https://yaya.life/2019/12/12/go-basic-接口/) 

### reflect.Value , reflect.ValueOf()

>reflect.ValueOf() 接受interface{}, 返回 reflect.Value, ValueOf和TypeOf类似，但Value可以持有接口值

```go
v := reflect.ValueOf(3)
fmt.Println(v) // "3"
fmt.Printf("%v\n", v) // "3"
fmt.Println(v.String()) // NOTE: "<int Value>"

//v.Type() 返回类型
fmt.Println(v.Type().String()) // "int"
```

>PS：reflect.Value 和interface{}都可以保存任意值，但空接口隐藏了值对应的表示方式和公开的方法

来个栗子。。。(￣ε(#￣)☆╰╮o(￣皿￣///)

```go
//Any formats any value as a string
func Any(value interface{}) string {
	return formatAtom(reflect.ValueOf(value))
}

func formatAtom(v reflect.Value) string {
	switch v.Kind() {
	case reflect.Invalid:
		return "invalid"
	case reflect.Int, reflect.Int16, reflect.Int32, reflect.Int64:
		return strconv.FormatInt(v.Int(), 10)
	case reflect.Uint, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
		return strconv.FormatUint(v.Uint(), 10)
    // ...还有其它的， 不写了
	default:
		return v.Type().String() + "value"
	}
}

```

## 通过reflect.ValueOf()修改值

>使用reflect.Value的CanAddr查看是否可以取地址，但是CanAddr并不代表可以修改， CanSet才代表可以修改

```go
x := 2
a := reflect.ValueOf(2) 
b := reflect.ValueOf(x)
c := reflect.ValueOf(&x)
d := c.Elem()

a.CanAddr() // false
b.CanAddr() //false
c.CanAddr() //false
d.CanAddr() //true
fmt.Println(d.CanAddr(), d.CanSet()) // "true" "true"
```

>PS：是否可以取地址的规则类似，通过指针间接地获取的reflect.Value都是可以取值的， 例如 slice的表达式 e[i] 隐式地包含一个指针， 是可取值的， 即使e不支持取值也没关系

>改变值，先使用Addr获取一个Value返回指向变量的指针，在调用Interface()， 返回一个interface{}, 之后就可以使用断言来更新变量

```go
x := 2
d := reflect.ValueOf(&x).Elem()
px := d.Addr().Interface().(*int)  // px := &x
*px = 3
fmt.Println(x) // "3"
```

也可以。。

```go
d.Set(reflect.ValueOf(4))
fmt.Println(x) // "4"
```

>Set方法会检查是否可赋值， 上面都是int，所以可以， 如果值不一致，就会panic

>也有很多用于基本数据类型的方法
>
>SetInt， SetUint， SetString， SetFloat等等， ，， 

## 获取结构体的Tag

```go
v := reflect.ValueOf(ptr).Elem() // 获取struct p的value
for i := 0;i<v.NumField();i++{
    fieldInfo := v.Type().Field(i) //reflect.StructField
    tag := fieldInfo.Tag	// reflect.StructTag
    name := tag.Get("tagName")
}
```

## 获取结构体的方法

```go
func printMethod(x interface{}){
    v := reflect.ValueOf(x)
    t := v.Type()
    fmt.Printf("type %s\n", t)
    
    for i := 0;i < v.NumMethod(); i++ {
        methodType := v.Method(i).Type()
        fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name, strings.TrimPrefix(methodType.String(), "func")
    }
}
```

```go
//调用这个方法
v.Method(i).Call(...in) //调用Method的Call
```

## 最后。。。

>反射是脆弱的，难用的，且运行速度比普通代码慢1，2个量级， 且相较下没有那么安全。能不用就不用吧

