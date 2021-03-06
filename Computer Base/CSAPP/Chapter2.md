---
Author: HammerLi
---

[TOC]

# 信息的表示和处理

## 信息存储

对于机器级程序，内存被视为一个极其大的字节数组，或者称作虚拟内存（virtual memory），其中的每一个字节都由一个唯一的数字来标识，作为它的地址（address），所有可能的地址构成一个集合，即虚拟地址空间。本质上的实现是由动态随机访问存储器、闪存、磁盘存储器、特殊硬件、操作系统结合起来为程序提供的这样一种机器级的抽象——看上去统一的字节数组。

### 十六进制（hexadecimal）

| 二进制 | 十进制 | 十六进制 |
| ------ | ------ | -------- |
| 0000   | 0      | 0        |
| 0001   | 1      | 1        |
| 0010   | 2      | 2        |
| 0011   | 3      | 3        |
| 0100   | 4      | 4        |
| 0101   | 5      | 5        |
| 0110   | 6      | 6        |
| 0111   | 7      | 7        |
| 1000   | 8      | 8        |
| 1001   | 9      | 9        |
| 1010   | 10     | A        |
| 1011   | 11     | B        |
| 1100   | 12     | C        |
| 1101   | 13     | D        |
| 1110   | 14     | E        |
| 1111   | 15     | F        |

对于计算机而言，一个字节由八个位组成，其值域用二进制表示为$00000000_{(2)}\sim 11111111_{(2)}$。显然，二进制表示法过于冗长，而十进制表示法与位模式互相转换非常繁琐，进而采取了十六进制表示法。

## 字数据大小

对于每一台计算机，虚拟地址是使用一个字来编码的，其长度被称为字长（word size），指明了指针数据的标称大小（nominal size），同时决定了最重要的系统参数——虚拟地址空间的最大大小。对于一个字长为$w$的计算机，虚拟地址的范围为$0 \sim 2^w-1$。对于32位字长的计算机，其虚拟地址空间位4千兆字节（4GB），而64位字长的计算机则为16EB。

向后兼容，大多数64位机器同样可以运行32位机器编译的程序，反之则不然。

### C语言数据类型典型大小

| 类型/编译器   | 16位编译器 | 32位编译器 | 64位编译器 |
| ------------- | ---------- | ---------- | ---------- |
| `void`        | 0          | 0          | 0          |
| `char`        | 1          | 1          | 1          |
| `char *`      | 2          | 4          | 8          |
| `short int`   | 2          | 2          | 2          |
| `int`         | 2          | 4          | 4          |
| `float`       | 4          | 4          | 4          |
| `double`      | 8          | 8          | 8          |
| `long`        | 4          | 4          | 8          |
| `long double` | 8          | 12         | 16         |
| `long long`   | 8          | 8          | 8          |
| `int16_t`     | 2          | 2          | 2          |
| `int32_t`     | -          | 4          | 4          |
| `int64_t`     | -          | 8          | 8          |

C语言标准对不同数据类型的数字范围设置了下界，但并没有上界。考虑到可移植性，使用确定大小的整数类型是程序员准确控制数据表示的最佳途径。

## 寻址和字节顺序

多字节对象一般被存储为连续的字节序列，对象的地址一般为所使用的字节中最小的地址。

### 大端法和小端法（little/big endian）

考虑一个$w$位的整数，其位表为$[x_{w-1},x_{w-2},\ldots,x_1,x_0]$，其中$x_{w-1}$为最高有效位，$x_0$为最低有效位，假设$w=8n,n\in N+$，则最高有效字节包含位$[x_{w-1},x_{w-2},\ldots,x_{w-8}]$，最低有效字节包含位$[x_7,x_6,\ldots,x_0]$。

在内存中按照从最低有效字节到最高有效字节的顺序存储对象的方法称为小端法，从最高有效字节到最低有效字节的称为大端法。

如`0x12345678`的存储：

