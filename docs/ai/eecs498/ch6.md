# Lec 6: Backpropagation

## Overview
上一讲我们有了神经网络的结构，但还需要一个方法高效地计算**所有参数的梯度**，才能用梯度下降做训练。

对于一个几十层的深度网络，参数可能有几千万个。如果对每个参数都用数值梯度（前向传播两次），代价是不可接受的。

解决方法是**反向传播（Backpropagation）**：利用链式法则，沿着计算图从输出到输入高效地反向计算梯度，每个参数只需 $O(1)$ 额外工作。

## Computational Graph
将计算过程用一个**有向无环图（DAG）**表示：

- **节点（node）**：操作（加、乘、激活等）
- **边（edge）**：数据（标量、向量、张量）

!!! info "例子：$f(x, y, z) = (x + y) \cdot z$"
    设 $q = x + y$，则 $f = q \cdot z$。

    计算图：

    ```
    x ──┐
         + ── q ──┐
    y ──┘          × ── f
              z ──┘
    ```

构建计算图之后，可以先做**前向传播（Forward Pass）**计算输出值，再做**反向传播（Backward Pass）**计算梯度。

## Forward Pass
前向传播就是按照计算图的拓扑顺序，从输入到输出依次计算每个节点的值。

```python
# f(x, y, z) = (x + y) * z
x, y, z = -2, 5, -4
q = x + y    # q = 3
f = q * z    # f = -12
```

## Backward Pass
反向传播沿着计算图**反向**传递梯度。

目标是计算 $\dfrac{\partial f}{\partial x}$、$\dfrac{\partial f}{\partial y}$、$\dfrac{\partial f}{\partial z}$。

### 链式法则（Chain Rule）
如果 $f = g(q)$ 且 $q = h(x)$，则：

$$
\frac{\partial f}{\partial x} = \frac{\partial f}{\partial q} \cdot \frac{\partial q}{\partial x}
$$

每个节点只需要知道：

1. **上游梯度（upstream gradient）**：$\dfrac{\partial f}{\partial q}$，来自下一层的反传
2. **局部梯度（local gradient）**：$\dfrac{\partial q}{\partial x}$，由当前节点的操作决定

**下游梯度（downstream gradient）** = 上游梯度 $\times$ 局部梯度：

$$
\frac{\partial f}{\partial x} = \underbrace{\frac{\partial f}{\partial q}}_{\text{upstream}} \cdot \underbrace{\frac{\partial q}{\partial x}}_{\text{local}}
$$

```python
# 继续上面的例子
# 初始：df/df = 1
dL_df = 1.0

# f = q * z，反传 × 门：
dL_dq = dL_df * z    # = 1 * (-4) = -4
dL_dz = dL_df * q    # = 1 * 3 = 3

# q = x + y，反传 + 门：
dL_dx = dL_dq * 1   # = -4
dL_dy = dL_dq * 1   # = -4
```

## Common Gate Patterns
计算图中有一些常见操作，它们的梯度形式有固定规律：

### Add Gate（加法门）
$$
q = x + y \implies \frac{\partial q}{\partial x} = 1, \quad \frac{\partial q}{\partial y} = 1
$$

- 加法门**均等地分发**上游梯度给所有输入

### Multiply Gate（乘法门）
$$
q = x \cdot y \implies \frac{\partial q}{\partial x} = y, \quad \frac{\partial q}{\partial y} = x
$$

- 乘法门**交换输入值**作为对方的局部梯度
- 如果某个输入非常大，对应的梯度也很大（梯度放大效应）

### Max Gate（最大值门）
$$
q = \max(x, y) \implies \frac{\partial q}{\partial x} = \mathbf{1}[x > y], \quad \frac{\partial q}{\partial y} = \mathbf{1}[y > x]
$$

- 上游梯度**全部流向最大值对应的分支**，其他分支梯度为 0
- ReLU 就是 $\max(0, x)$ 的特例

