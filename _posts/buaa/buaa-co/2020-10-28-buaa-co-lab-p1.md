---
layout: "post"
title: "「BUAA-CO Lab」 P1 Verilog 模块及状态机 (unfinished)"
subtitle: "Verilog 模块设计"
author: "roife"
date: 2020-10-25

tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "L「Verilog-HDL」", "BUAA", "计算机组成", "unfinished"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 课下总结

## `$signed()`

这里写一下关于 `$signed()` 的理解.

`$signed()` 的真正功能是决定数据如何进行补位. 一个表达式 (特别注意三目运算符) 中如果存在一个无符号数, 那么整个表达式都会被当作无符号数.

### signedness

- self-determined expression
  : 指一个位宽可以由该表达式本身独立决定的expression。
- context-determined expression
  : 指一个位宽由其本身以及其所属的表达式共同决定的expression (例如一个阻塞赋值语句右侧表达式的位宽同时受左右两侧位宽的影响).

Verilog 在计算表达式前, 会先计算表达式的 signedness. 计算方式如下:
- 仅由操作数决定，不由左侧决定 (如 `assign D = exp`，`exp` 符号与 `D` 无关. 这一点区别于位宽, 位宽由左右两侧所有表达式的最大位宽决定)
- 小数是有符号的, 显式声明了进制的数无符号, 除非用修饰符s声明了其有符号 (如 `'d12` 无符号，`'sd12` 有符号
- 位选/多位选择/位拼接的结果无符号 (如 `b[2]`, `b[3:4]`, `{b}` 均无符号，这也是隔壁bit extender有同学用三目运算符结果WA了的原因)
- 比较表达式的结果无符号 (要么是 `0`, 要么是 `1`)
- 由实数强转成整型的表达式有符号
- 一个 self-determined expression 的符号性仅取决与其操作数
- 对于context-determined expression, 只有所有操作数均为有符号数时表达式才有符号

在计算表达式时, 先由以上规则得出最外层表达式的符号性, 再向表达式里的操作数递归传递符号性.

`$signed()` 函数的机制是计算传入的表达式, 返回一个与原表达式的值和位宽完全相同的值, 并将其符号性设为有符号. 该函数可以屏蔽外部表达式的符号性传递.

另外就是算术右移运算符 (`>>>`) 的有操作数不影响结果的符号, 所以没必要加上 `$signed()`.

### 一些例子

下面是一些例子:
- `assign C = (ALUOp==3'b101) ? $signed(A)>>>B : $signed(A+B);` 正确
- `assign C = (ALUOp==3'b101) ? $signed(A)>>>B : A+B;` 错误
- `assign C = (ALUOp==3'b101) ? $signed(A)>>>B : 0;` 正确 (`0` 被看作有符号数)
- `assign C = (ALUOp==3'b101) ? $signed(A)>>>B : 32'b0;` 错误 (显式声明了进制, 被当成无符号数)

特别的, 还有:

- `assign C = (ALUOp==3'b101) ? $signed(signed(A)>>>B) : $signed(A+B);` 正确

因为外层推导出是无符号, 但是外面的 `$signed` 阻止了 unsignedness 向内层转移.

## 位扩展

`$signed()` 用法太玄妙了, 不如直接用位扩展替代.

<!-- {%raw%} -->
```verilog
{{16{imm[15]}}, imm} // 将 [15:0] 的 imm 扩展到 32 位
```
<!-- {%endraw%} -->

## testbench 的优雅写法

```verilog
```

## 字符串识别状态机的方便写法 (待填坑)