| 地址   | ... | `0x100` | `0x101` | `0x102` | `0x103` | ... |
| ------ | --- | ------- | ------- | ------- | ------- | --- |
| 大端法 | ... | 12      | 34      | 56      | 78      | ... |
| 小端法 | ... | 78      | 56      | 34      | 12      | ... |

较新的微处理器使用的是双端法（bi-endian），可配置不同的机器，但一旦选择了特定的操作系统，字节顺序随之确定。

## 字符串

### ASCII

使用ASCII码作为字符码在任何系统上都能得到相同的结果，与字节顺序和字长无关，因此文本数据比二进制数据具有更强的平台独立性。（ASCII字符集适合于编码英文文档）

### Unicode标准

UTF-8（8-bit Unicode Transformation Format）是一种针对Unicode的可变长度字符编码，又称万国码，由Ken Thompson于1992年创建。现在已经标准化为RFC 3629。UTF-8用1到6个字节编码Unicode字符。

~~~C
/* Bytes Bits Hex Min  Hex Max  Byte Sequence in Binary */
/*   1     7  00000000 0000007f 0vvvvvvv */
/*   2    11  00000080 000007FF 110vvvvv 10vvvvvv */
/*   3    16  00000800 0000FFFF 1110vvvv 10vvvvvv 10vvvvvv */
/*   4    21  00010000 001FFFFF 11110vvv 10vvvvvv 10vvvvvv 10vvvvvv */
/*   5    26  00200000 03FFFFFF 111110vv 10vvvvvv 10vvvvvv 10vvvvvv 10vvvvvv */
/*   6    31  04000000 7FFFFFFF 1111110v 10vvvvvv 10vvvvvv 10vvvvvv 10vvvvvv 10vvvvvv */
~~~

第一列表示编码所需字节数，第二列为能表示的Unicode的最大二进制位数，第三列和第四列为能表示的Unicode范围，最后一列表示编码后的字节布局。把编码中字母v表示的部分连接起来就是对应的Unicode编码。

~~~c
typedef struct {
    int cmask;
    int cval;
    int shift;
    long lmask;
    long lval;
} Tab;

static Tab tab[] = {
    0x80,   0x00,   0*6,    0x7F,       0,          /* 1 byte sequence */
    0xE0,   0xC0,   1*6,    0x7FF,      0x80,       /* 2 byte sequence */
    0xF0,   0xE0,   2*6,    0xFFFF,     0x800,      /* 3 byte sequence */
    0xF8,   0xF0,   3*6,    0x1FFFFF,   0x10000,    /* 4 byte sequence */
    0xFC,   0xF8,   4*6,    0x3FFFFFF,  0x200000,   /* 5 byte sequence */
    0xFE,   0xFC,   5*6,    0x7FFFFFFF, 0x4000000,  /* 6 byte sequence */
    0,                                              /* end of table */
};
~~~

每个`Tab`包含五个字段。其中`lmask`和`lval`最简单，对应所能表示的 Unicode 的最大值和最小值。`Tab`所在的行号代表表示该范围 Unicode 所需要的字节数。

