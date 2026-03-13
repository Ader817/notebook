# Lec 9: Hardware and Software

## Overview
深度学习的成功不只依赖算法，也依赖于**硬件与软件生态的协同发展**。

本讲讨论三个问题：

1. 深度学习的计算发生在什么硬件上？CPU vs GPU vs TPU 的区别是什么？
2. 深度学习框架（PyTorch / TensorFlow）做了什么，如何选择？
3. 在实际训练中，如何高效地组织数据和计算，让硬件跑满？

## Deep Learning Hardware

### CPU vs GPU

| 特性 | CPU | GPU |
|---|---|---|
| 核数 | 少（4～64 核） | 多（数千 CUDA 核） |
| 每核频率 | 高（3～5 GHz） | 低（~1 GHz） |
| 擅长任务 | 串行、复杂逻辑 | 大规模并行浮点计算 |
| 内存带宽 | 低（~50 GB/s） | 高（~900 GB/s，A100） |
| 典型用途 | 数据预处理、控制流 | 矩阵乘法、卷积 |

深度学习的核心计算是矩阵乘法（GEMM），天然适合 GPU 的大规模并行架构。

!!! info "为什么 GPU 更快？"
    训练一个 batch 时，每个样本的前向/反向传播在数学上相互独立，可以完全并行。

    CPU 的少量高频核无法充分利用这种并行性；GPU 的数千个低频核则能同时处理整个 batch。

### GPU 内存层级

GPU 的内存系统对性能至关重要：

```
寄存器（Register）     < 1 ns    每个 CUDA 核私有
  ↓
共享内存（Shared Mem） ~几 ns    同一线程块内共享
  ↓
L1/L2 Cache           ~几十 ns
  ↓
全局内存（VRAM/HBM）   ~百 ns    所有核共享，容量大（A100: 80GB）
  ↓
主机内存（CPU RAM）     PCIe 传输  带宽瓶颈
```

**内存带宽**往往是实际训练的瓶颈，而非浮点算力。

### TPU（Tensor Processing Unit）

Google 专为深度学习设计的 ASIC：

- 针对矩阵乘法做硬件级优化（脉动阵列，Systolic Array）
- 比 GPU 在特定任务（Transformer 训练）上快数倍
- 能耗效率更高

TPU 通过 Google Cloud / JAX 使用，PyTorch 也逐步支持 TPU。

### 数值格式

降低数值精度可以成倍提升吞吐量：

| 格式 | 位数 | 精度 | 典型用途 |
|---|---|---|---|
| FP64（double） | 64 位 | 最高 | 科学计算 |
| FP32（single） | 32 位 | 标准 | 训练默认 |
| FP16（half） | 16 位 | 较低 | 混合精度训练 |
| BF16 | 16 位 | 指数范围大 | Transformer 训练常用 |
| INT8 | 8 位 | 整数 | 推理量化 |

**混合精度训练（Mixed Precision Training）**：前向传播和梯度计算用 FP16，参数更新用 FP32 主备份，兼顾速度和稳定性。

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for X, y in dataloader:
    optimizer.zero_grad()
    with autocast():               # 前向传播用 FP16
        loss = model(X, y)
    scaler.scale(loss).backward()  # 反传时缩放梯度避免下溢
    scaler.step(optimizer)
    scaler.update()