### Copy Gate（复制门）
如果一个节点的输出被使用了多次，其梯度是所有使用处梯度的**累加**：

$$
\frac{\partial L}{\partial x} = \sum_k \frac{\partial L}{\partial z_k} \cdot \frac{\partial z_k}{\partial x}
$$

## Sigmoid Gate Shortcut
Sigmoid 函数 $\sigma(x) = \dfrac{1}{1 + e^{-x}}$ 的导数有一个优雅的形式：

$$
\frac{d\sigma}{dx} = \sigma(x)(1 - \sigma(x))
$$

!!! tip "推导验证"
    $$
    \frac{d\sigma}{dx} = \frac{e^{-x}}{(1 + e^{-x})^2} = \sigma(x) \cdot \frac{e^{-x}}{1 + e^{-x}} = \sigma(x)(1 - \sigma(x))
    $$

可以把 Sigmoid 里的一堆子操作打包成单个节点，直接用这个公式反传。

```python
# Sigmoid 的前向 + 反向
x = 1.0
sig = 1 / (1 + torch.exp(-x_tensor))
# 反传：上游梯度 * sig * (1 - sig)
```

## Vectorized Backpropagation
当输入从标量扩展到向量/矩阵时，梯度计算也要扩展。

设 $y = f(x)$，$x \in \mathbb{R}^n$，$y \in \mathbb{R}^m$，则**雅可比矩阵（Jacobian Matrix）**为：

$$
J = \frac{\partial y}{\partial x} \in \mathbb{R}^{m \times n}, \quad J_{ij} = \frac{\partial y_i}{\partial x_j}
$$

链式法则向量形式：

$$
\frac{\partial L}{\partial x} = J^T \cdot \frac{\partial L}{\partial y}
$$

!!! warning "不显式构造 Jacobian"
    对于一个大型网络，Jacobian 矩阵可能是 $4096 \times 4096$，存储就不现实。

    实践中不会显式构造 $J$，而是直接计算 $J^T \cdot v$（矩阵向量积），这叫 **vector-Jacobian product（VJP）**。

### 矩阵乘法反传
设 $Y = XW$，其中 $X \in \mathbb{R}^{N \times D}$，$W \in \mathbb{R}^{D \times M}$，$Y \in \mathbb{R}^{N \times M}$，则：

$$
\frac{\partial L}{\partial X} = \frac{\partial L}{\partial Y} \cdot W^T, \quad \frac{\partial L}{\partial W} = X^T \cdot \frac{\partial L}{\partial Y}
$$

验证形状（维度对齐原则）：

- $\dfrac{\partial L}{\partial X}$：形状应与 $X$ 相同，即 $N \times D$ ✓
- $\dfrac{\partial L}{\partial W}$：形状应与 $W$ 相同，即 $D \times M$ ✓

!!! info "记忆技巧"
    对 $Y = XW$：
    
    - $\dfrac{\partial L}{\partial X} = \dfrac{\partial L}{\partial Y} W^T$（上游梯度乘以 $W$ 的转置）
    - $\dfrac{\partial L}{\partial W} = X^T \dfrac{\partial L}{\partial Y}$（$X$ 的转置乘以上游梯度）
    
    只要确保形状和原始矩阵一致，就不会搞错 $W^T$ 在左还是右。

### 向量化代码示例
```python
import torch

x = torch.randn(4, 3, requires_grad=True)
w = torch.randn(3, 5, requires_grad=True)
y = x @ w                   # (4, 5)
loss = y.sum()
loss.backward()

print(x.grad.shape)         # (4, 3)
print(w.grad.shape)         # (3, 5)
```

## Backprop Through a Neural Network
把以上思想应用到两层神经网络：

$$
s = W_2 \underbrace{\max(0, W_1 x)}_h
$$

**前向传播：**

$$
h_1 = W_1 x + b_1, \quad h_2 = \max(0, h_1), \quad s = W_2 h_2 + b_2
$$

**反向传播：**

