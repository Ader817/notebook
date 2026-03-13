# Lec 7: Convolutional Networks

## Overview
前两讲介绍了神经网络的结构（线性层 + 激活函数）和训练方式（反向传播）。但对于图像任务，全连接网络（MLP）存在明显的不足：

- **参数量爆炸**：CIFAR-10 图像展平后有 $3072$ 维，一层隐藏层有 $1000$ 个神经元，就需要 $> 300$ 万参数；ImageNet（$224 \times 224 \times 3$）则需要数亿参数
- **丢失空间结构**：展平为向量后，像素之间的空间相邻关系完全丢失

本讲的核心思路：图像有两个天然的结构——**局部性（locality）**和**平移不变性（translation invariance）**。卷积（Convolution）正是对这两个结构的利用。

## Motivation: Why Convolution?

图像中的模式（边缘、纹理、物体）具有以下特点：

1. **局部性（Locality）**：边缘等低级特征只取决于局部像素，不需要全局感知
2. **平移不变性（Translation Invariance）**：猫不管出现在图像哪个位置，用于检测猫的特征应该是相同的

全连接层对图像每个位置都学一套独立的权重，既浪费参数，又无法共享跨位置的特征检测器。

**卷积层**的解决方案：用**同一个小滤波器（filter/kernel）**在图像的所有位置滑动，实现参数共享和局部感受野。

## Convolution Operation
### 单通道卷积
设输入图像大小为 $H \times W$，滤波器大小为 $F \times F$，则：

$$
\text{output}[i, j] = \sum_{m=0}^{F-1} \sum_{n=0}^{F-1} \text{input}[i+m, j+n] \cdot \text{filter}[m, n] + b
$$

滤波器在图像上的每个位置做**内积（dot product）**，输出一个数值，汇集为**特征图（feature map）**。

```python
import torch
import torch.nn as nn

# 单个卷积层：输入 1 通道，输出 1 张特征图，3×3 滤波器
conv = nn.Conv2d(in_channels=1, out_channels=1, kernel_size=3, padding=1)
# 输入: (batch, 1, H, W) → 输出: (batch, 1, H, W)
```

### 多通道卷积
实际图像有 $C$ 个输入通道（如 RGB 图像 $C = 3$），滤波器也要对应有 $C$ 个通道：

- 滤波器形状：$C \times F \times F$
- 用 $K$ 个不同滤波器 → 输出 $K$ 张特征图

$$
\text{output}[k, i, j] = \sum_{c=0}^{C-1} \sum_{m=0}^{F-1} \sum_{n=0}^{F-1} \text{input}[c, i+m, j+n] \cdot W_k[c, m, n] + b_k
$$

所以一个卷积层的参数量是：$K \times C \times F \times F + K$（$K$ 个偏置）

!!! info "参数量对比"
    全连接层：$H \times W \times C \to H \times W \times K$ 需要 $(H \cdot W \cdot C) \times (H \cdot W \cdot K)$ 个参数。

    卷积层：同样从 $C$ 通道到 $K$ 通道，只需要 $K \times C \times F^2$ 个参数（$F \ll H$），**参数共享**节省了巨量参数。

## Output Size

卷积输出的空间尺寸由四个超参数决定：

| 超参数 | 含义 |
|---|---|
| $F$（kernel size） | 滤波器大小 |
| $P$（padding） | 在输入周围填补的像素数 |
| $S$（stride） | 滤波器每次滑动的步长 |
| $K$（num filters） | 滤波器数量，即输出通道数 |

输入大小为 $W_\text{in} \times H_\text{in}$，输出大小：

$$
W_\text{out} = \frac{W_\text{in} - F + 2P}{S} + 1
$$

$$
H_\text{out} = \frac{H_\text{in} - F + 2P}{S} + 1
$$

!!! tip "常用设置"
    - $F = 3, P = 1, S = 1$：输出与输入**同尺寸**（same padding）
    - $F = 3, P = 0, S = 1$：输出每边各缩小 1 像素
    - $F = 3, P = 1, S = 2$：输出尺寸减半（下采样）

## Padding
**零填充（zero-padding）**在输入边界补 0，目的：

1. 控制输出尺寸（避免每层都缩小）
2. 让边界区域也能完整参与卷积计算

常见填充方式：

- `padding='valid'`（无填充）：输出 $< $ 输入
- `padding='same'`（保持尺寸）：输出 $=$ 输入

## Stride
**步长（stride）** $S > 1$ 时，滤波器每次跳跃 $S$ 个像素，输出尺寸缩小（用于下采样）。

等效于卷积后再做均匀间隔的采样。

## 1×1 Convolution
$1 \times 1$ 卷积看似没有空间汇聚，但它对**通道维度**做了线性变换：

$$
\text{output}[k, i, j] = \sum_{c=0}^{C-1} \text{input}[c, i, j] \cdot W_k[c] + b_k
$$

用途：

- **通道升降维**：用较少的 $K$ 实现通道压缩（降维），减少后续计算量
- **增加非线性**：$1 \times 1$ 卷积后接 ReLU，增加表达能力
- GoogLeNet/Inception 模块广泛使用 $1 \times 1$ 卷积

