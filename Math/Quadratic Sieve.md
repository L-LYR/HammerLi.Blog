---
Author: HammerLi
---

[TOC]

## 二次筛选法

### 前置知识

#### 费马小定理

> 若$p$是质数，且$gcd(a,p)=1$，那么$a^{p-1}\equiv 1 (mod \ p)$。

#### 费马因式分解

> 对于任意一个奇数$n$，有$n=ab=x^2-y^2$，其中$x$和$y$必为整数。

上述推导是很容易的，如下
$$
n=ab=x^2-y^2=(x-y)(x+y)\Rightarrow \begin{cases} a=(x-y)\\b=(x+y)\end{cases} \Rightarrow \begin{cases} x=\frac{a+b}{2} \\ y = \frac{a-b}{2} \end{cases}
$$
其中$n$为奇数，则$a,b$必为奇数，所以$(a-b),(a+b)$必为偶数，进而$x,y$必为整数，且$x>y$。

进而我们可以进行变换得到$y^2=x^2-n$，因此有以下推导
$$
y^2\ge 0 \Rightarrow x^2-n \ge 0 \Rightarrow x \ge \sqrt{n}
$$
我们可以从$x=\lceil \sqrt{n} \rceil$开始枚举，直到$x^2-n$为完全平方数为止，即可求得$x$与$y$，从而获得$a$和$b$。

显然我们知道质数（除$2$以外）都是奇数，所以质数$n$的费马因式分解一定是$n=n\cdot 1$，故$a=n,b=1$，这时我们枚举的区间即为$[\lceil \sqrt{n} \rceil, \frac{n+1}{2}]$。

### 算法简述

#### 算法思路

二次筛法即是来源于上述费马分解法的。由费马分解法我们有
$$
x^2-y^2=n\Rightarrow x^2\equiv y^2(mod \ n)
$$
这指导我们去寻找一些完全平方数之间的关系，而不是直接的去试除或者枚举。

我们构造二次函数
$$
Q(x)=(x+\lceil\sqrt{n}\rceil)^{2}-n
$$
其中$x$为非负整数，之所以构造这样一个二次函数，是因为它自然地满足我们想要的一些形式上的需求：
$$
Q(x) \equiv(x+\lceil\sqrt{n}\rceil)^{2}(\bmod \mathrm{n})
$$
进一步明确，我们只需要$Q(x)$是一个（模$n$的）平方数，即可以用费马分解法分解。但是在这个过程中，我们不会试图去**直接**寻找一个满足条件的完全平方数，而是去找一系列能够**被某个质数集合完全分解**的数，进而由这些数，**拼凑**出$Q(x)$。这样思考是因为**平方数的所有质因子的幂都是偶数**（如$400 = 2^4 \times 5^2$）。 所以我们接下来需要做的便很明晰了。

#### 引入名词

上文中提到的 “某个质数集合”，我们称之为**因子基**（Factor Base），能够被因子基中的质数完全分解的数，我们称其为**光滑数**（Smoothness）。比如对于因子基$\{2,5\}$，$400$相对于它就是光滑的，而$30$则是不光滑的。 

#### 算法步骤

- 步骤1.确定因子基

因子基$S=\{p_1,p_2,\ldots , p_m \}$，其中要求$p_m(m\ge 2)$满足两条性质：

1. $p_m$是素数；
2. $n\ mod\ p_m$是**二次剩余**的，即满足表达式$\exist x\ s.t.\ x^2\equiv n(mod \ p_n)$。

注：这里根据$n$的大小取一定数量的满足性质的即可。

- 步骤2.获取$Q(x_i)$

根据二次函数$Q(x)=(x+\lceil\sqrt{n}\rceil)^{2}-n$，我们取**一定范围**的$x$，计算其函数值构成初始的光滑数选取集合。（注：此范围根据$n$的大小来定，若接下来算法未能分解，则可进一步扩大。）

假定我们在经历过步骤2之后获取的光滑数$Q(x_i)$如下
$$
\mathrm{T}=\left\{\mathrm{Q}\left(x_{1}\right), \mathrm{Q}\left(x_{2}\right), \mathrm{Q}\left(x_{3}\right) \ldots \mathrm{Q}\left(x_{i}\right)\right\}
$$
则它们的乘积一定满足
$$
\begin{aligned} Q\left(x_{1}\right)  Q\left(x_{2}\right) \ldots Q\left(x_{i}\right) \left.\equiv\big(\left(x_{1}+\lceil\sqrt{n}\rceil\right) \left(x_{2}+\lceil \sqrt{n}\right\rceil\right)  \ldots \left(x_{i}+\lceil\sqrt{n}\rceil\right)\big)^{2}(\bmod \mathrm{n}) \end{aligned}
$$
这时，右边已经符合完全平方数，而左边，因为$Q(x_i)$均是光滑的，所以我们只需要找到合适的$Q(x_i)$相乘来确保所得到的乘积$Q(x)$的所有质因子的幂均为偶数即可，而这一过程本质上是一个求解线性方程的过程。

