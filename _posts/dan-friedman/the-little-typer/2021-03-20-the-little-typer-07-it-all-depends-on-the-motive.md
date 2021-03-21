---
layout: "post"
title: "「The Little Typer」 07 It all depends on the Motive"
subtitle: "DT 与 motive"
author: "roife"
date: 2021-03-20

tags: ["The Little Typer@Books@Series", "程序语言理论@Tags@Tags", "函数式编程@Tags@Tags", "Dependent Type@Tags@Tags", "Dan Friedman@Series@Series", "Pie@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

# DT

> **Dependent Types**
>
> A type that is determined by something that is not a type is called a *dependent type*.

- `(pears Nat)`：传入一个 `Nat`，返回指定数量个 `'pea`

```lisp
(claim peas
  (Π ((how-many-peas Nat))
    (Vec Atom how-many-peas)))
```

然而此处如果使用 `rec-Nat` 定义 `peas` 会有类型的问题。`(rec-Nat cond base step)` 要求 `base` 的类型和 `step` 的类型相同，但是这里使用了 DT 后就不再符合条件，所以要引入新的递归方式，即 `ind-Nat`。

```lisp
// Wrong Definition with rec-Nat
(define peas
  (λ (how-many-peas)
    (rec-Nat how-many-peas
      vecnil
      (λ (l-1 peas_l-1)
        (vec:: 'pea peas_l-1)))))
```

# `ind-Nat`

`ind-Nat` 和 `rec-Nat` 相似，但是它允许 `base` 和 `step` 的返回结果不同，因此`ind-Nat` 专门用于 DT，例如 `(Vec Atom how-many-peas)`。

但是使用 `ind-Nat` 时需要一个额外的参数，用来描述 `base` 的 `step` 的结果是怎样依赖于 `Nat` 的，或者说如何利用 `Nat` 构建出这个类型。这个参数被称为 **motive**，类型为 `(→ Nat U)`（注意返回类型为 `U`）。

一个 `int-Nat` 表达式的类型即为将 Nat 传入 motive 后返回的结果。

> **Use ind-Nat for Dependent Types**
>
> Use ind-Nat instead of rec-Nat when the rec-Nat- or ind-Nat-expression’s type depends on the target Nat. The ind-Nat-expression’s type is the motive applied to the target.

```lisp
(claim mot-peas
  (→ Nat
    U))

(define mot-peas
  (λ (k)
    (Vec Atom k)))
;; (mot-peas zero)，返回 U，即 (Vec Atom zero)
```

显然，`peas` 的 `base`（即 `vecnil`）应该是 `(mot-peas zero)`。

下面考虑 `step` 的类型：接受一个 `n-1` 和一个 almost-answer。almost-answer 的类型则是 `(mot n-1)`。则 `step` 的类型如下：

```lisp
(Π ((n-1 Nat))
  (→ (mot n-1)
    (mot (add1 n-1))))
```

> **The Law of ind-Nat**
>
> If `target` is a `Nat`, `mot` is an
>
> ```lisp
> (→ Nat
>   U)
> ```
>
> `base` is a `(mot zero)`, and `step` is a
>
> ```lisp
> (Π ((n-1 Nat))
>   (→ (mot n-1)
>     (mot (add1 n-1))))
> ```
>
> then
>
> ```lisp
> (ind-Nat target
>   mot
>   base
>   step)
> ```
>
> is a `(mot target)`.

> **The First Commandment of ind-Nat**
>
> The ind-Nat-expression
>
> ```lisp
> (ind-Nat zero
>   mot
>   base
>   step)
> ```
>
> is the same `(mot zero)` as `base`.

> **The Second Commandment of ind-Nat**
>
> The ind-Nat-expression
>
> ```lisp
> (ind-Nat (add1 n)
>   mot
>   base
>   step)
> ```
>
> and
>
> ```lisp
> (step n
>   (ind-Nat n
>     mot
>     base
>     step))
> ```
> are the same `(mot (add1 n))`.

## Induction

```lisp
(claim step-peas
  (Π ((l-1 Nat))
    (→ (mot-peas l-1)
      (mot-peas (add1 l-1)))))

(define step-peas
  (λ (l-1)
    (λ (peas_l-1)
      (vec:: 'pea peas_l-1))))
```

这里 `step-peas` 的类型声明中出现了两次 `mot-peas`，创建的类型分别为 `(Vec Atom l-1)` 和 `(Vec Atom (add1 l-1))`，后者是由前者构造的，这是一个数学归纳的过程。

> **Induction on Natural Numbers**
>
> Building a value for any natural number by giving a value for zero and a way to transform a value for `n` into a value for `n + 1` is called induction on natural numbers.

```lisp
(define peas
  (λ (how-many-peas)
    (ind-Nat how-many-peas
      mot-peas ; motive
      vecnil
      step-peas)))
```

## `rec-Nat` & `ind-Nat`

`rec-Nat` 可以用 `ind-Nat` 定义，只要忽略掉 motive 的参数，直接返回指定的类型就行了。

```lisp
(claim also-rec-Nat
  (Π ((X U))
    (→ Nat
        X
        (→ Nat X
          X)
      X)))

(define also-rec-Nat
  (λ (X)
    (λ (target base step)
      (ind-Nat target
        (λ (k)
          X)
        base
        step))))
```

# `last`

- `(last U l es)`：返回列表中最后一个元素

首先考虑 `last` 的类型：

```lisp
(claim last
  (Π ((E U)
      (l Nat))
    (→ (Vec E (add1 l))
      E)))
```

`last` 总共传入三个参数，前两个是类型，最后一个是数据。下面对于 `last` 的定义实际上定义了一个组合子，传入前两个类型参数，返回的是一个函数（而不是具体的值）。得到函数后，需要再传入参数利用 curry-ing 计算出结果。

## `base`

考虑 `last` 中 `base` 的写法。（注意，`base` 也写成了一个函数）

```lisp
(claim base-last
  (Π ((E U))
    (→ (Vec E (add1 zero))
      E)))

(define base-last
  (λ (E)
    (λ (es)
      (head es))))
```

如果 `base` 与 `step` 的 almost-answer 都是一个函数，那么 `ind-Nat` 也会返回一个函数。实际上函数和值没有区别，函数只是一个用 λ 作为 constructor 的数据。

> Are λ-expressions values?
>
> Yes, because λ is a constructor.
>
> Functions are indeed values.

`base` 的类型就是 motive 传入 `zero` 的结果。

> **`ind-Nat`’s Base Type**
>
> In `ind-Nat`, the `base`’s type is the motive applied to the target `zero`.

## motive

motive 反映了 `ind-Nat` 的类型！

```lisp
(claim mot-last
  (→ U Nat
    U))

(define mot-last
  (λ (E k)
    (→ (Vec E (add1 k))
      E)))

; 注意不要用 Π 表达式，因为 motive 是一个返回类型的函数，会被 applied，而不是本身是一个类型
;; 错误写法
; (Π ((E U)
;     (k Nat))
;   (→ (Vec E (add1 k))
;     E))
```

## `step`

`step` 会传入对于 `l-1` 的结果并返回对于 `(add1 l-1)` 的结果。在这里，即将一个返回值为 `(Vec E (add1 l-1))` 的函数变为一个返回值为 `(Vec E (add1 (add1 l-1)))` 的函数。

这里的两个 `add1` 有不同的含义。内层的 `add1` 是根据 `ind-Nat` 的规则传入的参数 `k`；而外层的 `add1` 目的在于确保列表至少有一个元素（这个 `add1` 来自于 motive 的函数体），保证 totoality。

所以 `step` 的类型为：

```lisp
(→ (→ (Vec E (add1 l-1)) ; add1 表示列表长度至少为 1，来自 motive
    E)
  (→ (Vec E (add1 (add1 l-1))) ; 内层来自于传入的参数 k，外层来自于 motive
    E))
```

> **`ind-Nat`’s Step Type**
>
> In `ind-Nat`, the `step` must take two arguments: some `Nat n` and an almost-answer whose type is the motive applied to `n`. The type of the answer from the `step` is the motive applied to `(add1 n)`. The `step`’s type is:
>
> ```lisp
> (Π ((n Nat))
>   (→ (mot n)
>     (mot (add1 n))))
> ```

```lisp
(claim step-last
  (Π ((E U)
      (l-1 Nat))
    (→ (mot-last E l-1)
      (mot-last E (add1 l-1)))))

(define step-last
  (λ (E l-1)
    (λ (last_l-1) ; last_l-1 是函数，类型为 (mot-last E l-1)
      (λ (es) ; es 类型为 (Vec E (add1 (add1 l-1)))
        (last_l-1 (tail es)))))) ; 整个 λ 表达式类型为 (mot-last E (add1 l-1))
; (tail es) 类型为 (Vec E (add1 l-1))
; 将传入的列表去掉头部，然后递归处理
```

## `last`

```lisp
(define last
  (λ (E l)
    (ind-Nat l
      (mot-last E)
      (base-last E)
      (step-last E))))
; 注意这里 `last` 的定义只有两个类型参数
```

可以发现，虽然这定义了一个组合子，但是 `base` 和 `step-last` 都有 `es` 这个参数。

# `drop-last`

- `drop-last`：去掉列表最后一个元素

```lisp
(claim drop-last
  (Π ((E U)
      (l Nat))
    (→ (Vec E (add1 l))
      (Vec E l))))

(claim base-drop-last
  (Π ((E U))
    (→ (Vec E (add1 zero))
      (Vec E zero))))

(define base-drop-last
  (λ (E)
    (λ (es)
      vecnil)))

(claim mot-drop-last
  (→ U Nat
    U))

(define mot-drop-last
  (λ (E k)
    (→ (Vec E (add1 k))
      (Vec E k))))

(claim step-drop-last
  (Π ((E U)
      (l-1 Nat))
    (→ (mot-last E l-1)
      (mot-last E (add1 l-1)))))

(define step-drop-last
  (λ (E l-1)
    (λ (drop-last_l-1)
      (λ (es)
        (vec:: (head es)
          (drop-last_l-1 (tail es)))))))

(define drop-last
  (λ (E l)
    (ind-Nat l
      (mot-drop-last E)
      (base-drop-last E)
      (step-drop-last E))))
```

## 化简过程

注意，这都是组合子运算。

1.
```lisp
(drop-last Atom
  (add1
    (add1 zero)))
```

2.
```lisp
(ind-Nat (add1
          (add1 zero))
  (mot-drop-last Atom)
  (base-drop-last Atom)
  (step-drop-last Atom))
```

3.
```lisp
(step-drop-last Atom (add1 zero)
  (ind-Nat (add1 zero)
    (mot-drop-last Atom)
    (base-drop-last Atom)
    (step-drop-last Atom)))
```

4.
```lisp
(λ (es)
  (vec:: (head es)
    ((ind-Nat (add1 zero)
      (mot-drop-last Atom)
      (base-drop-last Atom)
      (step-drop-last Atom))
    (tail es))))
```

5.
```lisp
(λ (es)
  (vec:: (head es)
    ((λ (ys)
      (vec:: (head ys)
        ((ind-Nat zero
          (mot-drop-last Atom)
          (base-drop-last Atom)
          (step-drop-last Atom))
        (tail ys)))) ; 防止名字冲突
      (tail es))))
```

6.
```lisp
(λ (es)
  (vec:: (head es)
    (vec:: (head (tail es))
    (base-drop-last Atom
      (tail (tail es))))))
```

7.
```lisp
(λ (es)
  (vec:: (head es)
    (vec:: (head (tail es))
      vecnil)))
```