```

## Deep Learning Software

深度学习框架提供三个核心能力：

1. **张量计算（Tensor Operations）**：GPU 加速的多维数组运算
2. **自动微分（Autograd）**：自动构建计算图并反传梯度
3. **神经网络抽象（nn.Module）**：层、优化器、Loss 等模块化封装

### PyTorch vs TensorFlow

| | PyTorch | TensorFlow |
|---|---|---|
| 计算图 | 动态（Define-by-Run） | 静态（Define-and-Run，TF1）/ 动态（TF2 Eager） |
| 调试 | 直接用 Python 调试器 | TF1 需要 Session，TF2 改善 |
| 学术界 | 主流 | 逐渐减少 |
| 工业部署 | TorchScript / ONNX | TensorFlow Serving / TFLite |
| 生态 | HuggingFace、Lightning 等 | Keras、TFHub 等 |

!!! tip "现在学什么？"
    学术研究和个人实践首选 **PyTorch**。工业部署中 TensorFlow / ONNX / TorchScript 也值得了解。

### PyTorch 核心组件

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset

# 1. 张量（Tensor）
x = torch.randn(3, 4)           # CPU 张量
x = x.cuda()                    # 移到 GPU
x = x.to('cuda')                # 等价写法

# 2. 自动微分
x = torch.randn(3, requires_grad=True)
y = (x ** 2).sum()
y.backward()                    # 计算梯度
print(x.grad)                   # dy/dx = 2x

# 3. 模型定义
class MyNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = nn.Linear(128, 10)

    def forward(self, x):
        return self.fc(x)

# 4. 训练循环
model = MyNet().cuda()
optimizer = optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.CrossEntropyLoss()

for X, y in dataloader:
    X, y = X.cuda(), y.cuda()
    optimizer.zero_grad()
    loss = loss_fn(model(X), y)
    loss.backward()
    optimizer.step()
```

### 静态图 vs 动态图

**动态图（Eager Mode）**：每次前向传播时即时执行并构建计算图，Python 控制流直接有效。

```python
# 动态图：if 分支直接生效
def forward(x):
    if x.sum() > 0:
        return x * 2
    return x * -1
```

**静态图（Graph Mode / TorchScript）**：先编译整个计算图，再重复执行，适合部署优化。

```python
# TorchScript：将模型编译为静态图
scripted = torch.jit.script(model)
torch.jit.save(scripted, 'model.pt')
```

## Data Loading

训练时 GPU 的利用率常被**数据加载**拖慢：GPU 还没算完，下一个 batch 的数据还没准备好。

### PyTorch DataLoader

PyTorch 的 `DataLoader` 支持多进程预取（prefetch），在 GPU 计算当前 batch 时，提前在 CPU 上加载和预处理下一个 batch：

```python
from torch.utils.data import DataLoader

# num_workers > 0：开启多进程数据加载
# pin_memory=True：使用固定内存，加速 CPU→GPU 传输
dataloader = DataLoader(
    dataset,
    batch_size=64,
    shuffle=True,
    num_workers=4,
    pin_memory=True,
)
```

!!! info "num_workers 如何选？"
    经验法则：`num_workers = CPU 核数 / GPU 数`（通常 4～8）。

    太多反而会因进程切换开销降低效率；若数据已在 SSD 上，2～4 通常足够。

### 数据预处理流水线

典型的 ImageNet 数据增强 pipeline：

```python
from torchvision import transforms

train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),     # 随机裁剪并 resize
    transforms.RandomHorizontalFlip(),     # 随机水平翻转
    transforms.ColorJitter(0.4, 0.4, 0.4), # 颜色抖动
    transforms.ToTensor(),                 # PIL → Tensor，归一化到 [0,1]
    transforms.Normalize(                  # 零均值单位方差
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225],
    ),
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])
```

## Computational Graphs and Memory

### 前向传播的内存消耗

在训练时，前向传播需要**保存所有中间激活值**（供反传使用），这是显存占用的主要来源。

一个典型 ResNet-50 在 batch_size=32 时，激活值约占 ~1.5 GB。

### 梯度检查点（Gradient Checkpointing）

以**时间换空间**：不保存全部中间激活，在反传时重新计算所需的激活值。

```python
from torch.utils.checkpoint import checkpoint

# 用 checkpoint 包裹前向传播，节省约 60% 显存
out = checkpoint(model_segment, x)
```

代价：大约增加 ~33% 的前向传播计算量。

### 模型并行 vs 数据并行

