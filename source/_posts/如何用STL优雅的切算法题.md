---
title: 如何用STL优雅的切算法题
date: 2023-04-25 21:05:44
tags:	cpp
categories:	Competitive Programming
excerpt: 
---

#  <algorithm>

## 谁是最大（最小）值？

问题：给定一个数组，找出最大、最小值。

我猜你会写个循环，把整个数组遍历一遍，找到最值。

什么你要排序？你要`priority_queue`？呃呃，也行吧。

<algorithm>为你提供了现成的取最值的函数，为你免去了每次都敲一个循环的烦恼。

它在时间复杂度上没有任何特殊的优化，只是简单的封装了一下罢了。

### `max_element()`

> This does what you think it does.