#### UTF-8  $\rightarrow$  Unicode

 ```c
/* s 指向 UTF-8 字节序列，n 表示字节长度 */
/* p 指向一个 wchar_t 变量 */
/* mbtowc 对 s 指定的字节进行解码，得到的 Unicode 存到 p 指向的变量 */
int
mbtowc(wchar_t *p, char *s, size_t n)
{
    long l;
    int c0, c, nc;
    Tab *t;

    if(s == 0)
        return 0;

    nc = 0;
    if(n <= nc)
        return -1;

    /* c0 保存第一个字节内容，后面会移动 s 指针，此处备份一下 */
    /* 汉字「吕」的 UTF-8 编码是 `11100101`, `10010000`, `10010101` */
    /* 此时 l = c0 = 11100101 */
    c0 = *s & 0xff;
    /* l 保存 Unicode 结果 */
    l = c0;

    /* 根据 UTF-8 的表示范围从小到大依次检查 */
    for(t=tab; t->cmask; t++) {
        /* nc 以理解为 tab 的行号 */
        /* tab 行号跟这个范围内 UTF-8 编码所需字节数量相同 */
        nc++;

        /* c0 指向第一个字节，不会变化 */
        /* l 在 n == 1 和 n == 2 时左移 6 位两次 */
        /* 到 nc == 3 时才会进入该分支 */
        /* 此时的 l 已经是 11100101+010000+010101 了 */
        if((c0 & t->cmask) == t->cval) {
            /* lmaxk 表示三字节能表示的 Unicode 最大值 */
            /* 使用 & 操作，移除最高位的 111 */
            /* 所以 l 最终为 00000101+010000+010101 */
            /* 也就是 l = 0x5415，对应 Unicode +U5415 */
            l &= t->lmask;

            if(l < t->lval)
                return -1;

            /* 保存结果并反回 */
            *p = l;
            return nc;
        }

        if(n <= nc)
            return -1;

        /* s 指向下一个字节 */
        s++;
        /* 0x80 = 10000000 */
        /* UTF-8 编码从第二个字节开始高两位都是 10 */
        /* 这一步是为了把最高位的 1 去掉 */
        c = (*s ^ 0x80) & 0xFF;
        /* n == 1 时 */
        /* c = 00010000 */
        /* n == 2 时 */
        /* c = 00010101 */

        /* 0xc0 = 1100000 */
        /* 这一上检查次高位是否为 1，如果是 1，则为非法 UTF-8 序列 */
        if(c & 0xC0)
            return -1;

        /* c 只有低 6 位有效 */
        /* 根据 UTF-8 规则，l 左移 6 位，将 c 的低 6 位填入 l */
        l = (l<<6) | c;
        /* n == 1 时 */
        /* l 的值变成 11100101+010000 */
        /* n == 2 时 */
        /* l 的值变成 11100101+010000+010101 */
    }
    return -1;
}
 ```

#### UTF-8  $\leftarrow$  Unicode

```c
/* wc 存有一个 Unicode */
/* s 指向存放 UTF-8 的内存 */
/* wctomb 返回最终 UTF-8 编码的字节长度 */
int
wctomb(char *s, wchar_t wc)
{
    long l;
    int c, nc;
    Tab *t;

    if(s == 0)
        return 0;

    /* l = wc = 101010000010101 */
    l = wc;
    nc = 0;
    /* tab 每一行表示的 Unicode 的范围是递增的 */
    for(t=tab; t->cmask; t++) {
        nc++; /* 记录行数，也就是字节数 */
        /* 找到第一个可以表示的 Tab */
        /* 对于「吕+U5412」nc == 3 */
        if(l <= t->lmask) {
            /* c->shift = 2 * 6 */
            /* c->cval = 11100000 */
            c = t->shift;
            /* UTF-8 共需要 3 个字节 */
            /* 第 2 个和第 3 个字节各有 6 个有效位 */
            /* 所以将 l 右移 2 * 6 位得到的结果需要存到第一个字节 */
            /* 首字节高 4 位还要存储后续字节长度标识 */
            *s = t->cval | (l>>c);

            /* 处理剩余字节 */
            while(c > 0) {
                c -= 6;
                /* s 依次指向下一个字节 */
                s++;
                /* 0x3f = 00111111 */
                /* 0x80 == 10000000 */
                /* c == 0 时，取 l 的低 6 位并将其高两位置成 10 */
                /* 此时 s 指向的位置存有 10010101 */
                /* c == 1 时，取 l 的次低 6 位并将其高两位置成 10 */
                /* 此时 s 指向的位置存有 10010000 */
                *s = 0x80 | ((l>>c) & 0x3F);
            }

            /* 最终 s 最初指向的区域存有 11100000, 10010101, 10010000 */
            return nc;
        }
    }
    return -1;
}
```

## 布尔代数

逻辑运算$\rightarrow$布尔代数$\rightarrow$位向量运算（位运算）

## 位运算

### 运算符

