---
title: 最长上升子序列问题的二分优化
description: 对 LIS 问题的二分查找解法的深度解析
slug: lis-w-binary-search
date: 2024-05-10 00:00:01+0800
math: true
categories:
    - notes
tags:
    - 动态规划
---

- 模板题目：[AcWing 896](https://www.acwing.com/problem/content/898/)
- 典型例题：[导弹拦截](https://www.luogu.com.cn/problem/P1020)

## 朴素做法

**最长上升子序列 (longest increasing subsequence, LIS)** 问题的动态规划朴素思路为，针对给定序列中的每一个元素，计算一遍以该元素结尾的所有 LIS，并比较这些序列的长度，取得最大值。为了减小复杂度，计算每个元素结尾的 LIS 长度这一操作，我们通过动态规划提供的状态转移来实现，即根据以前一个元素结尾的 LIS 最大长度，对当前元素结尾的 LIS 的最大长度进行更新。根据这种思想可以写出以下的代码：

``` cpp
const int N = 1e3 + 10;
int a[N], dp[N];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];

    int ans = 0;
    for (int i = n; i >= 1; i--) {
        dp[i] = 1;
        for (int j = n; j > i; j--)
            if (a[i] < a[j])
                dp[i] = max(dp[j] + 1, dp[i]);
        ans = max(dp[i], ans);
    }

    cout << ans << endl;
    return fflush(stdout), 0;
}
```

可以大致分析，这种思路下，代码的复杂度达到了 $\mathcal{O}(n^2)$，所以如果数据范围较大的话，就无法在时限内得出结果了。

## 优化思路

如果说上面的思路还比较符合动态规划的思路，那么优化方式就需要换换脑筋，换用一种贪心的思路。考虑一个序列 $A = \left \lbrace 1, 2, 5, 4, 3, 4\right \rbrace$，并维护一个序列 $F$，每一个元素 $F_i$ 存储长度为 $i$ 的 LIS 中末尾元素的最小值。这样做的好处在于，在遍历 $A$ 中元素时，假如遍历到的元素比 $F$ 中的所有元素都要大，那么恭喜，这个序列的 LIS 就可以增加一个元素了；假如遍历到的元素比 $F$ 中的某个元素要小，就用当前元素**覆盖掉第一个大于或等于它**的元素，在给剩下的元素留出更大空间的同时，也不会影响 LIS 的长度。

这么说可能有点抽象，不如拿实际的例子来说明，更加直观。

我们令序列中的下标从 $1$ 开始。

- 当 $i = 1$ 时，对于 $A_1 = 1$，长度为 $i = 1$ 的 LIS 的末尾元素最小当然只能是 $1$，所以将 $F_1$ 的值设为 $1$；
- 当 $i = 2$ 时，对于 $A_2 = 2$，由于目前 $F$ 中还没有比 $2$ 大的元素，所以把它放到 $F_2$，作为长度为 $2$ 的 LIS 末尾的最小元素；
- 当 $i = 3$ 时，对于 $A_3 = 5$，由于目前 $F$ 中还没有比 $5$ 大的元素，所以把它放到 $F_3$，作为长度为 $3$ 的 LIS 末尾的最小元素；
- 当 $i = 4$ 时，对于 $A_4 = 4$，它比 $F_3 = 5$ 要小，所以为了给后加入的元素腾空间，就把 $F_3$ 的值更新为 $4$；
- 当 $i = 5$ 时，对于 $A_5 = 3$，它比 $F_3 = 4$ 要小，所以为了给后加入的元素腾空间，就把 $F_3$ 的值更新为 $3$；
- 当 $i = 6$ 时，对于 $A_6 = 4$，由于目前 $F$ 中还没有比 $4$ 大的元素，所以把它放到 $F_4$，作为长度为 $4$ 的 LIS 末尾的最小元素。

看到这里，你可能已经发现之前我们设计 $F$ 序列及有关一系列操作的意义了。

如果不进行覆盖，在上面说 $i = 4$ 的时候没有替换掉原来的 $5$，那么当遍历到 $A_6 = 4$ 的时候，你会发现 $4$ 就无法添加到 LIS 末尾元素的序列中，也就无法得出 LIS 的正确长度了。将原序列中的更小元素存入 $F$ 序列，就能给后来的元素腾出更多的空间，从而尽可能加长 LIS。

但是，按照朴素写法，这种思路的复杂度依然是 $\mathcal{O}(n^2)$，所以还需要挖掘我们维护的 $F$ 序列所具有的性质来选取合适的算法以降低复杂度。按照我们对 $F$ 序列的描述，加上上面例子的解释，不难发现，$F$ 序列是单调递增的。而对于具有单调性的、需要查找某一个元素（要查找 $F$ 中第一个比 $A_i$ 大的元素）的问题，常用的解决方法就是二分了。（如果不替换**第一个**比 $A_i$ 大的元素，而是替换成了后面的数，就会破坏 $F$ 序列的单调性，导致无法使用二分解题。）

> $\mathbf{F}$ **序列单调性的证明**
>
> 假设存在 $i < j$，满足 $F_i \geqslant F_j$，则序列 $A$ 中一定有一个以 $F_j$ 结尾的 LIS，那么显然可以从该 $F_j$ 为结尾的 LIS 中不断删除末尾元素，直到留下一个长度为 $i$ 的 LIS，这个新 LIS 的末尾元素一定小于 $F_j$，所以也一定小于 $F_i$。但根据我们的定义，这个新 LIS 的末尾元素应当为 $F_i$，但它同时比 $F_i$ 小，因此产生矛盾，所以 $F_i \geqslant F_j$ 不成立，原序列单调递增。

不过这也会引出一个问题：为什么我们要**覆盖掉**这个大于等于 $A_i$ 的元素，而不是将更大的元素向后挪一位呢？后者明明也不会破坏 $F$ 序列的单调性。还是拿 $A = \left \lbrace 1, 2, 5, 4, 3, 4\right \rbrace$ 这个序列举例：如果我们在遍历到 $A_i = 4$ 时，将 $F_3 = 5$ 向后挪到 $F_4 = 5$，将 $A_i = 4$ 插入到了 $F_3$ 的位置上，那么显然 $5$ 不可能成为一个长度为 $4$ 的 LIS 的末尾元素，因为原序列中 $5$ 及之前一共都只有 $3$ 个元素。放大到任意一个序列 $A$ 来讲，由于我们遍历到 $A$ 中的每一个元素都只考虑其之前的元素所构成的 LIS，如果将 $F$ 中元素向后移动，就会导致 $F$ 中被移动的值无法构成合法的 LIS。

## 代码实现

根据这样的思路，可以写出如下代码：

``` cpp
const int N = 1e5 + 10;
int a[N], f[N];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];

    int len = 0;
    for (int i = 1; i <= n; i++) {
        int l = 1, r = len + 1;
        while (l < r) {
            int mid = (l + r) >> 1;
            if (f[mid] >= a[i])
                r = mid;
            else
                l = mid + 1;
        }
        len = max(len, l);
        f[l] = a[i];
    }

    cout << len << endl;
    return fflush(stdout), 0;
}
```

由于二分的复杂度在对数级别，所以算法的总复杂度就降低到了 $\mathcal{O}(n \log n)$，即可处理更大的数据范围。