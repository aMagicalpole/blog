---
title: 数组reduce方法的使用
date: 2020-12-01 19:51:56
tags: [ nodejs ]
categories: 前端
copyright: true
author: Jerry Liu
top:
description: reduce方法作为 Array 的一个高级方法，接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值；相比于 Array 的其他方法比较复杂，但是当你了解了过后，将会大大的提高你的效率。
---

&emsp;我们先看下reduce的语法：
>array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
>total:必需。初始值, 或者计算结束后的返回值。
>currentValue:必需。当前元素
>currentIndex:可选。当前元素的索引
>arr:可选。当前元素所属的数组对象。
>initialValue:可选。传递给函数的初始值

&emsp;我们先来看一个例子，也是reduce最基本的方法，累加和累乘：
```
const arr = [1, 2, 3, 4, 5, 6]
//  累加
const sum = arr.reduce((pre,cur) => {
	pre += cur
	return pre
}, 0)    // sum = 21
//  累乘
const muti = arr.reduce((pre, cur) => {
	pre *= cur
	return pre
}, 1)    // muti = 720

```

&emsp;下面我们来介绍一下reduce的一些高级用法：

- reduce对数组去重
```
const arr = [1, 2, 3, 4, 2, 4, 6]

const newArr = arr.reduce((pre,cur) => {
	if (!pre.includes(cur)) {
		pre.push(cur)
	}
	return pre
}, [])  // [1, 2, 3, 4, 6]
```

- 统计数组中元素个数
```
const arr = ['a', 'b', 'c', 'a', 'a', 'c', 'a']
const newObj = arr.reduce((pre,cur) => {
	if (cur in pre) {
		pre[cur]++
	} else {
		pre[cur] = 1
	}
	return pre
}, {})  // {a: 4, b: 1, c: 2}

```
- 合并相同类型数组
```
const arr = [
	{
		name: '椅子',
		value:10
	},
	{
		name: '桌子',
		value: 12
	},
	{
		name: '凳子',
		value: 7
	},
	{
		name: '椅子',
		value: 9
	},
	{
		name: '凳子',
		value: 11
	}
]

const newArr = arr.reduce((pre, cur) => {
	const index = pre.findIndex(item => item.name === cur.name)
	if (index !== -1) {
		pre[index].value += cur.value
	} else {
		pre.push(cur)
	}
	return pre
}, []) 
// [{name: "椅子", value: 19},{name: "桌子", value: 12} ,{name: "凳子", value: 18}]

```
- 多维数组降一维
```
const arr = [[1], [2, 3], 4]

const newArr = arr.reduce((pre, cur) => Array.isArray(cur) ? 
[...pre, ...cur] : [...pre, cur], []) // [1,2,3,4]
```
- 递归多维降为一维
```
function reductionArr(arr) {
	return arr.reduce((pre, cur) => Array.isArray(cur) ?
 [...pre, ...reductionArr(cur)] : [...pre, cur], [])
}

const arr = [[1], [2, [3, 4, [5, 6]]]]
console.log(reductionArr(arr)) // [1,2,3,4,5,6]
```
- 对数组排序
```
const arr = [12, 2, 3, 1, 5, 3, 10, 7, 4, 6]

const newArr = arr.reduce((pre, cur) => {
    const index = pre.findIndex(item => cur <= item)
    if (index === -1) {
        pre.push(cur)
    } else {
        pre.splice(index, 0, cur)
    }
    return pre
}, [])    // [1, 2, 3, 3, 4, 5, 6, 7, 10, 12]
```