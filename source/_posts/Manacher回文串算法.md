---
title: Manacher回文串算法
date: 2023-07-12 11:08:35
tags:
  - algorithm
categories: Competitive Programming
excerpt: 马拉车。
---

`Manacher`: $O(n)$的寻找回文串。

# [P3805【模板】manacher 算法](https://www.luogu.com.cn/problem/P3805)

## 题目描述

给出一个只由小写英文字符 $\texttt a,\texttt b,\texttt c,\ldots\texttt y,\texttt z$ 组成的字符串 $S$ ,求 $S$ 中最长回文串的长度 。

字符串长度为 $n$。

## 输入格式

一行小写英文字符 $\texttt a,\texttt b,\texttt c,\cdots,\texttt y,\texttt z$ 组成的字符串 $S$。$1\le n\le 1.1\times 10^7$。

## 输出格式

一个整数表示答案。

## 样例 #1

### 样例输入 #1

```
aaa
```

### 样例输出 #1

```
3
```

# Idea

### 预处理

在原字符串的每个字符用一个**字符集中不存在的字符**隔开。

比如，对于一个由`a`到`z`的字符组成的串 `aabbbaa` -> `|a|a|b|b|b|a|b|`

这样做是为了使每个回文串都能找到**对称中心**。

对于一个长度为偶数的字符串 `aaaa` 他的对称中心在2和3之间。如果插入字符，`|a|a|a|a|` ，第5个字符就是他的中心。

### 计算

定义`p[i]`为每个字符的回文半径。同时记录一个mid，一个r，分别代表**已经确定的右侧最靠右的回文串的对称中心和右边界**。

对于第 `i` 个字符：

* 如果 `i <= r`，`p[i] = min(p[m * 2 - i], r - i + 1)` 
	* `p[m * 2 - i]` 是与 `i` 关于点 `m` 对称的点。如果以这点为中心的最长回文串在 `[m-r, m]`范围内，`p[i]`显然也会具有这样一个回文串。
	
	* 如果以这点为中心的最长回文串超出了 `[m - r, m]`范围，至少在 `[m - r, m]`范围内的回文串是有效的，可以被传递给`p[i]`。
* 否则 `p[i] = 1`。

随后暴力计算`p[i]`。

该字符串中的最长字串为：`max(p[i]) - 1`。`p[i]` 中计入了额外的分隔符，数目与回文中心的另一端等长，也就相当于回文串的长度。

# Code

```cpp
#include <cstdio>
using namespace std;
using ll = long long;
// !!! N = n * 2, because you need to insert '#' !!!
const int N = 3e7;
#define min(A, B) ((A > B) ? B : A)
// p[i]: range of the palindrome i-centered. 
int p[N];
// s: the string.
char s[N] = "@#";
// l: length of s.
int l = 2;

int main()
{
    char tmp = getchar();
    while (tmp > 'z' || tmp < 'a')
        tmp = getchar();
    while (tmp <= 'z' && tmp >= 'a')
        s[l++] = tmp, s[l++] = '#', tmp = getchar();
    /*<--- input & preparation --->*/
    int m = 0, r = 0;
    ll ans = 0;
    for (int i = 1; i < l; i++)
    {
        // evaluate p[i]
        if (i <= r)
            p[i] = min(p[m * 2 - i], r - i + 1);
        else
            p[i] = 1;
        // brute force!
        while (s[i - p[i]] == s[i + p[i]])
            ++p[i];
        // maintain m, r
        if (i + p[i] > r)
        {
            r = i + p[i] - 1;
            m = i;
        }
        // find the longest p[i]
        if (p[i] > ans)
            ans = p[i]; 
    }
    printf("%lld", ans - 1);

    return 0;
}
```

