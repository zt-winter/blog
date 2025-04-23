title: golang中的零值、nil与空值
tags:
    - golang
    - nil
    - zero value
date: 2023/4/10
categories: go
---

# 零值
零值是指声明变量（分配内存时）并未显示初始化时，编译器为你的变量自动设置一个默认初始值。[零值官方说明](https://go.dev/ref/spec#The_zero_value)
零值默认初始值可以分为两大类：
* 值类型：int默认值为0，bool默认值为false，float64的默认值为0.000000，byte默认值为0，string默认值是“”,对于数组和结构体，会递归初始化其元素或者字段，取决于其类型。
* 引用类型：均为nil，包括指针pointer，函数func，接口interface，切片slice，管道channel，哈希表map。
注意`testNum := 0`已经完成了初始化，不属于零值。

# nil
最初接触nil，是错误处理时，通过返回的error是否为nil判断是否出现错误。nil是go中预先声明的标识符，主要用来表示引用类型的零值，表示它们未被初始化。
```
\\nil经典问题
var p *int
var i interface{}
var p1 *[3]int

fmt.Println(p1 == p) \\编译不通过，因为类型不一致
fmt.Println(i == p) \\返回结果为True
```
上述结果的原因在于，interface是没有类型，其他变量声明时已经指定了类型。一般不同类型的数据不能直接比较。有些比较是go内部进行了类型转换，然后比较。而interface可以接受任意类型，因此在interface比较时会比较类型与值两个部分。p与i值都是nil，但是类型不一致。
在判断`p1==p`时，编辑器会直接提示类型不一致，不能比较。


#空结构
空结构是没有任何字段的结构类型
```
type sample struct{}
```
一个结构体示例的大小由其字段的宽度与对齐标准决定。空结构占用零字节，也不用填充对齐。空结构数组或者切片也不占用空间。常见的两个用途：
* chan strcut{}代替chan bool传递信号，此时我们不关注传递的值，只关注传递的动作是否发生。
* 防止unkeyed初始化结构
```
type sample {
 x int
 y int
 _ struct{}
}
```
此时赋值sample{x:1,y:1}，但sample{1,1}出现错误。