| 策略 | 说明 | 适用场景 |
|---|---|---|
| **数据并行（Data Parallel）** | 每张 GPU 存一个模型副本，各自处理不同 batch，梯度汇总 | 单机多 GPU，最常用 |
| **模型并行（Model Parallel）** | 模型的不同层分布在不同 GPU 上 | 模型太大，单 GPU 放不下 |
| **张量并行（Tensor Parallel）** | 每个矩阵乘法拆分到多个 GPU 上并行 | 超大 Transformer（GPT 等） |

```python
# PyTorch 数据并行（单机多 GPU）
model = nn.DataParallel(model)

# 分布式数据并行（DDP，推荐）
from torch.nn.parallel import DistributedDataParallel as DDP
model = DDP(model, device_ids=[local_rank])
```

## Profiling and Optimization

检查训练效率，找到瓶颈：

```python
# PyTorch Profiler
with torch.profiler.profile(
    activities=[
        torch.profiler.ProfilerActivity.CPU,
        torch.profiler.ProfilerActivity.CUDA,
    ]
) as prof:
    model(x)

print(prof.key_averages().table(sort_by='cuda_time_total'))
```

常见瓶颈及优化方向：

| 瓶颈 | 诊断 | 优化方法 |
|---|---|---|
| GPU 利用率低 | `nvidia-smi` 显示 GPU 占用 < 80% | 增大 batch size，减少 CPU-GPU 同步 |
| 数据加载慢 | Profiler 显示大量 CPU 等待 | 增加 `num_workers`，使用更快存储 |
| 显存不足 | OOM 报错 | 减小 batch size，gradient checkpointing，FP16 |
| 通信开销大 | 多 GPU 训练时 GPU 利用率周期性低 | 使用 DDP 代替 DP，增大 batch size |

## Main Takeaways
- GPU 通过大规模并行加速矩阵运算，是深度学习训练的核心硬件
- GPU 内存带宽往往是实际瓶颈，而非峰值算力
- 混合精度训练（FP16 + FP32 主备份）可以 2× 提速
- PyTorch 动态图便于调试；TorchScript / ONNX 用于部署
- DataLoader 的多进程预取和 pin_memory 可以消除数据加载瓶颈
- 训练时中间激活值是显存的主要占用；Gradient Checkpointing 以算换存
- 多 GPU 首选 DDP（Distributed Data Parallel）

## Terms
!!! info "术语小结"
    - **CUDA（Compute Unified Device Architecture）**：NVIDIA GPU 的并行计算平台
    - **VRAM / HBM**：GPU 显存，高带宽内存（High Bandwidth Memory）
    - **TPU（Tensor Processing Unit）**：Google 专为矩阵运算设计的 AI 加速芯片
    - **Mixed Precision Training（混合精度训练）**：FP16 计算 + FP32 参数维护，兼顾速度和稳定
    - **BF16（Brain Float 16）**：Google 提出的 16 位浮点格式，指数位更多，数值范围与 FP32 相当
    - **Autograd（自动微分）**：框架自动构建计算图并计算梯度
    - **TorchScript**：将 PyTorch 模型编译为静态图，用于部署优化
    - **DataLoader**：PyTorch 的数据加载器，支持多进程预取和批次采样
    - **pin_memory**：将 CPU 张量放在固定（非分页）内存中，加速 CPU→GPU 传输
    - **num_workers**：DataLoader 使用的子进程数，用于并行数据加载
    - **Gradient Checkpointing（梯度检查点）**：减少显存占用，以重新计算替代存储中间激活
    - **Data Parallel（数据并行）**：每个 GPU 存一份模型，各处理不同 batch 后汇总梯度
    - **DDP（Distributed Data Parallel）**：PyTorch 推荐的多 GPU / 多机训练方式
    - **Profiler（性能分析器）**：定位训练中 CPU/GPU 的时间消耗，找到性能瓶颈
