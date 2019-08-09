---
title: Golang 二分法查找
date: 2019-08-09 14:58:17
tags:
  - 二分法查找
  - golang
  - 数据结构算法
---


## 递归二分法查找
```go
func BinarySearch(array []int, target int, params ...int) int {
	length := len(array)
	maxPoint := length-1
	var point int
	if len(params) == 0 {
		point = int(maxPoint / 2)
	} else {
		point = params[0]
	}

	if (point ==  0 || point == maxPoint) && target != array[point]{
		return -1
	}

	if target == array[point] {
		return point
	} else if target > array[point] {
		max := point + int(math.Floor(float64(point)/2))
		if max > maxPoint {
			max = maxPoint
		}
		return BinarySearch(array, target, max)
	} else { //target < array[point]
		min := point - int(math.Ceil(float64(point)/2))
		if min < 0 {
			min = 0
		}
		return BinarySearch(array, target, min)
	}
}

// testing
func TestBinarySearch(t *testing.T) {
	array := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}
	assert.Equal(t,11,BinarySearch(array, 12))
	assert.Equal(t,-1,BinarySearch(array, 15))
	assert.Equal(t,0,BinarySearch(array, 1))
	assert.Equal(t,-1,BinarySearch(array, -5))
	assert.Equal(t,-1,BinarySearch(array, 15))
	assert.Equal(t,10,BinarySearch(array, 11))
	assert.Equal(t,4,BinarySearch(array, 5))
}
```

> 二分法的提前是有序的排序

