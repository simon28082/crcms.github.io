---
title: Leetcode刷题之有效的括号#20    
date: 2019-08-13 14:45:38
tags:
---

## 题目描述 
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。

## 解题思路
由'('，')'，'{'，'}'，'['，']'等一一对应，想到的应该是数据栈。

左侧结构的符号入栈，遇到右侧的出栈对比，这样即可一一对应。

## 代码
```go
func IsValid(s string) bool {
	stack := make([]string, 0)
	for _,v := range strings.Split(s,``) {
		if v == `{` || v == `[` || v == `(` {
			stack = append(stack, v)
		} else {
			var sv string
			if len(stack) == 0 {
				return false
			} else if len(stack) == 1 {
				sv = stack[0]
				stack = stack[0:0]
			} else {
				sv = stack[len(stack)-1]
				stack = stack[:len(stack)-1]
			}

			if sv == `{` && v != `}`{
				return false
			} else if sv == `[` && v != `]` {
				return false
			} else if sv == `(` && v != `)`{
				return false
			}
		}
	}

	return len(stack) == 0
}
```


## 测试 

```golang
func TestIsValid(t *testing.T) {
	assert.Equal(t,false,IsValid("{}{{{{]}}"))
	assert.Equal(t,true,IsValid("{}()[]"))
	assert.Equal(t,true,IsValid("[{{{}}}]"))
	assert.Equal(t,true,IsValid("{([])}"))
}
```