# Lec 5: Neural Networks

## Overview
前三讲已经建立了整个监督学习的基本骨架：线性分类器 → 损失函数 → 梯度下降优化。

但线性分类器有一个根本局限：它只能在像素空间做线性划分，面对复杂的类内变化往往无能为力。

本讲讨论两个问题：

- 在进入神经网络之前，能否先对输入特征做一些变换让线性模型更好用？（特征变换）
- 如果允许堆叠多层，我们能得到什么？（神经网络）

## Feature Transform
线性分类器的公式是：

$$
f(x, W) = Wx + b
$$

$x$ 直接是展平后的像素向量。像素本身并不是描述图像内容最好的表示方式。

一种缓解思路是先对输入 $x$ 做特征变换 $\phi(x)$，然后在变换后的空间里做线性分类：

$$
f(\phi(x), W) = W\phi(x) + b
$$

常见的人工特征：

- **颜色直方图**（Color Histogram）：统计图像中各颜色出现的频率，抛弃空间信息，但对颜色分布敏感
- **HOG（Histogram of Oriented Gradients）**：统计局部区域内边缘方向的分布，捕捉物体轮廓和形状，抛弃颜色信息
- **词袋模型（Bag of Words）**：将图像视为一组视觉"词"的集合

!!! info "特征变换的本质"
    特征变换把原始输入映射到一个更适合线性分类的空间。如果变换选得好，线性模型就能处理原本线性不可分的问题。

    但这些人工特征依赖领域专家设计，神经网络的思路是让模型自己学习好的特征表示。

## From Linear to Neural Networks
线性分类器的表达能力有限，最直接的扩展思路是：把多个线性变换堆叠起来，并在中间加入非线性激活。

一个两层神经网络可以写作：

$$
f = W_2 \max(0, W_1 x + b_1) + b_2
$$

其中：

- $W_1 \in \mathbb{R}^{H \times D}$，$b_1 \in \mathbb{R}^H$：第一层权重与偏置
- $W_2 \in \mathbb{R}^{C \times H}$，$b_2 \in \mathbb{R}^C$：第二层权重与偏置
- $H$ 是隐藏层大小（hidden size）
- $\max(0, \cdot)$ 是激活函数 ReLU

用变量名更清晰地写：

$$
h = \max(0, W_1 x + b_1), \quad s = W_2 h + b_2
$$

$h$ 是隐藏层（hidden layer），$s$ 是输出分数。

### 为什么需要激活函数
如果没有激活函数，两层线性变换等价于一层：

$$
W_2(W_1 x) = (W_2 W_1) x = W x
$$

无论叠多少层，结果仍然是线性分类器，表达能力没有提升。

引入非线性激活后，模型才能学习复杂的非线性决策边界。

## Activation Functions
常见激活函数如下：

### Sigmoid
$$
\sigma(x) = \frac{1}{1 + e^{-x}}
$$

- 输出范围：$(0, 1)$，可以解释为概率
- 缺点：当 $x$ 过大或过小时梯度接近 0（**梯度消失**）；输出不以 0 为中心

### Tanh
$$
\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}}
$$

- 输出范围：$(-1, 1)$，以 0 为中心
- 缺点：梯度消失问题依然存在

### ReLU
$$
\text{ReLU}(x) = \max(0, x)
$$

- 计算简单，收敛快
- 缺点：$x < 0$ 时梯度为 0，可能出现 **dead ReLU**（某些神经元永远不激活）

### Leaky ReLU
$$
\text{LeakyReLU}(x) = \max(\alpha x, x), \quad \alpha = 0.01
$$

- 解决 dead ReLU 问题，$x < 0$ 时仍有微小梯度

### ELU
$$
\text{ELU}(x) = \begin{cases} x & x \geq 0 \\ \alpha(e^x - 1) & x < 0 \end{cases}
$$

- 比 ReLU 更平滑，负半轴有限输出
- 计算略贵（指数运算）

!!! tip "实践建议"
    - 默认使用 ReLU
    - 如果出现大量 dead ReLU，尝试 Leaky ReLU
    - 不要用 Sigmoid 或 Tanh 作为中间层激活函数（梯度消失）

## Fully-Connected Neural Networks
将输入层、若干隐藏层、输出层串联，每层均为线性变换 + 激活函数，这样的结构叫做：

