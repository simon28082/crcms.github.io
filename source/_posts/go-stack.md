---
title: Golang container/list 实现 简单stack
date: 2019-08-13 21:54:21
tags:
  - stack
  - golang
  - 数据结构算法
---


## 基础实现方法
```go
import "container/list"

type Stack struct {
	list *list.List
}

func NewStack() *Stack {
	return &Stack{
		list: list.New(),
	}
}

func (s *Stack) Push(value interface{}) {
	s.list.PushBack(value)
}

func (s *Stack) Pop() interface{} {
	e := s.list.Back()
	if e != nil {
		s.list.Remove(e)
		return e.Value
	}

	return nil
}

func (s *Stack) Length() int {
	return s.list.Len()
}

```


## 测试
```go
func TestNewStack(t *testing.T) {
	stack := NewStack()
	stack.Push("a")
	stack.Push("b")
	stack.Push("c")
	stack.Push("d")
	assert.Equal(t,4,stack.Length())
	v := stack.Pop()
	assert.Equal(t,"d",v.(string))
	assert.Equal(t,3,stack.Length())
	v = stack.Pop()
	assert.Equal(t,"c",v.(string))
	assert.Equal(t,2,stack.Length())
	v = stack.Pop()
	assert.Equal(t,"b",v.(string))
	assert.Equal(t,1,stack.Length())
	v = stack.Pop()
	assert.Equal(t,"a",v.(string))
	assert.Equal(t,0,stack.Length())
}
```