| C        | Pascal     |
| -------- | ---------- |
| `a & b ` | ` a and b` |
| `a | b ` | `a or b`   |
| `a ^ b ` | `a xor b`  |
| `  ~a  ` | `  not a`  |
| `a << b` | `a shl b`  |
| `a >> b` | `a shr b`  |

注意优先级，记不清带括号。

#### 对右移运算的细分

- 逻辑右移

在左端补齐相应个数的0。

- 算术右移

在左端补齐相应个数的最高有效位。

#### 移位位数超数据类型位数

机器会对位移量关于数据类型位数取模，但最好不要超过。

### 简单操作

| 功能                    | 示例                                   | 位运算                                 |
| ----------------------- | -------------------------------------- | -------------------------------------- |
| 去掉最后一位            | (101101->10110)                        | `x >> 1`                               |
| 在最后加一个0           | (101101->1011010)                      | `x << 1`                               |
| 在最后加一个1           | (101101->1011011)                      | `x << 1 + 1`                           |
| 把最后一位变成1         | (101100->101101)                       | `x | 1`                                |
| 把最后一位变成0         | (101101->101100)                       | `x | 1 - 1`                            |
| 最后一位取反            | (101101->101100)                       | `x ^ 1`                                |
| 把右起第k位变成1        | (101001->101101, k=3)                  | `x | (1 << (k - 1))`                   |
| 把右起第k位变成0        | (101101->101001, k=3)                  | `x & ( ~(1 << (k - 1)))`               |
| 右起第k位取反           | (101001->101101, k=3)                  | `x ^ (1 << (k - 1))`                   |
| 取末三位                | (1101101->101)                         | `x & 7`或`x & 0x111`                   |
| 取末k位                 | (1101101->01101, k=5)                  | `x & ((1 << k) - 1)`                   |
| 取右起第k位             | (1101101->1, k=4)                      | `(x >> (k - 1)) & 1`                   |
| 把末k位变成1            | (101001->101111, k=4)                  | `(x | (1 << (k - 1)))`                 |
| 末k位取反               | (101001->100110, k=4)                  | `x ^ ((1 << k) - 1)`                   |
| 把右起连续的1变成0      | (100101111->100100000)                 | `x & (x + 1)`                          |
| 把右起第一个0变成1      | (100101111->100111111)                 | `x | (x + 1)`                          |
| 把右起连续的0变成1      | (11011000->11011111)                   | `x | (x - 1)`                          |
| 取右起连续的1           | (100101111->1111)                      | `(x ^ (x + 1)) >> 1`                   |
| 取右起第一个1的右侧部分 | (100101000->1000)                      | `x & (x ^ (x - 1))`                    |
| 将k~1~位至k~2~位间反转  | (100101000->100011000, k~1~=6, k~2~=5) | `x ^ ((1 << (k2 - k1 + 1)) - 1) << k2` |

### 简单应用

位向量可以来表示有限集合，因此可以通过位向量对集合编码。

一些可移植性的细节：`~0`和`0xffffffff`对于32位机器来说都是全1的32位掩码，但后者不可移植。

## 整数表示

### 无符号数的编码（Binary to Unsigned）

$$
B2U_w(\vec x)=\sum_{i=0}^{w-1}x_i2^i\\
\vec x=[x_{w-1},w_{w-2},\ldots,x_0]
$$

- 该函数是一个双射——无符号数编码的唯一性。

### 补码编码（Binary to Two's-complement）

$$
B2T_w(\vec x)=-x_{w-1}2^{w-1}+\sum_{i=0}^{w-2}x_i2^i\\
\vec x=[x_{w-1},w_{w-2},\ldots,x_0]
$$

- 最高位$x_{w-1}$被称作符号位，1为正，0为负。
- 该函数是一个双射——补码编码的唯一性。
- 补码的不对称性，$|T_{min}|=|T_{max}|+1$
- 无符号数与有符号数的关系
  - $U_{max}=2T_{max}+1$
  - $U_{max}$与$-1$编码一样
  - 两种编码中$0$的表示一样。

