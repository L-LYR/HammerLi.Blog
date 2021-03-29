---
Author: HammerLi
Date: 2021.3.24
Tag: [Algorithm][Cantor Expansion]
---

康托展开是一个全排列到一个自然数的双射，常用于构建 hash 表时的空间压缩，或者某些问题中的状态压缩。

设有$n$个数$(1，2，3，4,…,n)$，可以有组成不同($n!$种)的排列，康托展开表示的就是是当前排列组合在n个不同元素的全排列中的名次。
$$
X = a[n] * (n - 1)!+a[n - 1] * (n - 2)!+...+a[i] * (i - 1)!+...+a[1] * 0!
$$
其中，$a[i]$为整数，并且$0 \le a[i] \le i, 0 \le i \lt n$，表示当前未出现的的元素中排第几个。

举个例子，对于数组：$1\ 2\ 3\ 4\ 5\ 6\ 7\ 8\ 9$，其中的一种排列：$8\ 6\ 9\ 1\ 4\ 5\ 2\ 3\ 7$，各个数位后面比他小的值，即运算式中的权值$a[n]$有：$7 5 6 0 2 2 0 0 0$，则他的名次为：$7 * 8! + 5 * 7! + 6 * 6! + 2 * 4! + 2 * 3! = 311820$。

逆康托展开就是上式的逆运算，从高到低除$i!$，储存商，获得权值数列，再利用权值数列反求排列。

C++实现如下：

```c++
class Cantor {
    vector<int64_t> f;

   public:
    Cantor(int n) {
        f.resize(n + 1, 1);
        for (int i = 1; i <= n; ++i) {
            f[i] = f[i - 1] * i;
        }
    }
    auto compress(const vector<int>& seq) -> int64_t {
        int64_t rank = 0;
        int n = seq.size();
        for (int i = 0; i < n; ++i) {
            int cnt = 0;
            for (int j = 0; j < i; ++j) {
                if (seq[j] < seq[i]) cnt++;
            }
            rank += f[n - 1 - i] * (seq[i] - cnt - 1);
        }
        return rank;
    }
    auto recover(int64_t rank, int n) -> vector<int> {
        assert(static_cast<int>(f.size()) == n + 1);

        vector<int> seq(n);
        vector<bool> used(n + 1, false);
        for (int i = 0; i < n; ++i) {
            int64_t r = rank / f[n - 1 - i];
            int j = 1;
            int cnt = 0;
            while (j <= n) {
                if (!used[j]) cnt++;
                if (cnt == r + 1) break;
                ++j;
            }
            seq[i] = j;
            used[j] = true;
            rank %= f[n - 1 - i];
        }
        return seq;
    }
};
```