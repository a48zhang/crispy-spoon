---
title: C++STL学习笔记
date: 2023-05-25 21:59:30
tags: C++
categories: Programming Skills
excerpt: C++ Standard Template Library (未完成)
---

# 什么是STL

C++ Standard Template Library (STL)，即C++ 标准模板库，是 C++ 标准库的一部分。

其包括六大组件：

- 容器（Containers）
- 算法（Algorithm）
- 迭代器（Iterators）
- 适配器（Adapters）
- 仿函数（Functors）
- 分配器（Allocators）

# 容器(Containers)

- 分类
    - 序列式容器：强调值的排序，序列式容器中的每个元素均有固定的位置
    	- **向量**(`vector`) (顺序表)
    	- **双端队列**(`deque`) (顺序表)
    	- **列表**(`list`) (链表)
    	- **单向列表**(`forward_list`) (链表)
    	- **数组** *C++11*(`array`) (顺序表)
    	
    - 关联式容器：二叉树（红黑树）结构，各元素之间没有严格的物理上的顺序关系.
    
		- **集合**(`set`) 
    
      - **映射**(`map`) `map`中所有元素都是`pair`，所有元素都会根据元素的键值自动排序
      
      - **多重映射**(`multimap`) 
      - **多重集合**(`multiset`) 

        

# 算法(Algorithms)

[`#include <algorithm>`](https://zh.cppreference.com/w/cpp/algorithm) 

- 各种常用算法，如sort、find、copy、for_each等
- 分类
    - 质变算法：运算过程中会更改区间内的元素内容。如拷贝、替换、删除等
    - 非质变算法：运算过程中不会更改区间内的元素内容。例如查找、计数、遍历、寻找极值等。
    
    `//To be continue...`
