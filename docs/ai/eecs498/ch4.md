# Lec 4: Optimization

## Overview
上一讲我们已经有了模型和损失函数：

- 用线性分类器输出类别分数
- 用 SVM loss 或 Softmax loss 衡量参数好坏
- 用正则化控制模型复杂度

但还缺最关键的一步：

已知损失函数 $L(W)$，怎样找到一组让损失尽可能小的参数 $W$？

这就是最优化（Optimization）问题。

从形式上看，我们要解决的是：

$$
W^* = \arg\min_W L(W)
$$

第四讲主要讨论：

- 为什么“瞎猜参数”不可行
- 梯度为什么能提供下降方向
- 如何用梯度下降迭代优化参数
- 为什么要用随机梯度下降（SGD）
- SGD 为什么还不够，以及 momentum / RMSProp / Adam 等改进方法

## Optimization as Searching
最直接的思路是随机搜索：

1. 随机初始化一组参数
2. 计算损失
3. 再随机改一点参数
4. 如果损失变小就接受，否则丢弃

这种方法理论上能用，但几乎没有实用价值。因为参数空间通常极其巨大。

例如在线性分类器里，若输入是 CIFAR-10 图像，则参数维度大约是：

$$
3072 \times 10 = 30720
$$

这还只是一个很浅的模型。对于神经网络，参数量通常是百万级、千万级甚至更多。

所以我们需要利用损失函数的局部结构，而不是盲目试探。

## Gradient
对多元函数来说，梯度（gradient）给出函数在当前位置增长最快的方向。

若损失函数为 $L(W)$，则：

$$
\nabla_W L
$$

表示损失对参数各维度的偏导数组成的向量。

因为梯度指向最陡上升方向，所以：

- 沿着梯度方向移动，损失通常会变大
- 沿着负梯度方向移动，损失通常会变小

这就是梯度下降（Gradient Descent）的核心直觉。

!!! info "梯度的直观理解"
    你可以把损失函数想成一片山地地形图，当前位置的梯度就是“最陡上坡方向”。

    如果我们想走到低谷，就应该朝反方向走。

## Numerical Gradient
如果暂时不会手推导数，可以先用数值方法近似梯度。

对于一维函数 $f(x)$，导数可以近似写成：

$$
f'(x) \approx \frac{f(x+h) - f(x-h)}{2h}
$$

这叫做 centered difference，比单边差分更稳定。

对于高维参数向量 $W$，我们可以对每个维度分别做这样的微小扰动，估计每个偏导数。

```python
def eval_numerical_gradient(f, x, h=1e-5):
    grad = np.zeros_like(x)
    it = np.nditer(x, flags=['multi_index'], op_flags=['readwrite'])

    while not it.finished:
        idx = it.multi_index
        old_value = x[idx]

        x[idx] = old_value + h
        fxph = f(x)
        x[idx] = old_value - h
        fxmh = f(x)
        x[idx] = old_value

        grad[idx] = (fxph - fxmh) / (2 * h)
        it.iternext()

    return grad
```

### Numerical Gradient 的用途
- 适合建立直觉
- 适合检查你手写梯度是否正确
- 不适合真正训练模型

原因是它太慢了。若参数有 $D$ 维，那么一次梯度计算通常需要约 $2D$ 次前向计算。

## Analytic Gradient
真正训练时，我们希望直接推导梯度公式，也就是解析梯度（analytic gradient）。

后面的反向传播（Lecture 6）本质上就是一种高效计算解析梯度的方法。

所以第四讲可以先把它理解成：

- Numerical gradient 用于验证
- Analytic gradient 用于训练

## Gradient Descent
有了梯度以后，最基本的更新规则是：

$$
W \leftarrow W - \eta \nabla_W L
$$

其中：

- $\eta$ 是学习率（learning rate）
- $\nabla_W L$ 是当前参数处的梯度

学习率控制每次更新走多远。

```python
w = initialize_weights()
for step in range(num_steps):
    dw = compute_gradient(loss_fn, data, w)
    w -= learning_rate * dw
```

### Learning Rate
学习率是最重要的超参数之一。

- 太小：每一步都走得很慢，收敛极慢
- 太大：容易来回震荡，甚至直接发散

所以优化的难点并不只是“知道往哪走”，还包括“每次走多远”。

## Full Batch Gradient Descent
如果数据集有 $N$ 个样本，总损失通常写成：

$$
L(W) = \frac{1}{N} \sum_{i=1}^{N} L_i(W) + \lambda R(W)
$$

对应梯度为：

$$
\nabla_W L(W) = \frac{1}{N} \sum_{i=1}^{N} \nabla_W L_i(W) + \lambda \nabla_W R(W)
$$

这意味着每次更新前，都需要对整个训练集做一遍前向和反向计算。

如果数据集很大，这会非常昂贵。

