# Softmax 

Softmax 函数是深度学习，尤其是在多分类问题中，一个至关重要的激活函数。它能够将一个包含任意实数的向量“压缩”成一个概率分布向量。

## 1. Softmax函数的原理

### 核心思想

在多分类任务中，神经网络的最后一层通常会为每个类别输出一个实数分数（score），这个分数也称为 logit。这些分数可正可负，大小不一，不方便直接作为概率来解释。

Softmax 函数的作用就是将这些原始分数转换成一个**概率分布**。这个分布具有两个关键特性：

1.  **非负性与归一性**：输出向量中的每个元素都在 (0, 1) 区间内。

2.  **总和为1**：输出向量的所有元素之和等于 1。

这样，输出向量的第 `k` 个元素就可以被解释为输入样本属于第 `k` 个类别的概率。

### 数学公式

假设我们有一个 n 维的输入向量 `a = (a_1, a_2, ..., a_n)`，这是全连接层的输出。经过 Softmax 函数处理后，我们得到输出向量 `y = (y_1, y_2, ..., y_n)`。其中，`y` 的第 `k` 个元素的计算公式为：

$$
y_k = \frac{e^{a_k}}{\sum_{j=1}^{n} e^{a_j}}
$$

这个公式分为两步：
1.  **分子 `e^{a_k}`**：使用指数函数 `exp()` 将所有输入分数转换为正数。指数函数还有一个很好的特性，就是它会放大输入之间的差异，使得分数高的元素在输出概率中占据更大的比重。

2.  **分母 `Σ e^{a_j}`**：将所有指数化后的分数相加，作为一个归一化常数。用分子除以这个常数，就保证了所有输出 `y_k` 的总和为 1。

### 实现中的注意事项：数值稳定性

在实际编程中，如果输入 `a_k` 的值非常大，`e^{a_k}` 可能会导致数值溢出（infinity）。为了解决这个问题，通常会在分子分母上同时乘以一个常数 `C`，这不会改变最终结果。一个常用的技巧是令 `C = e^{-max(a)}`，即：

$$
y_k = \frac{e^{a_k - C'}}{\sum_{j=1}^{n} e^{a_j - C'}} \quad \text{其中 } C' = \max(a_1, a_2, ..., a_n)
$$

这样可以保证指数函数的输入最大为0，有效避免了溢出问题。

## 2. Softmax函数的应用

Softmax 函数最核心的应用场景是**多分类 (Multi-class Classification)** 任务的输出层。

在一个典型的分类神经网络中：

1.  输入数据（如图片、文本）经过一系列的隐藏层（如卷积层、全连接层）进行特征提取。

2.  网络的最后一层是一个全连接层，它为每个类别输出一个 logit 分数。例如，对于一个10分类的数字识别任务，这一层的输出是一个10维向量。

3.  这个10维的 logit 向量被送入 Softmax 函数，转换成一个10维的概率向量。例如 `[0.01, 0.03, 0.85, ..., 0.02]`，这可以被解释为模型认为输入图片是数字“2”的概率为85%。

4.  这个概率向量 `y` 随后被送入损失函数（通常是交叉熵损失函数）中，与真实标签 `t` 进行比较，以计算损失。

## 3. 核心推导：Softmax与交叉熵损失的反向传播

这部分是理解分类网络如何学习的关键。我们将详细推导当损失函数为**交叉熵损失**时，损失 `L` 对 Softmax 层输入 `a` 的偏导数 `∂L/∂a`。

### 3.1 设定与目标

* **Softmax 函数**: $y_k = \frac{e^{a_k}}{\sum_{j} e^{a_j}}$
* **交叉熵损失**: $L = - \sum_{k} t_k \log(y_k)$
    * `t = (t_1, ..., t_n)` 是真实标签的 one-hot 编码向量。
* **目标**: 计算梯度 $\frac{\partial L}{\partial a_i}$。

### 3.2 链式法则的应用

损失 `L` 通过中间变量 `y` 间接依赖于 `a`。根据多元链式法则，我们有：

$$
\frac{\partial L}{\partial a_i} = \sum_{k=1}^{n} \frac{\partial L}{\partial y_k} \frac{\partial y_k}{\partial a_i}
$$

我们需要分别计算 `∂L/∂y_k` 和 `∂y_k/∂a_i`。

### 3.3 计算组件偏导数

#### 损失对Softmax输出的偏导 ($\partial L / \partial y_k$)
对损失函数 $L = - \sum_{j} t_j \log(y_j)$ 求关于 $y_k$ 的偏导，只有求和项中 `j=k` 的部分与 $y_k$ 相关：

