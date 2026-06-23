---
counter: True
---

# Block & Traces

!!! quote "参考笔记"

    本章可对照 HowJul 语雀笔记的 [第 8 章：基本块和轨迹](https://www.yuque.com/howjul/rt9ms6/bqf6xmz23er93kht)，以及 Cubic Y³ 的 [Part 14: 基本块与 traces](https://cubicy.icu/compiler-construction-principles/#Part-14-基本块与traces)。

由于中间代码与后端机器码之间往往存在一些 mismatch：比如虽然 IR Tree 很适合表达程序语义，但不适合直接变成机器码，因此我们需要对中间代码进行一些优化，使其变成更接近机器指令序列的形式

!!! note


    1. `CJUMP` 的问题

       Tree IR 里可能有：

       ```
       CJUMP(op, e1, e2, true_label, false_label)
       ```

       意思是：如果 `e1 op e2` 成立，跳到 `true_label`；否则跳到 `false_label`。

       但真实机器的条件跳转通常更像：

       ```
       if condition jump L_true
       // condition false 时自动执行下一条指令
       ```

       也就是说，机器码一般只有一个显式跳转目标，另一个分支靠 **fall through**，也就是“自然往下执行”。

       因此我们需要调整基本块顺序，让 IR 的 false 分支直接落到下一条 IR 指令处

    2.  `ESEQ` 的问题

       `ESEQ(s,e)` 主要表示顺序执行，表示先执行 s 产生 side-effect，再 evaluate e

       可见 ESEQ 在计算表达式时要考虑 s 可能产生副作用改变表达式的状态，因此在同一个树中，如果子表达式求值顺序不同，可能会产生不同的结果，这会让代码生成很麻烦，因为编译器通常希望可以自由安排子表达式的求值顺序，为此我们要尽可能消除 ESEQ 指令

       而 `SEQ` 本身不是语义上有问题，它只是说明“先执行这个，再执行那个 ”。但问题在于：机器码天然就是一条条线性指令，而 `SEQ` 是树状结构。

       所以后面要把很多嵌套的 `SEQ`：比如 SEQ(s1, SEQ(s2, SEQ(s3, s4))) 线性化成 s1 s2 s3 s4

    3. `CALL` 的问题

       `CALL` Node 也有和 ESEQ 一样类似的问题，因为函数调用也通常有副作用；此外如果用到寄存器传参，如果有嵌套的 CALL，可能会存在寄存器参数覆盖的问题

针对 IR Trees 这一种中间代码的优化，主要分为以下三步：

- A tree is rewritten into a list of **canonical trees** without SEQ or ESEQ nodes
- This list is grouped into a set of **basic blocks**, which contain no internal jumps or labels
- The basic blocks are ordered into a set of **traces** in which every CJUMP is immediately followed by its false label

本章的内容主要围绕这三步优化来展开

## Canonical Trees

To perform stage-one transform, we need to

- eliminate ESEQS （比较重要，好像经常考）
- move CALLs to top level
- eliminate SEQS
