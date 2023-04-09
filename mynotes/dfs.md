# DFS

DFS 为图论中的概念，在 **搜索算法** 中，该词常常指利用递归函数方便地实现暴力枚举的算法，与图论中的 DFS 算法有一定相似之处，但并不完全相同。

该类搜索算法的特点在于，将要搜索的目标分成若干「层」，每层基于前几层的状态进行决策，直到达到目标状态。

## 例题 [Luogu P1706 全排列问题](https://www.luogu.com.cn/problem/P1706)

按照字典序输出自然数 $1$ 到 $n$ 所有不重复的排列，即 $n$ 的全排列，要求所产生的任一数字序列中不允许出现重复的数字。

## 输入格式

一个整数 $n$。

## 输出格式

由 $1 \sim n$ 组成的所有不重复的数字序列，每行一个序列。

每个数字保留 $5$ 个场宽。

## 样例
输入

```
3
```

输出

```
    1    2    3
    1    3    2
    2    1    3
    2    3    1
    3    1    2
    3    2    1
```

## 提示

$1 \leq n \leq 9$。

---

C++ 代码：

```cpp
#include <iomanip>
#include <iostream>
using namespace std;
int n;
bool vis[50]; // 访问标记数组
int a[50];    // 排列数组，按顺序储存当前搜索结果

void dfs(int step)
{
    if (step == n + 1)
    { // 边界
        for (int i = 1; i <= n; i++)
        {
            cout << setw(5) << a[i]; // 保留5个场宽
        }
        cout << endl;
        return;
    }
    for (int i = 1; i <= n; i++)
    {
        if (vis[i] == 0)
        { // 判断数字i是否在正在进行的全排列中
            vis[i] = 1;
            a[step] = i;
            dfs(step + 1);
            vis[i] = 0; // 这一步不使用该数 置0后允许下一步使用
        }
    }
    return;
}

int main()
{
    cin >> n;
    dfs(1);
    return 0;
}
```