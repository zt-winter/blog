title: 反射
tags: 
	- reflect
date: 2022/06/13
categories: go
---

# reflect
反射是指在程序运行时对程序本身的进行访问和修改的能力。

## type和value
在golang中对所有的接口反射，都可以得到type和value。而golang中任意数据结构都实现了空接口,也就是说任意数据结构都有reflectt.type和reflect.value。
reflect.type表示一个数据结构的类型，这个类型的各种基本信息。reflect.value则表示了运行过程中，该数据结构类型的一个具体实现的值。

```
type test struct{
	a int
	b string
	c map[int]int
	d *test
}
a := new(test)
aT := reflect.Typeof(a)
aV := reflect.Valueof(a)
```
```
*main.test
0x47e1c0
```

[type接口的函数](https://pkg.go.dev/reflect@go1.18.3#Type)

[value结构体定义](https://cs.opensource.google/go/go/+/refs/tags/go1.18.3:src/reflect/value.go;l=39)

type和value的相关函数都在前面两处文档中有详细说明，这里不在赘述。

