# Lec 8: CNN Architectures

## Overview
上一讲介绍了卷积网络的基本组件：卷积层、池化层、BN 等。

本讲回顾 ImageNet 竞赛（ILSVRC）历史上具有里程碑意义的 CNN 架构，理解每次创新背后解决了什么问题：

- **AlexNet**（2012）：深度 CNN 首次击败传统方法
- **VGGNet**（2014）：更深的网络 + 只用 $3 \times 3$ 卷积
- **GoogLeNet / Inception**（2014）：Inception 模块大幅降低参数量
- **ResNet**（2015）：残差连接解决梯度消失，使超深网络成为可能
- **后 ResNet 时代**：DenseNet、MobileNet、EfficientNet 等

## ImageNet and ILSVRC
ImageNet 是一个包含 **140 万**张图像、**1000 个类别**的大规模图像分类数据集。

ILSVRC（ImageNet Large Scale Visual Recognition Challenge）是每年的竞赛，Top-5 Error 是主要衡量指标（模型给出的前 5 个预测中必须包含正确标签）。

| 年份 | 模型 | Top-5 Error |
|---|---|---|
| 2011 | 传统方法（HOG + SVM） | ~26% |
| 2012 | AlexNet | 16.4% |
| 2013 | ZFNet | 11.7% |
| 2014 | VGGNet / GoogLeNet | 7.3% / 6.7% |
| 2015 | ResNet | 3.6% |
| 2016+ | 各种改进 | ~3% |

人类判别误差约为 5%，ResNet 在 2015 年已超过人类水平。

## AlexNet（2012）

AlexNet 是第一个在 ImageNet 上大幅击败传统方法的深度 CNN，由 Krizhevsky、Sutskever 和 Hinton 提出。

### 网络结构

```
输入: 224×224×3
  Conv1: 96 × 11×11, stride=4 → 55×55×96 → ReLU → MaxPool (3×3, s=2)
  Conv2: 256 × 5×5, pad=2     → 27×27×256 → ReLU → MaxPool (3×3, s=2)
  Conv3: 384 × 3×3, pad=1     → 13×13×384 → ReLU
  Conv4: 384 × 3×3, pad=1     → 13×13×384 → ReLU
  Conv5: 256 × 3×3, pad=1     → 13×13×256 → ReLU → MaxPool (3×3, s=2)
  Flatten → FC(4096) → ReLU → Dropout
           → FC(4096) → ReLU → Dropout
           → FC(1000) → Softmax
```

总参数量约 **60M**。

### AlexNet 的关键创新

- **ReLU 激活函数**：训练速度比 sigmoid 快很多
- **Dropout**：正则化，减少 FC 层过拟合
- **数据增强（Data Augmentation）**：随机翻转、裁剪
- **GPU 训练**：首次用双 GPU 并行训练

!!! info "历史意义"
    AlexNet 的成功标志着深度学习时代的开始，其在 Top-5 Error 上将当年第二名甩开超 10 个百分点，震惊了整个 CV 社区。

## VGGNet（2014）

VGGNet 由 Oxford Visual Geometry Group 提出，核心思想是：**网络更深 + 全用 $3 \times 3$ 卷积**。

### 设计原则

- 只使用 $3 \times 3$、stride=1、padding=1 的卷积（保持尺寸）
- 只使用 $2 \times 2$、stride=2 的 MaxPool（尺寸减半）
- 网络每经过一次池化，通道数翻倍：64 → 128 → 256 → 512

### VGG-16 结构（简化）

```
输入: 224×224×3
  [Conv(64) × 2] → MaxPool  → 112×112×64
  [Conv(128) × 2] → MaxPool → 56×56×128
  [Conv(256) × 3] → MaxPool → 28×28×256
  [Conv(512) × 3] → MaxPool → 14×14×512
  [Conv(512) × 3] → MaxPool → 7×7×512
  Flatten → FC(4096) → FC(4096) → FC(1000)
```

总参数量约 **138M**（主要来自后三个 FC 层，占 ~102M）。

