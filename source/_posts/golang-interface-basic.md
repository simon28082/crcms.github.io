---
title: Golang Interface 基础示例一
date: 2019-05-05 23:49:04
tags:
  - Golang
  - Interface
  - 接口
---

路径`interfaces/demo.go`

```Golang
package interfaces

type Summationer interface {
	Sum(param1, param2 int) int
}

type Summation struct {
}

func (s *Summation) Sum(param1, param2 int) int{
	return param1 + param2
}

```

`main`调用
```Golang
package main

import (
	"fmt"
	"interfaces"
)

func main() {
	// Go的接口是动态的

	// 申明一个结构体
	sum_struct := new(interfaces.Summation)

	// 给当前结构体对象增加一个接口
	//var sumInter interfaces.Summationer
	//sumInter = sum_struct

	// 或者使用如下方法
	// sumInter := interfaces.Summationer(sum_struct)

	// 或者，这样其实接口也是通的，但并没有什么太大意义
	// 语法上看不出来结构体实现了此接口
	sumInter := sum_struct

	fmt.Println(testInterface(sumInter))
}

func testInterface(s interfaces.Summationer) int {
	return s.Sum(1, 2)
}

```
