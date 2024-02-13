---
title: 适用于写算法题的一些STL魔法
date: 2023-04-25 21:05:44
tags:	cpp
categories:	Competitive Programming
excerpt: 
---

#  [`#include <algorithm>`](https://zh.cppreference.com/w/cpp/algorithm) 

## 谁是最大（最小）值？

问题：给定一个数组，找出最大、最小值。

<algorithm>为你提供了现成的取最值的函数，为你免去了每次都写一个循环的烦恼。

它在时间复杂度上没有任何特殊的优化，只是简单的封装了一下罢了。

### `max_element()`

```cpp
template< class ForwardIt >
ForwardIt max_element( ForwardIt first, ForwardIt last );
```

> This does what you think it does.

类似的，你也可以使用 `min_element()`。

### `minmax_element()`

```cpp
template< class ForwardIt >
std::pair<ForwardIt,ForwardIt>
	minmax_element( ForwardIt first, ForwardIt last );
```

返回以指向最小元素的迭代器为第一元素，以指向最大元素的迭代器为第二元素的 `pair` 。

若多个元素等价于**最小**元素，则返回指向**首个**这种元素的迭代器。

若多个元素等价于**最大**元素，则返回指向**最后**一个这种元素的迭代器。