### 为什么只用 3×3 卷积？

两层 $3 \times 3$ 卷积的**感受野等于一层 $5 \times 5$**，但：

- 参数更少：$2 \times 9C^2 = 18C^2 < 25C^2$
- 两次 ReLU 比一次表达能力更强

!!! warning "VGGNet 的问题"
    参数量太大（138M），尤其是最后几个 FC 层极重。

    后续网络普遍用 **Global Average Pooling** 替代末尾的 FC 层，大幅减少参数。

## GoogLeNet / Inception（2014）

GoogLeNet 的目标：在不增加深度的前提下，**大幅降低参数量，提升计算效率**。

### Inception Module

Inception 模块的核心思路：**并行使用多种大小的卷积核，再拼接（concatenate）结果**：

```
输入 x
  ├── 1×1 Conv → 输出 A
  ├── 1×1 Conv → 3×3 Conv → 输出 B
  ├── 1×1 Conv → 5×5 Conv → 输出 C
  └── 3×3 MaxPool → 1×1 Conv → 输出 D
        ↓
  concat(A, B, C, D) 沿通道维拼接
```

$1 \times 1$ 卷积在进入大卷积核之前先做**通道降维（bottleneck）**，大幅减少计算量。

!!! info "降维效果举例"
    设输入 $256$ 通道，用 $32$ 个 $5 \times 5$ 卷积：

    - 直接：$32 \times 256 \times 5^2 = 204800$ 次乘加
    - 先用 $16$ 个 $1 \times 1$ 降维：$16 \times 256 + 32 \times 16 \times 25 = 16640$ 次乘加

    计算量约减少 12 倍。

### GoogLeNet 整体结构
- 深度 22 层（卷积层）
- 9 个 Inception 模块
- 无大型 FC 层，末尾用 Global Average Pooling
- 总参数量仅约 **5M**（AlexNet 的 1/12）
- 引入**辅助分类器（Auxiliary Classifiers）**：在中间层加两个分支分类头，帮助梯度流向更深处

## ResNet（2015）

何恺明等人提出 ResNet，解决了超深网络（>100 层）难以训练的问题。

### 问题：Degradation

随着网络加深，训练误差不降反升（不是过拟合，是优化问题）——**退化（degradation）**问题。

理论上更深的网络至少不应该比浅网络差（可以把多余的层学成恒等映射），但梯度下降很难学出恒等映射。

### 残差连接（Residual Connection / Skip Connection）

ResNet 的核心思想：不直接学 $H(x)$，而是学**残差 $F(x) = H(x) - x$**：

$$
H(x) = F(x) + x
$$

其中 $F(x)$ 是若干卷积层，$x$ 是输入的**跳跃连接（skip connection）**：

```
x ──→ Conv → BN → ReLU → Conv → BN → (+) ──→ ReLU ──→ 输出
  └───────────────────────────────────────┘
             shortcut (identity)
```

如果 $F(x) \approx 0$，整个模块就是恒等映射——学残差比学恒等映射容易得多。

梯度反传时，跳跃连接提供了**直接的梯度通道**，避免梯度消失。

### Bottleneck Block

为了减少参数量，深层 ResNet 用 bottleneck 结构：

```
输入
  → 1×1 Conv（降维：256 → 64）
  → 3×3 Conv（主体卷积）
  → 1×1 Conv（升维：64 → 256）
  → (+) skip connection
  → ReLU
```

参数量：$1 \times 1 \times 256 \times 64 + 3 \times 3 \times 64 \times 64 + 1 \times 1 \times 64 \times 256 = 69632$

vs 直接两层 $3 \times 3$：$3 \times 3 \times 256 \times 256 \times 2 = 1179648$，减少约 17 倍。

### ResNet 各版本

| 版本 | 层数 | Top-5 Error | 参数量 |
|---|---|---|---|
| ResNet-18 | 18 | — | 11M |
| ResNet-50 | 50 | 6.7% | 25M |
| ResNet-101 | 101 | 6.0% | 44M |
| ResNet-152 | 152 | 5.7% | 60M |