- 为防止一些细微的差错，建议善用一些数据类型，推荐使用`limits.h` `stdint.h`等头文件，使用`uintN_t` `intN_t`等类型，提高程序可移植性。

## 无符号数和有符号数的强制类型转换

#### Two's-complement to Unsigned 

$$
T2U_w(x)=
\begin{cases}
x+2^w&,x<0\\
x&,x\ge 0\
\end{cases}
\\
T_{min_w} \le x\le T_{max_w}
$$

#### Unsigned to Two's-complement

$$
U2T_w(x)=\begin{cases}
x&,x\le T_{max_w}\\
x-2^w&,x\ge T_{max_w}
\end{cases}
$$

- C语言中有符号数在运算过程中强制转换为无符号数的处理方式不会影响运算结果（从二进制串上来看），但是对于比较运算符来说，结果会有直观的差别，譬如$-1U<0 \Leftrightarrow 4294967295U<0U$。

## 扩展与截断（不同字长的整数间进行转换）

### 无符号数的零扩展

定义宽度为$w$的位向量$\vec x=[x_{w-1},\ldots,x_0]$和宽度为$w'$的位向量$\vec x'=[0,0,\ldots,0,x_{w-1},\ldots,x_0]$，其中有$w'>w$，则有$B2U_w(\vec x)=B2U_{w'}(\vec x')$。

这个很容易理解。因为只是二的幂次的加和，无需添加更多的有效位。

### 有符号数的符号扩展

定义宽度为$w$的位向量$\vec x=[x_{w-1},\ldots,x_0]$和宽度为$w'$的位向量$\vec x'=[x_{w-1},x_{w-1},\ldots,x_{w-1},x_{w-1},\ldots,x_0]$，其中有$w'>w$，则有$B2T_w(\vec x)=B2T(\vec x')$。

可由公式$2^n=\sum_{i=0}^{n-1}2^i-1$了解到符号位的移动一定会影响到二进制串转换为补码的过程，因此要添加有效位。

### 截断无符号数

已知宽度为$w$位的位向量$\vec x=[x_{w-1},\ldots,x_0]$，将其截断为$k$位，即$\vec x'=[x_{k-1},\ldots,x_0]$，所以存在以下关系$B2U_w(\vec x')=B2U_k(\vec x)\mod 2^k$。

这里由结果可知，截断到哪一位，在此位之前的所有位权值均化为0，之后保留，再根据取模运算的性质，以截断位为分界线可以得出截断公式。

### 截断有符号数

已知宽度为$w$位的位向量$\vec x=[x_{w-1},\ldots,x_0]$，将其截断为$k$位，即$\vec x'=[x_{k-1},\ldots,x_0]$，所以存在以下关系$B2T_k(\vec x')=U2T_k(B2U_w(\vec x)\mod 2^k)$。

这里先将该位向量视作无符号数进行截断操作，然后进行强制类型转换。

## 整数运算

### 无符号数加法

考虑到两个非负整数$x,y$ 均在无符号数的范围内，即$0\le x,y< 2^w$，但对于C语言等一些编程语言来说，不允许任意精度的整数运算，两数加和之后会存在溢出的情况，因此有
$$
x+^u_wy=\begin{cases}
x+y&,(x+y)<2^w\\
x+y-2^w&,(x+y)>2^w
\end{cases}
$$

很好理解，先假设计算机很聪明，再计算存在溢出情况时它也会按正常的进位得到溢出的结果，即一个字长为$w+1$的二进制数，这时它再依据死板的字长限制进行截断，将结果截断为字长为$w$的二进制数。

### 判断无符号数溢出的办法

记$s=x+y$，倘若有$s<x$或$s<y$成立，则说明发生了溢出。

这里很显然，从加法的两种结果来看即可。

```c
int uadd_ok(unsigned x, unsigned y)
{
    unsigned s = x + y;
    //return (x > s) || (y > s) ? 0 : 1;
    return s >= x;
}
```

### 无符号数求反（减法）