## Stochastic Gradient Descent
因此实际中更常用随机梯度下降（Stochastic Gradient Descent, SGD）。

它并不是每次都看全部数据，而是每次随机采样一个小批量（mini-batch）来近似整体梯度：

$$
\nabla_W L(W) \approx \frac{1}{B} \sum_{i \in \mathcal{B}} \nabla_W L_i(W) + \lambda \nabla_W R(W)
$$

其中：

- $\mathcal{B}$ 是当前 batch
- $B$ 是 batch size

对应伪代码：

```python
w = initialize_weights()
for step in range(num_steps):
    X_batch, y_batch = sample_batch(X_train, y_train, batch_size)
    dw = compute_gradient(loss_fn, X_batch, y_batch, w)
    w -= learning_rate * dw
```

### 为什么 SGD 更常用
- 每次更新更快
- 可以更频繁地更新参数
- 占用内存更少
- 对大数据集更现实

虽然每次梯度更 noisy，但总体上通常足够有效。

## Problems of SGD
SGD 很重要，但它并不完美。

### Zig-Zag Problem
如果损失曲面在某个方向很陡，在另一个方向很平缓，那么 SGD 容易出现锯齿形震荡。

直观上看：

- 在陡峭方向上容易来回摆动
- 在平坦方向上推进很慢

这会严重降低收敛速度。

### Saddle Point
高维优化里，鞍点（saddle point）比局部极小值更常见。

鞍点的特点是：

- 某些方向上像山谷
- 某些方向上像山峰
- 梯度可能很小，但并不是真正的最优点

SGD 在这些区域里容易停滞或推进很慢。

### Noisy Gradient
由于 batch 只是全数据的一小部分，估计出的梯度含有噪声。

这既有坏处，也有好处：

- 坏处：路径抖动、不稳定
- 好处：有时能帮助逃离某些糟糕区域

## Momentum
为了减少来回震荡、提升收敛速度，我们可以给 SGD 加上动量（momentum）。

直觉上，把参数更新想成一个带惯性的球：

- 如果一直朝同一方向下坡，就越滚越快
- 如果某个方向来回震荡，更新会被平滑掉

更新规则通常写作：

$$
v \leftarrow \rho v - \eta \nabla_W L
$$

$$
W \leftarrow W + v
$$

其中：

- $v$ 是速度项
- $\rho$ 是动量系数，常取 0.9 或 0.99

```python
v = 0
for step in range(num_steps):
    dw = compute_gradient(w)
    v = momentum * v - learning_rate * dw
    w += v
```

### Momentum 的效果
- 加速沿一致方向的移动
- 抑制高曲率方向上的震荡
- 有时能帮助越过浅局部坑或鞍点

## Nesterov Momentum
Nesterov momentum 可以理解为对普通 momentum 的一个小改进：

它不是在当前位置算梯度，而是先朝当前速度方向“看一眼”，再在前瞻位置计算梯度。

这个想法会让更新对前方地形更敏感，方向通常更准确。

可以把它理解成：

- 普通 momentum：先冲，再纠正
- Nesterov momentum：先预判，再冲

课程里知道这个直觉即可，真正实现时一般直接用框架提供的优化器。

## Adaptive Learning Rate Methods
除了给更新加惯性，还有一类方法会为不同参数自动调整步长。

直觉是：

- 某个维度梯度一直很大，说明这个方向更新要更谨慎
- 某个维度梯度一直很小，说明这个方向可以走得更快

### AdaGrad
AdaGrad 会累计历史梯度平方：

$$
G_t = G_{t-1} + g_t^2
$$

再用它缩放当前梯度：

$$
W_{t+1} = W_t - \frac{\eta}{\sqrt{G_t} + \epsilon} g_t
$$

其中 $g_t$ 是当前梯度。

优点是能自适应调整不同维度的学习率。

缺点也很明显：

- 历史平方梯度只增不减
- 训练到后面步长可能越来越小
- 最终几乎不再更新

```python
grad_squared = 0
for step in range(num_steps):
    dw = compute_gradient(w)
    grad_squared += dw * dw
    w -= learning_rate * dw / (np.sqrt(grad_squared) + 1e-7)
```

### RMSProp
RMSProp 可以看作“带遗忘机制的 AdaGrad”。

它不再无限累计所有历史梯度平方，而是做指数滑动平均：

$$
G_t = \beta G_{t-1} + (1 - \beta) g_t^2
$$

然后更新：

$$
W_{t+1} = W_t - \frac{\eta}{\sqrt{G_t} + \epsilon} g_t
$$

这样可以避免 AdaGrad 后期学习率过小的问题。

```python
cache = 0
for step in range(num_steps):
    dw = compute_gradient(w)
    cache = decay_rate * cache + (1 - decay_rate) * (dw * dw)
    w -= learning_rate * dw / (np.sqrt(cache) + 1e-7)
```

