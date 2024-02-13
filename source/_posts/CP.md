---
title: AVX指令集 in CP
date: 2023-11-27 10:08:39
tags:
  - CPP
  - TODO
---

# 概述

AVX（Advanced Vector Extensions）是一种为SIMD（Single Instruction, Multiple Data）操作设计的指令集扩展。它提供了更宽的向量寄存器和额外的指令，以实现对多个数据元素的**并行**处理。

2008年3月Intel宣布Sandy Bridge微架构将引入全新的AVX指令集，同年4月公布AVX指令集规范，随后开始不断进行更新，业界普遍认为支持AVX指令集是Sandy Bridge最重要的进步，没有之一。

2013年，英特尔正式发布了**AVX-512**指令集，和之前的 AVX/AVX2一样（只是为了迷惑大家，用位数512命名下一代），AVX-512是一组新的指令集，都属于向量运算指令，将指令宽度进一步扩展到了512bit，相比AVX2在数据寄存器宽度、数量以及FMA单元的宽度都增加了一倍，所以在每个时钟周期内可以打包32 次双精度和 64 次单精度浮点运算，或者8个 64 位和16个 32 位整数，因此在图像/音视频处理、数据分析、科学计算、数据加密和压缩以及人工智能/深度学习等密集型计算应用场景中，会带来前所未有的强大性能表现，**理论上浮点性能翻倍，整数计算则增加约33%的性能**。

# Usage in C++

```cpp
#include <immintrin.h>
```
这个头文件提供了必要的函数与数据类型，这些代码底层使用AVX指令集。使用这个头文件就不必在文件中内联汇编去实现AVX支持。

数据类型：AVX引入了新的数据类型，比如m256用于单精度浮点数（float）的256位宽向量。还有对应的双精度（m256d）和整数数据的类型。

实操

```cpp
#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4.1,sse4.2,avx,avx2,popcnt,tune=native")

#include <emmintrin.h>
#include <immintrin.h>
#include <iostream>

int main()
{
    __m256i a = _mm256_set_epi64x(1024, 2048, 4096, 8192);
    auto b = a;
    auto res = _mm256_add_epi64(a, b);
    for (int i = 0; i < 4; i++)
        std::cout << res[i] << ' ';
    return 0;
}
```

输出：

```
16384 8192 4096 2048
```



# References

* [AVX-512指令集的前世今生 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/136099964)
* [n方过百万 暴力碾标算——指令集优化的基础使用 - ouuan 的博客 - 洛谷博客 (luogu.com.cn)](https://www.luogu.com.cn/blog/ouuan/avx-optimize)
* [AVX — really nice thing to optimize everything - Codeforces](https://codeforces.com/blog/entry/122531)