根据以上的无符号数加法法则可很容易得到减法法则
$$
-^u_wx=\begin{cases}
0&,x=0\\
2^w-x&,0<x<2^w
\end{cases}
$$

### 补码加法

对于有符号数$-2^{w-1}\le x ,y \le 2^{w-1}-1$之间的加法有如下法则
$$
x+_w^t y=\begin{cases}
x+y-2^w&,2^{w-1}\le x+y\\
x+y&,-2^{w-1}\le(x+y)<2^{w-1}\\
x+y+2^w&,x+y<-2^{w-1}
\end{cases}
$$

看似复杂，实则推导过程简单粗暴。首先先将两个有符号数转换为无符号数，接着再用无符号数的加法进行运算，最后将结果截断之后转换为有符号数即可，写作函数映射关系即为如上所示。

### 判断有符号数溢出的办法

记$s=x+y$，倘若$s\le 0\ x>0\ y>0$有成立，则说明发生了上溢；倘若$s\ge 0\ x<0\ y<0$成立，则说明发生了下溢。

```c
int tadd_ok(int x, int y)
{
    int sum = x + y;
    if (x > 0 && y > 0 && sum <= 0)
        return 0;
    else if (x < 0 && y < 0 && sum >= 0)
        return 0;
    else
        return 1;
    /*
    int neg_over=(x < 0 && y < 0 && sum >= 0);
    int pos_over=(x > 0 && y > 0 && sum <= 0);
    return neg_over||pos_over;
    */
}
```

### 补码求反（减法）

同理，有
$$
-^t_wx=\begin{cases}
T_{min_w}&,x=T_{min_w}\\
-x&,x>T_{min_w}
\end{cases}
$$

### 无符号数乘法


$$
x \times^u_wy=(x\times y)\mod 2^w
$$

### 补码乘法

$$
x\times_w^t y=U2T_w((x\times y)\mod 2^w)
$$

### 无符号和补码乘法的位级等价性

这个地方无需用过于公式化的语言来表达，或者说正常语言表达更好理解。

位级等价性在于，两串二进制数进行乘法，将其结果以两种不同的编码方式读取所得的值即为原二进制串以相应的编码方式读取所得的数的乘积。

### 两种检验乘法溢出的方法

以32位乘法为例

~~~CQL
int tmult_ok_ver_1(int32_t x, int32_t y)
{
	int32_t p = x * y;
    return !x || p / y == x;
}

int tmult_ok_ver_2(int32_t x, int32_t y)
{
    int64_t p = (int64_t)x * y;
    return p == (int32_t)p;
}
~~~

这里使用了除法来检验乘法，但是上面的加法却不可以用减法检验。

```c
int tadd_ok(int x, int y)
{
    int sum = x + y;
    return (sum - x == y) && (sum - y == x);
}
```

`(sum - x == y) && (sum - y == x)`是一个永真式，因为补码加法的可逆性。接下来证明除法验证乘法的正确性。

首先假设$x,y$均是$w$位的补码数字，则$x\cdot y$则应为一个$2w$位的补码数字，接下来用有符号数$v$和无符号数$u$来分别表示其前$w$位和后$w$位，同时记乘法的实际结果为$p$，则相应的有（以下$x\cdot y$表示不限制位数的正常算术乘法）
$$
x\cdot y= v2^w+u
$$
进一步考虑$u=p+p_{w-1}2^w$（截断），其中$u=U2T_w(p)$，$p_{w-1}$是$p$的位向量中最高有效位（从0计数，也就是符号位），所以这里取$t=v+p_{w-1}$，有
$$
x\cdot y=t2^w+p
$$
所以当$t=0$时，$x\cdot y = p$ ，没溢出，当$t\ne 0$时，$x\cdot y \ne p$，溢出。

进一步考虑除法，根据带余除法有
$$
p=x\cdot q+r
$$
其中$|r|<|x|$。（这里加绝对值的缘故是考虑到有符号数的带余除法，如$-7=2*(-3)+(-1)$。所以余数应该限制在靠近零的范围里。）

