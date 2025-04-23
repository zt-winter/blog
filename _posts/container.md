title: container库
tags:
	- container
	- list
	- stack
categories: go
---

# container
	该库主要是实现三个数据结构
* list : 双向链表
* heap : 最小堆
* ring : 环形链表

## list

可以用list双向链表实现栈的操作
```
import(
	"container/list"
)

//stack初始化
stack := list.New()

//压栈操作
stack.PushBack(value)

//出栈操作
e := stack.Back()
if e != nil {
	stack.Remove(e)
}
return e.Value

//获取栈顶元素
e := stack.Back()
return e.Value

//栈长度
length := stack.Len()
```
