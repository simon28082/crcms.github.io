---
title: Golang 单向链表
date: 2019-08-11 14:58:17
tags:
  - 链表
  - golang
  - 数据结构算法
---


## 基础链表
```go

// 一个简单的单向链表
type Node struct {
	value string
	// 下一个节点
	next *Node
}

type List struct {
	// 表头
	header *Node
	// 总长度
	length int
}

func NewList() *List {
	return &List{
		header: nil,
		length: 0,
	}
}

func newNode(value string, next *Node) *Node {
	return &Node{
		value: value,
		next:  next,
	}
}

// 添加一个数据至链表
func (l *List) Add(value string) {
	if l.length == 0 {
		l.header = newNode(value, nil)
		l.length = 1
	} else {
		current := l.header
		for {
			if current.next == nil {
				current.next = newNode(value, nil)
				l.length += 1
				break
			}
			current = current.next
		}
	}
}

// 删除链接数据
func (l *List) Delete(value string) bool {
	currentNode := l.header
	prevNode := l.header
	for {
		if currentNode == nil {
			return false
		}

		if currentNode.value == value {
			// 如果是头节点
			if currentNode == prevNode {
				l.header = currentNode.next
			} else {
				// 否则，上一个节点的next指针等于当前删除的元素的next指针
				prevNode.next = currentNode.next
			}

			l.length -= 1
			return true
		}

		if currentNode.next == nil {
			return false
		}

		prevNode = currentNode
		currentNode = currentNode.next

	}
}

// 当前链表是否包指定值
func (l *List) Contains(value string) bool {
	currentNode := l.header
	for {
		if currentNode == nil {
			return false
		}

		if currentNode.value == value {
			return true
		}

		currentNode = currentNode.next
	}
}

// 当前链表长度
func (l *List) Length() int {
	return l.length
}
```

## 测试
```go
func TestList(t *testing.T) {
	list := NewList()
	list.Add("a")
	list.Add("b")
	list.Add("c")
	list.Add("d")
	assert.Equal(t, 4, list.Length())
	assert.Equal(t, true, list.Delete("c"))
	//fmt.Println(list.header.next.next)
	assert.Equal(t, 3, list.Length())
	assert.Equal(t,false,list.Contains("c"))
	assert.Equal(t,true,list.Contains("d"))
	assert.Equal(t,true,list.Contains("a"))

	assert.Equal(t,true,list.Delete("a"))
	assert.Equal(t, 2, list.Length())
	assert.Equal(t,false,list.Contains("a"))

	assert.Equal(t,true,list.Delete("d"))
	assert.Equal(t, 1, list.Length())
	assert.Equal(t,false,list.Contains("d"))

	assert.Equal(t,1,list.Length())
	assert.Equal(t,"b",list.header.value)
	assert.Nil(t,list.header.next)
}
```

