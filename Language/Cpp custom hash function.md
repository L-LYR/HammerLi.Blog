---
Author: HammerLi
Date: 2021.3.21
Tag: [Cpp][hash function]
---

# 自定义哈希函数

我们知道`unordered_map`和`unordered_set`底层实现是 Hash Table，它自然是需要用一个 Hash Function 来将 Key 定位到容器的一个桶或者槽中，目标即是使两个相同 Key 能够定位到相同的桶中，尽可能减少不同的 Key 的定位冲突（Hash 冲突），同时，输入一定范围内的 Key 能够给出一个分布较为均匀的 Hash Value。

我们自定的类型或者是`pair`，`tuple`类型需要自定义 Hash Function，网上给出的解决方案蛮多错误的，如下：

```cpp
struct pairhash {
public:
  template <typename T, typename U>
  std::size_t operator()(const std::pair<T, U> &x) const {
    return std::hash<T>()(x.first) ^ std::hash<U>()(x.second);
  }
};
```

看似可以起到一定的 Hash 的效果，其实冲突巨多，参看[评论区](https://stackoverflow.com/questions/20590656/error-for-hash-function-of-pair-of-ints)。

*The C++ Standard Library* 一书给出的解决办法：

```c++
#include <functional>
// https://www.boost.org/doc/libs/1_35_0/doc/html/hash/combine.html
template <typename T>
inline void hash_combine(std::size_t& seed, const T& val) {
    seed ^= std::hash<T>()(val) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
}

template <typename T>
inline void hash_val(std::size_t& seed, const T& val) {
    hash_combine(seed, val);
}

template <typename T, typename... Types>
inline void hash_val(std::size_t& seed, const T& val, const Types&... args) {
    hash_combine(seed, val);
    hash_val(seed, args...);
}

template <typename... Types>
inline std::size_t hash_val(const Types&... args) {
    std::size_t seed = 0;
    hash_val(seed, args...);
    return seed;
}
```

使用方法，以`pair<T1, T2>`为例：

```c++
struct hash_pair {
	template <typename T1, typename T2>
    std::size_t operator()(const std::pair<T1, T2>& p) const {
        return hash_val(p.first, p.second);
    }
};

unordered_map<pair<int, long long>, string, hash_pair> test_map;
```

关于 Hash Function 中的 Magic Numer，由来如下：
$$
\phi = \frac{1 + \sqrt{5}}{2}\\
\frac{2^{32}}{\phi} = \text{0x9E3779B9}
$$
其实随机比特位都可以，目的是防止全0映射到全0。