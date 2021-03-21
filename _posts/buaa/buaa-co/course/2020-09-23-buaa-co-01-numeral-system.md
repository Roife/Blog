---
layout: "post"
title: "「BUAA-CO」 01 数制"
subtitle: "数制，数字存储以及二进制的四则运算"
author: "roife"
date: 2020-09-23

tags: ["BUAA - 计算机组成@Courses@Series", "Digital Design and Computer Architecture@Books@Series", "Computer Organization and Design@Books@Series", "北航@Tags@Tags", "计算机组成@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 数制

$$
(b_{n - 1}b_{n - 2}\dots b_1 b_0)_b = \sum_{i = 0}^{n - 1} b_i \times b^i。
$$

## Bin, Oct, Hex 便捷转换

- Bin 转为 Hex:
  从低位开始，以 **四位** 为一组，分别转化后拼接
- Hex 转为 Bin:
  将 Hex 中的一位展开成四位 Bin 后拼接

- Bin 转为 Oct:
  从低位开始，以 **三位** 为一组，分别转化后拼接
- Oct 转为 Bin
  将 Oct 中的一位展开成三位 Bin 后拼接

# Byte, Nibble, Word

- Byte: 8 bit (Hex 中 2 位, `0x00 ~ 0xFF`)
- Nibble: 4 bit
- Word: 取决于处理器架构, 一般为 32 bit (Hex 中 8 位, `0x12345678`) 或者 64 bit

最低位称为 least significant bit (lsb), 最高位称为 most significant bit (msb).
类似有 Least Significant Byte (LSB) 和 Most Significant Byte (MSB).

# 原码, 反码, 补码

- 原码: 第一位作为符号位

  $$
    [X]_{\text{原}} = \begin{cases} X, & 0 \le X \le 2^{n - 1} - 1\\ 2^{n - 1} - X, & -2^{n - 1} + 1 \le X \le 0 \end{cases}
  $$

  缺点：`0` 有两种表示方式, 符号位处理复杂.

  浮点数的有效数字表示会用原码.

- 反码: 正数不变, 负数为其绝对值取反 (相当于原码除符号位取反).

  $$
  [X]_{\text{反}} = \begin{cases} X, & 0 \le X \le 2^{n - 1} - 1\\ 2^n - 1 + X, & -2^{n - 1} + 1 \le X \le 0 \end{cases}
  $$

  缺点：`0` 有两种表示方式, 需要[循环进位](https://en.wikipedia.org/wiki/Signed_number_representations#Ones.27_complement).

- 补码: 正数不变, 负数为反码 + 1

  $$
  [X]_{\text{补}} = \begin{cases} X, & 0 \le X \le 2^{n - 1} - 1\\ 2^{n} + X, & -2^{n - 1} \le X \le 0 \end{cases}。
  $$

  > 一个 $n$ 位的补码实际上是一个数在模 $2^n$ 意义上的表示.

# 浮点数

浮点数分为三部分: 尾数 (mantissa), 基数 (base), 指数 (exponent). 浮点数表示为 `±M * B^E`.

- 单精度浮点数(32 位, 精度 7 位) = 符号(1) + E(8) + M(23) + B(默认为 2, 省略).
- 双精度浮点数(64 位, 精度 15 位) = 符号(1) + E(11) + M(52) + B(默认为 2, 省略).

## 存储优化

浮点数存储时涉及到两个优化:

1. 由于科学计数法中, M 的首位一定为 `1`, 所以可以省略这一位.
  - `0 | 00001100 | 10100011011100000000000`
  - `0 | 00001100 | 01000110111000000000000` (`5.23*10^3`)

2. 存储负指数浮点数时, 指数部分 `1` 开头不好比较, 所以 `编码指数 = 实际指数 + 127 (b0111111)`. 即偏阶记数法.（双精度为 `1023`）
  - `0 | 01111110 | 00000000000000000000000`
  - `0 | 10000000 | 00000000000000000000000` (`2.0`)

浮点数计算前需要恢复这两个优化.

## 浮点数分类

| 数字         | 符号 | E        | M                       |
|--------------|------|----------|-------------------------|
| 0            | X    | 00000000 | 00000000000000000000000 |
| $\infty$     | 0    | 11111111 | 00000000000000000000000 |
| $-\infty$    | 1    | 11111111 | 00000000000000000000000 |
| NaN          | X    | 11111111 | 非零                    |
| 规格化浮点数   | X    | 非全零 & 非全一 | 非零                    |
| 非规格化浮点数 | X    | 00000000 | 非零                    |

- 规格化浮点数（Normalized Number）即最常见的浮点数。
- 非规格化浮点数（Denormalized Number）用于表示特别小的数字，比如 `1.23e-130` 可以表示为 `0.000123e-126`，此时 `E` 全为 `0`。

对于非规格化浮点数，不使用第一个优化（即恢复时也不用在前面加上 `1`）。

可以看出 IEEE 754 标准中有两个 0.

# 运算

## 加减乘除

- 减法: 先求补码再相加.

- 乘法: 无符号直接相乘. 带符号数相乘先算绝对值, 然后判断符号位.

- 除法: 试商只要试 `0` 和 `1`, 中间余数不断 `<<` 与 `|` 即可. 对于符号数, 先算绝对值再决定符号. 同号则商为正, 否则商为负. 余数的符号和被除数相同.

### 溢出检查
1. 当最高两位的进位不同 (即 $\text{carry}\_{\text{MSB}} \oplus \text{carry}_{\text{MSB} - 1} = 1$) 时, 就发生了溢出.
2. 加数符号位相同, 结果符号位改变.
3. 使用双符号位. 运算结果若为 `01` 则正溢; 若为 `10` 则负溢; 若为 `00` 或 `11` 未溢出.

## 位扩展

符号扩展: 复制符号位 (无符号用 `0` 扩展).

## 比较

`!=` 可以用 `==` 实现, 其他运算均可以用 `<` 实现, 如 `A >= B` 即 `not (A < B)`.

- `<`: 可以借助减法, 两个数字相减后比较符号位. 注意溢出的情况 (如 `b0000-b1000`), 因此在相减前需要先扩展一位 (带符号扩展符号位, 无符号扩展 `0`).
- `=`: 可以用 xor 或者减法.

# 参考资料

1. *Digital Design and Computer Architecture 2nd*, Chapter 1.4
2. *Digital Design and Computer Architecture 2nd*, Chapter 5.3