我们将获取的这些光滑数做质因子分解（实际上这一步已经在步骤2中完成），如下
$$
Q(x_i)={p_1}^{a_{i1}}{p_2}^{a_{i2}}\cdots{p_m}^{a_{im}}
$$
然后我们就得到了指数矩阵$M=\{a_{im}\bmod 2\}$，接着选取这些$Q(x_i)$只需要解如下线性方程即可：
$$
RM=\vec {0}_m
$$
所解得的列向量$R={r_j}$实际上是一个$01$串，它表示的就是光滑数集合的一个子集，即我们想要求取的用来拼凑结果的“积木”，于是我们有
$$
Q(x)=\prod_{r_j=1}Q(x_j)
$$
这便是我们最初的等式$x^2\equiv y^2(mod \ n)$中的$x$，而$y$则是
$$
\prod_{r_j=1}(x_j+\lceil\sqrt{n}\rceil)^{2}
$$

- 步骤4.分解$n$

$$
n=\gcd(x-y,n)\cdot \gcd(x+y,n)
$$

### 时间复杂度

以下论断摘自维基百科

>  一般来说，二次筛选法的执行时间为
>  $$
>  e^{(1+o(1)) \sqrt{\ln n \ln \ln n}}=L_{n}[1 / 2,1]
>  $$

### 算法案例

接下来我们针对$15347$进行二次筛选分解。

- 

- 因为$15347$比较小，所以首先选择因子基$S=\{2,17,23,29,31\}$（满足要求）。

- 计算$Q(x_i)$，这里我们取$0\le x_i\le 99$，接着有
  $$
  \begin{aligned} \mathrm{T}&=\{Q(x_{1}), Q(x_{2}), Q(x_{3}), Q(x_{4}), Q(x_{5}), Q(x_{6}), \ldots ,Q(x_{99})\} \\ &=\{29,278,529,782,1037,1294, \ldots ,34382\} \end{aligned}
  $$

- 使用筛法进行筛选：

  1. 将$T$中所有数中含有$2$的因子全部除去，得到$T_1$，如下：
     $$
     T_1=\{29,139,529,391,1037,647,\ldots,17191\}
     $$

  2. 将$T_1$中所有数中含有$17$的因子全部除去，得到$T_2$，如下：
     $$
     T_2=\{29,139,529,23,61,647,\ldots,17191\}
     $$

  3. 依次类推，遍历整个因子基$S$，最后得到
     $$
     T_n=\{1,139,1,1,61,647,\ldots,17191\}
     $$

- 最后$T_n$中出现了若干个$1$，这是预料之内的。

- 事实上，在该过程中我们发现
  $$
  Q(2)=(2+\lceil15347\rceil)^2-15347=126^2-15347=529=23^2
  $$
  这意味着$126^2\equiv 23^2\pmod{15347}$，则我们直接可以得到
  $$
  15347=\gcd(126+23,15347)\cdot\gcd(126-23,15347)=149\times103
  $$
  但一般情况下，我们不会有这么好运，所以抛去该数，我们希望用剩下的数据走完整个算法流程。因此我们对拼凑因子$Q(x_i)$有如下的选取
  $$
  Q(0)=29,Q(3)=782,Q(54)=16337,Q(71)=22678
  $$
  他们对应的指数矩阵$\pmod{2}$，如下：
  $$
  M=\left[\begin{array}{lllll}{0} & {0} & {0} & {1} & {0} \\ {1} & {1} & {1} & {0} & {0} \\ {0} & {1} & {0} & {0} & {0} \\ {1} & {1} & {1} & {1} & {0}\end{array}\right]
  $$

- 最后的任务便是解如下方程
  $$
  RM\equiv [0,0,0,0,0]\pmod{2}
  $$
  线性代数方面的解方程这里从略，因此解得
  $$
  R=[1,1,0,1]
  $$
  则我们需要构造的$Q(x)$如下
  $$
  Q(x)=Q(0)Q(3)Q(71)=29\times 782\times 22678=22678^2
  $$
  相对应的有
  $$
  (0+\lceil15347\rceil)^2(3+\lceil15347\rceil)^2(71+\lceil15347\rceil)^2=(124\times 127\times 195)^2=3070860^2
  $$
  构造出预期的等式
  $$
  22678^2\equiv 3070860^2\pmod{15347}
  $$
  
- 分解$15347$

$$
15347=\gcd(3070860+22678,15347)\cdot\gcd(3070860-22678,15347)=149\times103
$$

## 参考资料

[1] C. POMERANCE, "Analysis and comparison of some integer factoring algorithms," *Analysis and Comparison of some Integer Factoring Algorithms,* 1982.

[2] Quadratic sieve — Wikipedia, the free encyclopedia[EB/OL]. 2019.https://en.wikipedia.org/wiki/Quadratic_sieve