综合以上两方面的考虑，自然的，要用除法来检验乘法就应该证明$q=y$当且仅当$t=s=0$。

1. 若$q=y$，那么有
$$
p=x\cdot y-t2^w=x\cdot y+r
$$
即
$$
t2^w+r=0
$$

但根据对$r$的限制，有
$$
|r|<|x|<2^w\le|t2^w|
$$
因此只有$r=t=0$才能满足方程。

2. 若$r=t=0$，由上两式可以很快得出

$$
p=x\cdot q=x\cdot y\Rightarrow q=y
$$

综合以上的证明方法，有相应的两种检验方法，截断检验和除法检验。

注：在使用函数`malloc() realloc() calloc()`等函数时，对其确定空间大小的参数理应作出一定的溢出检查。他们的参数类型都是`size_t`。

### 乘以2的幂

对于无符号数和有符号数都有`x<<k`等价于`x*2^k`。

### 除以2的幂

对于无符号数和补码而言，`x>>k`等价于$\lfloor x/2^k\rfloor$。

仅对于补码而言，`(x+(1<<k)-1)>>k`等价于$\lceil x/2^k\rceil$。

 ## 浮点数表示

十进制表示法
$$
d_md_{m-1}\ldots d_0.d_{-1}d_{-2}\ldots d_{-n}
$$
数值表示
$$
d=\sum_{i=-n}^{m}10^id_{i}
$$
如果采用二进制表示法，则上述一式不变，二式权值变为2。

IEEE浮点标准用$V=(-1)^s\times M\times 2^E$的形式来表示一个数：

- 符号$s$：用一个单独的符号位表示。
- 尾数$M$：用$n$位字段存储，二进制小数，依赖于阶码，范围在$0 \sim 1-\epsilon$或$1\sim 2-\epsilon$。
- 阶码$E$：用$k$位字段存储。

一般来说对于`float`，$k=8$，$n=23$，对于`double`，$k=11$，$n=52$，对于`long double`，$k=15$，$n=52$。

格式如下：

| 符号$s$ | 阶码$E$ | 尾数$M$ |
| ------- | ------- | ------- |
| 31      | 30-23   | 22-0    |
| 63      | 62-52   | 51-0    |

实际上存储的情况有三种：

1. 规格化的值

$exp\in [1,2^k-1],f\in[0,1),Bias=2^{k-1}-1$

$E=exp-Bias\in[2-2^{k-1},2^{k-1}],M=1+f\in[1,2)$

即阶码$E$不全为0同时也不全为1。

2. 非规格化的值

$exp=0,f\in[0,1),Bias=2^{k-1}-1$

$E=1-Bias=2-2^{k-1},M=f\in[0,1)$

即阶码$E$全为0。

3. 特殊值

无穷：$exp=2^k,f=0$，阶码$E$全为1同时尾数$M$全为0。

`NaN`：$exp=2^k,f\ne 0$，阶码$E$全为1同时尾数$M$不为0。

- 为什么浮点数阶码要有偏移量？

原因还是为了计算机处理数据的方便，还记得为什么计算机要有补码吗？原因就是希望在加法运算中将减法运算一并处理了，简化CPU中运算器的设计，通过补码实现了加减法的统一。现在将浮点数用这种形式保存，那么计算机怎么比较浮点数的大小呢？浮点数表示有两个符号位置，一个是数符S，一个是阶码的符号，如果仅仅采用补码作为阶码，由于阶码有正有负，整个数的符号位和阶数的符号位将导致不能进行简单的大小比较，所以阶数采用了一个无符号的正整数存储。阶数的值直接进行二进制计算，符号位置是默认为0的，于是阶数的值可以为0到255。

-  舍入策略

向偶数舍入、向零舍入、向上舍入、向下舍入。后三者用来定上界和下界，前者用来找最接近的匹配。

- 注意事项

浮点加法和乘法只具有交换性但不具有结合性，且乘法对加法不具有分配性，这一点会体现在程序的运算式优化上。

浮点数相同阶码间相邻数的间隔相同。