```python
import torchvision.models as models

resnet50 = models.resnet50(pretrained=True)
# 修改最后一层适应自定义类别数
resnet50.fc = nn.Linear(2048, num_classes)
```

## 后 ResNet 时代

### DenseNet（2017）
将跳跃连接推广：每层的输入是**所有前面层的输出拼接**：

$$
x_l = H_l([x_0, x_1, \ldots, x_{l-1}])
$$

- 参数更少（特征复用）
- 梯度更容易反传
- 缺点：GPU 内存消耗大

### MobileNet（2017）
面向移动端，将标准卷积替换为**深度可分离卷积（Depthwise Separable Convolution）**：

- **Depthwise Conv**：对每个通道单独做 $F \times F$ 卷积（$C$ 个滤波器，每个只处理 1 个通道）
- **Pointwise Conv**：$1 \times 1$ 卷积混合通道

参数量减少约 $\dfrac{1}{F^2}$ 倍（$F = 3$ 时约减少 9 倍）。

### EfficientNet（2019）
系统性研究网络**宽度（width）、深度（depth）、分辨率（resolution）**三个维度的缩放，提出**复合缩放（Compound Scaling）**：

$$
\text{width} \propto \phi^{0.5}, \quad \text{depth} \propto \phi^{0.8}, \quad \text{resolution} \propto \phi^{0.45}
$$

在相同 FLOPS 下显著超过 ResNet。

## 架构选择指南

| 场景 | 推荐架构 |
|---|---|
| 快速原型 / 学习 | ResNet-50 |
| 移动端 / 低算力 | MobileNetV2 |
| 追求精度 | EfficientNet-B4/B5 |
| 迁移学习 | ResNet-50 / ResNet-101（ImageNet 预训练） |
| 特征提取骨干网络 | ResNet / EfficientNet |

!!! tip "迁移学习（Transfer Learning）"
    直接在 ImageNet 上预训练的模型已经学到了丰富的视觉特征，对于新任务：

    1. **特征提取（Feature Extraction）**：冻结所有卷积层，只训练最后的分类头
    2. **微调（Fine-tuning）**：解冻部分或全部层，用较小学习率在新数据上继续训练

## Main Takeaways
- AlexNet 标志深度学习时代，ReLU + Dropout + GPU 是关键
- VGGNet 证明深度比宽度更重要，小卷积核堆叠等价于大卷积核
- GoogLeNet 用 Inception 模块和 $1 \times 1$ 降维，5M 参数实现竞争力
- ResNet 残差连接解决退化问题，使超深网络训练成为可能
- 后续演进：DenseNet（特征复用）、MobileNet（深度可分离）、EfficientNet（复合缩放）
- 实践中优先使用 ResNet 预训练模型进行迁移学习

## Terms
!!! info "术语小结"
    - **Top-5 Error**：模型给出的前 5 个预测中不包含正确标签的比例
    - **Dropout**：训练时随机置零部分神经元，防止过拟合
    - **Bottleneck Block（瓶颈块）**：用 $1 \times 1$ 先降维再升维，减少参数量
    - **Skip Connection / Residual Connection（跳跃/残差连接）**：将输入直接加到若干层之后的输出
    - **Residual（残差）**：$F(x) = H(x) - x$，残差连接让网络学习残差而非直接目标
    - **Degradation（退化）**：网络加深后训练误差反而升高的现象
    - **Inception Module**：并行使用多种大小卷积核后拼接的模块
    - **Depthwise Separable Convolution（深度可分离卷积）**：把卷积拆成逐通道和逐点两步，大幅降低参数量
    - **Global Average Pooling（全局平均池化）**：将特征图每个通道取均值，替代大型 FC 层
    - **Transfer Learning（迁移学习）**：把在大数据集上预训练的模型迁移到新任务
    - **Fine-tuning（微调）**：在新任务数据上继续训练预训练模型（通常用较小学习率）
    - **Compound Scaling（复合缩放）**：EfficientNet 中同步缩放网络宽度、深度、分辨率的策略
