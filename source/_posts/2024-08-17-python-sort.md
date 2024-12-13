---
title: python sort
date: 2024-08-17 09:59:53
updated: 2024-08-17 09:59:53
categories: Python
tags: [Python]
description: sort
---

## sort
`l.sort()`，原地排序列表，sort 能够对很多内置类型排序。但是自定义的类没法直接排序，可以实现特定的魔法方法，让 sort 直接排序，但是真实情况下，可能需要很多种排序方式，自定义方法不能满足需求。

## key
key 参数接收一个函数，函数的参数是排序列表中的一项元素，函数的返回值应该是一个可以比较的值。

```python
places = ['home', 'work', 'New York', 'Paris']
places.sort()
places.sort(key=lambda x: x.lower())
```

## 多属性排序
如果要把一个对象按分数和姓名两个属性排序要怎么办？可以利用元祖排序特点，让排序函数返回一个元祖。

```python
students.sort(key=lambda s: (s.score, s.name))
students.sort(key=lambda s: (-s.score, s.name))
# 对于一些可以使用一元表达式取负数的属性，可以在前面加个负号。
```

## 分步骤排序
比如想要按姓名倒序排序，按分数正序排序，显然无法构造合适的元祖，可以先按姓名排序再按分数排序。

sort 排序是稳定的，这就让分步骤排序可行。