- 全连接网络（Fully-Connected Neural Network）
- 或多层感知机（Multi-Layer Perceptron, MLP）

一个 $L$ 层的 MLP 可以写成：

$$
h_0 = x, \quad h_l = f_l(W_l h_{l-1} + b_l), \quad l = 1, \dots, L-1, \quad s = W_L h_{L-1} + b_L
$$

其中 $f_l$ 是第 $l$ 层的激活函数，最后一层通常不加激活。

```python
import torch
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(3072, 100),
    nn.ReLU(),
    nn.Linear(100, 10),
)
```

### Template Matching 视角
$W_1$ 的每一行可以看作一个"模板"。与线性分类器不同，神经网络的隐藏层可以**为一个类别学习多个模板**，$W_2$ 再把它们组合成最终预测。

这种表示方式叫做**分布式表示（distributed representation）**。

## Universal Approximation
理论上，一个足够宽的两层神经网络可以逼近任意连续函数（万能逼近定理，Universal Approximation Theorem）。

直觉上：你可以把很多 ReLU 折线拼接起来逼近任何形状。

!!! warning "逼近能力 ≠ 具体可学习"
    万能逼近定理说的是"存在这样的网络"，但不保证用梯度下降能学出来，也不告诉你需要多宽。

    这个性质本身并没有 kNN 强（kNN 也能逼近任意函数），所以不能单靠它来证明神经网络好。

## Convexity and Non-Convex Optimization
理解神经网络优化的一个重要问题是：损失曲面是不是凸的？

对线性分类器配上 Softmax / SVM loss，可以证明优化是**凸问题**，局部最优即全局最优。

对任意层数的神经网络，**目前没有证明其损失曲面是凸的**。这意味着：

- 可能有很多局部极小值
- 梯度下降不保证找到全局最优
- 甚至不保证收敛

!!! info "实践中的现实"
    尽管理论保证缺失，实践中神经网络仍然能通过 SGD 和各种技巧找到足够好的解。

    很多局部最小值在损失值上接近全局最优，真正危险的反而是大量的**鞍点**（梯度为零但不是最优）。

## Biological Inspiration
神经网络的名称来自对生物神经元的粗浅类比：

- 神经元接收来自多个突触的输入信号，加权求和，再通过激活函数决定是否"激发"
- 人工神经元：$y = f(w^T x + b)$

这个类比帮助理解直觉，但不应该过分强调。现代深度学习的工程细节和生物神经科学差距很大。

## Depth vs. Width
拥有更多层的网络（deeper）和拥有更宽隐藏层的网络（wider）在理论上都能增加表达能力。

但在实践中：

- 更深的网络往往具有更强的**层级特征提取**能力，浅层学边缘、深层学语义
- 增加深度通常比增加宽度更参数高效
- 深层网络训练更难（梯度消失/爆炸），需要更多技巧辅助

## Neural Networks in Practice
用 PyTorch 定义和训练一个简单的 MLP：

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 定义网络
model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(3 * 32 * 32, 512),
    nn.ReLU(),
    nn.Linear(512, 256),
    nn.ReLU(),
    nn.Linear(256, 10),
)

optimizer = optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.CrossEntropyLoss()

# 训练一步
optimizer.zero_grad()
scores = model(X_batch)
loss = loss_fn(scores, y_batch)
loss.backward()
optimizer.step()
```

## Main Takeaways
- 人工特征变换可以在一定程度上改善线性模型
- 神经网络通过堆叠线性变换 + 非线性激活，获得比线性模型强得多的表达能力
- 激活函数的引入是神经网络非线性化的关键
- ReLU 是实践中最常用的激活函数
- 万能逼近定理保证了表达能力的上限，但不保证训练效果
- 神经网络优化问题通常是非凸的，但实践中往往仍能找到好的解

## Terms
!!! info "术语小结"
    - **MLP（Multi-Layer Perceptron）**：多层感知机，全连接神经网络
    - **Hidden Layer**：隐藏层，中间层
    - **Activation Function**：激活函数，为网络引入非线性
    - **ReLU**：$\max(0, x)$，最常用的激活函数
    - **Dead ReLU**：负输入时梯度为零、永远不更新的神经元
    - **Universal Approximation**：足够宽的两层网络可以逼近任意连续函数
    - **Distributed Representation**：分布式表示，多个神经元共同表达同一概念
    - **Depth**：网络层数
    - **Width**：每层神经元数量