$$
\frac{\partial L}{\partial y_k} = \frac{\partial}{\partial y_k} (-t_k \log(y_k)) = -t_k \frac{1}{y_k} = -\frac{t_k}{y_k}
$$

#### Softmax输出对输入的偏导 ($\partial y_k / \partial a_i$)
这部分需要分两种情况讨论：

* **情况 1: `i = k` (对角线元素)**
    使用除法求导法则对 $y_i = \frac{e^{a_i}}{\sum_{j} e^{a_j}}$ 求关于 $a_i$ 的偏导：

    $$
    \begin{aligned}
    \frac{\partial y_i}{\partial a_i} &= \frac{ (e^{a_i})'(\sum_{j}e^{a_j}) - e^{a_i}(\sum_{j}e^{a_j})' }{(\sum_{j}e^{a_j})^2} \\
    &= \frac{ e^{a_i}(\sum_{j}e^{a_j}) - e^{a_i}(e^{a_i}) }{(\sum_{j}e^{a_j})^2} \\
    &= \frac{e^{a_i}}{\sum_{j}e^{a_j}} - \left(\frac{e^{a_i}}{\sum_{j}e^{a_j}}\right)^2 \\
    &= y_i - y_i^2 = y_i(1 - y_i)
    \end{aligned}
    $$

* **情况 2: `i ≠ k` (非对角线元素)**
    对 $y_k = \frac{e^{a_k}}{\sum_{j} e^{a_j}}$ 求关于 $a_i$ 的偏导。此时分子 $e^{a_k}$ 是常数。

    $$
    \begin{aligned}
    \frac{\partial y_k}{\partial a_i} &= e^{a_k} \cdot \left( -1 \cdot (\sum_{j}e^{a_j})^{-2} \cdot (e^{a_i}) \right) \\
    &= - \frac{e^{a_k} e^{a_i}}{(\sum_{j}e^{a_j})^2} \\
    &= - \left(\frac{e^{a_k}}{\sum_{j}e^{a_j}}\right) \left(\frac{e^{a_i}}{\sum_{j}e^{a_j}}\right) = -y_k y_i
    \end{aligned}
    $$

### 3.4 组合与化简

现在，我们将这些结果代入链式法则的求和公式中。我们将求和拆分为 `k=i` 和 `k≠i` 两部分：

$$
\frac{\partial L}{\partial a_i} = \frac{\partial L}{\partial y_i}\frac{\partial y_i}{\partial a_i} + \sum_{k \neq i} \frac{\partial L}{\partial y_k}\frac{\partial y_k}{\partial a_i}
$$

代入我们求得的四个结果：

$$
= \left(-\frac{t_i}{y_i}\right) \cdot (y_i(1 - y_i)) + \sum_{k \neq i} \left(-\frac{t_k}{y_k}\right) \cdot (-y_k y_i)
$$

进行化简：

$$
= -t_i(1 - y_i) + \sum_{k \neq i} t_k y_i
$$

$$
= -t_i + t_i y_i + y_i \sum_{k \neq i} t_k
$$

将 $t_i y_i$ 移入求和项中：

$$
= -t_i + y_i \left( t_i + \sum_{k \neq i} t_k \right)
$$

$$
= -t_i + y_i \sum_{k=1}^{n} t_k
$$

由于 `t` 是 one-hot 向量，其所有元素之和 $\sum_{k=1}^{n} t_k$ 必然等于 1。因此：

$$
= -t_i + y_i \cdot 1
$$

最终我们得到了一个极其简洁的结果：
$$
\frac{\partial L}{\partial a_i} = y_i - t_i
$$

### 3.5 结论与直觉

这个推导的最终结果非常优美。对于 "Softmax-with-Cross-Entropy-Loss" 这一整个计算层，它反向传播的梯度就是：

$$
\frac{\partial L}{\partial a} = y - t
$$

这个结果非常直观，其含义是 **梯度 = 预测概率 - 真实概率**。

-   这个差值直接反映了模型预测的偏差。
-   如果模型对正确类别的预测概率 `y_i` 接近1，那么 `y_i - t_i` (即 `y_i - 1`) 接近0，梯度很小，参数更新也很小。
-   如果预测概率 `y_i` 很低，梯度 `y_i - 1` 就是一个较大的负数，这会促使参数向着能增大 `a_i` 的方向更新，从而提高正确率。

正是因为这个数学上的完美简化，**Softmax** 和 **交叉熵损失** 才成为了多分类任务中不可分割的“黄金搭档”。