$$
\frac{\partial L}{\partial s} \to \frac{\partial L}{\partial W_2}, \frac{\partial L}{\partial h_2} \to \frac{\partial L}{\partial h_1}\ (\text{ReLU gate}) \to \frac{\partial L}{\partial W_1}, \frac{\partial L}{\partial x}
$$

每一步都是"局部梯度 × 上游梯度"，链式法则自动处理层间传递。

## Autograd in PyTorch
现代深度学习框架（PyTorch、JAX 等）自动构建计算图并执行反向传播。

```python
import torch
import torch.nn as nn

x = torch.randn(1, 3072)
model = nn.Sequential(
    nn.Linear(3072, 100),
    nn.ReLU(),
    nn.Linear(100, 10),
)
loss_fn = nn.CrossEntropyLoss()
y = torch.tensor([3])

# 前向传播（自动构建计算图）
scores = model(x)
loss = loss_fn(scores, y)

# 反向传播（自动计算所有梯度）
loss.backward()

# 查看梯度
for name, param in model.named_parameters():
    print(name, param.grad.shape)
```

!!! info "PyTorch 的工作原理"
    - 每次前向传播，PyTorch 动态构建计算图（Dynamic Computation Graph）
    - `loss.backward()` 触发反向传播，沿图反向传递梯度
    - `.grad` 属性保存每个参数的梯度
    - `optimizer.step()` 用梯度更新参数

## Gradient Checking
验证反向传播实现是否正确的方法：用数值梯度与解析梯度对比。

```python
def grad_check(f, x, analytic_grad, h=1e-5):
    num_grad = torch.zeros_like(x)
    for i in range(x.numel()):
        old_val = x.flat[i]
        x.flat[i] = old_val + h
        pos = f(x)
        x.flat[i] = old_val - h
        neg = f(x)
        x.flat[i] = old_val
        num_grad.flat[i] = (pos - neg) / (2 * h)

    rel_err = (analytic_grad - num_grad).abs().max() / (
        analytic_grad.abs() + num_grad.abs() + 1e-8
    ).max()
    return rel_err  # 应小于 1e-5
```

!!! warning "梯度检查的注意事项"
    - 只在调试阶段使用，正常训练时关掉（太慢）
    - 用双精度（float64）以减少数值误差
    - 在问题规模的一小部分参数上检查，不要全部检查

## Main Takeaways
- 反向传播 = 在计算图上按链式法则反向传递梯度
- 每个节点只需要知道局部梯度和上游梯度，与其他节点无关
- 加法门分发梯度，乘法门交换梯度，Max 门路由梯度
- 向量化后用 Jacobian 矩阵和 VJP 高效计算，不显式构造大 Jacobian
- 矩阵乘法反传：$\partial L / \partial X = (\partial L / \partial Y) W^T$，$\partial L / \partial W = X^T (\partial L / \partial Y)$
- PyTorch autograd 自动完成计算图构建与反向传播

## Terms
!!! info "术语小结"
    - **Backpropagation（反向传播）**：利用链式法则在计算图上高效计算梯度的算法
    - **Computational Graph（计算图）**：表示计算过程的有向无环图
    - **Forward Pass（前向传播）**：从输入到输出计算数值
    - **Backward Pass（反向传播）**：从输出到输入计算梯度
    - **Local Gradient（局部梯度）**：当前节点输出对输入的偏导数
    - **Upstream Gradient（上游梯度）**：损失对当前节点输出的梯度（来自更靠近损失的节点）
    - **Downstream Gradient（下游梯度）**：损失对当前节点输入的梯度（传给更靠近输入的节点）
    - **Jacobian Matrix（雅可比矩阵）**：向量函数对向量输入的偏导数矩阵
    - **VJP（Vector-Jacobian Product）**：上游梯度向量与 Jacobian 的乘积，实践中用来替代显式 Jacobian
    - **Autograd**：自动微分，框架自动完成计算图构建与梯度计算
    - **Gradient Checking（梯度检查）**：用数值梯度验证解析梯度正确性的调试技巧