## Adam
Adam 是目前最常用的优化器之一，可以看作 momentum 和 RMSProp 的结合。

它同时维护：

- 一阶矩估计：梯度的滑动平均
- 二阶矩估计：梯度平方的滑动平均

公式写作：

$$
m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t
$$

$$
v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2
$$

由于初始化时 $m_t, v_t$ 都偏向 0，还需要偏差修正：

$$
\hat{m}_t = \frac{m_t}{1-\beta_1^t}, \quad \hat{v}_t = \frac{v_t}{1-\beta_2^t}
$$

最后更新：

$$
W_{t+1} = W_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t
$$

```python
m = 0
v = 0
for t in range(1, num_steps + 1):
    dw = compute_gradient(w)
    m = beta1 * m + (1 - beta1) * dw
    v = beta2 * v + (1 - beta2) * (dw * dw)
    m_unbias = m / (1 - beta1 ** t)
    v_unbias = v / (1 - beta2 ** t)
    w -= learning_rate * m_unbias / (np.sqrt(v_unbias) + 1e-7)
```

### Adam 的实践特点
- 通常开箱即用效果很好
- 对学习率没有纯 SGD 那么敏感
- 在很多任务里收敛更快

常见默认值是：

- $\beta_1 = 0.9$
- $\beta_2 = 0.999$
- $\epsilon = 10^{-8}$

## First-Order and Second-Order Methods
前面这些方法基本都只用到一阶导数，也就是梯度，因此叫一阶方法（first-order methods）。

另一类方法会利用二阶导数信息，例如 Hessian 矩阵，用来刻画曲率。

二阶方法理论上能：

- 更准确地估计地形
- 选择更合理的下降方向
- 在某些问题上收敛更快

但在深度学习中，二阶方法通常不够实用，因为：

- Hessian 很大
- 计算和存储代价都很高
- 求逆更加昂贵

因此大规模神经网络训练中通常还是以一阶方法为主。

## Convex and Non-Convex Optimization
优化问题里一个非常重要的概念是凸性（convexity）。

如果一个函数是凸函数，那么它有很好的性质：

- 局部最小值就是全局最小值
- 优化理论更完整
- 收敛分析更容易做

线性分类器配合 SVM 或 Softmax loss 时，通常对应的是凸优化问题。

但一旦进入神经网络，问题往往变成非凸优化：

- 可能有很多局部极值
- 可能有大量鞍点
- 很难证明一定找到全局最优

!!! warning "深度学习中的现实"
    我们通常无法保证找到全局最优解。

    但实践表明，只要优化方法、初始化和超参数选择合理，找到一个足够好的解通常就已经够用了。

## What Hyperparameters Matter
这一讲出现了很多优化相关超参数，最关键的包括：

- 初始化方式
- 学习率
- 训练步数
- batch size
- 正则化强度
- momentum 系数
- RMSProp / Adam 的衰减系数

其中最敏感、最常需要调的是学习率。

## Optimization in Practice
从工程角度看，第四讲最重要的结论不是“记住多少公式”，而是建立下面这组经验：

- 先确认 loss 能正常下降
- 如果 loss 完全不动，先怀疑学习率太小、梯度实现错误或初始化不当
- 如果 loss 爆炸，先怀疑学习率太大
- 如果更新方向左右摇摆，考虑用 momentum
- 如果不同维度尺度差异很大，自适应方法会更稳
- 默认起步时，Adam 往往比纯 SGD 更省事

## Main Takeaways
- Optimization 的目标是最小化损失函数，找到更好的参数
- 梯度给出局部最陡上升方向，因此负梯度是下降方向
- Numerical gradient 适合检查，analytic gradient 适合训练
- Full batch gradient descent 太贵，所以实践中常用 mini-batch SGD
- SGD 会遇到震荡、鞍点和 noisy gradient 等问题
- Momentum 通过引入速度项改善优化轨迹
- AdaGrad、RMSProp、Adam 会按维度自适应调整步长
- 大规模深度学习通常依赖一阶优化方法
- 线性模型优化通常是凸的，神经网络优化通常是非凸的

## Terms
!!! info "术语小结"
    - Gradient: 损失函数对参数的偏导向量，给出最陡上升方向
    - Learning Rate: 每次参数更新的步长
    - Mini-batch: 一次用于估计梯度的一小批训练样本
    - SGD: 使用随机小批量近似全数据梯度的优化方法
    - Momentum: 给参数更新加入历史速度的优化技巧
    - AdaGrad: 基于历史梯度平方自适应缩放学习率的方法
    - RMSProp: 带遗忘机制的 AdaGrad
    - Adam: 结合 momentum 与 RMSProp 的常用优化器
    - Saddle Point: 梯度接近 0 但并非最优解的区域
    - Convex Optimization: 局部最优等于全局最优的优化问题
