---
title: 单向链表优化
date: 2019-08-15 03:31:21
tags:
    - golang
    - 数据结构算法
    - 链表
---

## 说明 
在 [Golang 单向链表](./unidirectional-linked-list.md) 中是通过不断修改`next`来实现链表
本章通过一个虚拟`head`方法来优化链表，其原理就是，创建时直接创建`head`和`next`,`head`第一个值始终是`nil`
通过`head`的`next`来作为初始值，并且对原有链表进行相关优化

## 实现代码
```golang
type node struct {
	value interface{}
	next  *node
}

type LinkedList struct {
	dummyHead *node
	length    int
}

func newNode(value interface{}, next *node) *node {
	return &node{
		value: value,
		next:  next,
	}
}

func NewLinkedList() *LinkedList {
	return &LinkedList{
		// 一个虚拟的head value,只使用next
		dummyHead: newNode(nil, newNode(nil, nil)),
		length:    0,
	}
}

// 添加元素
func (l *LinkedList) Add(index int, value interface{}) {
	l.checkIndexRange(index)

	//移动指针到指定下标
	prev := l.dummyHead
	for i := 0; i < index; i++ {
		// 不断的移动链表指针
		// dummyHead 有个虚拟value，其实是从dummyHead的next作为第一个值
		prev = prev.next
	}

	// 每当数据更改时，自动修改其head，移动head头指针（往前增加），而不是直接修改next(不是往后增加)
	prev.next = newNode(value, prev.next)
	l.length ++
}

// 删除元素
func (l *LinkedList) Delete(index int) interface{} {
	l.checkIndexRange(index)

	prev := l.dummyHead
	var current *node
	for i := 0; i <= index; i++ {
		// 保存当前值
		// dummyHead 有个虚拟value，其实是从dummyHead的next作为第一个值
		current = prev.next

		// 需要删除的值
		if i == index {
			if current.next != nil {
				// 移除指定索引，移动指针
				// 当前值的next就等于当前的next，这样就移除了current
				prev.next = current.next
			} else {
				prev.next = nil
			}
		} else {
			prev = current
		}
	}

	l.length --

	return current.value
}

// 通过指定索引获取值
func (l *LinkedList) Index(index int) interface{} {
	l.checkIndexRange(index)

	prev := l.dummyHead
	for i := 0; i <= index; i++ {
		prev = prev.next
	}

	return prev.value
}

// 在链表开头插入值
func (l *LinkedList) Unshift(value interface{}) {
	l.Add(0, value)
}

// 在链表结尾插入值
func (l *LinkedList) Push(value interface{}) {
	l.Add(l.length, value)
}

// 弹出链表最后一个值
func (l *LinkedList) Pop() interface{} {
	return l.Delete(l.length - 1)
}

// 弹出链表第一个值
func (l *LinkedList) Shift() interface{} {
	return l.Delete(0)
}

//
func (l *LinkedList) Length() int {
	return l.length
}

func (l *LinkedList) checkIndexRange(index int) {
	if index < 0 || index > l.length {
		panic(`下标越界`)
	}
}
```

## 测试
```golang
func TestNewLinkedList(t *testing.T) {
	list := NewLinkedList()
	list.Add(0,"a")
	list.Push("b")
	list.Add(2,"c")
	list.Add(3,"d")

	assert.Equal(t,"a",list.Index(0).(string))
	assert.Equal(t,"b",list.Index(1).(string))
	assert.Equal(t,"c",list.Index(2).(string))
	assert.Equal(t,"d",list.Index(3).(string))

	// 中间追加
	list.Add(2,"e")

	assert.Equal(t,"e",list.Index(2).(string))
	assert.Equal(t,"c",list.Index(3).(string))
	assert.Equal(t,"d",list.Index(4).(string))
	assert.Equal(t,5,list.Length())

	assert.Equal(t,"a",list.Index(0).(string))
	assert.Equal(t,"b",list.Index(1).(string))
	assert.Equal(t,"e",list.Index(2).(string))
	assert.Equal(t,"c",list.Index(3).(string))
	assert.Equal(t,"d",list.Index(4).(string))


	assert.Equal(t,"e",list.Delete(2))
	assert.Equal(t,4,list.Length())

	assert.Equal(t,"a",list.Index(0).(string))
	assert.Equal(t,"b",list.Index(1).(string))
	assert.Equal(t,"c",list.Index(2).(string))
	assert.Equal(t,"d",list.Index(3).(string))

	assert.Equal(t,"d",list.Pop())
	assert.Equal(t,3,list.Length())
	assert.Equal(t,"a",list.Index(0).(string))
	assert.Equal(t,"b",list.Index(1).(string))
	assert.Equal(t,"c",list.Index(2).(string))

	assert.Equal(t,"a",list.Shift())
	assert.Equal(t,2,list.Length())
	assert.Equal(t,"b",list.Index(0).(string))
	assert.Equal(t,"c",list.Index(1).(string))
}
```