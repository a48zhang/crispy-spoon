---
title: C++中的编译期计算
date: 2023-06-25 11:54:49
tags: C++
categories: Programming Skills
excerpt: 让编译器替你打表 (未完成)
---

# 概述

编译期运算就是指在编译阶段由编译器所进行的运算，不占用运行期时间，有时也称模版元编程。

提示：`constexpr`在C++11被引入。建议把编译器版本开的尽可能高。

# 编译期常量

编译器的一些工作，如为数组分配空间、模板特化等，依赖于一些特定的参数（数组大小，模板元素类型）才能完成。很显然的，你不能像在运行期那样轻易的向程序中传入参数，而是需要用一些常量与变量进行运算。

**字面量**是编译期常量。

```cpp
int a[114514];
vector<int> b;
```

**`constexpr`变量**可以被用作编译期常量。

**`constexpr`变量**必须满足下列要求：

- 它的类型必须是[*字面类型* *(LiteralType)* ](https://zh.cppreference.com/w/cpp/named_req/LiteralType)。
- 它必须立即被初始化。
- 它的初始化包括所有隐式转换、构造函数调用等的[全表达式](https://zh.cppreference.com/w/cpp/language/eval_order)必须是[常量表达式](https://zh.cppreference.com/w/cpp/language/constant_expression)



# 实例

## 编译期计算素数表

```cpp
#include <array>
#include <iostream>

constexpr auto get_state()
{
    constexpr int N = 100;
    constexpr std::array<int, N> is_not_prime = []() {
        std::array<int, N> is_not_prime{};
        for (int i = 2; i * i < N; ++i)
        {
            if (!is_not_prime[i])
            {
                for (int j = i * i; j < N; j += i)
                {
                    is_not_prime[j] = true;
                }
            }
        }
        return is_not_prime;
    }();

    constexpr std::size_t M = [is_not_prime]() {
        std::size_t cnt = 0;
        for (std::size_t i = 2; i < N; ++i)
        {
            if (!is_not_prime[i])
            {
                ++cnt;
            }
        }
        return cnt;
    }();

    std::array<int, M> state{};
    for (std::size_t i = 2, j = 0; i < N; ++i)
    {
        if (!is_not_prime[i])
        {
            state[j++] = i;
        }
    }
    return state;
}

int main()
{
    constexpr auto state = get_state();
    for(auto i:state)
        std::cout << i << ' ';

    return 0;
}
```