## Pooling Layer
**池化层（Pooling Layer）**对特征图做空间下采样，减少特征图尺寸和参数量，同时引入一定程度的平移不变性。

池化层**没有可学习参数**。

### Max Pooling
在每个 $F \times F$ 窗口内取最大值：

$$
\text{output}[i, j] = \max_{0 \leq m, n < F} \text{input}[i \cdot S + m, j \cdot S + n]
$$

最常用：$F = 2, S = 2$，将特征图尺寸减半。

```python
pool = nn.MaxPool2d(kernel_size=2, stride=2)
# (batch, C, H, W) → (batch, C, H//2, W//2)
```

### Average Pooling
在窗口内取平均值，常用于网络末端（Global Average Pooling）将空间维度压缩为 $1 \times 1$：

```python
gap = nn.AdaptiveAvgPool2d((1, 1))
# (batch, C, H, W) → (batch, C, 1, 1)
```

## Receptive Field
**感受野（Receptive Field）**是输出特征图中某个神经元所对应的输入像素范围。

每经过一层 $F \times F$ 的卷积（stride=1），感受野增大 $F - 1$。经过 $L$ 层后感受野为：

$$
\text{RF} = 1 + L \times (F - 1)
$$

若使用 stride 或 pooling，感受野增长更快。

!!! info "为什么用小卷积核堆叠？"
    两层 $3 \times 3$ 卷积的感受野等于一层 $5 \times 5$ 卷积，但参数量更少（$2 \times 3^2 = 18 < 5^2 = 25$），且多一次非线性变换，表达能力更强。

    这也是 VGGNet 全用 $3 \times 3$ 卷积的原因。

## ConvNet Architecture

典型的卷积神经网络由以下模块堆叠而成：

```
输入图像
  → [Conv → BN → ReLU] × N    # 特征提取
  → [MaxPool]                  # 空间下采样
  → ...重复若干次               # 更高层特征
  → Flatten / Global Avg Pool  # 空间维度压缩
  → FC → Softmax               # 分类
```

用 PyTorch 定义一个简单的 ConvNet：

```python
import torch.nn as nn

class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, kernel_size=3, padding=1),  # 3→32 通道
            nn.ReLU(),
            nn.MaxPool2d(2, 2),                           # H,W 减半
            nn.Conv2d(32, 64, kernel_size=3, padding=1), # 32→64 通道
            nn.ReLU(),
            nn.MaxPool2d(2, 2),                           # H,W 再减半
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(64 * 8 * 8, 256),
            nn.ReLU(),
            nn.Linear(256, num_classes),
        )

    def forward(self, x):
        return self.classifier(self.features(x))
```

## Convolution as Feature Learning
卷积网络每层的滤波器都在学习不同层次的视觉特征：

- **第 1 层**：边缘、颜色斑块等低级特征
- **中间层**：纹理、局部形状
- **深层**：物体局部（眼睛、轮子）乃至整体语义

这种**层级特征（hierarchical features）**的自动学习是 CNN 强大的根本原因。

## Batch Normalization（简介）
卷积层后通常接**批标准化（Batch Normalization）**：

$$
\hat{x} = \frac{x - \mu_B}{\sigma_B + \epsilon}, \quad y = \gamma \hat{x} + \beta
$$

其中 $\mu_B, \sigma_B$ 是当前 mini-batch 的均值和标准差，$\gamma, \beta$ 是可学习的缩放和平移参数。

作用：
- 稳定训练，允许更大的学习率
- 轻度正则化效果
- 使网络对权重初始化不那么敏感

## Main Takeaways
- 卷积层通过参数共享和局部连接，高效处理图像的空间结构
- 输出尺寸公式：$(W - F + 2P) / S + 1$
- 多通道卷积：$K$ 个 $C \times F \times F$ 的滤波器 → $K$ 张特征图
- 池化层（Max Pooling）做空间下采样，无可学习参数
- $1 \times 1$ 卷积用于通道升降维
- 小卷积核堆叠比大卷积核参数更少、非线性更强
- ConvNet 自动学习层级视觉特征

## Terms
!!! info "术语小结"
    - **Convolution（卷积）**：滤波器在输入上滑动做内积，提取局部特征
    - **Filter / Kernel（滤波器/卷积核）**：卷积层的可学习参数，形状 $C \times F \times F$
    - **Feature Map（特征图）**：卷积层的输出，每个滤波器产生一张
    - **Padding（填充）**：在输入边界补零，控制输出尺寸
    - **Stride（步长）**：滤波器每次移动的像素数，$S > 1$ 实现下采样
    - **Receptive Field（感受野）**：某个输出神经元对应的输入区域大小
    - **Parameter Sharing（参数共享）**：同一滤波器在所有位置共享权重
    - **Max Pooling（最大池化）**：在局部窗口取最大值，做空间下采样
    - **Global Average Pooling（全局平均池化）**：将整个特征图压缩为一个数值
    - **1×1 Convolution**：对通道维度做线性变换，用于升降维
    - **Batch Normalization（批标准化）**：对每个 mini-batch 标准化激活值，稳定训练
